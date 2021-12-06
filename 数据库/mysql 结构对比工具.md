# mysql 结构对比工具

　　市面上有 2 中工具一个是 `MySQL Utilities` 和 `AmpNmp.DatabaseCompare`

### MySQL Utilities

　　是官方出品，支持三大平台，使用的 `python` 运行环境。

```
mysqldiff --force --difftype=sql --server1=user:password@ip --server2=user:password@ip --changes-for=server2  server1_schema:server2_schema
```

　　参数说明

```
server1_schema server1对应的库名称
server2_schema server2对应的库名称

其他参数对号入座 user、password、ip
```

　　对比的内容有基本的

* sql 注释、字段长度、名称等。但是不智能的地方是竟然字段大小写不忽略（可能是由于没有读大小写忽略的配置）。
* 存储过程、函数、触发器、视图。
* 自增长的序列（竟然这个都有，这个对比的时候需要酌情考虑要不要同步，毕竟两个系统的数据量不同）
* 可列出 A 库中存在而 B 库不存在的，反之也显示。

　　发现的 bug

* 对比存储过程、函数的时候，认为所有的都有差异。尽管已经使用差异出来的 sql 执行过了。

### AmpNmp.DatabaseCompare

　　没用过，据介绍这个是可视化的工具。具体请移步官网

## 使用环境

　　macOS 10.11.6
python 2.7

　　在使用的过程中发现无法运行 `mysqldiff` 原因是没有安装 `MySQL Connector`
点击下面地址下载对应的工具 http://dev.mysql.com/downloads/connector/python/

### 工具

　　我使用的是下面 2 个工具
mysql-utilities-1.6.5-macos10.12.dmg
mysql-connector-python-2.0.5-osx10.9.dmg

　　可以使用 https://pan.baidu.com/s/1cV2rHVED2fZzVDl72bbPcw 这地址下载

## 参考资料

　　https://stackoverflow.com/questions/21781598/cant-run-mysql-utilities
https://downloads.mysql.com/archives/utilities/
http://blog.sina.com.cn/s/blog_61d758500102w8ed.html
https://blog.csdn.net/kikajack/article/details/81567248
http://ampnmp.com/database-compare/
http://seanlook.com/2017/08/05/mysql_mysqldiff/
