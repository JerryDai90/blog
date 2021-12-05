# mysql 5.7、centos 7 运行一段时间后重启机器后不能启动数据库

> 错误日志看 /etc/my.cnf 里面的 `log-error=/var/log/mysqld.log`
>

　　如题，由于运维原因，重启了机器后发现Mysql 启动不了，查看日志后提示如下，

```
2020-03-02T07:57:46.856711Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-03-02T07:57:46.857040Z 0 [Warning] Can't create test file /home/work/mysql/data/localhost.lower-test
2020-03-02T07:57:46.857107Z 0 [Note] /usr/sbin/mysqld (mysqld 5.7.28) starting as process 27517 ...
2020-03-02T07:57:46.864004Z 0 [Warning] Can't create test file /home/work/mysql/data/localhost.lower-test
2020-03-02T07:57:46.864046Z 0 [Warning] Can't create test file /home/work/mysql/data/localhost.lower-test
2020-03-02T07:57:46.867890Z 0 [Note] InnoDB: PUNCH HOLE support available
2020-03-02T07:57:46.867940Z 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2020-03-02T07:57:46.867948Z 0 [Note] InnoDB: Uses event mutexes
2020-03-02T07:57:46.867955Z 0 [Note] InnoDB: GCC builtin __atomic_thread_fence() is used for memory barrier
2020-03-02T07:57:46.867961Z 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2020-03-02T07:57:46.867966Z 0 [Note] InnoDB: Using Linux native AIO
2020-03-02T07:57:46.868650Z 0 [Note] InnoDB: Number of pools: 1
2020-03-02T07:57:46.868925Z 0 [Note] InnoDB: Using CPU crc32 instructions
2020-03-02T07:57:46.873423Z 0 [Note] InnoDB: Initializing buffer pool, total size = 128M, instances = 1, chunk size = 128M
2020-03-02T07:57:46.890110Z 0 [Note] InnoDB: Completed initialization of buffer pool
2020-03-02T07:57:46.896321Z 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
2020-03-02T07:57:46.906476Z 0 [ERROR] InnoDB: The innodb_system data file 'ibdata1' must be writable
2020-03-02T07:57:46.906501Z 0 [ERROR] InnoDB: The innodb_system data file 'ibdata1' must be writable
2020-03-02T07:57:46.906510Z 0 [ERROR] InnoDB: Plugin initialization aborted with error Generic error
2020-03-02T07:57:47.507923Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2020-03-02T07:57:47.507975Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2020-03-02T07:57:47.507992Z 0 [ERROR] Failed to initialize builtin plugins.
2020-03-02T07:57:47.507996Z 0 [ERROR] Aborting
```

　　其中关键是

```
2020-03-02T07:57:46.906476Z 0 [ERROR] InnoDB: The innodb_system data file 'ibdata1' must be writable
2020-03-02T07:57:46.906501Z 0 [ERROR] InnoDB: The innodb_system data file 'ibdata1' must be writable
2020-03-02T07:57:46.906510Z 0 [ERROR] InnoDB: Plugin initialization aborted with error Generic error
```

　　其中查看 /home/work/mysql/data/ 已经赋予了 775 权限了，依旧不可写，还是报错。百度一通，与 enforce 有关。

```
setenforce 0 
```

> 设置完成后需要重启，如果重启后 setenforce 重启后依旧是 Enforcing，需要 Edit the /etc/selinux/config file and set the SELINUX to permissive
> ![/etc/selinux/config-w500](http://img.lsof.fun/2020-04-28-WeChatWorkScreenshot_f9478372-7a51-4d5f-9c07-b7932fa7d8e0.png)
>

　　也有可能会出现一下异常

```
2020-04-28T03:11:31.744451Z 0 [Warning] Changed limits: max_open_files: 5000 (requested 10000)
2020-04-28T03:11:31.744695Z 0 [Warning] Changed limits: table_open_cache: 1495 (requested 2000)
2020-04-28T03:11:31.910242Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2020-04-28T03:11:31.910274Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2020-04-28T03:11:31.911895Z 0 [Note] /usr/sbin/mysqld (mysqld 5.7.28) starting as process 17864 ...
2020-04-28T03:11:31.916110Z 0 [Note] InnoDB: PUNCH HOLE support available
2020-04-28T03:11:31.916149Z 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2020-04-28T03:11:31.916157Z 0 [Note] InnoDB: Uses event mutexes
2020-04-28T03:11:31.916165Z 0 [Note] InnoDB: GCC builtin __atomic_thread_fence() is used for memory barrier
2020-04-28T03:11:31.916174Z 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2020-04-28T03:11:31.916180Z 0 [Note] InnoDB: Using Linux native AIO
2020-04-28T03:11:31.917079Z 0 [Note] InnoDB: Number of pools: 1
2020-04-28T03:11:31.917205Z 0 [Note] InnoDB: Using CPU crc32 instructions
2020-04-28T03:11:31.919035Z 0 [Note] InnoDB: Initializing buffer pool, total size = 128M, instances = 1, chunk size = 128M
2020-04-28T03:11:31.927466Z 0 [Note] InnoDB: Completed initialization of buffer pool
2020-04-28T03:11:31.929542Z 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
2020-04-28T03:11:31.939664Z 0 [ERROR] InnoDB: Operating system error number 13 in a file operation.
2020-04-28T03:11:31.939688Z 0 [ERROR] InnoDB: The error means mysqld does not have the access rights to the directory.
2020-04-28T03:11:31.939697Z 0 [ERROR] InnoDB: os_file_get_status() failed on './ibdata1'. Can't determine file permissions
2020-04-28T03:11:31.939709Z 0 [ERROR] InnoDB: Plugin initialization aborted with error Generic error
2020-04-28T03:11:32.540484Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2020-04-28T03:11:32.540588Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2020-04-28T03:11:32.540603Z 0 [ERROR] Failed to initialize builtin plugins.
2020-04-28T03:11:32.540611Z 0 [ERROR] Aborting
```
