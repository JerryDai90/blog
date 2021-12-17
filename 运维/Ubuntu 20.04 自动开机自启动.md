# Ubuntu 20.04 自动开机自启动

## 1.准备 sh 脚本文件

```
sudo gedit /home/tom/test/my.sh
```

```bash
#! /bin/bash
echo "success..."
```

```
chmod +x /home/tom/test/my.sh
```

## 2.创建一个 service 文件

　　进入/etc/systemd/system/，创建一个 my.service 文件，

```shell
sudo gedit  /etc/systemd/system/my.service
```

　　内容如下：

```properties
[Unit]
#这里填简介
Description=just for test
#这里填上你这个脚本所需要的前置service，都在/etc/systemd/system/下
After=BBB.service　XXX.service  AAA.service

[Service]
#这里填sh文件路径，比如这里运行了这个my.sh，后面也可以跟参数，比如 -D -I
ExecStart=/home/tom/test/my.sh

[Install]
WantedBy=multi-user.target
```

　　范例 

```properties
[Unit]
After=network.service

[Service]
ExecStart=/work/my-demons.sh

[Install]
WantedBy=default.target
```

## 3.启动服务

　　使用以下命令使能这个服务开机启动：

　　重新加载配置文件

```properties
#service文件改动后要重新转载一下
sudo systemctl daemon-reload
#这句是为了设置开机启动
sudo systemctl enable my.service
```

　　如果你想不重启立刻使用这个 sh 脚本，就运行下面这句：

　　重启相关服务

```properties
#启动服务
sudo systemctl start my.service
```
