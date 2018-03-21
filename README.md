# Docker
关于CentOs的Docker的配置使用及问题，还有项目在Docker

[docker](https://yeasy.gitbooks.io/docker_practice/content/introduction/why.html)

## 概念

docker是一个容器,可以在上面部署项目,但是运行环境是宿主内核,自身并没有可运行环境

### 优势

1. 启动快: 秒,甚至是毫秒级启动
2. 不需要虚拟硬件等配置,高效利用系统资源
3. 提供除内核外完整的运行环境,保持环境一致性
4. 持续交付:一次创建配置和环境,任意地方运行,迁移方便

### 和传统虚拟机对比

1. 启动:传统启动分钟级,docker秒级
2. 硬盘:传统的GB,docker的MB
3. 系统支持量:传统的几十个,docker的上千个
4. 性能:传统弱于原生,docker接近


### 镜像 image

操作系统分为内核和用户操作空间,linux下用户操作空间为root文件系统,docker就如同root文件系统一般

采用分层存储(union Fs技术),镜像是一个虚拟的概念,为多个文件的系统组成的


### 容器 container

1. 动态的,实质是进程,进程运行于自己的命名空间
2. 也是分层存储,该层为容器存储层,以镜像为基础层创建
3. 容器存储层随容器的存在而存在
4. 不写入任何数据,保持无状态,数据在数据卷上操作或者绑定宿主目录,独立于容器之外.


### 仓库 Repositroy

1. docker registry: 一个集中存储,分发的公开服务,供用户使用,允许用户管理镜像的服务. Docker Hub是默认的 公开服务
2. 一个服务可有多个库,每个库包含多个标签Tag,类似一个仓库是一个软件,标签就是软件的不同版本.没有指定Tag则默认为latest

## 安装

### Ubuntu 16.04

1. 卸载旧版本

```sh
$ sudo apt remove docker \ docker-engine \ docker.io
```

2. 由于 apt 源使用 HTTPS 以确保软件下载过程中不被篡改。因此，我们首先需要添加使用 HTTPS 传输的软件包以及 CA 证书。

```sh
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```
3. 确保下载软件的合法性,添加软件源的GPG秘钥

```sh

$ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -


# 官方源
# $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

4. source.list中添加docker的软件源

```sh
$ sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"


# 官方源
# $ sudo add-apt-repository \
#    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
#    $(lsb_release -cs) \
#    stable"
```
5. 开始安装

```sh
$ sudo apt-get update

$ sudo apt-get install docker-ce
```
6. 启动 使用systemctl

```sh
$ sudo systemctl enable docker
$ sudo systemctl start docker

```
7. 添加权限

docker命令使用Unix socket与Docker引擎通讯.只有root组或者docker组的用户才可以访问

```sh
// 建立 docker 组

$ sudo groupadd docker

// 将当前用户加入 docker 组

$ sudo usermod -aG docker $USER

// 关闭终端重新登录
```

```sh
// 查看组
cat /etc/group
```

8. 检测docker是否安装成功

```sh
docker run hello-world

// 输出
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:be0cd392e45be79ffeffa6b05338b98ebb16c87b255f48e297ec7f98e123905c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/

```

## 配置国内加速器

### ubuntu 16.04 (systemd系统)

```sh
1. cd /etc/docker/
2. sudo touch daemon.json // 存在无需这步
3. vim daemon.json 

// 写入如下内容(官方为主)

{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ]
}

// 重新启动服务
4. sudo systemctl daemon-reload
5. sudo systemctl restart docker

```

## 命令和操作

### 获取镜像

```sh
$ docker pull <ip:port> <用户名>/<软件名>:<标签名> 
```

参数分析:

1. <ip:port> : docker registry 的地址, 不输入默认为docker hub
2. <用户名>/<软件名>: 不输入用户名默认为library(官方镜像)
3. <标签名>: 为软件版本号

### 以对应镜像启动容器

```sh
$ docker run -it --rm ubuntu:16.04 bash
```
参数:

1. -it:2个参数,i是交互式操作,t是终端
2. --rm : 退出容器后随之将其删除,一般退出后的容器是不会删除的
3. bash: 放在软件名后的是shell命令,bash指删除后的命令为交互式shell

### 列出存在的image

```sh
docker image ls (-a)

// -a可以列出包括中间镜像在内的所有镜像,否则为顶级镜像
```
注: 出现none镜像时,属于原版被新版替换了,这种属于虚悬镜像
```sh
// 查看虚悬镜像
$ docker image ls -f dangling=true

// 删除这些没用的镜像
$ docker image prune
```

### 查看镜像,数据卷还有容器所占用体积

```sh
docker system df
```
