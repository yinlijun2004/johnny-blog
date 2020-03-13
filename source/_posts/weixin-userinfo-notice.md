---
title: Spring Boot获取微信用户信息要注意的问题
date: 2020-03-13 15:48:10
tags: [Sprint Boog, 微信, Mysql]
---

在APP接入微信登录时，经常会有保存用户信息的需求，微信首选用户信息的内容，一般返回如下结构：
```json
{
  "openid": "OPENID",
  "nickname": "NICKNAME",
  "sex": 1,
  "province": "PROVINCE",
  "city": "CITY",
  "country": "COUNTRY",
  "headimgurl": "http://wx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ4eMsv84eavHiaiceqxibJxCfHe/0",
  "privilege": ["PRIVILEGE1", "PRIVILEGE2"],
  "unionid": " o6_bmasdasdsad6_2sgVt7hMZOPfL"
}
```
微信官方对解释如下：

参数|说明
- | :- | :- |
openid | 普通用户的标识，对当前开发者帐号唯一
nickname |	普通用户昵称
sex	| 普通用户性别，1 为男性，2 为女性
province |	普通用户个人资料填写的省份
city |	普通用户个人资料填写的城市
country |	国家，如中国为 CN
headimgurl |	用户头像，最后一个数值代表正方形头像大小（有 0、46、64、96、132 数值可选，0 代表 640*640 正方形头像），用户没有头像时该项为空
privilege |	用户特权信息，json 数组，如微信沃卡用户为（chinaunicom）
unionid |	用户统一标识。针对一个微信开放平台帐号下的应用，同一用户的 unionid 是唯一的。

其中province city sex是比较有用的信息，比如说我可以利用这个信息对用户进行归类，做响应的推送。

openid 和 unionid 是用来表示唯一用户的。

headimgurl需要注意，这个表示用户头像临时url，当用户换了头像之后，这个地址是失效的，如果最好不在数据库中存储，而是下载下来，转存到另外的位置（比如说oss），然后数据库中存这个地址。

下面重点要说的是nickname，nickname是ISO-8859-1编码的，需要转成UTF-8编码。
```java
String nickName = null;
try {
    nickName = new String(user.getNickname().getBytes("ISO-8859-1"), "UTF-8");
} catch (UnsupportedEncodingException e) {
    e.printStackTrace();
}
```

但是如果你的数据库里字段的编码是utf8，也还是不能直接存储的，会抛出如下异常：
```bash
Caused by: java.sql.SQLException: Incorrect string value: '\xF0\x9F\x91\x84\xF0\x9F...' for column 'nickname' at row 1
```

这时需要修改数据的字符集配置，具体步骤如下(以ubuntu18.04为例)：

1. 修改<b>/etc/mysql/my.cnf</b>文件，末尾增加如下内容：
```
[client]
default-character-set=utf8mb4
[mysql]
default-character-set=utf8mb4
[mysqld]
character-set-client-handshake=FALSE
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4’
```

2. 修改数据库字符集
```SQL
ALTER DATABASE [数据库名] CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
```

3. 切换数据库
```SQL
use [数据库名]
```

4. 修改表字符集
```SQL
ALTER DATABASE [表名] CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
```

5. 修改字段字符集
```SQL
ALTER TABLE [表名] CHANGE [字段名] [字段名] [字段类型] CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

6. 重启数据库
```bash
sudo service mysql restart
```

7. 检查设置结果
```SQL
SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
```

输出如下表示设置成功:
```SQL
+--------------------------+--------------------+
| Variable_name            | Value              |
+--------------------------+--------------------+
| character_set_client     | utf8mb4            |
| character_set_connection | utf8mb4            |
| character_set_database   | utf8mb4            |
| character_set_filesystem | binary             |
| character_set_results    | utf8mb4            |
| character_set_server     | utf8mb4            |
| character_set_system     | utf8               |
| collation_connection     | utf8mb4_unicode_ci |
| collation_database       | utf8mb4_unicode_ci |
| collation_server         | utf8mb4_unicode_ci |
+--------------------------+--------------------+
10 rows in set (0.00 sec)
```

同时，修改application.yml的数据源设置
```json
spring:
  datasource:
    url: jdbc:mysql://[数据库服务器IP]:[端口]/[数据库名]?useLegacyDatetimeCode=false&serverTimezone=UTC&useUnicode=true&characterEncoding=utf8
```
其中<font color='red'><b>useUnicode=true&characterEncoding=utf8</b></font>是必须的。

另外，还需要加上：
```
spring:
  datasource:
    tomcat:
      init-s-q-l: SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci
```

这样，就可以完整的保存用户的昵称了，用mysql命令行查看，如图：

![昵称截图](1584087342832.jpg)



参考资料：
> https://developers.weixin.qq.com/doc/oplatform/Mobile_App/WeChat_Login/Authorized_API_call_UnionID.html
> https://blog.csdn.net/WeiHao1024k/article/details/88554476?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task



