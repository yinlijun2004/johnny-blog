---
title: 解决 /dev/kvm is not found 的问题
tags:
  - android
  - ubuntu
  - android studio
date: 2016-11-28 19:44:23
---

## 问题出现环境
- Ubuntu 12.04
- Android Studio 2.2.2

## 解决步骤
### 开启VT-x
在ubuntu上使用Android Studio创建模拟器时，会提示一个错误：
```
/dev/kvm is not found
```
并且提示要在*BIOS*里面开启<font size='4em'>**VT-x**</font>。

<!-- more -->
重启电脑，按*DEL*键进入*BIOS*,发现确实没有启用，于是启用后再此重启电脑。

此时打开Android Studio，仍然提示一样的错误。

再次在网上搜寻，发现如下解决方案，记录一下。

[http://askubuntu.com/questions/600727/replacement-for-haxm-on-ubuntu-says-intel-x86-emulator-accelerator-is-not-comp](http://askubuntu.com/questions/600727/replacement-for-haxm-on-ubuntu-says-intel-x86-emulator-accelerator-is-not-comp)

### 
Check if your CPU supports hardware virtualization, by typing:
```
egrep -c '(vmx|svm)' /proc/cpuinfo
```
If the result is 0, your CPU does not support hardware virtualization, which is necessary to run the KVM. If you get 1 or more, that means you’re fine.

Next, install KVM. First make sure if your processor supports KVM by typing:
```
kvm-ok
```
You will see this if that’s the case:

INFO: Your CPU supports KVM extensions INFO: /dev/kvm exists KVM acceleration can be used
If this is the result, you need to turn on Intel VT in BIOS:

INFO: KVM is disabled by your BIOS HINT: Enter your BIOS setup and enable Virtualization Technology (VT), and then hard poweroff/poweron your system KVM acceleration can NOT be used
The next step is to install the KVM and a few other packages needed. To do so, type:
```
sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils
```
Add your user to some groups, replacing by your own username:
```
sudo adduser <user> libvirtd
sudo adduser <user> kvm
```
Check if everything is ok:
```
sudo virsh -c qemu:///system list
```