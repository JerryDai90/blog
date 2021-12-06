# mysql 8.0 安装（centos 7.0）

## 1. 准备安装包

## 2. 卸载现有的

```
rpm -qa|grep mariadb
```

　　如果有结果，就执行

```
yum remove mariadb*
```

## 3. 安装服务

　　依次支持一下脚本，一般来说不会有依赖问题

```
rpm -ivh mysql-community-common-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-client-plugins-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-libs-8.0.25-1.el7.x86_64.rpm 
rpm -ivh mysql-community-devel-8.0.25-1.el7.x86_64.rpm 
```

> 执行会报错
>

```
warning: mysql-community-devel-8.0.25-1.el7.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID 5072e1f5: NOKEY
error: Failed dependencies:
	pkgconfig(openssl) is needed by mysql-community-devel-8.0.25-1.el7.x86_64
```

　　安装依赖

```
yum install openssl-devel
```

　　再次安装

```
rpm -ivh mysql-community-devel-8.0.25-1.el7.x86_64.rpm 
```

　　安装 mysql-community-client-8.0.25-1.el7.x86_64.rpm

```
rpm -ivh mysql-community-client-8.0.25-1.el7.x86_64.rpm 
```

　　安装依赖

```
yum install net-tools
yum install perl
```

　　安装最后一个服务

```
rpm -ivh mysql-community-server-8.0.25-1.el7.x86_64.rpm 
```

## 4. 初始化数据

```
mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql --console
```

> 如果需要设置忽略大小写，先修改配置文件/etc/my.cnf 再进行初始化。如果已经初始化了可以删除掉 /var/lib/mysql 下的所有文件，再次初始化
>

　　初始化后会生产随机 root 密码 /var/log/mysqld.log

```
2021-07-08T08:42:38.153215Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2021-07-08T08:42:39.020174Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2021-07-08T08:42:41.556118Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: ;Hvv,r*OV4<y
```

## 5. 修改密码

```
mysql -u root -p 
```

　　输入 `/var/log/mysqld.log ` 中的密码

　　执行修改密码操作

```
alter user 'root'@'localhost' identified by '123456';
```

> 默认一定是需要修改密码才行执行其他命令的，否则报一下错误
>

```
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

　　进入 mysql 库，执行下面命令，让外部可以使用 root 登陆。

```
use mysql
update user set host='%' where user ='root';
```

　　如果使用 Navicat 登陆的需要重新设置一下密码

```
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
```

　　开放端口

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent  
firewall-cmd --reload
```

## 6. 附录

　　如果出现权限问题则执行

```
chown -R mysql:mysql /var/lib/mysql/
chown -R mysql:mysql /var/run/mysqld/
```
