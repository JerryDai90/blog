# mysql执行存储过程抛出错误（ Error Code: 1366. Incorrect string value: '\xE9\x93\x81\xE6\x9C\xA8...' for column ...）

经过搜索和排查，最终得知是字符集的问题，恰好在创建里面用到的表和存储过程的时候不知道哪位兄台修改了数据的字符集。排查步骤如下

* **查看数据库的编码**

```sql
-- databaseName数据库名称
show create database databaseName;
```
会返回数据库的字符集编码。

* **查看存储过程编译的时候的编码**

```SQL
-- procedureName存储过程名称
show create procedure procedureName;
```
这个编码要和数据库的编码要一致，如果不一致，重新修改一下执行就设定为当前数据库设置的字符了。

* **查看里面使用的表字段字符集**

```sql
-- tableName需要查询的表名
show full columns from tableName;  
```
这个编码也是要和数据库的编码要一致。结构如下，需要看 Collation 的字符集（主要在字符类型的字段）
![](http://img.lsof.fun/2018-08-08-15337089427558.jpg)

如果上面三点的字符集一致的情况下存储过程是可以执行的。

