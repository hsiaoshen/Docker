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
