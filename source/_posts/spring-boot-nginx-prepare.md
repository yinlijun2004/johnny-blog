---
title: AWS Ubuntu 18.04准备Spring Boot运行环境。
date: 2019-06-21 20:03:44
tags: [AWS, Linux, Spring Boot]
---

安装nginx软件包:
```bash
sudo apt update
sudo apt install nginx
```

查看nginx的运行状态:
```bash
sudo systemctl status nginx
```

安装mysql软件包:
```bash
sudo apt-get install mysql-server
```
这个命令会让你输入root的密码，记住这个密码，如果你安装的版本是5.7，没让输入密码，请本文参考最下方方法。
```bash
sudo apt install mysql-client
sudo apt install libmysqlclient-dev
```
<!-- more -->
查看mysql安装结果:
```bash
sudo netstat -tap | grep mysql
```
输出如下:
```bash
tcp        0      0 localhost:mysql         *:*                     LISTEN      22144/mysqld    
```

修改默认字符集为utf8
在/etc/mysql/my.cnf加上如下配置
```
[mysqld]
bind-address            = 0.0.0.0
character_set_server=utf8
collation_server=utf8_general_ci
```

安装redis软件包
```bash
sudo apt-get install redis-server
```
检查安装结果:
```bash
ps -aux|grep redis
```

安装jdk
```bash
sudo apt-get install openjdk-8-jdk openjdk-8-jdk-headless
sudo apt-get install openjfx openjfx-source
```


准备AWS EBS，并且将mysql挂载到新盘。
```bash
ubuntu@ip-172-31-40-13:~$ sudo fdisk -l
Disk /dev/nvme0n1: 10 GiB, 10737418240 bytes, 20971520 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x1e7ee857

Device         Boot Start      End  Sectors Size Id Type
/dev/nvme0n1p1 *     2048 20971486 20969439  10G 83 Linux


Disk /dev/nvme1n1: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

```
格式化设备
```bash
ubuntu@ip-172-31-40-13:~$ sudo mkfs.ext4 /dev/nvme1n1
mke2fs 1.42.13 (17-May-2015)
Creating filesystem with 5242880 4k blocks and 1310720 inodes
Filesystem UUID: 3dc4c546-58ae-423c-84d2-894d26ed7512
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information:        
done
```

挂载磁盘
```bash
ubuntu@ip-172-31-40-13:~$ mkdir mysql
ubuntu@ip-172-31-40-13:~$ sudo mount /dev/nvme1n1 mysql
```

如果需要取消挂载
```bash
ubuntu@ip-172-31-40-13:~$ sudo umount mysql
```
查看挂载结果
```bash
ubuntu@ip-172-31-40-13:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           789M  8.8M  780M   2% /run
/dev/nvme0n1p1  9.7G  2.2G  7.5G  23% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
/dev/loop0       90M   90M     0 100% /snap/core/6818
/dev/loop1       18M   18M     0 100% /snap/amazon-ssm-agent/1335
/dev/loop2       89M   89M     0 100% /snap/core/7169
/dev/nvme1n1     20G   44M   19G   1% /home/ubuntu/mysql
tmpfs           789M     0  789M   0% /run/user/1000
```

但是这个时候重启，挂载磁盘会消失，因此，需要设置启动时挂载。

查看/etc/fstab文件
```bash
ubuntu@ip-172-31-40-13:~$ cat /etc/fstab
LABEL=cloudimg-rootfs	/	 ext4	defaults,discard	0 0
```
可以很明显的看到文件有6列。
第1列是设备名或者卷标

第2列是挂载点（也就是挂载目录）

第3列是所要挂载设备的文件系统或者文件系统类型

第4列是挂载选项，通常使用defaults就可以

第5列设置是否使用dump备份，置0为不备份，置1，2为备份，但2的备份重要性比1小

第6列设置是否开机的时候使用fsck检验所挂载的磁盘，置0为不检验，置1，2为检验，但置2盘比置1的盘晚检验。

增加一行“/dev/nvme1n1       /home/ubuntu/mysql        ext4    defaults        1       1”
```bash
ubuntu@ip-172-31-40-13:~$ cat /etc/fstab
/dev/nvme1n1       /home/ubuntu/mysql        ext4    defaults        1       1
LABEL=cloudimg-rootfs	/	 ext4	defaults,discard	0 0
```

mysql迁移到新盘

把MySQL服务进程停掉：
```bash
ubuntu@ip-172-31-40-13:~$ mysqladmin -u root -p shutdown
Enter password: 
```

把/var/lib/mysql整个目录移到/home/data
```bash
sudo mv /var/lib/mysql　/home/ubuntu/mysql
```
这样吧数据库都移到了新盘

为原来的mysql目录建立软连接，这样可以省去修改配置的。
```bash
sudo ln -s /home/ubuntu/mysql/mysql/ /var/lib/mysql
```
修改/var/lib/mysql所属用户和组
```bash
sudo chown -h mysql:mysql /var/lib/mysql
```

重启mysql服务
```bash
sudo /etc/init.d/mysql start
```

这个时候发现重启不成功，需要修改安全配置文件
```
sudo vim /etc/apparmor.d/usr.sbin.mysqld
```
添加如下两行
```bash
/home/ubuntu/mysql/mysql/ r,
/home/ubuntu/mysql/mysql/** rwk,
```
重新载入配置文件
```bash
sudo service apparmor reload
```

启动 mysql
```bash
sudo /etc/init.d/mysql start
```
重新启动mysql，正常。


---

## MySQL 5.7 重设默认密码的方法

如果安装mysql-server时，没有让输入密码，请按照如下步骤，修改默认root密码


```
sudo cat /etc/mysql/debian.cnf
```
会输入如下内容
```
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = XwSXXXXLXXXXLR15
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = XwSXXXXLXXXXLR15
socket   = /var/run/mysqld/mysqld.sock
```
其中debian-sys-maint和XwSXXXXLXXXXLR15是默认的用户名和随机密码，用这个用户名密码登录。
```
mysql -udebian-sys-maint -pXwSXXXXLXXXXLR15
```

按如下方式修改默认root密码
```
ubuntu@ip-111-31-27-111:~$ mysql -udebian-sys-maint -pXwSXXXXLXXXXLR15
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.27-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update user set authentication_string=PASSWORD('xxxxxxxxxxx') where User='root';
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

mysql> update user set plugin="mysql_native_password";
Query OK, 1 row affected (0.00 sec)
Rows matched: 4  Changed: 1  Warnings: 0

mysql> flush privileges; 
Query OK, 0 rows affected (0.00 sec)

mysql> quit;
Bye

```
这样，就可以用root和新密码登录了。
```
ubuntu@ip-111-31-27-111:~$ sudo /etc/init.d/mysql stop
[ ok ] Stopping mysql (via systemctl): mysql.service.
ubuntu@ip-111-31-27-111:~$ sudo /etc/init.d/mysql start
[ ok ] Starting mysql (via systemctl): mysql.service.
ubuntu@ip-111-31-27-111:~$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> exit
Bye

```