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
