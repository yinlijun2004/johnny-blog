---
title: 从零搭建Android OTA系统——用户管理规则实现
date: 2018-04-19 10:16:18
tags:  [express, react, nodejs, antd]
categories: 从零搭建Android OTA系统
---

OTA系统提供两套API，一套用于普通的查询功能，比如以发布的版本列表，版本下载地址等，另一套提供管理功能，比如新建，编辑，删除等。

管理功能API，需要区分用户类型，不同的用户有不同的权限，用户分为以下四类：

- 系统管理员
    - 批准用户注册申请
    - 删除用户
    - 重置用户密码
    - 管理测试SN
    - 管理Android版本
    - 管理版本类别

<!--more-->

- 软件工程师
    - 新建版本
    - 修改版本
    - 上传固件包
    - 发布Alpha测试

- 测试工程师
    - 撤回Alpha测试
    - 发布Beta测试

- 运营工程师
    - 撤回Beta测试
    - 发布Release版本

### 注册用户

#### 前端注册请求

saga/homeSaga.js
```javascript
export function* register (data) {
    yield put({type:IndexActionTypes.FETCH_START});
    try {
        return yield call(puter, '/user', data)
    } catch (error) {
        yield put({type:IndexActionTypes.SET_MESSAGE,msgContent:'注册失败',msgType:0});
    } finally {
        yield put({type: IndexActionTypes.FETCH_END}); 
    }
}
```

#### 后端处理注册请求

routes/user.js
```javascript
//注册
router.put('/', validRequest.validUsernamePassword, user.register);
```
validRequest中间件用来校验用户名密码。

controller/user.js
```javascript
exports.register = (req, res, next) => {
  const {
    type,
    username,
    password} = req.body;
  var newUser = new User();
  newUser.username = username;
  newUser.password = newUser.encryptPassword(password);
  newUser.type = type;
  newUser.state = "pending";
  
  newUser.save()
    .then(user => {
      responseClient(res, 200, errorCode.ERROR_CODE_SUCCESS, "注册成功，请联系管理员批准");
    })
    .catch(err => {
      console.log(err.message);
      responseClient(res, 406, errorCode.ERROR_CODE_SERVER, err.message);
    })
}
```
创建用户时，初始状态为pending，存储密码用密文。


### 批准用户

批准用户需要管理员身份。

#### 前端发送批准请求
adminManagerUserSaga.js
```javascript
function* acceptUser(username) {
    yield put({type: IndexActionTypes.FETCH_START});
    try {
        return yield call(post, `/user_manager/accept`, {username});
    } catch (err) {
        yield put({type: IndexActionTypes.SET_MESSAGE, msgContent: '网络请求错误', msgType: 0});
    } finally {
        yield put({type: IndexActionTypes.FETCH_END})
    }
}
```

#### 后端处理批准请求
routes/user_manager.js
```javascript
//批准用户
router.post('/accept', validRequest.validUserType('admin'), user.acceptUser);
```

controller/user_manager.js
```javascript
exports.acceptUser = (req, res, next) => {
  const {
    username,
  } = req.body;
  
  if(!username) {
    responseClient(res, 401, errorCode.ERROR_CODE_CLIENT, "未知用户名");
    return;
  }
  User.findOne({username: username}).exec()
    .then(user => {
      if(!user) {
        throw new Error(`未知用户名:${username}`)
      } else if(user.state === 'accept') {
        return Promise.resolve(user);
      }
      user.state = 'accept';
      return user.save();
    })
    .then(user => {
      responseClient(res, 200, errorCode.ERROR_CODE_SUCCESS, `${username}已批准`);
    })
    .catch(error => {
      console.log('acceptUser', error.message);
      responseClient(res, 401, errorCode.ERROR_CODE_CLIENT, `批准用户失败`);
    })
}
```

其他用户管理操作的流程是类似的。

以上所有的源代码都可以在[https://github.com/yinlijun2004/android_ota_system](https://github.com/yinlijun2004/android_ota_system)中找到。