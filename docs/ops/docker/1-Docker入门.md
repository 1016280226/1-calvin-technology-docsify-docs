# 笔记一 Docker

## 1. Docker 简介

![images](./../../../statics/images/docker/logo.jpg)

- **Docker** 是基于**Go**语言实现的云开源项目，诞生于2013年初，最初发起者是**dotClouw**公司。
- **Docke** 自开源后受到广泛的关注和讨论，目前已有多个相关项目，逐断形成了围**Docker**的生态体系。**dotCloud** 公司后来也改名为**Docker Ine**。
  
> - Docker官方网址 https://docs.docker.com 
> - Docker中文网址 http://www.docker.org.cn

&nbsp;

## 2. 为什么要使用Docker

- **Docker** 作为一种**轻量级的虚拟化**方式，**Docker**在运行应用上跟**传统的虚拟机方式相比**具有显著优势。
  - **节省开支**：虚拟化容器体积小，占用率低。
  - **简化程序**：启动速度快, 启动和停止可以在**秒级实现**，这相比传统的虚拟机方式要快得多。
  - **避免选择恐惧症**：系统隔离级别高，可以作为一台虚拟电脑系统。
  
- **Docker** 与  **开发人员** 和 **运维人员** 提供了好处有：
  - 运维人员：方便安装软件，方便部署（像使用yum一样）,  应为隔离级别高，所以一个容器一个应用软件，方便管理和维护。
  - 开发人员：使应用程序的生命周期从开发、测试、打包、发布、管理减少了手动操作，提高了工作效率，同时还对系统应用管理，减少了虚拟机到处存放文件的问题。
  
- 使用 **Docker** 容器引擎，让我们技术人员，为**云计算时代**做好相应的准备。

&nbsp;

## 3. Docker 架构

![images](./../../../statics\images\docker\docker-01.jpg)

重点主要分为3层
a. **Docker daemon**（ Docker守护进程）

> Docker daemon是一个运行在**宿主机**（ **DOCKER-HOST**）的后台进程。
>
> 作用：可通过 **Docker客户端与之通信**。

 

b. **Client**（ Docker客户端）

> **Docker客户端是 Docker的用户界面**，它可以接受用户命令和配置标识，并与 Docker daemon通信。
>
> 图中， docker build等都是 Docker的相关命令。

 

c. **Images**（ Docker镜像）

> Docker镜像是一个只读模板，它包含创建 Docker容器的说明。
>
> 它和系统安装光盘有点像，使用系统安装光盘可以安装系统，同理，使用Docker镜像可以运行 Docker镜像中的程序。

 

d. **Container**（容器）

> 容器是镜像的**可运行实例**。
>
> 镜像和容器的关系有点类似于面向对象中，类和对象的关系。
>
> 可通过 Docker API或者 CLI命令来启停、移动、删除容器。



f. 运行原理：

> **容器→镜像→仓库**



## 4.Linux 环境上安装Docker

### 4.1 Docker 安装

1. **Docker** 是一个**开源的商业产品**，有两个版本：

> - **社区版**（Community Edition，缩写为 CE）
>
> - **企业版**（Enterprise Edition，缩写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到



2. **Docker** 要求 CentOS 系统的内核版本在 **3.10以上** ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

   a. 通过 **uname -r** 命令查看你当前的内核版本

   ```bash
   uname -r  
   ```

   b. 使用 root 权限登录 Centos。确保 yum 包更新到最新。

   ```bash
    yum -y update   
   ```

   c. 卸载旧版本(如果安装过旧版本的话)。

   ```bash
    yum remove docker docker-common   docker-selinux docker-engine   
   ```

   d. 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的。

   ```bash
    yum install -y yum-utils   device-mapper-persistent-data lvm2   
   ```

   e. 设置yum源。

   ```bash
    yum-config-manager --add-repo   https://download.docker.com/linux/centos/docker-ce.repo   
   ```

   f. 可以查看所有仓库中所有docker版本，并选择特定版本安装。
   ```bash
   yum list docker-ce --showduplicates | sort -r   
   ```

   g.  安装docker。

   ```bash
   sudo yum install -y docker-ce     #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版18.03.1   
   ```

   h.  启动并加入开机启动。

   ```bash
   systemctl start docker   
   systemctl enable docker   
   ```

   i. 验证安装是否成功(有client和service两部分表示docker安装启动都成功了)。

   ```bash
   docker version   
   ```



### 4.2  配置镜像加速器

> [阿里云镜像加速器地址](https://cr.console.aliyun.com/cn-hangzhou/mirrors)

针对Docker客户端版本大于 1.10.0 的用户

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://m0p90m3l.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 5.Docker 常用的命令

1. 查看镜像文件。

   ```bash
   docker images
   ```

2. 删除镜像文件（根据镜像文件IMAGE ID，来删除镜像文件）。

   > -f  强制

   ```bash
   docker rmi -f IMAGE_ID
   ```

3. 查看运行中的容器。

   > -a 所有

   ```bash
   docker ps 
   ```

4. 查看容器中的的信息

   ```bash
   docker inspect CONTAINER_ID
   ```

5. 删除容器

   ```bash
   docker rm -f
   ```

6. 查看 安装软件 版本

   ```bash
   docker search install_softe_name 
   ```

7. 下载软件镜像

   > : 后指定版本号

   ```bash
   docker pull install_softe_name:version
   ```

8. 启动容器

   - 方法一：安装并且启动运行

     > -d 后台运行
     >
     > -p 宿主机端口 : 容器端口  #开放容器端口到宿主机端口

     ```bash
     docker run -d -p 81:80 install_softe_name
     ```

     > *提醒 ：使用 docker run命令创建容器时，会先检查本地是否存在指定镜像。如果本地不存在该名称的镜像， Docker就会自动从 Docker Hub下载镜像并启动一个 Docker容器。*

   - 方法二：启动运行

     ```bash
     docker start container_name #容器名称或容器ID
     ```

9. 进入容器目录

   ```bash
   docker container exec -it CONTAINER_ID /bin/bash 
   ```
   
10. 卸载老版本docker

    ```bash
    yum remove docker \
              docker-client \
              docker-client-latest \
              docker-common \
              docker-latest \
              docker-latest-logrotate \
              docker-logrotate \
              docker-engine
    ```
11. 卸载Docker

    ```bash
    # 卸载Docker软件包
    yum remove docker-ce
    # 主机上的映像，容器，卷或自定义配置文件不会自动删除。要删除所有图像，容器和卷
    rm -rf /var/lib/docker
    ```
12. 可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)
    ```bash
    docker system prune
    ```
    > *注意: -a命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了…所以使用之前一定要想清楚.*
    
13. 容器导出
    ```bash
    docker export CONTAINER_NAME > /存储路径/文件名称.tar
    ```


## 6. 基于Docker 部署SpringBoot 项目

1. 将 jar 包上传到Linux 服务器 /usr/local/docker/app 目录下，在 jar 包所在目录创建名为Dockerfile 的文件。

2. 在Dockerfile 中添加以下内容：

   ```shell
   ###指定java8环境镜像
   FROM java:8
   ###复制文件到容器app-springboot
   ADD docker-spring-boot-swagger-1.0.jar /spring-boot-swagger-1.0.jar
   ###声明启动端口号
   EXPOSE 8080
   ###配置容器启动后执行的命令
   ENTRYPOINT ["java","-jar","/spring-boot-swagger-1.0.jar"]
   ```

3. 使用docker build 命令构建镜像。

   > 格式：docker build -t 镜像名称:标签 Dockerfile的相对位置

   ```bash
   docker build -t docker-spring-boot-swagger-1.0 ./
   ```

4. 启动 docker-spring-boot-swagger-1.0.jar 镜像文件

   ```bash
   docker run -p 8080:8080 docker-spring-boot-swagger-1.0
   ```

5. 在防火墙iptables 配置文件中或者使用命令，添加允许访问的端口号。

   ```bash
   # 关闭firewalld
   systemctl stop firewalld   
   systemctl mask firewalld
   
   #开放80端口(HTTPS)
   iptables -A INPUT -p tcp --dport 80 -j ACCEPT
   
   #保存上述规则
   service iptables save
    
   #开启服务
   systemctl restart iptables.service
   ```
   
## 7. Docker 磁盘空间占满,迁移/var/lib/docker目录

> **问题：** *no space left on device*

1. 查看磁盘使用情况

    ```
    df -hl
    ```
   ![image](B2AAA5AB01D24EA3BEC3106A3795C5E2)
   
   
   
2. 查看docker 的存储根目录

    ```bash
    docker info | grep -i "docker root dir"
    ```
   ![image](6EAF9B2829094816BD3BD3143A34C203)
   
   
   
3. 查看目录剩余空间
    ```bash
    df -hl /docker-root
    ```
    ![image](9F33B7E2E0304F4EA37D152517DFC3B9)
    
4. 停止Docker服务

    ```bash
    systemctl stop docker
    ```

5. 创建一个存储空间较大的docker目录

    ```bash
    # 找一个大的磁盘
    df -h 
    # 创建一个大磁盘docker目录
    mkdir -p /home/docker/lib
    ```
    
6. 迁移到新目录下面的文件

    ```bash
    rsync -avz /docker-root /home/docker/lib
    ```
    
    > *ps： 未安装rsync， 输入 yum install rsync -y*
    
7. 配置 /etc/systemd/system/docker.service.d/devicemapper.conf

    ```bash
    # 如果存在该目录，创建该目录
    mkdir -p /etc/systemd/system/docker.service.d/
    
    # 修改devicemapper.conf
    vi /etc/systemd/system/docker.service.d/devicemapper.conf
    
    # 同步的时候把父文件夹一并同步过来，实际上的目录应在 /home/docker/lib/docker 
    
    
    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd  --graph=/home/docker/lib/docker
    ```

8. 重新加载 docker

    ```bash
    systemctl daemon-reload
    systemctl restart docker
    systemctl enable  docker
    ```

9. 查看docker信息

    ```bash
    docker info 
    ```
    ![image](8D21449811F341CB91FC2DC21C79C047)

10. 查看挂载大目录后信息

    ```bash
    df -h /home/docker/lib
    ```
    ![image](C482089EA52D499E9A490B19128AAA92)
    


## 8. Dockerfile 构建镜像

### 8.1 Dockerfile 简介

1. **Dockerfile** 是由一系列命令构成的脚本，这些命令是基于镜像，最终创建出一个新的镜像。(应用于基础镜像)

2. 对于**DevOps**来说 :

    a. **开发人员**: 提供开发环境。

    > *例如: 搭建jdk环境变量、RabbitMQ 等消息中间件*

    b. **测试人员**: 通过Docker文件直接进行工作。
    
    > *例如：使用Dokcer搭建jemter测试直接进行工作，或编写Dockerfiler脚本直接进行测试。*
    
    c. **运维人员**: 在部署时，可以实现应用的无缝移植。
    > *例如：将一个Docker中打包好的文件，编写Dokcerfile可以实现跨主机，实现无缝的移植。*
    
### 8.2 Dockerfile 常用的命令


| 命令                                   | 作用                                                               |
| -------------------------------------- | ------------------------------------------------------------------ |
| **FROM** image_name:tag                | 定义使用哪个基础镜像启动构建流程                                   |
| **MAINTAINER** user_name               | 声明镜像的创建者                                                   |
| **ENV** key value                      | 设置环境变量                                                       |
| **RUN** command                        | 运行Docker容器镜像                                                 |
| **ADD** source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会复制后自动解压 |
| **COPY** source_dir/file dest_dir/file | 和ADD相似,但是如果有压缩文件不能解压                               |
| **WORKDIR** path_dir                   | 设置工作目录                                                       |


### 8.3 Dockerfile 脚本创建镜像
#### 8.3.1 案例-构建Jdk8 Dockerfile
**步骤：**
```
# 1.创建一个目录
mkdir -p /home/docker/data/dockerfile/jdk8

# 2.将下载压缩包放入该目录
mv jdk1.8.0_171.tar.gz /home/docker/data/dockerfile/jdk8

# 3.进入该目录下创建Dockerfile 文件
cd /home/docker/data/dockerfile/jdk8
touch Dockerfile

# 4.编辑 Dockerfile
vi Dockerfile

FROM centos:7                               # 镜像基础信息为 centos7 操作系统        
MAINTAINER calvin                           # 创建镜像的创建者为 Calvin
WORKDIR /usr                                # 设置工作目录为 /user
RUN mkdir /usr/local/java                   # 创建java目录
ADD jdk1.8.0_171.tar.gz /usr/local/java/    # 添加压缩包到该目录，会自动解压

                                            # 配置JAVA环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV ClASSPATH $JAVA_HOME/bin/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

# 5.运行执行构建 
# -t:指定当前dokcerfile文件的名称
docker build -t='jdk1.8'

# 6.可以在docker镜像中看到新建的镜像
docker images

```

## 9. Docker 私有仓库

> *场景: 企业内部所使用私有仓库 (相当于内部的git或svn,但是这里是个镜像仓库)* 

**操作步骤:**

### 9.1 搭建Docker 私有仓库搭建与配置

#### 9.1.1 拉取私有仓库镜像
```
docker pull registry
```
#### 9.1.2 启动私有仓库镜像
```
docker run -d --name=registry -p 5000:5000 registry
```
#### 9.2.3 访问私有仓库

>- 打开chmore浏览器 http://localhost:5000/v2/_catalog 可以看到 
>- *{"repositories":[]}* 表示私有仓库搭建成功，并且里面没有镜像。

![image](877A902B32A641EBB2F528BA19B2D3C2)

#### 9.2.4 给dokcer 授权私有仓库地址

(1) 修改daemon.json
```
vi /etc/docker/daemon.json
```
(2) 添加私有仓库地址到docker注册表中去。

> *作用是将本地的镜像上传到私有仓库。*
```
{
  "registry-mirrors": ["https://m0p90m3l.mirror.aliyuncs.com"], # 注册镜像: 阿里云仓库获取镜像
  "insecure-registries":["192.168.2.98:5000"]                   # 不安全注册表: 从本地私有仓库
}
```

(3) 重启docker服务，使其生效
```
systemctl restart docker
docker    restart registry
```
#### 9.2.5 将镜像上传到私有仓库
(1) 标记此镜像为私有仓库镜像
```
# tag 指定镜像名 私服ip地址
docker tag jdk1.8 192.168.2.98:5000/jdk1.8 
```
(2) 上传标记的镜像
```
docker push 192.168.2.98:5000/jdk1.8
```

#### 9.2.6 访问私有仓库

打开chmore浏览器 http://localhost:5000/v2/_catalog 可以看到 










​    


​    
