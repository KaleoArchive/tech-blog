+++
draft = false
title = "Docker Tips"
date = "2016-12-27T22:33:52+08:00"

+++

# 垃圾清理

Docker 1.13引进了`prune`，这个命令让清理垃圾看起来更优雅。但如果是在 Docker 1.13以前的版本，可能下面这些江湖流传的小技巧仍对你有用。



```shell
# 清理所有none镜像
$ docker rmi $(docker images -q -f "dangling=true")

# 删除指定镜像
$ docker rmi $(docker images | grep "hub.kaleocheng.com" | awk '{print $3}')
# 或者下面这个更好
$ docker rmi $(docker images | grep hacker | awk '{print $1','$2}' | sed  "s/ /:/g")

# 停止所有容器
$ docker stop $(docker ps -a -q)

# 删除所有停止的容器
$ docker rm $(docker ps -a -q)
```

# 封装成命令

你知道，当人们不想在电脑上安装许多依赖的时候，就会想要封装。

这时候，你可以使用`alias`。比如我是这么使用 sass 的：

```shell
alias sass="docker run -it --rm -v \$(pwd):\$(pwd) -w \$(pwd) kaleocheng/sass"
```

# 网络不通

你刚刚在一台主机上安装好 Docker，然后主机就 Ping 不通了。

首先看看你的本机网络是不是`172.17.xxx.xxx`，如果不是， Google 在等你呢。如果是，哥们，别担心，我们一分钟就能搞定它。



Docker 安装好后会新建一个默认的 `docker0` 网桥为`172.17.0.1`，如果你本地网络的网段也是这样，就会造成网段的冲突。解决办法是更改 Docker 的默认网段：

```shell
$ sudo service docker stop 
```

```shell
$ sudo vim /lib/systemd/system/docker.service
# 在ExecStart这行加上 --bip=10.40.0.1/16 即可
# 有些版本的service文件可能有些许不同，没关系，你试着加一下，多半会成功的
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/docker daemon -H fd:// --bip=10.40.0.1/16
MountFlags=slave
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

[Install]
WantedBy=multi-user.target
```

```shell
$ sudo systemctl daemon-reload
$ sudo service docker start 
```



如果你还在使用`docker-compose`，那你得再解决一个问题才能到达炼狱，因为 Docker Compose 默认会给每个项目新建一个网桥，而这个网桥默认也是`172.17.0.1`。解决方法是手动新建一个网桥，然后在docker-compose 中使用外部网络。

# 进入Mac里的Docker虚机

相信你没有再使用 docker2boot 了吧？Docker for Mac 使用了 MacOS 原生的虚拟化技术，底层运行了一个很轻量级的系统来跑 Docker，如果你想进去瞧一瞧：

```shell
$ screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty

# 退出： Ctrl - A  and Ctrl - \
```
