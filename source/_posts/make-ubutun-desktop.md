---
title: Ubuntu 12.04 生成桌面图标 
date: 2016-11-15 18:04:19
tags: ubuntu, android studio
---

在Ubuntu上从网上下载压缩包版本（非.deb包）的应用程序之后，是不生成桌面图表的，比如网上下载的[Android Studio](http://www.android-studio.org/), 所以需要自己做一个桌面图标。

1. 进入到/usr/share/applications/目录下
```bash
cd /usr/share/applications/
```

2. 新建一个android-studio.desktop文件。
```bash
vim android-studio.desktop
```

3. 输入一下内容
```bash
Version=2.2
Name=Android Studio
GenericName=Android IDE
Comment=Android Development
Exec=/home/yinlijun/android_toolchain/android-studio/bin/studio.sh %U
Terminal=false
Icon=/home/yinlijun/android_toolchain/android-studio/bin/studio.png
Type=Application
Categories=Android;IDE;
```

保存退出之后，在应用程序里面就可以找到，绑定了图标的应用程序，可以将其固定到启动器上。