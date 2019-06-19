---
title: AWS虚拟机实例ssh登录注意事项
date: 2019-06-19 17:29:19
tags: [AWS, ssh]
---

在AWS上创建好实例时，会要求输入私钥对名称，AWS会让你将私钥下载下来，利用私钥登录，这样做的目的是省去口令登录。毕竟口令登录是不安全的。

登录命令：
```bash
ssh -v -i /path/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com
```
但是会报错，
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'my-key-pair.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "my-key-pair.pem": bad permissions
ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com: Permission denied (publickey).
```

这里要求/path/my-key-pair.pem文件，必须是别的用户不能修改的，因此要改下权限。

```bash
chmod 0600 /path/my-key-pair.pem
```

改好之后，再次执行ssh登录，可能还是会报错
```bash
ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com: Permission denied (publickey).
```

这是因为ec2-user用户名错了，不同的系统实例，用户名是不一样的。

- 对于 Amazon Linux AMI，用户名称是 ec2-user。
- 对于 RHEL5 AMI，用户名称为 root 或 ec2-user。
- 对于 Ubuntu AMI，用户名称是 ubuntu。
- 对于 Fedora AMI，用户名称为 fedora 或 ec2-user。
- 对于 SUSE Linux，用户名称是 root 或 ec2-user。
- 另外，如果 ec2-user 和 root 无法使用，请与 AMI 供应商核实。

我选的是ubuntu，所以应该是ubuntu。

