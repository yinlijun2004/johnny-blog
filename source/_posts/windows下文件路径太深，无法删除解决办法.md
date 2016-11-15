---
title: windows下文件路径太深，无法删除解决办法
date: 2016-11-8 18:02:04
tags:
---

windows下npm开发时，有时候node_modules/下的目录嵌套太深，导致无法删除项目。

npm社区贡献了一个工具[windows-node-deps-deleter](https://www.npmjs.com/package/windows-node-deps-deleter)可供删除这样的目录。

<!--more-->

```
E:\vscode>npm install -g windows-node-deps-deleter
C:\Users\xx\AppData\Roaming\npm\wnddel -> C:\Users\xx\AppData\Roaming\npm\no
de_modules\windows-node-deps-deleter\wnddel.js
windows-node-deps-deleter@0.1.1 C:\Users\xx\AppData\Roaming\npm\node_modules\w
indows-node-deps-deleter
└── fs-extra@0.13.0 (ncp@1.0.1, jsonfile@2.4.0, rimraf@2.5.4)

E:\vscode>wnddel react-todo-list
Deleting "react-todo-list" ...
"react-todo-list" deleted.
```

参考：[windows-node-deps-delete](https://www.npmjs.com/package/windows-node-deps-deleter#readme)
