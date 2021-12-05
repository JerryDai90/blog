# Ubuntu 20.04 自动开机自启动

## 1.准备sh脚本文件

```
sudo gedit /home/tom/test/my.sh
```

```
#! /bin/bash
echo "success..."
```

```
chmod +x /home/tom/test/my.sh
```

## 2.创建一个service文件

　　进入/etc/systemd/system/，创建一个my.service文件，

```
sudo gedit  /etc/systemd/system/my.service
```

　　内容如下：

```
[Unit]
Description=just for test                     #这里填简介
After=BBB.service　XXX.service  AAA.service   #这里填上你这个脚本所需要的前置service，都在/etc/systemd/system/下
 
[Service]
ExecStart=/home/tom/test/my.sh                 #这里填sh文件路径，比如这里运行了这个my.sh，后面也可以跟参数，比如 -D -I                                                                                                                                  
 
[Install]
WantedBy=multi-user.target
```

## 3.启动服务

　　使用以下命令使能这个服务开机启动：

　　重新加载配置文件

```
sudo systemctl daemon-reload               #service文件改动后要重新转载一下
sudo systemctl enable my.service          #这句是为了设置开机启动
```

　　如果你想不重启立刻使用这个sh脚本，就运行下面这句：

　　重启相关服务

```
$ sudo systemctl start my.service           #启动服务
```
