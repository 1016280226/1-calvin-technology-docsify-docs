# 笔记二 Docker 安装

## 基于 Linux 环境上安装Docker

### 1. **Docker** 是一个**开源的商业产品**，有两个版本：
> - **社区版**（Community Edition，缩写为 CE）
> - **企业版**（Enterprise Edition，缩写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到

### 2. **Docker** 要求 CentOS 系统的内核版本在 **3.10以上** ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

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
   
### 3. 配置镜像加速器

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


