# 笔记三 Docker 常用的命令

### 1. 查看镜像文件。

   ```bash
   docker images
   ```

### 2. 删除镜像文件（根据镜像文件IMAGE ID，来删除镜像文件）。

   > -f  强制

   ```bash
   docker rmi -f IMAGE_ID
   ```

### 3. 查看运行中的容器。

   > -a 所有

   ```bash
   docker ps 
   ```

### 4. 查看容器中的的信息

   ```bash
   docker inspect CONTAINER_ID
   ```

### 5. 删除容器

   ```bash
   docker rm -f
   ```

### 6. 查看 安装软件 版本

   ```bash
   docker search install_softe_name 
   ```

### 7. 下载软件镜像

   > : 后指定版本号

   ```bash
   docker pull install_softe_name:version
   ```

### 8. 启动容器

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

### 9. 进入容器目录

   ```bash
   docker container exec -it CONTAINER_ID /bin/bash 
   ```
   
### 10. 卸载老版本docker

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
### 11. 卸载Docker

    ```bash
    # 卸载Docker软件包
    yum remove docker-ce
    # 主机上的映像，容器，卷或自定义配置文件不会自动删除。要删除所有图像，容器和卷
    rm -rf /var/lib/docker
    ```
### 12. 可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)
    ```bash
    docker system prune
    ```
    > *注意: -a命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了…所以使用之前一定要想清楚.*
    
### 13. 容器导出
    ```bash
    docker export CONTAINER_NAME > /存储路径/文件名称.tar
    ```
  