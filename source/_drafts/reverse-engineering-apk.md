---
title: ubuntu如何反编译apk（2016最新方法）
tags: [ubuntu, android, apk, reverse engineering]
---

# 下载apktool
1. Download Linux [wrapper script](https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool) (Right click, Save Link As apktool)
```
cd ~/
wget https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool
```

2. Download apktool-2 ([find newest here](https://bitbucket.org/iBotPeaches/apktool/downloads))
```
wget https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.2.1.jar
```

3. Make sure you have the 32bit libraries (ia32-libs) downloaded and installed by your linux package manager, if you are on a 64bit unix system.
(This helps provide support for the 32bit native binary aapt, which is required by apktool)
```
sudo apt-get install ia32-libs
```

4. Rename downloaded jar to apktool.jar
```
mv apktool_2.2.1.jar apktool.jar
```

5. Move both files (apktool.jar & apktool) to /usr/local/bin (root needed)
```
sudo mv apktool.jar apktool /usr/local/bin/
```

6. Make sure both files are executable (chmod +x)
```
chmod +x /usr/local/bin/apktool /usr/local/bin/apktool.jar
```

7. Try running apktool via cli
```
apktool d xxx.apk
```

# 下载dex2jar
dex2jar，功能：反编译出jar文件，即apk的源程序文件的字节码，
下载地址：[https://github.com/pxb1988/dex2jar](https://github.com/pxb1988/dex2jar)

```
git clone https://github.com/pxb1988/dex2jar
```

## 参考
[https://ibotpeaches.github.io/Apktool/](https://ibotpeaches.github.io/Apktool/)

