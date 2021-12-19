　　在 `macOS` 下，使用 brew 安装了 `rabbitMQ`，发现外部是访问不了 `5672` 端口的。使用命令查看后改端口只监听了 `localhost`，那就可以解释为什么不能访问了。如下图：

　　![w500](http://img.lsof.fun/2020-03-09-15836496895109.jpg)  
① 处只监听了 localhost，② 处是不限制。所以需要把 `amqp` 监听修改为指定 IP 等。

　　修改此文件 `/usr/local/Cellar/rabbitmq/3.7.16/sbin/rabbitmq-env`，（文件路径按实际目录来，这个是我本机地址）增加如下代码：

```elang
...
# 增加这行，并且修改为本机IP（多网卡的需要注意绑定的网卡是否是其他机器可访问）
NODE_IP_ADDRESS=192.168.0.105

DEFAULT_NODE_IP_ADDRESS=auto
DEFAULT_NODE_PORT=5672
...
```

　　重启即可生效

## 参考文献

　　[http://liubin.nanshapo.com/2013/09/04/rabbitmq-on-os-x/](http://liubin.nanshapo.com/2013/09/04/rabbitmq-on-os-x/)
