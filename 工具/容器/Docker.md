# Docker
## 1. 容器技术简介
 - 容器 : ==**就是运行 APP 的**==
     - 对软件和其依赖的标准化打包
     - 应用之间相互隔离
     - 共享一个 OSKernel
     - 可以运行在很多主流操作系统之上
- 容器是在 APP 层面的隔离
- 容器编排工具 : 对容器的创建,运行,调度等

## 2. Docker 环境的各种搭建
### 2.1 Mac 
- ==**Mac 安装**==

### Centos

```shell
<!--查看是否有旧版本的 Docker,如果有就删除-->
[root@linux ~]# sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
<!--安装一些必须的包-->
[root@linux ~]# sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  
<!--设置稳定的库-->
[root@linux ~]# sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    

<!-- 安装最新版本的 Docker -->
[root@linux ~]# sudo yum install docker-ce docker-ce-cli containerd.io

<!--启动 Docker-->
[root@linux ~]# sudo systemctl start docker
```

### 2.2 Docker Machine
- Docker Machine 可以自动在一台虚拟机上安装
- Mac 上安装 Docker,Docker machine 就自动安装好了

```properties
Docker-machine create demo : 创建一个 Docker-machine,名称为 demo
docker-machine ls : 列出当前创建好或者运行的 Docker
docker-machine ssh demo : 进入到名称为 demo 的 docker-machine

```

## 3. Docker 的镜像和容器
### 3.1 Docker 的架构和底层技术
- 后台进程 dockerd,就是 Docker 的 server: 用于维护 
- Rest API Server :
- CLI 接口 : 

- ==*Docker 底层技术支持*==
    - Namespaces : 做隔离
    - Control groups : 做资源控制
    - Union file systems : container 和 image 的分层


### 3.2 Docker ==**Image(镜像)**==
- Image
    - 文件和 meta data 的集合
    - 分层的,并且每一层都可以添加改变删除文件,成为一个新的 image
    - 不同的 image 可以共享相同的 layer
    - image 本身是 read-only 的

#### 3.2.1 docker image 的获取
##### 3.2.1.1 ==*从 Registry 上获取*==
- docker pull xxx
    - 如果不加版本号,默认获取最新版

#### 3.2.2 创建一个 base image





## 4. Docker 的网络
## 5. Docker 的持久化存储和数据共享
## 6. Docker Compose 多容器部署
## 7. 容器编排工具 : Docker Swarm
## 8. Docker Cloud 和 Docker 企业版
## 9. 容器编排 Kubernetes
## 10. 容器的运维和监控
## 11. Docker + DevOps 实战--过程和工具

## 用户手册
### 命令

```properties
docker version
docker image ls : 查看已安装的 image
```