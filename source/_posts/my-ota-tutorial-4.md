---
title: 从零搭建Android OTA系统——版本发布规则实现
date: 2018-04-19 09:32:37
tags:  [express, react, nodejs, antd]
categories: 从零搭建Android OTA系统
---

版本包含两部分
- 版本元信息
存放在业务服务器上，存储版本号，版本类型，Android版本号等等。
- 版本固件包 存放在七牛云服务器上。

<!-- more -->

总体框图如下：
![](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/5e891902dcc45c5a8384d25d7779d5eb937fd0183a1fd67496650f84621404c5ea46a384b1d2b8836819fa09c44df7e0?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=OTA%E6%A1%86%E5%9B%BE.png&size=1024)


发布版本规则，其实就是改变版本的状态，我将版本分为四个状态。
- new 新建状态，可以上传固件包，编辑版本信息。
- alpha Alpha状态，可以在后台配置支持alpha测试的SN号，对应的终端设备能收到升级提示。
- beta Beta状态，可以在后台配置支持beta测试的SN号，对应的终端设备能收到升级提示。
- Release状态，所有的设备都能收到升级提示。

版本状态迁移：

![版本状态迁移：](https://picabstract-preview-ftn.weiyun.com:8443/ftn_pic_abs_v2/f8279f93bdf2d68e2d599d0498b747cfe83430bbd7161ce4a39268a1156af75ff42effdec511f08e0dc4d6489e58dbbe?pictype=scale&from=30113&version=2.0.0.2&uin=474724984&fname=firmware_process.png&size=1024)

### 创建版本

#### 前端上传版本信息
adminManagerFirmwareSaga.js
```javascript
export function* createFirmwareInfo(data) {
  yield put({type: IndexActionTypes.FETCH_START});
  try {
    data.releaseNotes = JSON.stringify(data.releaseNotes);
    return yield call(puter, '/firmware_manager/firmware', data);
  } catch (error) {
    //
  } finally {
    yield put({type: IndexActionTypes.FETCH_END});
  }
}
```

#### 后端处理新建版本
routes/firmware_manager.js
```javascript
//新建版本
router.put('/firmware', 
    validRequest.validUserType('program'), firmwareManager.createFirmware);
```
这里用一个中间件过滤用户类型，只有program的用户才允许创建版本。

controller/firwmare_manager.js
```javascript
exports.createFirmware = (req, res, next) => {
  const {
    firmwareCode, 
    firmwareDescription, 
    androidVersion, 
    firmwareClass,
    releaseNotes,
  } = req.body; 
  if(!firmwareCode || !firmwareDescription || !androidVersion || !firmwareClass || !releaseNotes) {
    responseClient(res, 401, errorCode.ERROR_CODE_CLIENT, "invalid param");
  }
  let notes;
  try {
     notes = JSON.parse(releaseNotes);
  } catch(err) {
    console.log(err.message);
    responseClient(res, 401, errorCode.ERROR_CODE_CLIENT, "release notes parse failed");
    return;
  }
  Promise.all(
    [
      Firmware.findOne({firmwareClass: firmwareClass, firmwareCode: firmwareCode}).exec(),
      Firmware.find({firmwareClass: firmwareClass}).exec()
    ]).then(([newFirmware, allFirmwares]) => {
      console.log("newFirmware:", newFirmware, "allFirmwares:", allFirmwares);
      if(newFirmware) {
        throw new Error(`firmware code: ${firmwareCode} in class ${firmwareClass} already exsist!`);
      }
      console.log("release notes:", notes);
      let data = {
        firmwareCode: firmwareCode,
        firmwareDescription: firmwareDescription,
        androidVersion: androidVersion,
        firmwareClass: firmwareClass,
        releaseNotes: notes,
        firmwareState: "new",
      };
      if(allFirmwares && allFirmwares.length > 0) {
        let diffPath = allFirmwares.filter(f => {
          return firmwareUtils.compareFirmwareCode(firmwareCode, f.firmwareCode) > 0;
        }).map(f => {
          return {firmwareCode: f.firmwareCode, firmwareId: f._id};
        })
        data.diffPackagePath = diffPath;
      }
      firmwareUtils.assertFirmwareContent(data);
      return new Firmware(data).save();
    })
    .then(firmware => {
      responseClient(res, 200, errorCode.ERROR_CODE_SUCCESS, "firmware created");
    })
    .catch(error => {
      console.log('firmware save error', error.message);
      responseClient(res, 406, errorCode.ERROR_CODE_CLIENT, error.message);
    })
}
```
创建版本后，初始化版本状态为new。

### 修改版本状态

#### 前端发送请求
```javascript

function* firmwarePublishAlpha(id) {
  yield put({type: IndexActionTypes.FETCH_START})
  try {
    return yield call(post, `/firmware_manager/${id}/alpha`);
  } catch(error) {
    //
  } finally {
    yield put({type: IndexActionTypes.FETCH_END});
  }
}

function* firmwareReverseAlpha(id) {
  yield put({type: IndexActionTypes.FETCH_START})
  try {
    return yield call(deleter, `/firmware_manager/${id}/alpha`);
  } catch(error) {
    //
  } finally {
    yield put({type: IndexActionTypes.FETCH_END});
  }
}

function* firmwarePublishBeta(id) {
  yield put({type: IndexActionTypes.FETCH_START})
  try {
    return yield call(post, `/firmware_manager/${id}/beta`);
  } catch(error) {
    //
  } finally {
    yield put({type: IndexActionTypes.FETCH_END});
  }
}

function* firmwareReverseBeta(id) {
  yield put({type: IndexActionTypes.FETCH_START})
  try {
    return yield call(deleter, `/firmware_manager/${id}/beta`);
  } catch(error) {
    //
  } finally {
    yield put({type: IndexActionTypes.FETCH_END});
  }
}

function* firmwarePublishRelease(id) {
  yield put({type: IndexActionTypes.FETCH_START})
  try {
    return yield call(post, `/firmware_manager/${id}/release`);
  } catch(error) {
    //
  } finally {
    yield put({type: IndexActionTypes.FETCH_END});
  }
}
```
接口设计遵循restful原则。

#### 后端处理

routes/firwmare_manager.js
```javascript
//发布alpha测试
router.post('/:id/alpha',
    validRequest.validUserType('program'), firmwareManager.changeState("new", "alpha"));

//撤回alpha测试
router.delete('/:id/alpha',
    validRequest.validUserType('test'), firmwareManager.changeState("alpha", "new"));

//发布beta测试
router.post('/:id/beta',
    validRequest.validUserType('test'), firmwareManager.changeState("alpha", "beta"));

//撤回beta测试
router.delete('/:id/beta',
    validRequest.validUserType('operation'), firmwareManager.changeState("beta", "alpha"));

//发布release版本
router.post('/:id/release',
    validRequest.validUserType('operation'), firmwareManager.changeState("beta", "release"));
```
上面的api接口，统一放在changeState里面做。

controller/firwmare_manager.js
```javascript
exports.changeState = (prevState, nextState) => {
  return (req, res, next) => {
    const id = req.params.id;

    Firmware.findById(id).exec() 
    .then(item => {
      if(!item) {
        throw new Error(`firmware id ${id} not found`);
      }
      if(item.firmwareState != prevState) {
        throw new Error(`user has no permission change ${prevState} to ${nextState}`);
      }
      
      if(prevState === "new" && (!item.fullPackagePath || item.fullPackagePath.length === 0)) {
        throw new Error(`state change failed, full package not found`);
      }
      item.firmwareState = nextState;
      item.save();
    })
    .then(item => {
      responseClient(res, 200, errorCode.ERROR_CODE_SUCCESS, "firmware publish success");
    })
    .catch(error => {
      responseClient(res, 404, errorCode.ERROR_CODE_SERVER, error.message);
    })
  } 
}
```
以上所有的源代码都可以在[https://github.com/yinlijun2004/android_ota_system](https://github.com/yinlijun2004/android_ota_system)中找到。