# Docker
关于Docker的配置使用及问题，还有项目在Docker

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

采用分层存储(union Fs技术,ubuntu下为overlay2),镜像是一个虚拟的概念,为多个文件的系统组成的,每一层都是在上一层的基础上修改的


### 容器 container

1. 动态的,实质是进程,进程运行于自己的命名空间
2. 也是分层存储,该层为容器存储层,以镜像为基础层上创建一层作为存储层
3. 容器存储层随容器的存在而存在
4. 不写入任何数据,保持无状态,数据在数据卷上操作或者绑定宿主目录,独立于容器之外.
6. 以前台执行


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
docker image ls (-a) (-q) (--digests)(-f)

// -a可以列出包括中间镜像在内的所有镜像,否则为顶级镜像
// -q可以列出所有镜像的id
// --digests可以列出镜像摘要
// -f可以按满足要求的列出镜像
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
### 删除镜像

```sh
$ docker image rm XXX

// 可以删除根据命令获取到的镜像
$ docker image rm $(docker image ls -q)
```

### 使用Dockerfile定制镜像

Dockerfile是一个脚本文件,描述了该镜像层所有修改,安装,操作,配置命令

```sh
// 创建文件,每一次的修改镜像是在原来的基础上再创建一个镜像层
$ touch Dockerfile

// file内容

//FROM imagename   // 若imagename为scratch,则创建的为镜像第一层
// RUN echo '<h1>hello</h1>' > /usr/share/nginx/html/index.html
```

```sh
$ docker build [选项] <上下文路径/URL/->

//example

$ docker build -t nginx:v3 .

// . 为上下文路径,本地操作(客户端)docker命令是通过rest api来和docker引擎(服务器端)进行交互,build命令实际上是在服务器端进行构建的.制定上下文路径,该命令会将该路径下的所有文件打包传至docker引擎
```

#### Dockerfile中常用命令

1. FROM
2. RUN
3. COPY:使用该命令是把源文件的元数据(读写权限,变更时间等)都会保存

```sh
COPY <源路径>... <目标路径>
COPY ["<源路径1>",... "<目标路径>"]

// 源路径为构建上下文路径,目标路径为新一层镜像路径
```
4. CMD:指定默认容器主进程启动命令的(容器相当于一个进程,启动时需要指定所需要的程序和参数)
```sh
shell 格式：CMD <命令>
exec 格式：CMD ["可执行文件", "参数1", "参数2"...]
参数列表格式：CMD ["参数1", "参数2"...]。在指定了 ENTRYPOINT 指令后，用 CMD 指定具体的参数。

// 注意:一般使用exec格式,可以解析时格式解析为json数组,所以必须使用双引号.使用shell格式会先解析为 sh -c 参数格式进行执行
```
5. ENTRYPOINT:和CMD相同.使用该命令可以run时在镜像名后面可以带参数,比如-i

```sh
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]

$ docker run xxx -i
```
```sh
ENTRYPOINY ["xxx.sh"]

CMD ["redis-server"]

// 注意:如果使用ENTRYPOINT执行,那CMD制定的为参数可以调用
```

6. ENV:设置环境变量,有点类似于代码中的const定义的变量,定义好的变量可以被其他命令拿来用,使用$envname来获取
