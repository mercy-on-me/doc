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
- docker 命令 : ==*docker +命令关键字command +一系列参数*==
- 如果不知道某个 command的作用,可以使用 ==* docker command --help*== 查看帮助文档

```properties
docker version
docker image ls / docker images : 查看已安装的 image

docker ps 
docker search xxx : 查找 image
docker pull xxx : 下载镜像

容器 : 
docker run : 创建和启动 container
    例如 : docker run -d -p 80:80 httpd
    指定名称 : docker run --name 指定名称 -d -p 80:80 httpd
    
docker container ls / docker ps: 查看运行的容器
    比如 ID 等,ID 默认 128 位,此命令查询出的是 16 位的简略 ID,要查看完整的加上 --no-trunc

docker inspect 容器 ID : 查看容器信息
    可使用 docker run 指定的名称代替 ID
    查看指定信息 : docker inspect -f {{.NetworkSettings.IPAddress}} 名称/ID

docker ps -a | grep container_id : 查看容器状态   
    Up什么什么的 : 说明运行中. 
    Exited : 说明停止
    
docker stop container_id : 停止容器

docker start container_id : 启动容器
    也可以使用在 docker run 时设置的名称代替 ID
    
docker logs 名称/ID : 查看日志

```