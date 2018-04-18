---
title: 从零搭建Android OTA系统——七牛云存储接口实现
date: 2018-04-18 15:34:06
tags:  [express, react, nodejs, antd]
categories: 从零搭建Android OTA系统
---

我们的OTA的固件存放在七牛，需要用到他们的一些SDK。
- [Nodejs sdk](https://developer.qiniu.com/kodo/sdk/1289/nodejs)
- [js sdk](https://developer.qiniu.com/kodo/sdk/1283/javascript)
- [Android sdk](https://developer.qiniu.com/kodo/sdk/1236/android) 

### 上传流程

<!-- more -->

#### 客户端请求生成上传凭证

adminManagerFirmwareSaga.js
```javascript
function* getUpToken(tokenInfo) {
    return yield call(get, `/firmware_manager/${tokenInfo.curFirmwareId}/uptoken`, {prevFirmwareId: tokenInfo.prevFirmwareId});
}
```

#### 服务端路由返回上传凭证
routes/firwmare_manager.js
```javascript
router.get('/:id/uptoken',
    validRequest.validUserType('program'), firmwareManager.createUploadToken);
```

controller/firmware_manager.js
```javascript
exports.createUploadToken = (req, res, next) => {
  const {
    prevFirmwareId,
  } = req.query;
  const id = req.params.id;
  Firmware.findById(id).exec() 
    .then(item => {
      if(!item) {
        throw new Error(`firmware id ${id} not found`);
      }
      const isFullPackage = !prevFirmwareId;
      const options = {
          expires: 36000,
          callbackUrl: `${config.domain}/api/firmware_manager/${item._id}/notify_package_uploaded`,
      };
      
      const { _id, firmwareClass } = item;
      const currentFirmwareCode = item.firmwareCode;

      if(isFullPackage) {
        const path = `${firmwareClass}/${currentFirmwareCode}/${currentFirmwareCode}.zip`;
        options.scope = `${qiniuConfig.bucketFirmware}:${path}`;
        options.callbackBody = `{"currentFirmwareCode": "${currentFirmwareCode}", "hash":"$(etag)"}`;
        options.callbackBodyType= 'application/json';
        const token = qiniu.createUptoken(qiniuConfig.bucketFirmware, path, options);

        responseClient(res, 200, errorCode.ERROR_CODE_SUCCESS, "create upload token success", {token: token, key: path});
      } else {
        //diff package
        Firmware.findById(prevFirmwareId).exec()
          .then(item => {
            if(!item) {
              throw new Error(`previous firmware id ${prevFirmwareId} not found`);
            }
            if(item.firmwareClass !== item.firmwareClass) {
              throw new Error(`firmware class not match`);
            }
            const prevFirmwareCode = item.firmwareCode;
            const path = `${firmwareClass}/${currentFirmwareCode}/diff_${currentFirmwareCode}_${prevFirmwareCode}.zip`;
            options.scope = `${qiniuConfig.bucketFirmware}:${path}`;
            options.callbackBody =`{"prevFirmwareCode": "${prevFirmwareCode}", "hash":"$(etag)"}`;
            options.callbackBodyType = 'application/json'
            const token = qiniu.createUptoken(qiniuConfig.bucketFirmware, path, options);

            responseClient(res, 200, errorCode.ERROR_CODE_SUCCESS, 
                    "create upload token success", {token: token, key: path}, options);
          });
      }
    })
    .catch(error => {
      responseClient(res, 406, errorCode.ERROR_CODE_SERVER, error.message);
    })
}
```
根据prevFirmwareId参数是否存在，判断是否是全包还是差分包，两者的路径不同，最后调用封装的qiniu api接口。
callbackBody是七牛的回调请求参数，etag用来保存文件的hash，用来Android客户端校验固件完整性用的。

qiniu_proxy.js
```javascript
var qiniu = require('qiniu');
var qiniuConfig = require('../config').qiniuConfig;

const accessKey = qiniuConfig.accessKey;
const secretKey = qiniuConfig.secretKey;

qiniu.conf.ACCESS_KEY = accessKey;
qiniu.conf.SECRET_KEY = secretKey;
const mac = new qiniu.auth.digest.Mac(accessKey, secretKey);
var config = new qiniu.conf.Config();
var bucketManager = new qiniu.rs.BucketManager(mac, config);

exports.createUptoken = (bucket, key, options) => {
  if(!key) {
    throw new Exception("empty key");
  }
  if(!bucket) {
    throw new Exception("empty bucket");
  }
  var keyToOverwrite = key;
  options = options || {
      scope: bucket + ":" + keyToOverwrite,
  }
  var putPolicy = new qiniu.rs.PutPolicy(options);
  const token = putPolicy.uploadToken(mac);

  return token;
}

const getDomainByBucketName = (bucket) => {
  const buckets = qiniuConfig.buckets;
  for(let item of buckets) {
    if(item.name === bucket) {
      return item.domain;
    }
  }
}

exports.createPrivateDownloadUrl = (bucket, key) => {
  const privateDomain = getDomainByBucketName(bucket);
  if(privateDomain) {
    const deadline = parseInt(Date.now() / 1000) + 3600; // 1小时过期
    const privateDownloadUrl = bucketManager.privateDownloadUrl(privateDomain, key, deadline);

    return privateDownloadUrl;
  }
}

exports.createPublicDownloadUrl = (bucket, key) => {
  const publicDomain = getDomainByBucketName(bucket);
  if(publicDomain) {
    const publicDownloadUrl = bucketManager.publicDownloadUrl(privateDomain, key);

    return publicDownloadUrl;
  }
}
```
#### 前端上传固件
FileUploader.js
```javascript
uploadFirmwarePackage(file, key, uptoken) {
    var observer = {
    next: function (res) {
        this.setState({uploading: true, status:'active', percent: Math.floor(res.total.percent)})
    }.bind(this),
    error: function (err) {
        console.log("upload err", err);
        this.setState({uploading: false, status:'exception', token: null});
    }.bind(this),
    complete: function () {
        this.setState({key: undefined, uploading: false, status:'success', fileList: [], token: null});
        this.props.onUploadSuccess();
    }.bind(this)
    };
    startUploadFirmwarePackage(file, key, uptoken, observer);
}
```
FileUploader是个React组件，将上传状态放在组件内部，本来想放在saga里面统一管理，但是发现generate跟七牛上传observer回调函数搞不通。

qiniuApi.js
```javascript
export function startUploadFirmwarePackage(file, key, uptoken, observer) {
  let observable = qiniu.upload(file, key, uptoken, putExtra, config);
  observable.subscribe(observer);
}
```

#### 服务器处理上传通知
七牛上传完之后，会给业务服务器发一个成功的请求，将固件信息在保存到数据库中。

controller/firmware_manager.js
```javascript
exports.notifyPackageUploaded = (req, res, next) => {
  const id = req.params.id;
  const {
    prevFirmwareCode,
    hash,
  } = req.body;
  
  Firmware.findById(id).exec()
    .then(item => {
      if(!item) {
        throw new Error(`firmware id ${id} not found`);
      }
      const firmwareClass  = item.firmwareClass;
      const currentFirmwareCode = item.firmwareCode;
      const isFullPackage = !prevFirmwareCode;
      if(isFullPackage) {
        //full package uploaded notification
        item.fullPackagePath = {
          firmwarePath: `${firmwareClass}/${currentFirmwareCode}/${currentFirmwareCode}.zip`,
          packageHash: hash,
        }
        return item.save();
      } else {
        //diff package uploaded notification
        const path = `${firmwareClass}/${currentFirmwareCode}/diff_${currentFirmwareCode}_${prevFirmwareCode}.zip`;
        const package = item.diffPackagePath.find(p => {
          return p.firmwareCode === prevFirmwareCode;
        });
        if(package) {
          //override original path
          package.firmwarePath = path;
          package.packageHash = hash;
        } else {
          item.diffPackagePath.push({firmwareCode: prevFirmwareCode, firmwarePath: path, packageHash: hash});
        }
        return item.save();
      }
    })
    .then(item => {
        responseClient(res, 200, errorCode.ERROR_CODE_SUCCESS, "upload success");
    }) 
    .catch(error => {
      console.log("error", error.message);
      responseClient(res, 500, errorCode.ERROR_CODE_SERVER, error.message);
    });
}
```


### 客户端下载流程

#### 获取下载凭证
routes/firmware_common.js
```javascript
router.get('/firmware', firmwareCommon.getDownloadUrl);
```

controller/firmware_common.js
```javascript
exports.getDownloadUrl = (req, res, next) => {
  const { 
    firmwareClass, 
    currentFirmwareCode,
    targetFirmwareCode
  } = req.query;
  //console.log("getDownloadUrl", req.body);
  if(!firmwareClass || !targetFirmwareCode) {
    responseClient(res, 404, errorCode.ERROR_CODE_CLIENT, `invalid firmware class or target firmware code`);
    return;
  }
  
  Firmware.findOne({firmwareClass: firmwareClass, firmwareCode: targetFirmwareCode}).exec()
    .then(item => {
      if(!item) {
        throw new Error(`${targetFirmwareCode} for class ${firmwareClass} not found`);
      }
      
      //first try diff package download
      let diffPackage;
      if(currentFirmwareCode) {
        diffPackage = item.diffPackagePath.find(p => {
          if(firmwareUtils.compareFirmwareCode(p.firmwareCode, currentFirmwareCode) === 0) {
            //bingo! found it
            return p;
          }
        })
      }
      if(diffPackage && diffPackage.firmwarePath) {
        //ok make a download url
        const url = qiniu.createPrivateDownloadUrl(qiniuConfig.bucketFirmware, diffPackage.firmwarePath);
        //console.log("diffpackage", "key", diffPackage.firmwarePath, "hash", diffPackage.packageHash, "url", url);
        responseClient(res, 200, errorCode.ERROR_CODE_SUCCESS, "found url success", {url: url, hash: diffPackage.packageHash});  
      } else {
        //no diff package, try full package download
        if(!item.fullPackagePath) {
          throw new Error(`${targetFirmwareCode} package for class ${firmwareClass} not found, may be has not uploaded`);
        }
        let key = item.fullPackagePath.firmwarePath;
        let hash = item.fullPackagePath.packageHash;
        const url = qiniu.createPrivateDownloadUrl(qiniuConfig.bucketFirmware, key);
        //console.log("fullpackage", "key", key, "hash", hash, "url", url);
        responseClient(res, 200, errorCode.ERROR_CODE_SUCCESS, "found url success", {url: url, hash: hash});
      }
    })
    .catch(error => {
      responseClient(res, 404, errorCode.ERROR_CODE_CLIENT, error.message);
    })
}
```

#### Android下载
```java
public void downloadFirmware(final String filePath, final Subscriber<Integer> subscriber) {
        if(downloadListener != null) {
            downloadListener.onStartDownload();
        }
        setUpdateState(UpdateStateManager.STATE_DOWNLOADING);
        Map<String, String> params = OtaParamFactory.createDownloadFirmwareParam(mContext);
        otaServerApi.getDownloadUtl(params)
                .subscribeOn(Schedulers.io())
                .map(new Func1<VersionDownloadUrl, String>() {
                    @Override
                    public String call(VersionDownloadUrl url) {
                        mFirmwareManager.savePackageHash(url.getData().getHash());
                        return url.getData().getUrl();
                    }
                })
                .flatMap(new Func1<String, rx.Observable<ResponseBody>>() {
                    @Override
                    public rx.Observable<ResponseBody> call(String url) {
                        return qiniuServiceApi.downloadFirmware(url);
                    }
                })
                 .subscribeOn(Schedulers.io())
                 .unsubscribeOn(Schedulers.io())
                 .map(new Func1<ResponseBody, String>() {
                     @Override
                     public String call(ResponseBody responseBody) {
                         saveToDisk(responseBody, filePath);
                         return mPackagePath;
                     }
                 })
                 .observeOn(Schedulers.io())
                 .map(new Func1<String, Integer>() {
                    @Override
                    public Integer call(String path) {
                        setUpdateState(UpdateStateManager.STATE_VERIFYING);
                        int result = checkPackageHash();
                        if(result != Util.OTAresult.CHECK_OK) {
                            return result;
                        }
                        return checkUpdagePackage(path);
                    }
                 })
                 .observeOn(AndroidSchedulers.mainThread())
                 .subscribe(new Subscriber<Integer>() {
                     @Override
                     public void onCompleted() {
                         subscriber.onCompleted();
                     }

                     @Override
                     public void onError(Throwable e) {
                         setUpdateState(UpdateStateManager.STATE_FAIL);
                         subscriber.onError(e);
                     }

                     @Override
                     public void onNext(Integer result) {
                         if(result == Util.OTAresult.CHECK_OK) {
                             if(downloadListener != null) {
                                 downloadListener.onFinishDownload();
                             }
                             setUpdateState(UpdateStateManager.STATE_READY);
                         } else {
                             if(downloadListener != null) {
                                 downloadListener.onFail("download fail code:" + result);
                             }
                             setUpdateState(UpdateStateManager.STATE_FAIL);
                         }
                         subscriber.onNext(result);
                     }
                 });
    }
    ```