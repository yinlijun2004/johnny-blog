---
title: https双向验证功能的实现
date: 2016-11-28 08:51:23
tags: [android, nodejs, https, openssl]
---
本文介绍一个简单echo服务器的实现，服务端用nodejs，客户端用android。

## <font size='6em'>用openssl一系列证书</font>

### <font size='5em'>生成自己的CA根证书</font>

#### <font size='4em'>生成跟证书私钥ca.key</font>
```
$ openssl genrsa -des3 -out ca.key 1024
```
#### <font size='4em'>生成X.509证书签名请求文件ca.csr</font>
在生成ca.csr的过程中，会让输入一些组织信息等。
```
$ openssl req -new -key ca.key -out ca.csr 
```
<!-- more --> 

输出如下
```
Enter pass phrase for ca.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:GuangDong
Locality Name (eg, city) []:ShenZhen
Organization Name (eg, company) [Internet Widgits Pty Ltd]:IBoxPay
Organizational Unit Name (eg, section) []:IBoxPay
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:admin@iboxpay.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

#### <font size='4em'>生成X.509格式的CA根证书ca.crt</font>
```
$ openssl x509 -req -days 365 -in ca.csr -out ca.crt -signkey ca.key
```
输出如下
```
Signature ok
subject=/C=CN/ST=GuangDong/L=ShenZhen/O=IBoxPay/OU=IBoxPay/emailAddress=admin@iboxpay.com
Getting Private key
Enter pass phrase for ca.key:
```

### <font size='5em'>生成服务端的证书</font>

#### <font size='4em'>生成服务端私钥文件 server.key</font>
```
$ openssl genrsa -des3 -out server.key 1024
```
#### <font size='4em'>服务端需要向CA机构申请签名证书，在申请签名证书之前依然是创建自己的证书签名请求文件server.csr</font>
这一步需要填写一个组织信息，不要跟根证书的组织的一样。另外Common Name填一个自己的域名（如果没有实际的域名也可以写，后面在/etc/hosts映射一个，我写的就是yinlijun.com），不要填localhost，android会报错。
```
openssl req -new -key server.key -out server.csr
```
输出如下
```
Enter pass phrase for server.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:GuangDong
Locality Name (eg, city) []:ShenZhen
Organization Name (eg, company) [Internet Widgits Pty Ltd]:yinlijun
Organizational Unit Name (eg, section) []:yinlijun
Common Name (e.g. server FQDN or YOUR name) []:yinlijun.com
Email Address []:admin@yinlijun.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []: 
```

#### <font size='4em'>删除私钥的密码，这一步非常**重要**，一定要执行,否则会影响后面的步骤。</font>
```
$ cp server.key server.key.passphrase
$ openssl rsa -in server.key.passphrase -out server.key
```
输出如下
```
Enter pass phrase for server.key.passphrase:
writing RSA key
```
#### <font size='4em'>签发服务器证书server.crt：</font>
```
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
输出如下：
```
Signature ok
subject=/C=CN/ST=GuangDong/L=ShenZhen/O=yinlijun/OU=yinlijun/CN=yinlijun.com/emailAddress=admin@yinlijun.com
Getting Private key
```
到现在为止，你目录下的文件应该有：
```
ls -la
total 36
drwxr-xr-x  2 user user 4096 Sep  5 16:19 .
drwxr-xr-x 12 user user 4096 Sep  5 16:09 ..
-rw-r--r--  1 user user  757 Sep  5 16:12 ca.crt
-rw-r--r--  1 user user  603 Sep  5 16:10 ca.csr
-rw-r--r--  1 user user  963 Sep  5 16:09 ca.key
-rw-r--r--  1 user user  757 Sep  5 16:19 server.crt
-rw-r--r--  1 user user  603 Sep  5 16:16 server.csr
-rw-r--r--  1 user user  887 Sep  5 16:18 server.key
-rw-r--r--  1 user user  951 Sep  5 16:17 server.key.passphrase
```

#### 生成之后察看服务器证书信息。
```
openssl x509 -in server.crt -text -noout
```

#### 生成服务器的pfx文件，这个文件node服务器要用到。
```
openssl pkcs12 -export -in server.crt -inkey server.key -certfile ca.crt -out server.pfx
```
输出如下
```
Enter Export Password:
Verifying - Enter Export Password:
```
#### 生成服务端的p12文件。，这个是为了生成服务端bks文件用的
```
openssl pkcs12 -export -clcerts -in server.crt -inkey server.key -out server.p12
```
### 下载一个bcprov-jdk16-141.jar，也是为了生成服务端bks文件要用到的。
```
下载地址：[http://www.java2s.com/Code/JarDownload/bcprov/bcprov-jdk16-141.jar.zip](http://www.java2s.com/Code/JarDownload/bcprov/bcprov-jdk16-141.jar.zip)
```
### 生成服务端的bks文件，这个android程序要用到
```
keytool -importkeystore -srckeystore server.p12 -srcstoretype pkcs12 -destkeystore server.bks -deststoretype bks -provider org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath bcprov-jdk16-141.jar
```
输出如下
```
输入目标密钥库口令:  
再次输入新密码: 
输入源密钥库口令:  
已成功导入别名 1 项。
已完成导入命令: 1 项成功导入，0 项失败或取消
```
因为要进行双向验证，还需要生成客户端证书。
#### 生成客户端密钥
```
openssl genrsa -des3 -out client.key 1024
```
#### 生成客户端证书请求签名文件
```
openssl req -new -out client.csr -key client.key
```
输出如下
```
Enter pass phrase for client.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:GuangDong 
Locality Name (eg, city) []:ShenZhen
Organization Name (eg, company) [Internet Widgits Pty Ltd]:ruochen
Organizational Unit Name (eg, section) []:ruochen
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:admin@ruochen.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

#### 创建一个自当前日期起有效期为十年的客户端证书，需要根证书和根密钥参与。
```
openssl x509 -req -in client.csr -out client.cert -signkey client.key -CA ca.crt -CAkey ca.key -CAcreateserial -days 3650
```
输入如下
```
Signature ok
subject=/C=CN/ST=GuangDong/L=ShenZhen/O=ruochen/OU=ruochen/emailAddress=admin@ruochen.com
Getting Private key
Enter pass phrase for client.key:
Getting CA Private Key
Enter pass phrase for ca.key:
yinlijun@yinlijun:~/personal_github/echo-https-server/keys$ ls
ca.crt  ca.csr  ca.key  ca.srl  client.cert  client.csr  client.key  server.crt  server.csr  server.key  server.key.passphrase  server.pfx
yinlijun@yinlijun:~/personal_github/echo-https-server/keys$ openssl pkcs12 -export -clcerts -in client.cert -inkey client.key -out client.p12
Enter pass phrase for client.key:
Enter Export Password:
Verifying - Enter Export Password:
```

#### 生成浏览器支持的p12文件
```
openssl pkcs12 -export -clcerts -in client.cert -inkey client.key -out client.p12
```
#### 将客户端证书文件client.crt和客户端证书密钥文件client.key合并成客户端证书安装包client.pfx
```
openssl pkcs12 -export -in client.crt -inkey client.key -out client.pfx
```

以上的文件我只用到了一部分，应该有替代关系，具体我也搞不清楚:)。 不同的实现方式有用到不同的文件。
我的android客户端用到了
- server.bks
- client.p12

看了网上的一些例子，好像server.bks可以用server.crt替代。

node用到了
- server.pfx

察看node的文档，server.pfx可以用server.crt和server.key替代。


## 服务端（nodejs）的代码
```javascript
var https =require('https'), fs = require('fs');

var options = {
    key: fs.readFileSync('./keys/server.key'),
    cert: fs.readFileSync('./keys/server.crt'),
};

var app = express();
var server = https.createServer(options, app);
server.listen(443, function() {
    console.log('Https server listening on port ' + 443);
});
```

## android应用自有证书的验证方式
将服务端证书拷贝到app资源目录下，一般是<project_dir>/assets/server.crt

### 方法一：直接根据server.crt初始化TrustManagerFactory
```java
    CertificateFactory cf = CertificateFactory.getInstance("X.509");
    InputStream caInput = new BufferedInputStream(getAssets().open("server.crt"));
    final Certificate ca;
    try {
        ca = cf.generateCertificate(caInput);
        Log.i(TAG, "ca=" + ((X509Certificate) ca).getSubjectDN());
        Log.i(TAG, "key=" + ((X509Certificate) ca).getPublicKey());
    } finally {
        caInput.close();
    }

    String keyStoreType = KeyStore.getDefaultType();
    Log.d(TAG, "keystore type:" + keyStoreType);
    KeyStore keyStore = KeyStore.getInstance(keyStoreType);
    keyStore.load(null, null);
    keyStore.setCertificateEntry("cert", ca);

    String tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
    Log.d(TAG, "tmfAlgorithm:" + tmfAlgorithm);
    TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(tmfAlgorithm);
    trustManagerFactory.init(keyStore);

    mSSLContext = SSLContext.getInstance("TLS");
    mSSLContext.init(null, trustManagerFactory.getTrustManagers(), null);

    URL url = new URL("https://yinlijun.com");
    HttpsURLConnection urlConnection =
            (HttpsURLConnection)url.openConnection();
    urlConnection.setSSLSocketFactory(mSSLContext.getSocketFactory());
    InputStream in = urlConnection.getInputStream();
    copyInputStreamToOutputStream(in, System.out);
} catch (CertificateException e) {
    e.printStackTrace();
} catch (IOException e) {
    e.printStackTrace();
} catch (NoSuchAlgorithmException e) {
    e.printStackTrace();
} catch (KeyManagementException e) {
    e.printStackTrace();
} catch (KeyStoreException e) {
    e.printStackTrace();
}
```
copyInputStreamToOutputStream方法如下：

```java
    private void copyInputStreamToOutputStream(InputStream in, PrintStream out) throws IOException {
        byte[] buffer = new byte[1024];
        int c = 0;
        while ((c = in.read(buffer)) != -1) {
            out.write(buffer, 0, c);
        }
    }
```

### 方法二 

```java
    try {
        CertificateFactory cf = CertificateFactory.getInstance("X.509");
        InputStream caInput = new BufferedInputStream(getAssets().open("server.crt"));
        final Certificate ca;
        try {
            ca = cf.generateCertificate(caInput);
            Log.i("Longer", "ca=" + ((X509Certificate) ca).getSubjectDN());
            Log.i("Longer", "key=" + ((X509Certificate) ca).getPublicKey());
        } finally {
            caInput.close();
        }

        // Create an SSLContext that uses our TrustManager
        SSLContext context = SSLContext.getInstance("TLSv1","AndroidOpenSSL");
        context.init(null, new TrustManager[]{
                new X509TrustManager() {
                    @Override
                    public void checkClientTrusted(X509Certificate[] chain,
                                                    String authType)
                            throws CertificateException {

                    }

                    @Override
                    public void checkServerTrusted(X509Certificate[] chain,
                                                    String authType)
                            throws CertificateException {
                        for (X509Certificate cert : chain) {

                            // Make sure that it hasn't expired.
                            cert.checkValidity();

                            // Verify the certificate's public key chain.
                            try {
                                cert.verify(((X509Certificate) ca).getPublicKey());
                            } catch (NoSuchAlgorithmException e) {
                                e.printStackTrace();
                            } catch (InvalidKeyException e) {
                                e.printStackTrace();
                            } catch (NoSuchProviderException e) {
                                e.printStackTrace();
                            } catch (SignatureException e) {
                                e.printStackTrace();
                            }
                        }
                    }

                    @Override
                    public X509Certificate[] getAcceptedIssuers() {
                        return new X509Certificate[0];
                    }
                }
        }, null);

        URL url = new URL("https://yinlijun.com/");
        HttpsURLConnection urlConnection =
                (HttpsURLConnection)url.openConnection();
        urlConnection.setSSLSocketFactory(context.getSocketFactory());
        InputStream in = urlConnection.getInputStream();
        copyInputStreamToOutputStream(in, System.out);
```


## 双向认证
单向验证只能验证服务器，如果服务器也想对客户端进行验证，即所谓（双向验证），需要在连接是一起发送客户端证书。

### 双向认证，服务器代码
```javascript
const tls = require('tls');
const fs = require('fs');

const options = {
  pfx: fs.readFileSync('./server.pfx'),
  passphrase: "123456",
  // This is necessary only if using the client certificate authentication.
  requestCert: true,
  rejectUnauthorized: true //如果接受也非认证链接，可以删除此行。
};
//需要双向认证才需要配置requestCert为true。
const server = tls.createServer(options, (socket) => {
  console.log('server connected',
              socket.authorized ? 'authorized' : 'unauthorized');
  socket.setEncoding('utf8');
  socket.on('data', (data) => {
      console.log(data);
      socket.write(data);
  });
  socket.on('end', (socket) => {
    console.log("socket closed");
  });
});
server.listen(8000, () => {
  console.log('server bound');
});

```
### 双向认证，android的代码，实现方式一

server.pfx和client.p12放到<project_dir>/assets/目录下
```java
    try {
        KeyStore trustStore = KeyStore.getInstance("bks");
        InputStream tsIn = getResources().getAssets().open("server.bks");

        KeyStore keyStore = KeyStore.getInstance("PKCS12");
        InputStream ksIn = getResources().getAssets().open("client.p12");

        try {
            keyStore.load(ksIn, "123456".toCharArray());
            trustStore.load(tsIn, "123456".toCharArray());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                ksIn.close();
            } catch (Exception ignore) {
            }
            try {
                tsIn.close();
            } catch (Exception ignore) {
            }
        }
        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance("X509");
        keyManagerFactory.init(keyStore, "123456".toCharArray());
        TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
        trustManagerFactory.init(trustStore);
        mSSLContext = SSLContext.getInstance("TLS");
        mSSLContext.init(null, trustManagerFactory.getTrustManagers(), null);

        mSSLSocket = (SSLSocket) mSSLContext.getSocketFactory().createSocket("yinlijun.com", 8000);
        mSSLSocket.startHandshake();
        //...
    } catch (IOException e) {
        e.printStackTrace();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
    } catch (KeyManagementException e) {
        e.printStackTrace();
    } catch (KeyStoreException e) {
        e.printStackTrace();
    } catch (UnrecoverableKeyException e) {
        e.printStackTrace();
    }

```

### 双向认证，android的代码，实现方式2，用server.crt替代server.bks

server.crt和client.p12放到<project_dir>/assets/目录下

```java
    try {
        CertificateFactory cf = CertificateFactory.getInstance("X.509");
        InputStream caInput = new BufferedInputStream(getAssets().open("server.crt"));
        final Certificate ca;
        try {
            ca = cf.generateCertificate(caInput);
            Log.i("Longer", "ca=" + ((X509Certificate) ca).getSubjectDN());
            Log.i("Longer", "key=" + ((X509Certificate) ca).getPublicKey());
        } finally {
            caInput.close();
        }

        KeyStore keyStore = KeyStore.getInstance("PKCS12");
        InputStream ksIn = getResources().getAssets().open("client.p12");

        try {
            keyStore.load(ksIn, "123456".toCharArray());
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                ksIn.close();
            } catch (Exception ignore) {
            }
        }
        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance("X509");
        keyManagerFactory.init(keyStore, "123456".toCharArray());

        // Create an SSLContext that uses our TrustManager
        SSLContext context = SSLContext.getInstance("TLSv1","AndroidOpenSSL");
        context.init(keyManagerFactory.getKeyManagers(), new TrustManager[]{
                new X509TrustManager() {
                    @Override
                    public void checkClientTrusted(X509Certificate[] chain,
                                                    String authType)
                            throws CertificateException {

                    }

                    @Override
                    public void checkServerTrusted(X509Certificate[] chain,
                                                    String authType)
                            throws CertificateException {
                        for (X509Certificate cert : chain) {

                            // Make sure that it hasn't expired.
                            cert.checkValidity();

                            // Verify the certificate's public key chain.
                            try {
                                cert.verify(((X509Certificate) ca).getPublicKey());
                            } catch (NoSuchAlgorithmException e) {
                                e.printStackTrace();
                            } catch (InvalidKeyException e) {
                                e.printStackTrace();
                            } catch (NoSuchProviderException e) {
                                e.printStackTrace();
                            } catch (SignatureException e) {
                                e.printStackTrace();
                            }
                        }
                    }

                    @Override
                    public X509Certificate[] getAcceptedIssuers() {
                        return new X509Certificate[0];
                    }
                }
        }, null);
        //...
```

项目地址：

## 参考文档
- [How to generate self-signed certificate for usage in Express4 or Node.js HTTP](https://matoski.com/article/node-express-generate-ssl/)
- [SSL证书生成方法](http://blog.csdn.net/fyang2007/article/details/6180361)
- [Android安全开发之安全使用HTTPS](https://zhuanlan.zhihu.com/p/22816331)
- [通过 HTTPS 和 SSL 确保安全](https://developer.android.com/training/articles/security-ssl.html#nogotofail) (要翻墙)
- [https://nodejs.org/api/tls.html](https://nodejs.org/api/tls.html)
- [https://gist.github.com/Frank-Zhu/41e21a00df26d63cd38d](https://gist.github.com/Frank-Zhu/41e21a00df26d63cd38d)
