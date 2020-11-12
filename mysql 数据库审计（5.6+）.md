# mysql 数据库审计（5.6+）

数据库中需要增加审计功能。

>审计记录包括事件日期、时间、发起者信息（如用户名、IP地址等）、类型、描述和结果（是否成功等）等内容。

`mysql` 孪生兄弟 `mariadb` 的安装包里面有相关的模块，我们只需要对应下载下来即可。本次测试的是windows 所以插件都是dll文件后缀，如果是linux的话后缀是 .so

## 1. 下载 mariadb 版本
下载地址：[https://archive.mariadb.org]()，针对对应的部署平台下载相应的版本。由于找不到 mysql 与 mariadb对应的版本。可以先拿到最新的 mariadb-5.5.68 进行尝试。

### 1.1.1 获取 dll or so
拿到压缩包后，路径：lib\plugin\server_audit.dll

### 1.1.2 复制
复制到 mysql 目录下 lib\plugin\

### 1.1.3 安装
登陆数据库后执行

```sql
INSTALL PLUGIN server_audit SONAME 'server_audit.dll';
```
## 2. 配置 my.ini or my.cnf

```
server_audit=FORCE_PLUS_PERMANENT
server_audit_events=CONNECT,QUERY,TABLE,QUERY_DDL
server_audit_logging=on
server_audit_file_rotate_size=200000001
server_audit_file_rotations=200
server_audit_file_rotate_now=ON

# 审计文件路径，注意，这里的目录都务必存在。会在指定的目录增加 server_audit.log 文件。另外还需要注意反斜杠的转义问题
server_audit_file_path=D:\\MySQL\\logs
```

## 3. 验证
重启mysql 后，谁便执行语句即可在 server_audit.log 中看到数据




