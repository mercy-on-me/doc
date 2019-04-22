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


## 3 Docker 的架构和底层技术
- 后台进程 dockerd,就是 Docker 的 server: 用于维护 
- Rest API Server :
- CLI 接口 : 

- ==*Docker 底层技术支持*==
    - Namespaces : 做隔离
    - Control groups : 做资源控制
    - Union file systems : container 和 image 的分层


## 4 Docker ==**Image(镜像)**==
- Image
    - 文件和 meta data 的集合
    - 分层的,并且每一层都可以添加改变删除文件,成为一个新的 image
    - 不同的 image 可以共享相同的 layer
    - image 本身是 read-only 的

### 4.1 docker image 的获取
#### 4.1.1 ==*从 Registry 上获取*==
- docker pull xxx
    - 如果不加版本号,默认获取最新版

### 4.2 通过编写 Dockerfile 创建 Docker image
- 步骤 :
    - 编写一个 dockerfile[dockerfile语法](#dockerfile)( 文件约定名为 Dockerfile,不推荐其他的名称)
    - 使用 Docker build 构建 image
#### 4.2.1 简单演示

    ```properties
    创建 Dockerfile
    FROM alpine:latest : 要产生的新 image 有一个 alpine 基础镜像
    MAINTAINER cy : 由谁构建,作者名称
    CMD echo 'hello docker' : 镜像的功能
    
    构建 image
    docker build -t image_name .
    (-t : 给创建的 image 一个标识,名称, 最后的 . 是 Dockerfile 文件的路径,因为使用 Docker build 命令的时候就在 Dokcerfile 路径下,所以使用 . )
    ```

#### 4.2.2 稍复杂点的构建演示

```properties
# 创建 Dockerfile 文件
FROM ubuntu : 要使用的基础镜像
MAINTAINER cy : 此镜像的作者
RUN sed -i 's/archive.ubunt.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN apt-get update : 更新 ubunt 的库
RUN apt-get install -y nginx : 安装 Nginx
COPY index.html /var/www/html : 拷贝本地文件到 image,路径为 Ubuntu 的HTML 的路径
ENTRYPOINT ["/usr/share/nginx", "-g", "daemon off;"] : 给出一个入口
EXPOSE 80 : 暴露的端口

# 构建
docker build -t image_name
```

#### 4.3 构建自己的 image
==**docker commit**== -- create a new image from a container`s changes
- 场景: pull centos 后,centos 内没有 vim.可以安装 vim,然后根据 container commit 成一个新的 image
    
```shell
# pull censot
docker pull centos
# 交互式运行image
docker run -it centos
# 在 centos 中安装 vim
yum install -y vim
# 退出
exit
# 查看实例化出的 container 的 ID
docker ps -a
# 根据 container_id commit 出一个新的 image
docker commit old_container new_image_name
# 查看 image 列表, 此时可以看到创建出的新的 image
docker images
# 使用 docker history image_id 分别查看 centos 和 comit 出的 image 的分层信息
docker history image_id
# 可发现,centos 和 commit 出的 image 共享了很多层
```

==**docker build**== -- build an image form Dockerfile
- 通过一个Dockerfile 文件和 Docker build 命令构建 image

```shell
# vim Dockerfile
FROM centos
RUN yum install -y vim

# 执行 Docker build ,构建 image
docker build -t image_name .
# image 是只读的,所以会先构建一个临时的 container,然后在构建 image
```

#### 4.4 Dockerfile 语法以及最佳实践
- ==**FROM**==
    - 指定要 build 的 image 的 base image.就是要在哪个 base image 之上构建 image
    - 当要制作 base image 时,使用 ==*FROM scratch*==
    - 尽量使用官方 image 作为 base image
- ==**LABEL**==
    - 指定 image 的 metadata
- ==**RUN**==
    - 构建 image 过程中要安装,或者升级软件,或者执行一些命令
    - ==*每一个 RUN,就会多一层,所以尽量少分层,要指定多个安装或者升级的时候,使用 && 分开,如果太长,需要换行,就使用 \ 进行换行*==
- ==**WORKDID**==
    - 设定当前工作目录
    - 尽量使用绝对路径,不要使用相对路径
    
        ```properties
        WORDEIR /test : 如果没有会自动创建 test 目录
        WORKDIR demo
        RUN pwd   : 执行上边两条命令后,次数输出结果为 : /test/demo
        ```
        
    - ==**ADD 和 COPY**==
        - 都是将==*本地文件*==添加到 build 的 image 里边去.
        - ADD 不管可以添加文件到 image 中,还可以解压缩
        - 大部分情况,CPOY 优先于 ADD 使用.
        - ==**如果想添加远程文件,使用 curl 或者 wget**==

        ```properties
        ADD test.tar.gz /  : 将test.tar.gz 添加到根目录,并解压
        #
        WORKDIR /root
        ADD hello test/     : 指定工作目录为/root,并将 hello 文件添加到 test/里边,所以最后是 /root/test/hello
        
        ```

    - ==**ENV**==
        - 设置一些常量,增加可维护性
        - 尽量使用 ENV 来增加可维护性
        
        ```properties
         # 设置 MySQL 版本常量
        ENV MYSQL_VERSION 5.6 
        
        # 使用设置的常量
        RUN apt-get install -y mysql-server = "${MYSQL_VERSION}" \
            && rm -rf /var/lib/apt/lists/*
        ```
        
    - CMD 和 
    

## 5. docker container
- 通过 image 创建 container
- 在 image layer 之上 ==**增加了一层**==,建立一个 container layer(container layer 是可读可写的)
- 比如面向对象 : image 就是类, container 就是实例出来对象.
- ==**image 负责 APP 的存储和分发,container 负责运行 APP**==

### 5.1 创建 container
- 直接通过 docker run image 就可以创建一个 container
- ==*交互式运行*== : docker run -it image



## 4. Docker 的网络
## 5. Docker 的持久化存储和数据共享
## 6. Docker Compose 多容器部署
## 7. 容器编排工具 : Docker Swarm
## 8. Docker Cloud 和 Docker 企业版
## 9. 容器编排 Kubernetes
## 10. 容器的运维和监控
## 11. Docker + DevOps 实战--过程和工具

## 用户手册
### 1. 命令
- docker 命令 : ==*docker +命令关键字command +一系列参数*==
- 如果不知道某个 command的作用,可以使用 ==* docker command --help*== 查看帮助文档

```properties
<!--全局命令-->
docker version

<!--关于 image -->
docker image ls / docker images : 查看已安装的 image
docker rmi images_id : 删除指定的镜像(前提是没有容器使用这个镜像,停止容器,删除容器)
docker commit -m 'xxx' image_id new_image_name : 对 image_id 的 image 进行修改之后,生成一个信息的 image,名称为 image_name, 就是基于某个改变的 container 创建出一个新的 image

docker search xxx : 查找 image
docker pull xxx : 下载镜像

docker build -t image_name : 构建 image

docker history image_id : 查看 image 的 分层

<!-- 关于 container -->
docker run : 创建和启动 container
    例如 : docker run -d -p 80:80 httpd
    指定名称 : docker run --name 指定名称 -d -p 80:80 httpd
    Docker 会看本地有没有这个 container,如果没有,就下载最新版
    
docker container ls / docker ps: 查看运行的容器
    比如 ID 等,ID 默认 128 位,此命令查询出的是 16 位的简略 ID,要查看完整的加上 --no-trunc

docker ps : 查看正在运行的容器
docker ps -a : 查看所有的容器  
docker ps -aq : 列出所有的容器的 ID  
docker ps -a | grep container_id : 查看容器状态   
    Up什么什么的 : 说明运行中. 
    Exited : 说明停止
docker ps -f "status=exited"  : 列出所有停止的 container
docker ps -f "status=exited" -q : 列出所有停止的 container 的 ID

docker stop container_id : 停止容器
docker start container_id : 启动容器
    也可以使用在 docker run 时设置的名称代替 ID
docker rm container_id : 删除 container
docker rm $(docker ps -aq) : 删除所有的 container

docker inspect 容器 ID : 查看容器信息
    可使用 docker run 指定的名称代替 ID
    查看指定信息 : docker inspect -f {{.NetworkSettings.IPAddress}} 名称/ID

      
docker logs 名称/ID : 查看日志

docker cp custom_file container_id://path : 替换容器中的文件,但是此时重启的话,依然是改变前的内容. 使用 commit 可以保存修改(其实就是新创建了一个 image)
```

<span id="dockerfile"/>
### 2. dockerfile 语法

```properties
FROM : 指定 base image (基础 image)
RUN : 构建过程中
ADD : 添加文件,比 copy 强大,可以添加远程的文件
COPY : 向 image 内 copy 文件
CMD : 执行命令
EXPOSE : 要暴露的端口
WORKDIR : 指定运行命令的路径
MAINTAINER : 维护者
ENV : 设定环境变量
ENTRYPOINT : 指定入口
USER : 指定执行该容器的用户
```

### 3. docker 使用技巧
#### 3.1 虚拟机中使用Docker
- 不使用 sudo 前缀也可以使用 Docker 命令
    
```properties
第一步 : sudo groupadd docker
第二步 : sudo gpasswd -a vagrant docker
第三步 : sudo service docker restart : 重启 Docker 服务
第四步 : 重新登录 shell (重新登录 Linux 主机)
```


