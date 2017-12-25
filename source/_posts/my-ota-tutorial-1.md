---
title: 小白的全栈梦之从零搭建Android OTA系统（1）
date: 2017-12-20 18:23:04
tags: [express, react, nodejs]
---

本文实现简单的服务端的注册，登录功能，暂时不实现前端界面，用curl模拟前端请求。

### 创建后端项目
``` bash
express ota_be
```

#### 目录结构
model存放数据库代码，routes存放路由代码，controller存放处理代码，middleware存放中间件代码。
```
├─app.js
├─config.js
├─model/
├─routes/
├─controller/
├─middleware/
```

### 用户注册

#### 实现用户集合
创建model/user.js，目前只保存username和password，其中username唯一。
``` javascript
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var UserSchema = new Schema({
  username: {
    type: String,
    unique: true,
  },
  password: {
    type: String, 
  },
})
mongoose.model('User', UserSchema);
```

创建model/index.js，连接数据库。
``` javascript
var mongoose = require('mongoose');
var config = require('../config');

mongoose.connect(config.mongodb, {
  server: {poolSize: 20},
  useMongoClient: true,
}, err => {
  if(err) {
    console.error("connect to %s failed", config.mongodb, err
    .message);
    process.exit(-1);
  } 
});

require('./user');

exports.User = mongoose.model('User');
```

#### 路由实现
创建routes/user.js。
这里的路由，都分发到控制器的user实现功能。

```javascript
var express = require('express');
var user = require('../controller/user');
var router = express.Router();
var passport = require('passport');

router.post('/user/register',  user.register);

module.exports = router;
```

创建routes/index.js，引入路由文件。
```` javascript
var express = require('express');
var router = express.Router();
var user = require('./user');

router.use(user);

module.exports = router;
````

修改app.js

```javascript
var router = require('./routes');
var app = express();
//...
app.use('/', router);
```
#### 实现注册操作
创建controller/user.js。
用户密码需要利用bcyptsjs加密，不存储明文密码。

```javascript
var Model = require('../model');
var User = Model.User;
var bcrypt = require('bcryptjs')

validUserRequest = body => {
  //TODO 更详细校验
  if(!body.username || !body.password) {
      return {code: -1, message: "valid param"};
  }
  return null;
}

exports.register = (req, res, next) => {
  var error = validUserRequest(req.body);
  if(error) {
    res.status(400).send(error);
    return;
  }
  const {username, password} = req.body;
  bcrypt.hash(password, 10)
    .then(hash => {
      var user = new User({
        username: username,
        password: hash
      });
      return user.save()
    })
    .then(user => {
      res.json({code: 0, message: "create user success"});
    })
    .catch(err => {
      res.status(406).send({code: -1, message: err.message});
    })
}

```

#### 运行
因为用到了mongoose库，需要本地先启用mongod服务，端口需要跟config.js中的一致。
config.js
``` javascript
var config = {
  mongodb: 'mongodb://127.0.0.1:50000/ota_server',
}

module.exports = config;
```
然后启动应用。
``` bash
yarn start
```

#### 测试注册接口
在终端里面用如下指令模拟注册请求。
``` bash
$ curl -d "username=yinlijun&password=123456" "http://127.0.0.1:3020/user/register"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    75  100    42  100    33    168    132 --:--:-- --:--:-- --:--:--   168{"code":0,"message":"create user success"}
```

利用mongo客户端打开集合，查看集合数据，插入成功。
``` bash
$ mongo mongodb://127.0.0.1:50000
MongoDB shell version v3.4.9
connecting to: mongodb://127.0.0.1:50000
MongoDB server version: 3.4.9

$ use ota_server
switched to db ota_server

$ db.users.find()
{ "_id" : ObjectId("5a3b61b1db2e604ebca6dfc4"), "username" : "yinlijun", "password" : "$2a$10$Zc33sn8Zj1kJslfTTXM0deFNUjVMJyWk.tMsuh.aaLtZEFUSedLQW", "__v" : 0 }
```

再次注册同名用户，报错。
``` bash
$ curl -d "username=yinlijun&password=123456" "http://127.0.0.1:3020/user/register"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   158  100   125  100    33   1344    354 --:--:-- --:--:-- --:--:--  1344{"code":-1,"message":"E11000 duplicate key error collection: ota_server.users index: username_1 dup key: { : \"yinlijun\" }"}
```

### 用户登录

#### 路由实现
添加路由routes/user.js
``` javascript
router.post('/user/login', user.login);
```

#### 实现登录操作
在controller/user.js中添加如下代码，校验用户名和密码，校验密码用到了bcrypt.compareSync。

然后生成cookie发到前端。

``` javascript
exports.login = (req, res, next) => {
  let {username, password} = req.body;
  User.findOne({username: username})
    .then(user => {
      if(!user) {
        res.status(400).send({code: -1, message:"invalid username or password"});
        return;
      }
      if(!bcrypt.compareSync(password, user.password)) {
        res.status(400).send({code: -1, message:"invalid username or password"});
        return;
      }
      genCookie(user, res);    
      res.send({code:0, message:"登录成功"});
    })
};

genCookie = (user, res) => {
  var token = user._id + '$$$';
  var opts = {
    path: '/',
    maxAge: config.session_age,
    httpOnly: true,
  };
  res.cookie(config.cookie_name, token);
}
```

这里可以看到有个重复调用的validUserRequest函数，可以做成一个middleware。

创建middleware/validRequest.js。
``` javascript
exports.validUsernamePassword = (req, res, next) => {
  //TODO 更详细校验
  let {username, password} = req.body;
  if(!username || !password) {
      res.status(400).send({code: -1, message: "valid param"});
      return;
  }
  next();
}
```
然后修改routes/user.js
``` javascript
var validRequest = require('../middleware/validRequest');

router.post('/user/register', validRequest.validUsernamePassword, user.register);

router.post('/user/login', validRequest.validUsernamePassword, user.login);
```

把controller/user.js的register中validUserRequest调用去掉。

测试一下这个middleware：
```` bash
 $ curl -d "username=yinlijun&password=" "http://127.0.0.1:3020/user/register"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    62  100    35  100    27   1093    843 --:--:-- --:--:-- --:--:--  1093{"code":-1,"message":"valid param"}
````

#### 测试登录接口
输入错误的用户名:
``` bash
$ curl -d "username=liudehua&password=123456" "http://127.0.0.1:3020/user/login"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    85  100    52  100    33   1106    702 --:--:-- --:--:-- --:--:--  1106{"code":-1,"message":"invalid username or password"}
```
输入错误密码：
```bash
$ curl -d "username=yinlijun&password=654321" "http://127.0.0.1:3020/user/login"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    85  100    52  100    33    553    351 --:--:-- --:--:-- --:--:--   553{"code":-1,"message":"invalid username or password"}
```
输入正确的用户名密码:
```
$ curl -d "username=yinlijun&password=123456" "http://127.0.0.1:3020/user/login"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    68  100    35  100    33    372    351 --:--:-- --:--:-- --:--:--   372{"code":0,"message":"登录成功"}
```

### 利用passport校验登录功能。
社区里面有个passport的组件，可以用来实现校验登录功能。

#### 封装passport中间件
创建middleware/passport.js，封装passport模块。

将校验密码的操作挪到passport中，校验成功之后用户信息将会保存到req.user。
```javascript
var passport = require('passport');
var User = require('../model').User;
var LocalStrategy = require('passport-local').Strategy;

passport.use('local.login', new LocalStrategy((username, password, done) => {
  User.findOne({username: username})
    .exec()
    .then(user => {
      if(!user) {
        return done(null, false, {code: -1, message: "invalid username or password"});
      } 
      if(!user.validPassword(password)) {
        return done(null, false, {code: -1, message: "invalid username or password"});
      } 
      return done(null, user);
    })
    .catch(err => {
      done(null, false, {code: -1, message: err.message});
    })
}))

passport.serializeUser((user, done) => {
  done(null, user._id);
});

passport.deserializeUser((id, done) => {
  User.findById(id)
    .exec()
    .then(user => {
      done(null, user);
    })
    .catch(err => {
      done(null, false, {code: -1, message: err.message});
    })
})

module.exports = passport;
```

#### 重新实现登录操作
修改controller/user.js的login函数，去掉验证用户名密码的代码。
```javascript
exports.login = (req, res, next) => {
  const user = req.user;
  genCookie(user, res);    
  res.send({code:0, message:"login success"});
};
```
在middleware/validRequest.js添加一个校验中间件
```javascript
exports.validByPassport = (req, res, next) => {
    passport.authenticate('local.login', {
      failureMessage: 'invalid username or password',
    })(req,  res, next);
}
```

#### 修改路由
修改router/user.js路由
```javascript
router.post('/user/login', validRequest.validByPassport, user.login);
```
这样，用户登录就会走passport的校验流程。

### 退出功能
主要是清除cookie，controller/user.js中添加代码。
``` javascript
exports.logout = (req, res, next) => {
  clearCookie(res);
  res.send({code:0, message:"exit success"});
}

clearCookie = (res) => {
  res.clearCookie(config.cookie_name, {path: '/'});
}
```

测试
``` 
$ curl "http://127.0.0.1:3020/user/logout"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    35  100    35    0     0   2187      0 --:--:-- --:--:-- --:--:--  2187{"code":0,"message":"exit success"}
```