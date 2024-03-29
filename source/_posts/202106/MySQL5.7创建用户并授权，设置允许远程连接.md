title: MySQL5.7创建用户并授权，设置允许远程连接
date: 2021-06-18 22:32:28
toc: true
category: 
 - 数据库
tags: 
 - MySQL
 - 远程
 - 授权
 - 设置
 - 允许
 - 连接
 - 创建
 - 用户
---

## 示例

创建允许远程访问的 root 账号，密码为 123456，且授予所有表的所有操作。

```
CREATE USER 'root'@'%' IDENTIFIED BY '123456';
GRANT ALL ON *.* TO 'root'@'%';
FLUSH PRIVILEGES;
```

## 创建用户

命令：

```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

说明：

- username：你将创建的用户名
- host：指定该用户在哪个主机上可以登陆，如果是本地用户可用localhost，如果想让该用户可以**从任意远程主机登陆** ，可以使用通配符`%`
- password：该用户的登陆密码，密码可以为空，如果为空则该用户可以不需要密码登陆服务器
  
例子：

```mysql
CREATE USER 'dog'@'localhost' IDENTIFIED BY '123456';
CREATE USER 'pig'@'192.168.1.101' IDENTIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '123456';
CREATE USER 'pig'@'%' IDENTIFIED BY '';CREATE USER 'pig'@'%';
```

## 授权

命令：

```mysql
GRANT privileges ON databasename.tablename TO 'username'@'host';
```

说明：

- privileges：用户的操作权限，如`SELECT`，`INSERT`，`UPDATE`等，如果要授予所的权限则使用`ALL`
- databasename：数据库名
- tablename：表名，如果要授予该用户对所有数据库和表的相应操作权限则可用`*`表示，如`*.*`例子:

```mysql
GRANT SELECT, INSERT ON test.user TO 'pig'@'%';
GRANT ALL ON *.* TO 'pig'@'%';
GRANT ALL ON maindataplus.* TO 'pig'@'%';
```

注意：

用以上命令授权的用户不能给其它用户授权，如果想让该用户可以授权，用以下命令:

```mysql
GRANT privileges ON databasename.tablename TO 'username'@'host' WITH GRANT OPTION;
```

## 设置与更改用户密码

命令：

```mysql
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
```

如果是当前登陆用户用：

```mysql
SET PASSWORD = PASSWORD("newpassword");
```

例子：

```mysql
SET PASSWORD FOR 'pig'@'%' = PASSWORD("123456");
```

## 撤销用户权限

命令：

```html hljs xml
REVOKE privilege ON databasename.tablename FROM 'username'@'host';
```

说明：

privilege, databasename, tablename：同授权部分

例子：

```mysql
REVOKE SELECT ON *.* FROM 'pig'@'%';
```

注意：

假如你在给用户`'pig'@'%'`授权的时候是这样的（或类似的）：`GRANT SELECT ON test.user TO 'pig'@'%'`，则在使用`REVOKE SELECT ON *.* FROM 'pig'@'%';`命令并不能撤销该用户对test数据库中user表的`SELECT` 操作。

相反，如果授权使用的是`GRANT SELECT ON *.* TO 'pig'@'%';`则`REVOKE SELECT ON test.user FROM 'pig'@'%';`命令也不能撤销该用户对test数据库中user表的`Select`权限。

具体信息可以用命令`SHOW GRANTS FOR 'pig'@'%';` 查看。

## 删除用户

命令：

```mysql
DROP USER 'username'@'host';
```

**最后一步** 就是刷新权限：

```mysql
FLUSH PRIVILEGES;
```

## 建库

```mysql
CREATE DATABASE IF NOT EXISTS 库名 DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
CREATE DATABASE IF NOT EXISTS 库名 DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci;
```