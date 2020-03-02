# mysql 崩溃
导入了一张100W 多万的表，导入到一半mysql 就挂了，怎么重启服务都启动不了。系统页没有沾满空间。

这个时候直接使用 mysqld 来启动，这个时候系统也是启动失败的，这个时候我就删除掉有问题的库。再次启动的时候就可以了。

```
/usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/mysql/plugin --user=mysql --skip-log-error --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/lib/mysql/mysql.sock &
```

```
018-11-15T02:25:26.216519Z 0 [ERROR] InnoDB: Datafile './itsf4_hn/wfexpression.ibd' is corrupted. Cannot determine the space ID from the first 64 pages.
2018-11-15T02:25:26.219646Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfform.ibd' with space ID 24779, since the redo log references ./itsf4_hn/wfform.ibd with space ID 24505.
2018-11-15T02:25:26.223857Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfjsstrategy.ibd' with space ID 24780, since the redo log references ./itsf4_hn/wfjsstrategy.ibd with space ID 24506.
2018-11-15T02:25:26.230234Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wflastsubmitlog.ibd' with space ID 24781, since the redo log references ./itsf4_hn/wflastsubmitlog.ibd with space ID 24507.
2018-11-15T02:25:26.247740Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wflatelyactor.ibd' with space ID 24782, since the redo log references ./itsf4_hn/wflatelyactor.ibd with space ID 24508.
2018-11-15T02:25:26.250235Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wflockinfo.ibd' with space ID 24783, since the redo log references ./itsf4_hn/wflockinfo.ibd with space ID 24509.
2018-11-15T02:25:26.252935Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfoperaterlog.ibd' with space ID 24784, since the redo log references ./itsf4_hn/wfoperaterlog.ibd with space ID 24510.
2018-11-15T02:25:26.261046Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wforgstrategy.ibd' with space ID 24785, since the redo log references ./itsf4_hn/wforgstrategy.ibd with space ID 24511.
2018-11-15T02:25:26.263893Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfparticipant.ibd' with space ID 24786, since the redo log references ./itsf4_hn/wfparticipant.ibd with space ID 24512.
2018-11-15T02:25:26.266675Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfprinter.ibd' with space ID 24787, since the redo log references ./itsf4_hn/wfprinter.ibd with space ID 24513.
2018-11-15T02:25:26.269732Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfprocess.ibd' with space ID 24788, since the redo log references ./itsf4_hn/wfprocess.ibd with space ID 24514.
2018-11-15T02:25:26.272724Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfprocessattach.ibd' with space ID 24789, since the redo log references ./itsf4_hn/wfprocessattach.ibd with space ID 24515.
2018-11-15T02:25:26.276628Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfprocessformprint.ibd' with space ID 24790, since the redo log references ./itsf4_hn/wfprocessformprint.ibd with space ID 24516.
2018-11-15T02:25:26.281096Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfprocessinstance.ibd' with space ID 24791, since the redo log references ./itsf4_hn/wfprocessinstance.ibd with space ID 24517.
2018-11-15T02:25:26.284610Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfprocessinstext.ibd' with space ID 24792, since the redo log references ./itsf4_hn/wfprocessinstext.ibd with space ID 24518.
2018-11-15T02:25:26.292722Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfrecycledata.ibd' with space ID 24793, since the redo log references ./itsf4_hn/wfrecycledata.ibd with space ID 24519.
2018-11-15T02:25:26.307107Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfroute.ibd' with space ID 24794, since the redo log references ./itsf4_hn/wfroute.ibd with space ID 24520.
2018-11-15T02:25:26.314730Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfselectstrategy.ibd' with space ID 24795, since the redo log references ./itsf4_hn/wfselectstrategy.ibd with space ID 24521.
2018-11-15T02:25:26.319170Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfshowstrategy.ibd' with space ID 24796, since the redo log references ./itsf4_hn/wfshowstrategy.ibd with space ID 24522.
2018-11-15T02:25:26.320955Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfsyscode.ibd' with space ID 24797, since the redo log references ./itsf4_hn/wfsyscode.ibd with space ID 24523.
2018-11-15T02:25:26.324213Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wftoread.ibd' with space ID 24798, since the redo log references ./itsf4_hn/wftoread.ibd with space ID 24524.
2018-11-15T02:25:26.335248Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfworkday.ibd' with space ID 24799, since the redo log references ./itsf4_hn/wfworkday.ibd with space ID 24525.
2018-11-15T02:25:26.338531Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfworkitemn.ibd' with space ID 24800, since the redo log references ./itsf4_hn/wfworkitemn.ibd with space ID 24526.
2018-11-15T02:25:26.341828Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfworkitemnlog.ibd' with space ID 24801, since the redo log references ./itsf4_hn/wfworkitemnlog.ibd with space ID 24527.
2018-11-15T02:25:26.344871Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfworkitemo.ibd' with space ID 24802, since the redo log references ./itsf4_hn/wfworkitemo.ibd with space ID 24528.
2018-11-15T02:25:26.350222Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wfworkoperationlog.ibd' with space ID 24803, since the redo log references ./itsf4_hn/wfworkoperationlog.ibd with space ID 24529.
2018-11-15T02:25:26.353173Z 0 [Note] InnoDB: Ignoring data file './itsf4_hn/wx_menu.ibd' with space ID 24804, since the redo log references ./itsf4_hn/wx_menu.ibd with space ID 24530.
2018-11-15T02:25:26.443005Z 0 [Note] InnoDB: Doing recovery: scanned up to log sequence number 7312203052
2018-11-15T02:25:26.597373Z 0 [Note] InnoDB: Database was not shutdown normally!
2018-11-15T02:25:26.597401Z 0 [Note] InnoDB: Starting crash recovery.
2018-11-15T02:25:26.598619Z 0 [ERROR] InnoDB: Tablespace 24778 was not found at ./itsf4_hn/wfexpression.ibd.
2018-11-15T02:25:26.598635Z 0 [ERROR] InnoDB: Set innodb_force_recovery=1 to ignore this and to permanently lose all changes to the tablespace.
2018-11-15T02:25:26.598663Z 0 [ERROR] InnoDB: Cannot continue operation.

```


