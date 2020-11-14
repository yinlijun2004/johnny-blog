---
title: CocosCreator实践
date: 2020-11-14 11:47:06
tags: [CocosCreator, bash]
categories: CocosCreator
---

最近一段实践接触过CoCosCreator，经过一段时间的熟悉，总结出三个方面的时间：多环境构建、多语言支持、Protobuf支持，在此做个总结。

### 多环境构建

CocosCreator利用GUI可以一键构建出游戏版本，但是如果需要切换构建环境，比如说开发环境，测试环境、正式环境，发布不同的版本，需要手动的修改代码，然后进行构建，就比较麻烦。

此时，可以利用命令行脚本进行构建。
```bash
CocosCreator --path . --build
```
既然可以利用脚本构建，那可以做的事情就多了，可以利用bash做出很多随心所欲的功能。

将环境变量抽离出来，放在单独的目录。
```bash
GameEnv.dev.ts
GameEnv.test.ts
GameEnv.prod.ts
```
根据当前脚本脚本，动态将对应的环境文件，拷贝到项目中，然后进行编译。

一个典型的GameEnv.ts，应该像如下所示
```javascript
export default class GameEnv {
    static apiUrl:string = "xxxxxx";
    static wsUrl:string = "xxxxx";
}
```

不同的版本最好发布到不同的目标目录，同理，可以将不同的builder.json抽离出来。
```javascript
builder.dev.json
builder.test.json
builder.prod.json
```
不同的json里面配置不同的编译变量。
```javascript
{
  "platform": "web-mobile",
  "actualPlatform": "web-mobile",
  "template": "link",
  "buildPath": "xxxxxxxxxxxxxxxxxxxxxxx",
  "debug": false,
  "sourceMaps": false,
  "embedWebDebugger": false,
  "previewWidth": "1280",
  "previewHeight": "720",
  "useDebugKeystore": true,
  "keystorePath": "",
  "keystorePassword": "",
  "keystoreAlias": "",
  "keystoreAliasPassword": "",
  "apiLevel": "",
  "appABIs": [],
  "vsVersion": "auto",
  "buildScriptsOnly": false
}

```
编译时将不同的配置替换到源目录的local/builder.json文件。

### 多语言支持
更新了2.3版本后，官方i18n已经不维护了，只好自己实现。
我看网上也有很多人方法，大致分为两种。
- 动态加载语言包js 适合字符串数量非常大的情况
- 写在一个文件里 适合字符串数量不多的情况

我的项目适用于后一种情况。

新增一个GameLocal.ts类，内容如下：
```javascript
const LANGS = {
    "zh-Hans": {
        "t_title": "游戏"
    },
    "ja": {
        "t_title": "ゲーム"
    },
    "en": {
        "t_title": "Game"
    }
}
    
    
function getLanguage() {
    const lang1 = cc.sys.language.split("-"), lang2 = cc.sys.language.split("_");
    const lang = lang2.length > lang1.length ? lang2 : lang1;
    const langAsset = ["en", "zh-Hans", "zh-Hant", "ja", "ko"]
    let index = -1;
    if(lang.length > 0) {
        const l = lang[0];
        index = langAsset.indexOf(l);
        if(lang.length == 2 && l === 'zh') {
            //判断中文
            const c = lang[1];
            if(c === "TW" || c === 'tw' || c === 'HK' || c === 'hk') {
                //繁体
                index = langAsset.indexOf("zh-Hant");
            }  else {
                //简体
                index = langAsset.indexOf("zh-Hans");
            }
        }
    } 

    if(index < 0) {
        return "zh-Hans";
    }
    return langAsset[index];
};

let lang = getLanguage();
let curLanMap = LANGS[lang];
export default curLanMap;
```

使用的时候，只要根据key就取内容就可以进行动态设置。
如果是不用动态设置的Label，可以只写key，然后重写Label的onLoad方法。
参考：
```
https://blog.csdn.net/qq_15009269/article/details/107014716
```

### protobuf支持

我的项目需要玩家与玩家之间实时传输数据，要用到websocket，并且数据量还挺大，对序列化、反序列化、传输的性能要求较高，所以将序列化工具由原先的json方式改为protobuf。

据说protobuf的性能是json的2~50倍，尤其是数据有大量数值型类型的时候，因此还是值得的。

proto文件格式，和转换成js就不说了，这里只记录用typescript的websocket库发送json的时候踩的坑。

#### 设置binaryType
发送protobuf数据的时候，websocket的binaryType要设为<b>arraybuffer</b>
```typescript
this.ws_.binaryType = "arraybuffer";
```

#### 接收处理
protobuf的数据是ArrayBuffer类型的，并且要将<b>ArrayBuffer</b>转换成<b>Uint8Array</b>。
```typescript
onMessage(event:MessageEvent) {
    if (event.data) {
        if (typeof(event.data) === "object") {
            let udata = new Uint8Array(event.data);
            //protobuf数据处理
        }
        else if (typeof(event.data) === "string"){
            let data = JSON.parse(event.data);
            //json数据处理
        }
    }
}
```

#### 发送处理
同理，发送的时候也需要protobuf打包的数据(<b>Uint8Array</b>类型)转换成<b>ArrayBuffer</b>
```typescript
typedArrayToBuffer(array: Uint8Array): ArrayBuffer {
    return array.buffer.slice(array.byteOffset, array.byteLength + array.byteOffset)
}

sendMsg(msg:any) {
    var buffer = this.typedArrayToBuffer(msg);
    this.ws_.send(buffer);
}

```