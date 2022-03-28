title: Centos7 单机模式部署 FastDFS
date: 2022-03-28 10:12:00
toc: true
index_img: https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_10-18-42-15a04261.png
category:
- 分享
- Linux
tags:
- Centos7
- 单机
- 模式
- 部署
- FastDFS
---

## 环境准备

查看 Centos 版本（需要 7.x 版本）：`cat /etc/redhat-release`。

### 依赖安装

执行命令完成依赖安装：

```shell
yum install git gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim -y
```

![Snipaste20220328102343.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_10-23-43-887025c2.png)

### 准备数据目录

新建 `/www/apps/fastdfs` 目录（可自定义）用来存储数据；切换到 `/usr/local/src/` 目录下准备下载安装包。

```shell
mkdir /www/apps/fastdfs
cd /usr/local/src
```

### 安装 `libfastcommon`

```shell
git clone https://github.com/happyfish100/libfastcommon.git --depth 1
cd libfastcommon/
./make.sh && ./make.sh install
```

![Snipaste20220328102945.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_10-29-45-bbb52f28.png)

![Snipaste20220328103140.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_10-31-40-d9309e96.png)

> Tips: 如果访问 Github 较慢， 可以使用加速镜像，见：[【转载】GitHub文件下载加速浏览器脚本插件+加速下载网站汇总 - biuaxia](https://www.biuaxia.cn/articles/2021/07/31/1627691040000.html)

### 安装 FastDFS

```shell
cd ../
git clone https://github.com/happyfish100/fastdfs.git --depth 1
cd fastdfs/
./make.sh && ./make.sh install
cp /usr/local/src/fastdfs/conf/http.conf /etc/fdfs/
cp /usr/local/src/fastdfs/conf/mime.types /etc/fdfs/
```

![Snipaste20220328103906.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_10-39-06-e29d5adc.png)

> 需要注意的是，官方教程中有三个拷贝的文件，在实际操作的时候建议 `cd /etc/fdfs` 查看是否存在 `tracker.conf`, `storage.conf`, `client.conf` 这三个文件，如果有就不用执行了。

```shell
# cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
# cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
# cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf
```

![Snipaste20220328104348.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_10-43-48-4e16e977.png)

### 安装 `fastdfs-nginx-module`

```shell
cd ../
git clone https://github.com/happyfish100/fastdfs-nginx-module.git --depth 1
cp /usr/local/src/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs
```

### `fastdfs-nginx` 配置

```shell
vim /etc/fdfs/mod_fastdfs.conf
# 需要修改的内容如下
tracker_server=192.168.10.13:22122  # tracker服务器IP和端口
url_have_group_name=true
store_path0=/www/apps/fastdfs
```

### 使用宝塔自带 `Nginx`

```shell
/www/server/nginx/sbin/nginx -V
```

![Snipaste20220328104954.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_10-49-54-144af0e1.png)

输出结果为：

```shell
[root@server1013 src]# /www/server/nginx/sbin/nginx -v
nginx version: nginx/1.20.2
[root@server1013 src]# /www/server/nginx/sbin/nginx -V
nginx version: nginx/1.20.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC)
built with OpenSSL 1.1.1m  14 Dec 2021
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/www/server/nginx --add-module=/www/server/nginx/src/ngx_devel_kit --add-module=/www/server/nginx/src/lua_nginx_module --add-module=/www/server/nginx/src/ngx_cache_purge --add-module=/www/server/nginx/src/nginx-sticky-module --with-openssl=/www/server/nginx/src/openssl --with-pcre=pcre-8.43 --with-http_v2_module --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_stub_status_module --with-http_ssl_module --with-http_image_filter_module --with-http_gzip_static_module --with-http_gunzip_module --with-ipv6 --with-http_sub_module --with-http_flv_module --with-http_addition_module --with-http_realip_module --with-http_mp4_module --with-ld-opt=-Wl,-E --with-cc-opt=-Wno-error --with-ld-opt=-ljemalloc --with-http_dav_module --add-module=/www/server/nginx/src/nginx-dav-ext-module
[root@server1013 src]#
```

拷贝内容：

```shell
--user=www --group=www --prefix=/www/server/nginx --add-module=/www/server/nginx/src/ngx_devel_kit --add-module=/www/server/nginx/src/lua_nginx_module --add-module=/www/server/nginx/src/ngx_cache_purge --add-module=/www/server/nginx/src/nginx-sticky-module --with-openssl=/www/server/nginx/src/openssl --with-pcre=pcre-8.43 --with-http_v2_module --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_stub_status_module --with-http_ssl_module --with-http_image_filter_module --with-http_gzip_static_module --with-http_gunzip_module --with-ipv6 --with-http_sub_module --with-http_flv_module --with-http_addition_module --with-http_realip_module --with-http_mp4_module --with-ld-opt=-Wl,-E --with-cc-opt=-Wno-error --with-ld-opt=-ljemalloc --with-http_dav_module --add-module=/www/server/nginx/src/nginx-dav-ext-module
```

> Tips: 执行 `cd /www/server/nginx/src` 提示 `-bash: cd: /www/server/nginx/src: 没有那个文件或目录` 就将 `Nginx` 卸载，重新以编译安装并且 `添加自定义模块`就行了。

执行命令：

```shell
cd /www/server/nginx/src
./configure --user=www --group=www --prefix=/www/server/nginx --add-module=/www/server/nginx/src/ngx_devel_kit --add-module=/www/server/nginx/src/lua_nginx_module --add-module=/www/server/nginx/src/ngx_cache_purge --add-module=/www/server/nginx/src/nginx-sticky-module --with-openssl=/www/server/nginx/src/openssl --with-pcre=pcre-8.43 --with-http_v2_module --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_stub_status_module --with-http_ssl_module --with-http_image_filter_module --with-http_gzip_static_module --with-http_gunzip_module --with-ipv6 --with-http_sub_module --with-http_flv_module --with-http_addition_module --with-http_realip_module --with-http_mp4_module --with-ld-opt=-Wl,-E --with-cc-opt=-Wno-error --with-ld-opt=-ljemalloc --with-http_dav_module --add-module=/www/server/nginx/src/nginx-dav-ext-module --add-module=/usr/local/src/fastdfs-nginx-module/src
make && make install
/www/server/nginx/sbin/nginx -s stop    #这是停止
/www/server/nginx/sbin/nginx            #这是启动
/www/server/nginx/sbin/nginx -V
```

## 单机部署

### `tracker` 配置

```shell
vim /etc/fdfs/tracker.conf
# 需要修改的内容如下
port=22122                      # tracker服务器端口（默认22122，一般不改，也无需对外开放）
base_path=/www/apps/fastdfs     # 存储日志和数据的根目录
```

完整配置：

```conf
# is this config file disabled
# false for enabled
# true for disabled
disabled = false

# bind an address of this host
# empty for bind all addresses of this host
bind_addr =

# the tracker server port
port = 22122

# connect timeout in seconds
# default value is 30
# Note: in the intranet network (LAN), 2 seconds is enough.
connect_timeout = 5

# network timeout in seconds for send and recv
# default value is 30
network_timeout = 60

# the base path to store data and log files
base_path = /www/apps/fastdfs

# max concurrent connections this server support
# you should set this parameter larger, eg. 10240
# default value is 256
max_connections = 1024

# accept thread count
# default value is 1 which is recommended
# since V4.07
accept_threads = 1

# work thread count
# work threads to deal network io
# default value is 4
# since V2.00
work_threads = 4

# the min network buff size
# default value 8KB
min_buff_size = 8KB

# the max network buff size
# default value 128KB
max_buff_size = 128KB

# the method for selecting group to upload files
# 0: round robin
# 1: specify group
# 2: load balance, select the max free space group to upload file
store_lookup = 2

# which group to upload file
# when store_lookup set to 1, must set store_group to the group name
store_group = group2

# which storage server to upload file
# 0: round robin (default)
# 1: the first server order by ip address
# 2: the first server order by priority (the minimal)
# Note: if use_trunk_file set to true, must set store_server to 1 or 2
store_server = 0

# which path (means disk or mount point) of the storage server to upload file
# 0: round robin
# 2: load balance, select the max free space path to upload file
store_path = 0

# which storage server to download file
# 0: round robin (default)
# 1: the source storage server which the current file uploaded to
download_server = 0

# reserved storage space for system or other applications.
# if the free(available) space of any stoarge server in
# a group <= reserved_storage_space, no file can be uploaded to this group.
# bytes unit can be one of follows:
### G or g for gigabyte(GB)
### M or m for megabyte(MB)
### K or k for kilobyte(KB)
### no unit for byte(B)
### XX.XX% as ratio such as: reserved_storage_space = 10%
reserved_storage_space = 20%

#standard log level as syslog, case insensitive, value list:
### emerg for emergency
### alert
### crit for critical
### error
### warn for warning
### notice
### info
### debug
log_level = info

#unix group name to run this program,
#not set (empty) means run by the group of current user
run_by_group=

#unix username to run this program,
#not set (empty) means run by current user
run_by_user =

# allow_hosts can ocur more than once, host can be hostname or ip address,
# "*" (only one asterisk) means match all ip addresses
# we can use CIDR ips like 192.168.5.64/26
# and also use range like these: 10.0.1.[0-254] and host[01-08,20-25].domain.com
# for example:
# allow_hosts=10.0.1.[1-15,20]
# allow_hosts=host[01-08,20-25].domain.com
# allow_hosts=192.168.5.64/26
allow_hosts = *

# sync log buff to disk every interval seconds
# default value is 10 seconds
sync_log_buff_interval = 1

# check storage server alive interval seconds
check_active_interval = 120

# thread stack size, should >= 64KB
# default value is 256KB
thread_stack_size = 256KB

# auto adjust when the ip address of the storage server changed
# default value is true
storage_ip_changed_auto_adjust = true

# storage sync file max delay seconds
# default value is 86400 seconds (one day)
# since V2.00
storage_sync_file_max_delay = 86400

# the max time of storage sync a file
# default value is 300 seconds
# since V2.00
storage_sync_file_max_time = 300

# if use a trunk file to store several small files
# default value is false
# since V3.00
use_trunk_file = false

# the min slot size, should <= 4KB
# default value is 256 bytes
# since V3.00
slot_min_size = 256

# the max slot size, should > slot_min_size
# store the upload file to trunk file when it's size <=  this value
# default value is 16MB
# since V3.00
slot_max_size = 1MB

# the alignment size to allocate the trunk space
# default value is 0 (never align)
# since V6.05
# NOTE: the larger the alignment size, the less likely of disk
#       fragmentation, but the more space is wasted.
trunk_alloc_alignment_size = 256

# if merge contiguous free spaces of trunk file
# default value is false
# since V6.05
trunk_free_space_merge = true

# if delete / reclaim the unused trunk files
# default value is false
# since V6.05
delete_unused_trunk_files = false

# the trunk file size, should >= 4MB
# default value is 64MB
# since V3.00
trunk_file_size = 64MB

# if create trunk file advancely
# default value is false
# since V3.06
trunk_create_file_advance = false

# the time base to create trunk file
# the time format: HH:MM
# default value is 02:00
# since V3.06
trunk_create_file_time_base = 02:00

# the interval of create trunk file, unit: second
# default value is 38400 (one day)
# since V3.06
trunk_create_file_interval = 86400

# the threshold to create trunk file
# when the free trunk file size less than the threshold,
# will create he trunk files
# default value is 0
# since V3.06
trunk_create_file_space_threshold = 20G

# if check trunk space occupying when loading trunk free spaces
# the occupied spaces will be ignored
# default value is false
# since V3.09
# NOTICE: set this parameter to true will slow the loading of trunk spaces
# when startup. you should set this parameter to true when neccessary.
trunk_init_check_occupying = false

# if ignore storage_trunk.dat, reload from trunk binlog
# default value is false
# since V3.10
# set to true once for version upgrade when your version less than V3.10
trunk_init_reload_from_binlog = false

# the min interval for compressing the trunk binlog file
# unit: second, 0 means never compress
# FastDFS compress the trunk binlog when trunk init and trunk destroy
# recommand to set this parameter to 86400 (one day)
# default value is 0
# since V5.01
trunk_compress_binlog_min_interval = 86400

# the interval for compressing the trunk binlog file
# unit: second, 0 means never compress
# recommand to set this parameter to 86400 (one day)
# default value is 0
# since V6.05
trunk_compress_binlog_interval = 86400

# compress the trunk binlog time base, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
# default value is 03:00
# since V6.05
trunk_compress_binlog_time_base = 03:00

# max backups for the trunk binlog file
# default value is 0 (never backup)
# since V6.05
trunk_binlog_max_backups = 7

# if use storage server ID instead of IP address
# if you want to use dual IPs for storage server, you MUST set
# this parameter to true, and configure the dual IPs in the file
# configured by following item "storage_ids_filename", such as storage_ids.conf
# default value is false
# since V4.00
use_storage_id = false

# specify storage ids filename, can use relative or absolute path
# this parameter is valid only when use_storage_id set to true
# since V4.00
storage_ids_filename = storage_ids.conf

# id type of the storage server in the filename, values are:
## ip: the ip address of the storage server
## id: the server id of the storage server
# this paramter is valid only when use_storage_id set to true
# default value is ip
# since V4.03
id_type_in_filename = id

# if store slave file use symbol link
# default value is false
# since V4.01
store_slave_file_use_link = false

# if rotate the error log every day
# default value is false
# since V4.02
rotate_error_log = false

# rotate error log time base, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
# default value is 00:00
# since V4.02
error_log_rotate_time = 00:00

# if compress the old error log by gzip
# default value is false
# since V6.04
compress_old_error_log = false

# compress the error log days before
# default value is 1
# since V6.04
compress_error_log_days_before = 7

# rotate error log when the log file exceeds this size
# 0 means never rotates log file by log file size
# default value is 0
# since V4.02
rotate_error_log_size = 0

# keep days of the log files
# 0 means do not delete old log files
# default value is 0
log_file_keep_days = 0

# if use connection pool
# default value is false
# since V4.05
use_connection_pool = true

# connections whose the idle time exceeds this time will be closed
# unit: second
# default value is 3600
# since V4.05
connection_pool_max_idle_time = 3600

# HTTP port on this tracker server
http.server_port = 8080

# check storage HTTP server alive interval seconds
# <= 0 for never check
# default value is 30
http.check_alive_interval = 30

# check storage HTTP server alive type, values are:
#   tcp : connect to the storge server with HTTP port only,
#        do not request and get response
#   http: storage check alive url must return http status 200
# default value is tcp
http.check_alive_type = tcp

# check storage HTTP server alive uri/url
# NOTE: storage embed HTTP server support uri: /status.html
http.check_alive_uri = /status.html

```

### `storage` 配置

```shell
vim /etc/fdfs/storage.conf
# 需要修改的内容如下
port=23000                          # storage服务端口（默认23000,一般不修改）
base_path=/www/apps/fastdfs         # 数据和日志文件存储根目录
store_path0=/www/apps/fastdfs       # 第一个存储目录
tracker_server=192.168.10.13:22122  # tracker服务器IP和端口，不能填写 127.0.0.1 ！！！
http.server_port=18888              # http访问文件的端口(默认8888，看情况修改)
```

完整配置：

```conf
# is this config file disabled
# false for enabled
# true for disabled
disabled = false

# the name of the group this storage server belongs to
#
# comment or remove this item for fetching from tracker server,
# in this case, use_storage_id must set to true in tracker.conf,
# and storage_ids.conf must be configured correctly.
group_name = group1

# bind an address of this host
# empty for bind all addresses of this host
bind_addr =

# if bind an address of this host when connect to other servers
# (this storage server as a client)
# true for binding the address configured by the above parameter: "bind_addr"
# false for binding any address of this host
client_bind = true

# the storage server port
port = 23000

# connect timeout in seconds
# default value is 30
# Note: in the intranet network (LAN), 2 seconds is enough.
connect_timeout = 5

# network timeout in seconds for send and recv
# default value is 30
network_timeout = 60

# the heart beat interval in seconds
# the storage server send heartbeat to tracker server periodically
# default value is 30
heart_beat_interval = 30

# disk usage report interval in seconds
# the storage server send disk usage report to tracker server periodically
# default value is 300
stat_report_interval = 60

# the base path to store data and log files
# NOTE: the binlog files maybe are large, make sure
#       the base path has enough disk space,
#       eg. the disk free space should > 50GB
base_path = /www/apps/fastdfs

# max concurrent connections the server supported,
# you should set this parameter larger, eg. 10240
# default value is 256
max_connections = 1024

# the buff size to recv / send data from/to network
# this parameter must more than 8KB
# 256KB or 512KB is recommended
# default value is 64KB
# since V2.00
buff_size = 256KB

# accept thread count
# default value is 1 which is recommended
# since V4.07
accept_threads = 1

# work thread count
# work threads to deal network io
# default value is 4
# since V2.00
work_threads = 4

# if disk read / write separated
##  false for mixed read and write
##  true for separated read and write
# default value is true
# since V2.00
disk_rw_separated = true

# disk reader thread count per store path
# for mixed read / write, this parameter can be 0
# default value is 1
# since V2.00
disk_reader_threads = 1

# disk writer thread count per store path
# for mixed read / write, this parameter can be 0
# default value is 1
# since V2.00
disk_writer_threads = 1

# when no entry to sync, try read binlog again after X milliseconds
# must > 0, default value is 200ms
sync_wait_msec = 50

# after sync a file, usleep milliseconds
# 0 for sync successively (never call usleep)
sync_interval = 0

# storage sync start time of a day, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
sync_start_time = 00:00

# storage sync end time of a day, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
sync_end_time = 23:59

# write to the mark file after sync N files
# default value is 500
write_mark_file_freq = 500

# disk recovery thread count
# default value is 1
# since V6.04
disk_recovery_threads = 3

# store path (disk or mount point) count, default value is 1
store_path_count = 1

# store_path#, based on 0, to configure the store paths to store files
# if store_path0 not exists, it's value is base_path (NOT recommended)
# the paths must be exist.
#
# IMPORTANT NOTE:
#       the store paths' order is very important, don't mess up!!!
#       the base_path should be independent (different) of the store paths

store_path0 = /www/apps/fastdfs
#store_path1 = /home/yuqing/fastdfs2

# subdir_count  * subdir_count directories will be auto created under each
# store_path (disk), value can be 1 to 256, default value is 256
subdir_count_per_path = 256

# tracker_server can ocur more than once for multi tracker servers.
# the value format of tracker_server is "HOST:PORT",
#   the HOST can be hostname or ip address,
#   and the HOST can be dual IPs or hostnames seperated by comma,
#   the dual IPS must be an inner (intranet) IP and an outer (extranet) IP,
#   or two different types of inner (intranet) IPs.
#   for example: 192.168.2.100,122.244.141.46:22122
#   another eg.: 192.168.1.10,172.17.4.21:22122

#tracker_server = 192.168.209.121:22122
tracker_server = 192.168.10.13:22122

#standard log level as syslog, case insensitive, value list:
### emerg for emergency
### alert
### crit for critical
### error
### warn for warning
### notice
### info
### debug
log_level = info

#unix group name to run this program,
#not set (empty) means run by the group of current user
run_by_group =

#unix username to run this program,
#not set (empty) means run by current user
run_by_user =

# allow_hosts can ocur more than once, host can be hostname or ip address,
# "*" (only one asterisk) means match all ip addresses
# we can use CIDR ips like 192.168.5.64/26
# and also use range like these: 10.0.1.[0-254] and host[01-08,20-25].domain.com
# for example:
# allow_hosts=10.0.1.[1-15,20]
# allow_hosts=host[01-08,20-25].domain.com
# allow_hosts=192.168.5.64/26
allow_hosts = *

# the mode of the files distributed to the data path
# 0: round robin(default)
# 1: random, distributted by hash code
file_distribute_path_mode = 0

# valid when file_distribute_to_path is set to 0 (round robin).
# when the written file count reaches this number, then rotate to next path.
# rotate to the first path (00/00) after the last path (such as FF/FF).
# default value is 100
file_distribute_rotate_count = 100

# call fsync to disk when write big file
# 0: never call fsync
# other: call fsync when written bytes >= this bytes
# default value is 0 (never call fsync)
fsync_after_written_bytes = 0

# sync log buff to disk every interval seconds
# must > 0, default value is 10 seconds
sync_log_buff_interval = 1

# sync binlog buff / cache to disk every interval seconds
# default value is 60 seconds
sync_binlog_buff_interval = 1

# sync storage stat info to disk every interval seconds
# default value is 300 seconds
sync_stat_file_interval = 300

# thread stack size, should >= 512KB
# default value is 512KB
thread_stack_size = 512KB

# the priority as a source server for uploading file.
# the lower this value, the higher its uploading priority.
# default value is 10
upload_priority = 10

# the NIC alias prefix, such as eth in Linux, you can see it by ifconfig -a
# multi aliases split by comma. empty value means auto set by OS type
# default values is empty
if_alias_prefix =

# if check file duplicate, when set to true, use FastDHT to store file indexes
# 1 or yes: need check
# 0 or no: do not check
# default value is 0
check_file_duplicate = 0

# file signature method for check file duplicate
## hash: four 32 bits hash code
## md5: MD5 signature
# default value is hash
# since V4.01
file_signature_method = hash

# namespace for storing file indexes (key-value pairs)
# this item must be set when check_file_duplicate is true / on
key_namespace = FastDFS

# set keep_alive to 1 to enable persistent connection with FastDHT servers
# default value is 0 (short connection)
keep_alive = 0

# you can use "#include filename" (not include double quotes) directive to
# load FastDHT server list, when the filename is a relative path such as
# pure filename, the base path is the base path of current/this config file.
# must set FastDHT server list when check_file_duplicate is true / on
# please see INSTALL of FastDHT for detail
##include /home/yuqing/fastdht/conf/fdht_servers.conf

# if log to access log
# default value is false
# since V4.00
use_access_log = false

# if rotate the access log every day
# default value is false
# since V4.00
rotate_access_log = false

# rotate access log time base, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
# default value is 00:00
# since V4.00
access_log_rotate_time = 00:00

# if compress the old access log by gzip
# default value is false
# since V6.04
compress_old_access_log = false

# compress the access log days before
# default value is 1
# since V6.04
compress_access_log_days_before = 7

# if rotate the error log every day
# default value is false
# since V4.02
rotate_error_log = false

# rotate error log time base, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
# default value is 00:00
# since V4.02
error_log_rotate_time = 00:00

# if compress the old error log by gzip
# default value is false
# since V6.04
compress_old_error_log = false

# compress the error log days before
# default value is 1
# since V6.04
compress_error_log_days_before = 7

# rotate access log when the log file exceeds this size
# 0 means never rotates log file by log file size
# default value is 0
# since V4.02
rotate_access_log_size = 0

# rotate error log when the log file exceeds this size
# 0 means never rotates log file by log file size
# default value is 0
# since V4.02
rotate_error_log_size = 0

# keep days of the log files
# 0 means do not delete old log files
# default value is 0
log_file_keep_days = 0

# if skip the invalid record when sync file
# default value is false
# since V4.02
file_sync_skip_invalid_record = false

# if use connection pool
# default value is false
# since V4.05
use_connection_pool = true

# connections whose the idle time exceeds this time will be closed
# unit: second
# default value is 3600
# since V4.05
connection_pool_max_idle_time = 3600

# if compress the binlog files by gzip
# default value is false
# since V6.01
compress_binlog = true

# try to compress binlog time, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
# default value is 01:30
# since V6.01
compress_binlog_time = 01:30

# if check the mark of store path to prevent confusion
# recommend to set this parameter to true
# if two storage servers (instances) MUST use a same store path for
# some specific purposes, you should set this parameter to false
# default value is true
# since V6.03
check_store_path_mark = true

# use the ip address of this storage server if domain_name is empty,
# else this domain name will ocur in the url redirected by the tracker server
http.domain_name =

# the port of the web server on this storage server
http.server_port = 18888

```

### `client` 配置

```shell
vim /etc/fdfs/client.conf
# 需要修改的内容如下
base_path=/www/apps/fastdfs
tracker_server=192.168.10.13:22122    # tracker服务器IP和端口，不能填写 127.0.0.1 ！！！
```

完整配置：

```conf
# connect timeout in seconds
# default value is 30s
# Note: in the intranet network (LAN), 2 seconds is enough.
connect_timeout = 5

# network timeout in seconds
# default value is 30s
network_timeout = 60

# the base path to store log files
#base_path = /home/yuqing/fastdfs
base_path = /www/apps/fastdfs

# tracker_server can ocur more than once for multi tracker servers.
# the value format of tracker_server is "HOST:PORT",
#   the HOST can be hostname or ip address,
#   and the HOST can be dual IPs or hostnames seperated by comma,
#   the dual IPS must be an inner (intranet) IP and an outer (extranet) IP,
#   or two different types of inner (intranet) IPs.
#   for example: 192.168.2.100,122.244.141.46:22122
#   another eg.: 192.168.1.10,172.17.4.21:22122

#tracker_server = 192.168.0.196:22122
tracker_server = 192.168.10.13:22122

#standard log level as syslog, case insensitive, value list:
### emerg for emergency
### alert
### crit for critical
### error
### warn for warning
### notice
### info
### debug
log_level = info

# if use connection pool
# default value is false
# since V4.05
use_connection_pool = false

# connections whose the idle time exceeds this time will be closed
# unit: second
# default value is 3600
# since V4.05
connection_pool_max_idle_time = 3600

# if load FastDFS parameters from tracker server
# since V4.05
# default value is false
load_fdfs_parameters_from_tracker = false

# if use storage ID instead of IP address
# same as tracker.conf
# valid only when load_fdfs_parameters_from_tracker is false
# default value is false
# since V4.05
use_storage_id = false

# specify storage ids filename, can use relative or absolute path
# same as tracker.conf
# valid only when load_fdfs_parameters_from_tracker is false
# since V4.05
storage_ids_filename = storage_ids.conf


#HTTP settings
http.tracker_server_port = 80

#use "#include" directive to include HTTP other settiongs
##include http.conf

```

### 启动 `tracker` 和 `storage`

```shell
pkill -9 fdfs
fdfs_trackerd /etc/fdfs/tracker.conf && fdfs_storaged /etc/fdfs/storage.conf
```


### `client` 测试

测试：

```shell
# /usr/local/src/nginx-1.15.4.tar.gz 是服务器绝对路径，可以换成实际存在的文件。
# 保存后测试,返回ID表示成功 如：group1/M00/00/00/xx.tar.gz
fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/nginx-1.15.4.tar.gz
# 例如：`fdfs_upload_file /etc/fdfs/client.conf /www/apps/fastdfs/Snipaste_2022-03-17_16-11-51.png`
```

![Snipaste20220328112017.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_11-20-17-346b98fa.png)

```shell
[root@server1013 src]# fdfs_upload_file /etc/fdfs/client.conf /www/apps/fastdfs/Snipaste_2022-03-17_16-11-51.png
group1/M00/00/00/wKgKDWJBKieAFy55AAA1IWpVuRE893.png
[root@server1013 src]#
```

### 配置 `Nginx` 访问

1. 新建网站；

    ![Snipaste20220328112153.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_11-21-53-9b0072ab.png)

2. 选择配置，添加反向代理；

    ![Snipaste20220328112331.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_11-23-31-b57406f6.png)

3. 编辑配置文件。

    ![Snipaste20220328112428.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_11-24-28-ebfe775c.png)

    ```shell
#PROXY-START/
location ~/group[0-9]/ {
    ngx_fastdfs_module;
}
#PROXY-END/
    ```

访问之前 `client` 测试得到的结果地址：[wKgKDWJBKieAFy55AAA1IWpVuRE893.png (364×278)](http://192.168.10.13:10001/group1/M00/00/00/wKgKDWJBKieAFy55AAA1IWpVuRE893.png)

![Snipaste20220328121016.png](https://b3logfile.com/file/2022/03/Snipaste_2022-03-28_12-10-16-665a7067.png)

## FAQ

### 配置好了无法通过浏览器访问上传结果

检查 `fastdfs-nginx` 配置，然后重启 nginx。

```shell
vim /etc/fdfs/mod_fastdfs.conf

/www/server/nginx/sbin/nginx -s stop
/www/server/nginx/sbin/nginx -t
/www/server/nginx/sbin/nginx
```

## 参考资料

- [happyfish100/fastdfs Wiki](https://github.com/happyfish100/fastdfs/wiki)
- [Centos7安装FASTDFS整合宝塔Nginx，配合nginx-fastdfs插件，实现文件上传](https://www.codeleading.com/article/97515655967/)
