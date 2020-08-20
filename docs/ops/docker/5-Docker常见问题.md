# 笔记五 Docker 常见问题

## 问题 1：在VM中安装CentOS，强制关机，导致Docker无法启动？

### <font color=red>报错问题:  Job for docker.service canceled.</font>

``` bash
$ systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since 三 2020-08-12 14:55:02 CST; 3s ago
     Docs: https://docs.docker.com
  Process: 36258 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock (code=exited, status=1/FAILURE)
 Main PID: 36258 (code=exited, status=1/FAILURE)

8月 12 14:55:02 docker-zentao dockerd[36258]: time="2020-08-12T14:55:02.213645133+08:00" level=info msg="ClientConn switching balance...le=grpc
8月 12 14:55:02 docker-zentao dockerd[36258]: time="2020-08-12T14:55:02.218196231+08:00" level=info msg="parsed scheme: \"unix\"" module=grpc
8月 12 14:55:02 docker-zentao dockerd[36258]: time="2020-08-12T14:55:02.218264532+08:00" level=info msg="scheme \"unix\" not register...le=grpc
8月 12 14:55:02 docker-zentao dockerd[36258]: time="2020-08-12T14:55:02.218303257+08:00" level=info msg="ccResolverWrapper: sending u...le=grpc
8月 12 14:55:02 docker-zentao dockerd[36258]: time="2020-08-12T14:55:02.218329022+08:00" level=info msg="ClientConn switching balance...le=grpc
8月 12 14:55:02 docker-zentao dockerd[36258]: failed to start daemon: couldn't create plugin manager: failed to dial "/run/containerd...ailable
8月 12 14:55:02 docker-zentao systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
8月 12 14:55:02 docker-zentao systemd[1]: Stopped Docker Application Container Engine.
8月 12 14:55:02 docker-zentao systemd[1]: Unit docker.service entered failed state.
8月 12 14:55:02 docker-zentao systemd[1]: docker.service failed.
Hint: Some lines were ellipsized, use -l to show in full.

```

### 解决步骤：

#### 1. 删除运行容器

``` bash
$ rm -rf /var/lib/containerd/
```

#### 2. 删除docker 容器下的内容

```bash
$ rm -rf /var/lib/docker
```

#### 3. 重启 Docker 服务

```bash
$ systemctl restart docker
```

