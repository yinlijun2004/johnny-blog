---
title: android native代码debug开关
date: 2018-04-26 18:06:18
tags: [android, debug, c/c++]
---

调试本地代码的时候，有时候要加上一些log，特别是一些关键路径，但是生产的时候又不需要打印，有几个办法。

### 宏定义实现

```c
#ifdef DEBUG
LOGD("i am a log");
#endif
```

这种方法有个不好的地方是，程序一旦编译完了，就不能加修改了，特别是user版本出了状况，因为system分区是只读的。

<!-- more -->
### 用动态开关

用动态开关量，也有两种方法，一种是给出接口，给上一级打开或者关闭这个开关。

#### 提供开关接口
```c
static int enable_log = 0;

void set_enable_log(int enable) {
    enable_log = enable;
}

int is_log_enable() {
    return enable_log;
}
```
在需要加log的地方写：
```c
if(is_log_enable()) {
    LOGD("i am a log");
}
```
这种方法，适用于上层控制开关的方式，比如说只有经过远程授权才允许打印的log。

#### android的属性控制开关

利用属性来控制log开关时，最好可以在user版本的shell下使用，这对属性名称有讲究。

查看系统代码 system/core/init/property_service.c，有关属性权限的限制。
```c
struct {
    const char *prefix;
    unsigned int uid;
    unsigned int gid;
} property_perms[] = {
    ...
    {"debug.",  AID_SHELL,  0},
    ...
}
```
可以看出， adb shell只能对debug.开始的属性有读写权限，所以属性名不能随意写。

设置完属性之后，可以在代码里面读取该属性值。
```c
#define DEBUG_PROP_NAME "debug.log_switch"

char debug_info[PROPERTY_VALUE_MAX];

int is_log_enable() {
    memset(debug_info, 0, PROPERTY_VALUE_MAX);
    propery_get(DEBUG_PROP_NAME, debug_info);

    return !strcmp(debug_info, "on");
}
```
然后在需要打log的地方写
```c
if(is_log_enable()) {
    LOGD("i am a log");
}
```

以上代码纯手打，没有经过编译，可能会有编译错误，仅提供一种log开关的思路。