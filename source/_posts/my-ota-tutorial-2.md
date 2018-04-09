---
title: 从零搭建Android OTA系统——用户注册登录前端界面实现
date: 2018-04-09 17:17:31
tags: [express, react, nodejs, antd]
categories: 从零搭建Android OTA系统
---
OTA的前端界面，打算用react实现，采用的库有
- [antd](https://ant.design/index-cn) 目前比较流行UI组件库。
- [axios](https://www.npmjs.com/package/axios) 网络请求库。
- [redux-saga](https://www.npmjs.com/package/redux-saga) 一个处理异步流处理的库。

目录结构：
```
src
 - components
 - containers
 - reducers
 - sagas
```
components用来存放无状态组件

containers用来存放状态组件

reducers用来存放reducer文件

sagas用来存放redux-saga文件

#### component和container的区别
我的理解，component是无状态、可复用的，只负责渲染的组件，它的输入是Props，输出是render的结果。

container负责和redux连接，从store里面获取数据。

#### 界面
登录和注册组件，可以作为一个component，用一个Tabs组件包装。
```javascript
export default class Login extends Component {
    constructor(props) {
        super(props);
        this.shouldComponentUpdate = PureRenderMixin.shouldComponentUpdate.bind(this)
    }


    render() {
        const { login, register } = this.props;
        return (
            <Tabs defaultActiveKey="1" tabBarStyle={{ textAlign: 'center' }} className={ style.container }>
                <TabPane tab="登录" key="1">
                    <LoginForm login={ login }/>
                </TabPane>
                <TabPane tab="注册" key="2">
                    <RegisterForm register={ register }/>
                </TabPane>
            </Tabs>
        )
    }
}
```

##### reducer
```javascript
export const actions = {
    login: function (username, password) {
        return {
            type: actionsTypes.USER_LOGIN,
            username,
            password
        }
    },
    logout: function () {
        return {
            type: actionsTypes.USER_LOGOUT,
        }
    },
    register: function (data) {
        return {
            type: actionsTypes.USER_REGISTER,
            data
        }
    },
}
```
data是注册表单的内容

#### saga
```javascript
export function* login(username, password) {
    yield put({type: IndexActionTypes.FETCH_START});
    try {
        return yield call(post, '/user/login', {username, password})
    } catch (error) {
        console.log("login failed", error.message);
    } finally {
        yield put({type: IndexActionTypes.FETCH_END});
    }
}

export function* loginFlow() {
    while (true) {
        let request = yield take(IndexActionTypes.USER_LOGIN);
        let response = yield call(login, request.username, request.password);
        if(response && response.code === 0){
            yield put({type:IndexActionTypes.SET_MESSAGE,msgContent:'登录成功!',msgType:1});
            yield put({type:IndexActionTypes.RESPONSE_USER_INFO,data:response.data})
        } else {
            yield put({type:IndexActionTypes.SET_MESSAGE,msgContent: (response && response.message) || '登录失败',msgType:0});
        }
    }
}

export function* logout() {
    yield put({type: IndexActionTypes.FETCH_START});
    try {
        return yield call(get, '/user/logout')
    } catch (error) {
        console.log("logout failed", error.message);
    } finally {
        yield put({type: IndexActionTypes.FETCH_END});
    }
}

export function* logoutFlow() {
    while(true) {
        let request = yield take(IndexActionTypes.USER_LOGOUT);
        let response = yield call(logout);
        yield put({type:IndexActionTypes.SET_MESSAGE,msgContent:'已退出当前账号',msgType:1});
        yield put({type:IndexActionTypes.RESPONSE_USER_INFO,data:{}})
    }
}

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

export function* registerFlow () {
    while(true){
        let request = yield take(IndexActionTypes.USER_REGISTER);
        let response = yield call(register, request.data);
        if(response && response.code === 0){
            yield put({type:IndexActionTypes.SET_MESSAGE,msgContent:'注册成功，请联系管理员批准!',msgType:1});
            yield put({type:IndexActionTypes.RESPONSE_USER_INFO,data:response.data})
        }
    }
}
```

#### 网络请求
为了和saga的操作put分开，axios的put方法被我封装成了puter。
```javascript
import axios from 'axios'

let config = {
    baseURL: '/api',
    transformRequest: [
        function (data) {
            let ret = '';
            for (let it in data) {
                ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
            }
            return ret
        }
    ],
    transformResponse: [
        function (data) {
            return data
        }
    ],
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8',
    },
    timeout: 10000,
    responseType: 'json'
};

axios.interceptors.response.use(function(res){
    //相应拦截器
    return res.data;
});

export function get(url, params) {
    console.log("url", url);
    config.params = params;
    return axios.get(url, config)
}

export function post(url, data) {
    console.log("url", url, "data", data);
    if(config.params) {
        delete config.params
    }
    return axios.post(url, data, config)
}

export function puter(url, data) {
    console.log("url", url, "data", data);
    if(config.params) {
        delete config.params
    }
    return axios.put(url, data, config);
}

export function deleter(url, param) {
    console.log("url", url, "param", param);
    config.params = param;
    return axios.delete(url, config);
}
```

以上所有的源代码都可以在[https://github.com/yinlijun2004/android_ota_system](https://github.com/yinlijun2004/android_ota_system)中找到。


