title: 宝塔linux面板命令大全
date: 2022-03-16 14:23:14
toc: true
category:
- Linux
tags:
- linux
- 宝塔
- 面板
- 命令
- 大全
---

## 官方命令

### 安装宝塔

Centos安装脚本

```
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

Ubuntu/Deepin安装脚本

```
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh
```

Debian安装脚本

```
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && bash install.sh
```

Fedora安装脚本

```
wget -O install.sh http://download.bt.cn/install/install_6.0.sh && bash install.sh
```

### 管理宝塔

宝塔工具箱**(包含下列绝大部分功能 直接ssh中执行bt命令 仅限6.x以上版本面板)**


<!-- more -->


```
bt
```

停止

```
/etc/init.d/bt stop
```

启动

```
/etc/init.d/bt start
```

重启

```
/etc/init.d/bt restart
```

卸载

```
/etc/init.d/bt stop && chkconfig --del bt && rm -f /etc/init.d/bt && rm -rf /www/server/panel
```

查看当前面板端口

```
cat /www/server/panel/data/port.pl
```

修改面板端口，如要改成8881（centos 6 系统）

```
echo '8881' > /www/server/panel/data/port.pl && /etc/init.d/bt restart
iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport 8881 -j ACCEPT
service iptables save
service iptables restart
```

修改面板端口，如要改成8881（centos 7 系统）

```
echo '8881' > /www/server/panel/data/port.pl && /etc/init.d/bt restart
firewall-cmd --permanent --zone=public --add-port=8881/tcp
firewall-cmd --reload
```

强制修改MySQL管理(root)密码，如要改成123456

```
cd /www/server/panel && python tools.py root 123456
```

修改面板密码，如要改成123456

```
cd /www/server/panel && python tools.py panel 123456
```

查看宝塔日志

```
cat /tmp/panelBoot.pl
```

查看软件安装日志

```
cat /tmp/panelExec.log
```

站点配置文件位置

```
/www/server/panel/vhost
```

删除域名绑定面板

```
rm -f /www/server/panel/data/domain.conf
```

清理登陆限制

```
rm -f /www/server/panel/data/*.login
```

查看面板授权IP

```
cat /www/server/panel/data/limitip.conf
```

关闭访问限制

```
rm -f /www/server/panel/data/limitip.conf
```

查看许可域名

```
cat /www/server/panel/data/domain.conf
```

关闭面板SSL

```
rm -f /www/server/panel/data/ssl.pl && /etc/init.d/bt restart
```

查看面板错误日志

```
cat /tmp/panelBoot
```

查看数据库错误日志

```
cat /www/server/data/*.err
```

站点配置文件目录(nginx)

```
/www/server/panel/vhost/nginx
```

站点配置文件目录(apache)

```
/www/server/panel/vhost/apache
```

站点默认目录

```
/www/wwwroot
```

数据库备份目录

```
/www/backup/database
```

站点备份目录

```
/www/backup/site
```

站点日志

```
/www/wwwlogs
```

### Nginx服务管理

nginx安装目录

```
/www/server/nginx
```

启动

```
/etc/init.d/nginx start
```

停止

```
/etc/init.d/nginx stop
```

重启

```
/etc/init.d/nginx restart
```

启载

```
/etc/init.d/nginx reload
```

nginx配置文件

```
/www/server/nginx/conf/nginx.conf
```

### Apache服务管理

apache安装目录

```
/www/server/httpd
```

启动

```
/etc/init.d/httpd start
```

停止

```
/etc/init.d/httpd stop
```

重启

```
/etc/init.d/httpd restart
```

启载

```
/etc/init.d/httpd reload
```

apache配置文件

```
/www/server/apache/conf/httpd.conf
```

### MySQL服务管理

mysql安装目录

```
/www/server/mysql
```

phpmyadmin安装目录

```
/www/server/phpmyadmin
```

数据存储目录

```
/www/server/data
```

启动

```
/etc/init.d/mysqld start
```

停止

```
/etc/init.d/mysqld stop
```

重启

```
/etc/init.d/mysqld restart
```

启载

```
/etc/init.d/mysqld reload
```

mysql配置文件

```
/etc/my.cnf
```

### FTP服务管理

ftp安装目录

```
/www/server/pure-ftpd
```

启动

```
/etc/init.d/pure-ftpd start
```

停止

```
/etc/init.d/pure-ftpd stop
```

重启

```
/etc/init.d/pure-ftpd restart
```

ftp配置文件

```
/www/server/pure-ftpd/etc/pure-ftpd.conf
```

### PHP服务管理

php安装目录

```
/www/server/php
```

启动**(请根据安装PHP版本号做更改，例如：/etc/init.d/php-fpm-54 start)**

```
/etc/init.d/php-fpm-{52|53|54|55|56|70|71|72|73|74} start
```

停止**(请根据安装PHP版本号做更改，例如：/etc/init.d/php-fpm-54 stop)**

```
/etc/init.d/php-fpm-{52|53|54|55|56|70|71|72|73|74} stop
```

重启**(请根据安装PHP版本号做更改，例如：/etc/init.d/php-fpm-54 restart)**

```
/etc/init.d/php-fpm-{52|53|54|55|56|70|71|72|73|74} restart
```

启载**(请根据安装PHP版本号做更改，例如：/etc/init.d/php-fpm-54 reload)**

```
/etc/init.d/php-fpm-{52|53|54|55|56|70|71|72|73|74} reload
```

配置文件**(请根据安装PHP版本号做更改，例如：/www/server/php/52/etc/php.ini)**

```
/www/server/php/{52|53|54|55|56|70|71|72|73|74}/etc/php.ini
```

### Redis服务管理

redis安装目录

```
/www/server/redis
```

启动

```
/etc/init.d/redis start
```

停止

```
/etc/init.d/redis stop
```

redis配置文件

```
/www/server/redis/redis.conf
```

### Memcached服务管理

memcached安装目录

```
/usr/local/memcached
```

启动

```
/etc/init.d/memcached start
```

停止

```
/etc/init.d/memcached stop
```

重启

```
/etc/init.d/memcached restart
```

启载

```
/etc/init.d/memcached reload
```

## 外部命令

### 宝塔面板7.7

原版安装脚本

```
wget -O install.sh http://f.cccyun.cc/bt/install_6.0.sh && bash install.sh
```

老版本升级脚本

```
curl http://f.cccyun.cc/bt/update6.sh|bash
```

一键优化补丁

> 优化内容
> 
> 1. 去除宝塔面板强制绑定账号
> 2. 去除各种删除操作时的计算题与延时等待
> 3. 去除创建网站自动创建的垃圾文件（index.html、404.html、.htaccess）
> 4. 关闭未绑定域名提示页面，防止有人访问未绑定域名直接看出来是用的宝塔面板
> 5. 关闭活动推荐与在线客服
> 
> 注意：仅适用宝塔 7.7 版本

```
wget -O optimize.sh http://f.cccyun.cc/bt/optimize.sh && bash optimize.sh
```

## 参考资料

- [宝塔面板 - 简单好用的Linux/Windows服务器运维管理面板](https://www.bt.cn/)
- [宝塔Linux面板安装教程 - 2022年2月18日更新 - 7.9.0正式版 - Linux面板 - 宝塔面板论坛](https://www.bt.cn/bbs/thread-19376-1-1.html)
- [宝塔linux面板命令大全 - 宝塔面板](https://www.bt.cn/btcode.html)
- [宝塔面板7.7原版安装脚本 - 教程 - 论坛](https://hu60.cn/q.php/bbs.topic.102316.html)
- [自用的宝塔面板一键优化补丁 - Shell脚本 - 论坛](https://hu60.cn/q.php/bbs.topic.102061.html)
- [宝塔面板站点与数据库一键备份与恢复脚本 - Shell脚本 - 论坛](https://hu60.cn/q.php/bbs.topic.102708.html)
