# mysql 5.7.28 安装（centos 7）

```
本文只适用于通过 rpm 二进制的方式安装，使用源码安装的请自行百度。
```
## 1. 准备文件
下载地址：[https://dev.mysql.com/downloads/mysql/5.7.html](https://dev.mysql.com/downloads/mysql/5.7.html)。我下载的是一整个 tar 包，如下图
![w400](http://img.lsof.fun/2020-03-02-15831588856536.jpg)

解压后只需要安装以下包

```
mysql-community-common-5.7.28-1.el7.x86_64.rpm
mysql-community-libs-5.7.28-1.el7.x86_64.rpm
mysql-community-client-5.7.28-1.el7.x86_64.rpm
mysql-community-server-5.7.28-1.el7.x86_64.rpm
```

**需要使用 root 的账号进行操作**

## 2. 卸载现有的版本数据库
### 2.1 卸载包

找出所有已经安装的包

```
rpm -qa | grep -i mysql
```
移除上面命令列出来的包

```
rpm -e pagename --nodeps

参数 pagename 上面出现的包名
参数 --nodeps 表示不检查依赖进行删除
```
### 2.2 删除文件
查找 mysql 的文件

```
find / -name mysql
```
删除

```
rm -rf folderORFileNamw
```

## 3 安装
### 3.1 安装软件
依次执行

```
rpm -ivh mysql-community-common-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.28-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.28-1.el7.x86_64.rpm
```

**安装过程中可能会遇到的问题**

①包冲突，异常如下图：
![](http://img.lsof.fun/2020-03-02-15831593202506.jpg)
*此图引用于（https://blog.csdn.net/hao134838/article/details/80163181）*

请卸载：

```
[root@localhost ~]# rpm -qa | grep postfix
postfix-2.10.1-6.el7.x86_64
[root@localhost ~]#  rpm -qa | grep mariadb
mariadb-libs-5.5.35-3.el7.x86_64

[root@localhost ~]# rpm -ev postfix-2.10.1-6.el7.x86_64
Preparing packages...
postfix-2:2.10.1-6.el7.x86_64
[root@localhost ~]# rpm -ev mariadb-libs-5.5.52-1.el7.x86_64
Preparing packages...
mariadb-libs-1:5.5.52-1.el7.x86_64
```

②缺失依赖包，异常如下图：

![67FA2904-492C-4599-AE7B-AD561634BE43](http://img.lsof.fun/2020-03-02-67FA2904-492C-4599-AE7B-AD561634BE43.png)

安装相关依赖包

```
yum install -y perl net-tools
```

## 4 初始化
### 4.1 初始化data目录文件

```
mysqld --initialize --user=mysql
```

运行完成后就会在 `/var/lib/mysql` 输出相关文件，就完成初始化动作。

###  4.2 修改 root 密码

数据库 `root` 密码在 `/var/log/mysqld.log ` 最后一行，可以使用命令查看密码

```
cat /var/log/mysqld.log 
```

启动 `Mysql` 服务

```
service mysqld restart
```

修改 `root` 密码

```
mysql -u root -p
#输入 `/var/log/mysqld.log` 中的密码登陆，这个时候会让你重置新密码

mysql> SET password=PASSWORD('新密码');
mysql> flush privileges;
```

设置 `root` 可以在外面任何机器可以访问。

```
mysql> use mysql;
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '新密码' WITH GRANT OPTION;
mysql> select host, user from user;
+-----------+--------+
| host      | user   |
+-----------+--------+
| %         | root   |
| %         | root@% |
| 127.0.0.1 | root   |
| ::1       | root   |
| bogon     | root   |
+-----------+--------+
5 rows in set (0.00 sec)
```
存在 `%         | root ` 即可成功添加

到此为止就安装完成了 `Mysql`，一下是调优配置

## 5 其他配置

### 5.1 配置忽略大小写

修改文件 `/etc/my.cnf` 增加一行

```
lower_case_table_names = 1
```

附上我的 `my.cnf` 文件

```shell
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
read_rnd_buffer_size = 128M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

max_connections=2000
event_scheduler = 1
lower_case_table_names = 1

join_buffer_size = 512M
sort_buffer_size = 20M
read_rnd_buffer_size = 128M 

sql_mode = STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

```

### 5.2 修改 datadir 目录
由于默认的目录（/var/lib/mysql）存储是不够的，所以需要迁移到其他地方 。

#### 5.2.1 修改 /etc/my.cnf 文件
假设：
新目录地址 `/home/mysql/data`。
原目录地址 `/var/lib/mysql`

找到 `datadir` 配置的目录地址，修改至目标目录，如下

```shell
.....
# sort_buffer_size = 2M
read_rnd_buffer_size = 128M
# 只能修改这里
datadir=/home/mysql/data
# 试过修改sock后会报错，不允许修改这里
socket=/var/lib/mysql/mysql.sock
.....
```

复制 `/var/lib/mysql` 到 `/home/mysql/data`，命令如下

```
cp -R /var/lib/mysql/* /home/mysql/data/
```

关闭掉 `setenforce`，命令如下：

```shell
[root@localhost ~]# setenforce 0
[root@localhost ~]# getenforce 
Permissive
```

重启即可生效。

### 5.3 window 版
#### 5.3.1 注册服务
进入到 `mysql` 目录，修改下面命令中的 `MYSQL_Sername_name` 为注册服务的名称，后面是具体 `my.ini` 路径

```sheel
mysqld install MYSQL_Sername_name --defaults-file="D:\mysql-5.7.26-winx64\mysql-5.7.26-winx64\my.ini"
```

#### 5.3.2 初始化服务

```sheel
mysqld.exe --initialize --user=root --console
```
 
## 6 参考文章
https://blog.csdn.net/wudinaniya/article/details/82979645?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task

https://blog.csdn.net/hao134838/article/details/80163181

