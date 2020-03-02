# mysql 5.6.38 安装（redhat 6）

```
本文只适用于通过 rpm 二进制的方式安装，使用源码安装的请自行百度。
```

## 1. 准备步骤
下载地址 ：[https://dev.mysql.com/downloads/mysql/5.6.html](https://dev.mysql.com/downloads/mysql/5.6.html)

只需要以下安装文件

```
MySQL-client*.rpm
MySQL-devel*.rpm
MySQL-server*.rpm 
```
**需要使用 root 的账号进行操作**

## 2. 卸载

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
rpm -ivh MySQL-devel*.rpm
rpm -ivh MySQL-client*.rpm
rpm -ivh MySQL-server*.rpm 
```

安装输出的日志如下

```
[root@bogon MySQL-5.6.43-1.el7.x86_64.rpm-bundle]# rpm -ivh MySQL-devel-5.6.43-1.el7.x86_64.rpm 
warning: MySQL-devel-5.6.43-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:MySQL-devel-5.6.43-1.el7         ################################# [100%]
[root@bogon MySQL-5.6.43-1.el7.x86_64.rpm-bundle]# rpm -ivh MySQL-client-5.6.43-1.el7.x86_64.rpm 
warning: MySQL-client-5.6.43-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:MySQL-client-5.6.43-1.el7        ################################# [100%]
[root@bogon MySQL-5.6.43-1.el7.x86_64.rpm-bundle]# rpm -ivh MySQL-s
MySQL-server-5.6.43-1.el7.x86_64.rpm         MySQL-shared-5.6.43-1.el7.x86_64.rpm         MySQL-shared-compat-5.6.43-1.el7.x86_64.rpm
[root@bogon MySQL-5.6.43-1.el7.x86_64.rpm-bundle]# rpm -ivh MySQL-server-5.6.43-1.el7.x86_64.rpm 
warning: MySQL-server-5.6.43-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:MySQL-server-5.6.43-1.el7        ################################# [100%]
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_CTYPE = "UTF-8",
	LANG = "zh_CN.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to the standard locale ("C").
2019-02-13 14:09:05 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2019-02-13 14:09:05 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2019-02-13 14:09:05 0 [Note] /usr/sbin/mysqld (mysqld 5.6.43) starting as process 15754 ...
2019-02-13 14:09:05 15754 [Note] InnoDB: Using atomics to ref count buffer pool pages
2019-02-13 14:09:05 15754 [Note] InnoDB: The InnoDB memory heap is disabled
2019-02-13 14:09:05 15754 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2019-02-13 14:09:05 15754 [Note] InnoDB: Memory barrier is not used
2019-02-13 14:09:05 15754 [Note] InnoDB: Compressed tables use zlib 1.2.11
2019-02-13 14:09:05 15754 [Note] InnoDB: Using Linux native AIO
2019-02-13 14:09:05 15754 [Note] InnoDB: Using CPU crc32 instructions
2019-02-13 14:09:05 15754 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2019-02-13 14:09:05 15754 [Note] InnoDB: Completed initialization of buffer pool
2019-02-13 14:09:05 15754 [Note] InnoDB: The first specified data file ./ibdata1 did not exist: a new database to be created!
2019-02-13 14:09:05 15754 [Note] InnoDB: Setting file ./ibdata1 size to 12 MB
2019-02-13 14:09:05 15754 [Note] InnoDB: Database physically writes the file full: wait...
2019-02-13 14:09:05 15754 [Note] InnoDB: Setting log file ./ib_logfile101 size to 48 MB
2019-02-13 14:09:06 15754 [Note] InnoDB: Setting log file ./ib_logfile1 size to 48 MB
2019-02-13 14:09:07 15754 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
2019-02-13 14:09:07 15754 [Warning] InnoDB: New log files created, LSN=45781
2019-02-13 14:09:07 15754 [Note] InnoDB: Doublewrite buffer not found: creating new
2019-02-13 14:09:07 15754 [Note] InnoDB: Doublewrite buffer created
2019-02-13 14:09:07 15754 [Note] InnoDB: 128 rollback segment(s) are active.
2019-02-13 14:09:07 15754 [Warning] InnoDB: Creating foreign key constraint system tables.
2019-02-13 14:09:07 15754 [Note] InnoDB: Foreign key constraint system tables created
2019-02-13 14:09:07 15754 [Note] InnoDB: Creating tablespace and datafile system tables.
2019-02-13 14:09:07 15754 [Note] InnoDB: Tablespace and datafile system tables created.
2019-02-13 14:09:07 15754 [Note] InnoDB: Waiting for purge to start
2019-02-13 14:09:07 15754 [Note] InnoDB: 5.6.43 started; log sequence number 0
A random root password has been set. You will find it in '/root/.mysql_secret'.
2019-02-13 14:09:08 15754 [Note] Binlog end
2019-02-13 14:09:08 15754 [Note] InnoDB: FTS optimize thread exiting.
2019-02-13 14:09:08 15754 [Note] InnoDB: Starting shutdown...
2019-02-13 14:09:10 15754 [Note] InnoDB: Shutdown completed; log sequence number 1625977


2019-02-13 14:09:10 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2019-02-13 14:09:10 0 [Note] Ignoring --secure-file-priv value as server is running with --bootstrap.
2019-02-13 14:09:10 0 [Note] /usr/sbin/mysqld (mysqld 5.6.43) starting as process 15776 ...
2019-02-13 14:09:10 15776 [Note] InnoDB: Using atomics to ref count buffer pool pages
2019-02-13 14:09:10 15776 [Note] InnoDB: The InnoDB memory heap is disabled
2019-02-13 14:09:10 15776 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2019-02-13 14:09:10 15776 [Note] InnoDB: Memory barrier is not used
2019-02-13 14:09:10 15776 [Note] InnoDB: Compressed tables use zlib 1.2.11
2019-02-13 14:09:10 15776 [Note] InnoDB: Using Linux native AIO
2019-02-13 14:09:10 15776 [Note] InnoDB: Using CPU crc32 instructions
2019-02-13 14:09:10 15776 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2019-02-13 14:09:10 15776 [Note] InnoDB: Completed initialization of buffer pool
2019-02-13 14:09:10 15776 [Note] InnoDB: Highest supported file format is Barracuda.
2019-02-13 14:09:10 15776 [Note] InnoDB: 128 rollback segment(s) are active.
2019-02-13 14:09:10 15776 [Note] InnoDB: Waiting for purge to start
2019-02-13 14:09:10 15776 [Note] InnoDB: 5.6.43 started; log sequence number 1625977
2019-02-13 14:09:10 15776 [Note] Binlog end
2019-02-13 14:09:10 15776 [Note] InnoDB: FTS optimize thread exiting.
2019-02-13 14:09:10 15776 [Note] InnoDB: Starting shutdown...
2019-02-13 14:09:12 15776 [Note] InnoDB: Shutdown completed; log sequence number 1625987




A RANDOM PASSWORD HAS BEEN SET FOR THE MySQL root USER !
You will find that password in '/root/.mysql_secret'.

You must change that password on your first connect,
no other statement but 'SET PASSWORD' will be accepted.
See the manual for the semantics of the 'password expired' flag.

Also, the account for the anonymous user has been removed.

In addition, you can run:

  /usr/bin/mysql_secure_installation

which will also give you the option of removing the test database.
This is strongly recommended for production servers.

See the manual for more instructions.

Please report any problems at http://bugs.mysql.com/

The latest information about MySQL is available on the web at

  http://www.mysql.com

Support MySQL by buying support/licenses at http://shop.mysql.com

WARNING: Found existing config file /usr/my.cnf on the system.
Because this file might be in use, it was not replaced,
but was used in bootstrap (unless you used --defaults-file)
and when you later start the server.
The new default config file was created as /usr/my-new.cnf,
please compare it with your file and take the changes you need.
```

root 密码在 `/root/.mysql_secret`, 可以使用命令查看密码 

```
cat /root/.mysql_secret
``` 

此时安装完成后 mysql 已经自动启动。查看是否启动成功可以使用 `lsof -i:3306` 来查看。

```
如果没有启动，则说明安装失败了（或者使用 service mysql start 启动一下），需要看一下 `/var/lib/mysql/机器名称.err`，如果 /var/lib/mysql/ 什么都没有，没有 `mysql` 库，重新安装一下  `MySQL-server*.rpm ` 即可。下图是正常的
```
![](http://img.lsof.fun/2019-02-04-15482366261875.jpg)



### 3.2 配置 my.cnf
使用命令 `find / -name my-default.cnf` 找到此文件。
复制配置文件到 `/etc/` 目录下

```
cp thisFilePath/my-default.cnf /etc/my.cnf
```

编辑修改 my.cnf 文件，只需要修改以下配置

```
 # mysql 安装的基础目录
 basedir = /usr
 # mysql 数据库文件目录（默认目录）
 datadir = /var/lib/mysql
 # 端口（可随意修改）
 port = 3306
 # 大小写忽略（用于表名、字段）
 lower_case_table_names=1
```

附上我的 my.cnf 配置

```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html
# *** DO NOT EDIT THIS FILE. It's a template which will be copied to the
# *** default location during install, and will be replaced if you
# *** upgrade to a newer version of MySQL.

[mysqld]

# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M

# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin

# These are commonly set, remove the # and set as required.
 basedir = /usr
 datadir = /var/lib/mysql
 port = 3306
# server_id = .....
# socket = .....

# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
 join_buffer_size = 128M
 sort_buffer_size = 16M
# read_rnd_buffer_size = 2M
 lower_case_table_name = 1

thread_cache_size = 16 
query_cache_size = 128M
max_connections = 3000 
 
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 

```


### 3.3 root 密码修改和外部访问

```
mysql -u root -p
#输入 `/root/.mysql_secret` 中的密码登陆，这个时候会让你重置新密码

mysql> SET password=PASSWORD('新密码');
mysql> flush privileges;
```

设置 root 可以在外面任何机器可以访问。

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

## 4. 备注

以下为mysql 安装后的目录说明

| Directory | Contents of Directory |
| --- | --- |
|/usr/bin | Client programs and scripts|
|/usr/sbin |The mysqld server|
|/var/lib/mysql|Log files, databases|
|/usr/share/info|Manual in Info format|
|/usr/share/man|Unix manual pages|
|/usr/include/mysql|Include (header) files|
|/usr/lib/mysql|Libraries|
|/usr/share/mysql|Miscellaneous support files, including error messages,character set files, sample configuration files, SQL for database installation|
|/usr/share/sql-bench|Benchmarks|

## 5. 额外加餐

注意，window 下的文件夹路径需要注意，如果盘符后面是有转义的字符，需要加上2个反斜杠
![-w500](http://img.lsof.fun/2019-02-04-15492117790460.jpg)



## 6. 安装参考链接
[http://blog.csdn.net/u010257584/article/details/51320542](http://blog.csdn.net/u010257584/article/details/51320542)

[mysql my.cnf 说明](http://m.blog.itpub.net/26736162/viewspace-2135880/)
[https://blog.csdn.net/hrayha/article/details/46128731](https://blog.csdn.net/hrayha/article/details/46128731)

[https://blog.csdn.net/hqmanwangkai/article/details/83225446](https://blog.csdn.net/hqmanwangkai/article/details/83225446)

[https://www.cnblogs.com/cnblogsfans/archive/2009/09/21/1570942.html](https://www.cnblogs.com/cnblogsfans/archive/2009/09/21/1570942.html)



