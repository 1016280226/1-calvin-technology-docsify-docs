# 笔记二 Docker 安装

## 1. 基于 Linux 环境 CentOS 7 上安装 Docker

### **Docker** 是一个**开源的商业产品**，有两个版本：
> - **社区版**（Community Edition，缩写为 CE）
> - **企业版**（Enterprise Edition，缩写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到



## 2. Docker 安装步骤

## 1. 前提条件

> ### 操作系统要求
>
> 要安装Docker Engine，您需要一个CentOS 7的维护版本。不支持或未测试存档版本。
>
> 该`centos-extras`库必须启用。默认情况下，此存储库是启用的，但是如果已禁用它，则需要 [重新启用它](https://wiki.centos.org/AdditionalResources/Repositories)。
>
> `overlay2`建议使用存储驱动程序。

### 1.1 卸载旧版本

较旧的Docker版本称为`docker`或`docker-engine`。如果已安装这些程序，请卸载它们以及相关的依赖项。

```docker
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

如果`yum`报告未安装这些软件包，则可以。

的内容（`/var/lib/docker/`包括图像，容器，卷和网络）被保留。现在将Docker Engine软件包称为`docker-ce`。

## 2. 安装方法

您可以根据需要以不同的方式安装Docker Engine：

- 大多数用户会 [设置Docker的存储库](https://docs.docker.com/engine/install/centos/#install-using-the-repository)并从中进行安装，以简化安装和升级任务。这是推荐的方法。
- 一些用户下载RPM软件包并 [手动安装它，](https://docs.docker.com/engine/install/centos/#install-from-a-package)并完全手动管理升级。这在诸如在无法访问互联网的空白系统上安装Docker的情况下非常有用。
- 在测试和开发环境中，一些用户选择使用自动 [便利脚本](https://docs.docker.com/engine/install/centos/#install-using-the-convenience-script)来安装Docker。



### 2.1 使用存储库安装

在新主机上首次安装Docker Engine之前，需要设置Docker存储库。之后，您可以从存储库安装和更新Docker。

#### 设置存储库

安装`yum-utils`软件包（提供`yum-config-manager` 实用程序）并设置**稳定的**存储库。

```bash
# 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的。
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2   

# 设置yum源。
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```



### 2.2 安装DOCKER引擎

1. 安装*最新版本*的Docker Engine和容器，或转到下一步以安装特定版本：

   ```bash
   $ sudo yum install docker-ce docker-ce-cli containerd.io
   ```

   如果提示您接受GPG密钥，请验证指纹是否匹配 `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`，如果是，则接受它。

   > 有多个Docker存储库吗？
   >
   > 如果您启用了多个Docker存储库，则在未在`yum install`or `yum update`命令中指定版本的情况下进行安装或更新将始终安装可能的最高版本，这可能不适合您的稳定性需求。

   Docker已安装但尚未启动。`docker`创建该组，但没有用户添加到该组。

2. 要安装*特定版本*的Docker Engine，请在存储库中列出可用版本，然后选择并安装：

   一个。列出并排序您存储库中可用的版本。此示例按版本号（从高到低）对结果进行排序，并被截断：

   ```
   $ yum list docker-ce --showduplicates | sort -r
   
   docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
   docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
   docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
   docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
   ```

   返回的列表取决于启用了哪些存储库，并且特定于您的CentOS版本（`.el7`在本示例中以后缀表示）。

   b。通过其完全合格的软件包名称安装特定版本，该软件包名称是软件包名称（`docker-ce`）加上版本字符串（第二列），从第一个冒号（`:`）到第一个连字符，以连字符（`-`）分隔。例如，`docker-ce-18.09.1`。

   ```bash
   $ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
   ```

   Docker已安装但尚未启动。`docker`创建该组，但没有用户添加到该组。

3. 启动并加入开机启动。

   ```bash
   $ sudo systemctl start docker && systemctl enable docker   
   ```

4. 验证安装是否成功(有client和service两部分表示docker安装启动都成功了)。

   ``` bash
   $ docker version   
   ```



### 2.3 配置镜像加速器

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


