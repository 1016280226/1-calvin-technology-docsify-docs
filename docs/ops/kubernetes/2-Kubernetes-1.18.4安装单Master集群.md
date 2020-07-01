# 笔记二 Kubernetes 1.18.x 安装单Master集群 

## 一、配置要求

对于 Kubernetes 初学者，在搭建K8S集群时，推荐在阿里云或腾讯云采购如下配置：（您也可以使用自己的虚拟机、私有云等您最容易获得的 Linux 环境）

### 1. 硬件配置要求：

- 至少3台，2核4G的服务器。
- CentOS 7.6 以上。

### 2. 安装软件版本要求：

- Kubernetes v1.18.x
  - calico 3.13.1
  - nginx-ingress 1.5.5
- Docker 19.03.8

### 3. 网络配置要求:

- 集群中所有机器之间网络互通。
- 可以访问外网，拉取镜像。
- 禁止swap分区

### 4. 节点配置

| 主机名             | IP            | 角色   | 系统                     | CPU  | 内存 | 磁盘  |
| ------------------ | ------------- | ------ | ------------------------ | ---- | ---- | ----- |
| kubernetes-master  | 192.168.2.100 | Master | CentOS Linux release 7.8 | 2核  | 4G   | 40GiB |
| kubernetes-node-01 | 192.168.2.110 | Node   | CentOS Linux release 7.8 | 2核  | 4G   | 40GiB |
| kubernetes-node-02 | 192.168.2.120 | Node   | CentOS Linux release 7.8 | 2核  | 4G   | 40GiB |

> 注意：由于演示问题，这里只有2个节点，如果服务器配置足够可以添加多个节点。

## 二、Kubernetes 1.18.x 搭建单Master集群 

### 1. 安装前准备工作

#### 1.1 检查 CentOS 系统配置。

```bash
# 查看系统是否为7.6以上版本 - 在master节点和node节点都要执行
$ cat /etc/redhat-release

--------------------------------- 输出以下信息 --------------------------------------------------
CentOS Linux release 7.8.2003 (Core)
-----------------------------------------------------------------------------------------------
```

#### 1.2 查看 CPU 内核数，不能低于2核。

```bash
$ lscpu

--------------------------------- 输出以下信息 --------------------------------------------------
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                2  # CPU 内核数量不能低于 2
......
-----------------------------------------------------------------------------------------------
```

#### 1.3 查看主机名称，不能使用 locahost 作为主机或节点名称。

```bash
# 在master节点和node节点都要执行
$ hostname
---------------------------------在 master 输出以下信息 -----------------------------------------
kubernetes-master
-----------------------------------------------------------------------------------------------

---------------------------------在 node 各输出以下信息 -----------------------------------------
kubernetes-node-01
kubernetes-node-02
-----------------------------------------------------------------------------------------------
```

#### 1.4 如果前期没有设置好主机名称，需要修改 hostname。

- 在 master 节点执行

  ```bash
  # 修改 hostname - master 节点执行
  $ hostnamectl set-hostname kubernetes-master 
  
  # 查看修改结果 - master 节点和 worker 节点
  $ hostnamectl status
  
  # 设置 hostname 解析 - master 节点
  $ echo "127.0.0.1   $(hostname)" >> /etc/hosts
  
  # 重启才生效
  reboot
  ```

- 在 node 节点执行

  ```bash
  # 修改 hostname - node 节点执行
  $ hostnamectl set-hostname kubernetes-node-01  
  $ hostnamectl set-hostname kubernetes-node-02
  
  # 查看修改结果 - master 节点和 worker 节点
  $ hostnamectl status
  
  # 设置 hostname 解析 - master 节点和 worker 节点
  $ echo "127.0.0.1   $(hostname)" >> /etc/hosts
  
  # 重启才生效 
  reboot
  ```

#### 1.5 检查网络状态

- 在所有master节点 和 worker节点执行命令

```bash
# 查看IP 网关
$ ip route show
--------------------------------- 输出以下信息 --------------------------------------------------
default via 192.168.2.1 dev em1 proto static metric 100 
------------------------------------------------------------------------------------------------

# 查看IP地址和网卡名称
$ ip address
--------------------------------- master 输出以下信息 -------------------------------------------
# em1 就是网卡名称
em1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 80:18:44:e6:07:d8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.100/24 brd 192.168.2.255 scope global noprefixroute em1
       valid_lft forever preferred_lft forever
    inet6 fe80::8218:44ff:fee6:7d8/64 scope link 
       valid_lft forever preferred_lft forever
------------------------------------------------------------------------------------------------
       
--------------------------------- node-01 输出以下信息 ------------------------------------------
# em1 就是网卡名称
em1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether ac:1f:6b:e1:42:18 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.110/24 brd 192.168.2.255 scope global noprefixroute em1
       valid_lft forever preferred_lft forever
    inet6 fe80::c2f:5705:a8ee:1436/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
------------------------------------------------------------------------------------------------

--------------------------------- node-02 输出以下信息 ------------------------------------------
# em1 就是网卡名称
em1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 56:1f:5b:c1:32:53 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.110/24 brd 192.168.2.255 scope global noprefixroute em1
       valid_lft forever preferred_lft forever
    inet6 fe80::c2f:5705:a8ee:1436/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
------------------------------------------------------------------------------------------------
```

> 注意：网卡名称必须统一，不然后期会出现 calico 网络问题，如 master calico 节点 和 node calico 网络冲突问题。

#### 1.6 如果网卡名称不一样，修改网卡名称。

```bash
# 例如：ifcfg-ens33 是你的网络配置也是你的网卡名称, 修改成 ifcfg-em1
$ cp /etc/sysconfig/network-scripts/ifcfg-ens33  /etc/sysconfig/network-scripts/ifcfg-em1

# 修改拷贝后的网络配置信息
$ vim /etc/sysconfig/network-scripts/ifcfg-em1

--------------------------------- 修改以下2项信息 -----------------------------------------------
NAME=em1
DEVICE=em1
------------------------------------------------------------------------------------------------

# 重点在这里，编辑网卡规则文件
$ vi /usr/lib/udev/rules.d/60-net.rules

--------------------------------- 修改以下2项信息 ---------------------------------------------------------------------------------------
# ACTION=="add", SUBSYSTEM=="net", DRIVERS=="?*", ATTR{type}=="1", PROGRAM="/lib/udev/rename_device", RESULT=="?*", NAME="$result"

# ATTR{address}="56:1f:5b:c1:32:53" 添加物理地址 
# NAME="em1" 添加网卡名称
ACTION=="add", SUBSYSTEM=="net", DRIVERS=="?*", ATTR{type}=="1", ATTR{address}="56:1f:5b:c1:32:53", NAME="em1"
----------------------------------------------------------------------------------------------------------------------------------------

# 重启才生效 
$ reboot
```

####  1.7 关闭防火墙、SeLinux、swap(临时交换空间)

```bash
# 关闭防火墙 - master 节点和 worker 节点
$ systemctl stop firewalld
$ systemctl disable firewalld

# 关闭 SeLinux - master 节点和 worker 节点
$ setenforce 0
$ sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

# 关闭 swap  - master 节点和 worker 节点
$ swapoff -a
$ yes | cp /etc/fstab /etc/fstab_bak
$ cat /etc/fstab_bak |grep -v swap > /etc/fstab
```

#### 1.8 修改系统配置

```bash
# 在 master 节点和 worker 节点都要执行
$ vim /etc/sysctl.conf

------------------------------------- 添加以下信息 ----------------------------------------------
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
------------------------------------------------------------------------------------------------

# 添加bridge-nf-call-ip6tables出现No such file or directory, 执行以下命令
$ modprobe br_netfilter

# 执行使其生效
$ sysctl -p
```



### 2. 安装指定版本 Docker

#### 2.1 卸在旧版本 Docker

```bash
# 卸载旧版本- master节点 和 worker节点都要执行
$ sudo yum remove -y docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine
```

> 提醒：如果安装最新版本卸载不了，可以使用 rpm 查看需要卸载版本
>
> - rpm -qa | grep docker 
> -  yum remove docker-ce-版本号 docker-ce-cli-版本号 containerd.io

### 2.2 安装 Docker 19.03.8

- 设置 yum repository - master 节点和 worker 节点执行

  ```bash
  $ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  sudo yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo
  ```

- 安装并启动 docker - master 节点和 worker 节点执行

  ```bash
  $ sudo yum install -y docker-ce-19.03.8 docker-ce-cli-19.03.8 containerd.io
  $ sudo systemctl enable docker
  $ sudo systemctl start docker
  ```

- 安装完后，查看Docker 版本 - master 节点和 worker 节点

  ```bash
  $ docker version
  
  --------------------------------- 输出以下信息, 代表安装成功-----------------------------------
  # 版本最好一定要对应上安装k8s不懂版本需求
  Client: Docker Engine - Community
   Version:           19.03.8  
   API version:       1.40
   Go version:        go1.12.17
   Git commit:        afacb8b
   Built:             Wed Mar 11 01:27:04 2020
   OS/Arch:           linux/amd64
   Experimental:      false
  
  # 版本最好一定要对应上安装k8s不懂版本需求
  Server: Docker Engine - Community
   Engine:
    Version:          19.03.8 
    API version:      1.40 (minimum version 1.12)
    Go version:       go1.12.17
    Git commit:       afacb8b
    Built:            Wed Mar 11 01:25:42 2020
    OS/Arch:          linux/amd64
    Experimental:     false
   containerd:
    Version:          1.2.13
    GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
   runc:
    Version:          1.0.0-rc10
    GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
   docker-init:
    Version:          0.18.0
    GitCommit:        fec3683
  -------------------------------------------------------------------------------------------
  ```

- 修改docker Cgroup Driver为systemd - master 节点和 worker 节点，<font color=red> (如果不修改，在添加 worker 节点时可能会碰到如下错误) </font>

  ```bash
  $ vim /etc/docker/daemon.json
  
  # 1. registry-mirrors 添加镜像仓库为：aliyun 镜像仓库
  # 2. 修改docker Cgroup Driver为systemd 
  ----------------------------- 添加以下内容 --------------------------------------------------
  {
    "registry-mirrors": ["https://m0p90m3l.mirror.aliyuncs.com"],
    "exec-opts": ["native.cgroupdriver=systemd"]
  }
  ---------------------------------------------------------------------------------------------
  # 加载系统配置
  $ systemctl daemon-reload
  
  # 重启 docker 服务
  $ systemctl restart docker
  
  # 查看 systmed 是否生效
  $ docker info
  
  --------------------------------- 输出以下信息-----------------------------------------------
  Client:
   Debug Mode: false
  
  Server:
   Containers: 28
    Running: 12
    Paused: 0
    Stopped: 16
   Images: 24
   Server Version: 19.03.11
   Storage Driver: overlay2
    Backing Filesystem: xfs
    Supports d_type: true
    Native Overlay Diff: true
   Logging Driver: json-file
   Cgroup Driver: systemd  # 代表修改成功
   ......
  -------------------------------------------------------------------------------------------
  ```



### 4. 安装 kubelet、kubeadm、kubectl 版本为1.18.4

#### 1. 配置K8S的yum源

```bash
# 在 master 节点和 worker 节点都要执行
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

#### 2. 安装kubelet、kubeadm、kubectl 版本为 1.18.4

```bash
$ yum install -y kubelet-1.18.4 kubeadm-1.18.4 kubectl-1.18.4

# 配置开机自启 
$ systemctl enable kubelet && systemctl start kubelet
```



### 5. 搭建 Kubernetes 集群

#### 5.1 初始化Master 节点

```bash
# 只在 master 节点执行

# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令
$ export MASTER_IP=192.168.2.100

# 替换 apiserver.master 为您想要的 dnsName
$ export APISERVER_NAME=apiserver.master

#Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于您的物理网络中
$ export POD_SUBNET=10.100.0.1/16

# 写入到主机网络中
$ echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts
```

#### 5.2 创建启动 K8S 脚本

```bash
# 创建一个k8s集群脚本目录
$ mkdir -p /usr/local/kubernetes/cluster/shell

# 添加编写初始master 的脚本
$ vim /usr/local/kubernetes/cluster/shell/init_master.sh

--------------------------------- k8s 初始脚本信息如下 -------------------------------------------
#!/bin/bash

# 只在 master 节点执行

# 脚本出错时终止执行
set -e

if [ ${#POD_SUBNET} -eq 0 ] || [ ${#APISERVER_NAME} -eq 0 ]; then
  echo -e "\033[31;1m请确保您已经设置了环境变量 POD_SUBNET 和 APISERVER_NAME \033[0m"
  echo 当前POD_SUBNET=$POD_SUBNET
  echo 当前APISERVER_NAME=$APISERVER_NAME
  exit 1
fi

# 查看完整配置选项 https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2
rm -f ./kubeadm-config.yaml
cat <<EOF > ./kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.4
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
controlPlaneEndpoint: "${APISERVER_NAME}:6443"
networking:
  serviceSubnet: "10.96.0.0/16"
  podSubnet: "${POD_SUBNET}"
  dnsDomain: "cluster.local"
EOF

# kubeadm init
# 根据您服务器网速的情况，您需要等候 3 - 10 分钟
kubeadm init --config=kubeadm-config.yaml --upload-certs

# 配置 kubectl
rm -rf /root/.kube/
mkdir /root/.kube/
cp -i /etc/kubernetes/admin.conf /root/.kube/config

# 安装 calico 网络插件
# 参考文档 https://docs.projectcalico.org/v3.13/getting-started/kubernetes/self-managed-onprem/onpremises
echo "安装calico-3.13.1"
rm -f calico-3.13.1.yaml
wget https://kuboard.cn/install-script/calico/calico-3.13.1.yaml
kubectl apply -f calico-3.13.1.yaml
-----------------------------------------------------------------------------------------------

# 执行脚本
$ sh /usr/local/kubernetes/cluster/shell/init_master.sh
```

#### 5.3 检查 master 初始化结果

```bash
# 执行如下命令，等待2-3分钟，直到所有的容器组处于 Running 状态
$ watch kubectl get pod -n kube-system -o wide

----------------------------------------------------------------------------------------------------------------------------------------------------------
NAME                                        READY   STATUS    RESTARTS   AGE   IP             NODE                NOMINATED NODE   READINESS GATES
calico-kube-controllers-74c9747c46-wf4fx    1/1     Running   0          61s   10.100.237.1   kubernetes-master   <none>           <none>
calico-node-vc77p                           1/1     Running   0          61s   192.168.2.100   kubernetes-master   <none>           <none>
coredns-7f9c544f75-dwd79                    1/1     Running   0          61s   10.100.237.2   kubernetes-master   <none>           <none>
coredns-7f9c544f75-v8xgv                    1/1     Running   0          61s   10.100.237.3   kubernetes-master   <none>           <none>
etcd-kubernetes-master                      1/1     Running   0          56s   192.168.2.100   kubernetes-master   <none>           <none>
kube-apiserver-kubernetes-master            1/1     Running   0          56s   192.168.2.100   kubernetes-master   <none>           <none>
kube-controller-manager-kubernetes-master   1/1     Running   0          55s   192.168.2.100   kubernetes-master   <none>           <none>
kube-proxy-6r4sr                            1/1     Running   0          61s   192.168.2.100   kubernetes-master   <none>           <none>
kube-scheduler-kubernetes-master            1/1     Running   0          56s   192.168.2.100   kubernetes-master   <none>           <none>
-----------------------------------------------------------------------------------------------------------------------------------------------------------

# 查看 master 节点初始化结果
$ kubectl get nodes -o wide
------------------------------------------------------------------------------------------------------------------------------------------------------------------
NAME                 STATUS     ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
kubernetes-master    Ready      master   8d    v1.18.4   192.168.2.100   <none>        CentOS Linux 7 (Core)   3.10.0-957.21.3.el7.x86_64    docker://19.3.8
------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

#### 5.4 给CNI中calico指定固定网卡

```bash
# 关闭 K8S 中 calico 
$ kubectl delete -f /usr/local/kubernetes/cluster/shell/calico-3.13.1.yaml

# 修改 calico 中配置，添加固定的网卡为em1
$ vim  /usr/local/kubernetes/cluster/shell/calico-3.13.1.yaml
```

```yml
# 以上部分省略...................
# ----------------------------------- 修改以下部分-----------------------------------------------
# Source: calico/templates/calico-node.yaml
# This manifest installs the calico-node container, as well
# as the CNI plugins and network config on
# each master and worker node in a Kubernetes cluster.
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: calico-node
  namespace: kube-system
  labels:
    k8s-app: calico-node
spec:
  selector:
    matchLabels:
      k8s-app: calico-node
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        k8s-app: calico-node
      annotations:
        # This, along with the CriticalAddonsOnly toleration below,
        # marks the pod as a critical add-on, ensuring it gets
        # priority scheduling and that its resources are reserved
        # if it ever gets evicted.
        scheduler.alpha.kubernetes.io/critical-pod: ''
    spec:
      nodeSelector:
        kubernetes.io/os: linux
      hostNetwork: true
      tolerations:
        # Make sure calico-node gets scheduled on all nodes.
        - effect: NoSchedule
          operator: Exists
        # Mark the pod as a critical add-on for rescheduling.
        - key: CriticalAddonsOnly
          operator: Exists
        - effect: NoExecute
          operator: Exists
      serviceAccountName: calico-node
      # Minimize downtime during a rolling upgrade or deletion; tell Kubernetes to do a "force
      # deletion": https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods.
      terminationGracePeriodSeconds: 0
      priorityClassName: system-node-critical
      initContainers:
        # This container performs upgrade from host-local IPAM to calico-ipam.
        # It can be deleted if this is a fresh installation, or if you have already
        # upgraded to use calico-ipam.
        - name: upgrade-ipam
          image: calico/cni:v3.13.1
          command: ["/opt/cni/bin/calico-ipam", "-upgrade"]
          env:
            - name: KUBERNETES_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: CALICO_NETWORKING_BACKEND
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: calico_backend
          volumeMounts:
            - mountPath: /var/lib/cni/networks
              name: host-local-net-dir
            - mountPath: /host/opt/cni/bin
              name: cni-bin-dir
          securityContext:
            privileged: true
        # This container installs the CNI binaries
        # and CNI network config file on each node.
        - name: install-cni
          image: calico/cni:v3.13.1
          command: ["/install-cni.sh"]
          env:
            # Name of the CNI config file to create.
            - name: CNI_CONF_NAME
              value: "10-calico.conflist"
            # The CNI network config to install on each node.
            - name: CNI_NETWORK_CONFIG
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: cni_network_config
            # Set the hostname based on the k8s node name.
            - name: KUBERNETES_NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            # CNI MTU Config variable
            - name: CNI_MTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            # Prevents the container from sleeping forever.
            - name: SLEEP
              value: "false"
          volumeMounts:
            - mountPath: /host/opt/cni/bin
              name: cni-bin-dir
            - mountPath: /host/etc/cni/net.d
              name: cni-net-dir
          securityContext:
            privileged: true
        # Adds a Flex Volume Driver that creates a per-pod Unix Domain Socket to allow Dikastes
        # to communicate with Felix over the Policy Sync API.
        - name: flexvol-driver
          image: calico/pod2daemon-flexvol:v3.13.1
          volumeMounts:
          - name: flexvol-driver-host
            mountPath: /host/driver
          securityContext:
            privileged: true
      containers:
        # Runs calico-node container on each Kubernetes node.  This
        # container programs network policy and routes on each
        # host.
        - name: calico-node
          image: calico/node:v3.13.1
          env:
            # Use Kubernetes API as the backing datastore.
            - name: DATASTORE_TYPE
              value: "kubernetes"
            # Wait for the datastore.
            - name: WAIT_FOR_DATASTORE
              value: "true"
            # Set based on the k8s node name.
            - name: NODENAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            # Choose the backend to use.
            - name: CALICO_NETWORKING_BACKEND
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: calico_backend
            # Cluster type to identify the deployment type
            - name: CLUSTER_TYPE
              value: "k8s,bgp"
            # Auto-detect the BGP IP address.
            # ----------------- 重点指定固定的网卡为em1----------------------------------------
            - name: IP_AUTODETECTION_METHOD
              value: "interface=em1"
            #-------------------------------------------------------------------------------
            - name: IP
              value: "autodetect"
            # Enable IPIP
            - name: CALICO_IPV4POOL_IPIP
              value: "Always"
            # Set MTU for tunnel device used if ipip is enabled
            - name: FELIX_IPINIPMTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            # The default IPv4 pool to create on startup if none exists. Pod IPs will be
            # chosen from this range. Changing this value after installation will have
            # no effect. This should fall within `--cluster-cidr`.
            # - name: CALICO_IPV4POOL_CIDR
            #   value: "192.168.0.0/16"
            # Disable file logging so `kubectl logs` works.
            - name: CALICO_DISABLE_FILE_LOGGING
              value: "true"
            # Set Felix endpoint to host default action to ACCEPT.
            - name: FELIX_DEFAULTENDPOINTTOHOSTACTION
              value: "ACCEPT"
            # Disable IPv6 on Kubernetes.
            - name: FELIX_IPV6SUPPORT
              value: "false"
            # Set Felix logging to "info"
            - name: FELIX_LOGSEVERITYSCREEN
              value: "info"
            - name: FELIX_HEALTHENABLED
              value: "true"
          securityContext:
            privileged: true
          resources:
            requests:
              cpu: 250m
          livenessProbe:
            exec:
              command:
              - /bin/calico-node
              - -felix-live
              - -bird-live
            periodSeconds: 10
            initialDelaySeconds: 10
            failureThreshold: 6
          readinessProbe:
            exec:
              command:
              - /bin/calico-node
              - -felix-ready
              - -bird-ready
            periodSeconds: 10
          volumeMounts:
            - mountPath: /lib/modules
              name: lib-modules
              readOnly: true
            - mountPath: /run/xtables.lock
              name: xtables-lock
              readOnly: false
            - mountPath: /var/run/calico
              name: var-run-calico
              readOnly: false
            - mountPath: /var/lib/calico
              name: var-lib-calico
              readOnly: false
            - name: policysync
              mountPath: /var/run/nodeagent
      volumes:
        # Used by calico-node.
        - name: lib-modules
          hostPath:
            path: /lib/modules
        - name: var-run-calico
          hostPath:
            path: /var/run/calico
        - name: var-lib-calico
          hostPath:
            path: /var/lib/calico
        - name: xtables-lock
          hostPath:
            path: /run/xtables.lock
            type: FileOrCreate
        # Used to install CNI.
        - name: cni-bin-dir
          hostPath:
            path: /opt/cni/bin
        - name: cni-net-dir
          hostPath:
            path: /etc/cni/net.d
        # Mount in the directory for host-local IPAM allocations. This is
        # used when upgrading from host-local to calico-ipam, and can be removed
        # if not using the upgrade-ipam init container.
        - name: host-local-net-dir
          hostPath:
            path: /var/lib/cni/networks
        # Used to create per-pod Unix Domain Sockets
        - name: policysync
          hostPath:
            type: DirectoryOrCreate
            path: /var/run/nodeagent
        # Used to install Flex Volume Driver
        - name: flexvol-driver-host
          hostPath:
            type: DirectoryOrCreate
            path: /usr/libexec/kubernetes/kubelet-plugins/volume/exec/nodeagent~uds
---
# -------------------------------- 以下部分省略 -------------------------------------------------
```

```bash
# 执行重新安装 K8S 中 calico 
$ kubectl apply -f /usr/local/kubernetes/cluster/shell/calico-3.13.1.yaml
```

### 5.3 初始化 Node 节点

##### 1. 获得join 命令参数- 在master 节点上执行

``` bash
$ kubeadm token create --print-join-command

-------------------------------------- 输出如下系信息，需要复制 -----------------------------------------------------------------------------------------------------------------

kubeadm join apiserver.master:6443 --token 4o2q6l.kjjs9pcoinuo4kpc --discovery-token-ca-cert-hash sha256:1b3eadbf7cbabe17adeee8e71ff297d9a02dcf82fbfda9bfccf2de9b07a8b969 
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

> 注意：有效时间该 token 的有效时间为 2小时内，您可以使用此 token > >初始化任意数量的 node 节点。

##### 2. 初始化Node，针对所有得Node节点上执行

```bash
# 只在 worker 节点执行
$ export MASTER_IP=192.168.2.100

# 替换 apiserver.demo 为初始化 master 节点时所使用的 APISERVER_NAME
$ export APISERVER_NAME=apiserver.master

# 写入到主机网络中 
$ echo "${MASTER_IP}    ${APISERVER_NAME}" >> /etc/hosts

# 将 worker 节点加入到master 中去
$ kubeadm join apiserver.master:6443 --token 4o2q6l.kjjs9pcoinuo4kpc --discovery-token-ca-cert-hash sha256:1b3eadbf7cbabe17adeee8e71ff297d9a02dcf82fbfda9bfccf2de9b07a8b969 
```

##### 3. 检查是节点是否加入到K8S中。

- 在 Master 执行。

```bash
$ kubectl get pod -n kube-system -o wide

----------------------------------------输出如下信息，当所有状态为Running 代码K8s 安装成功 -------------------------------------------------------------
NAME                                        READY   STATUS    RESTARTS   AGE   IP              NODE                 NOMINATED NODE   READINESS GATES
calico-kube-controllers-788d6b9876-m6vjb    1/1     Running   1          8d    10.100.237.5    kubernetes-master    <none>           <none>
calico-node-jnm95                           1/1     Running   0          8d    192.168.2.110   kubernetes-node-01   <none>           <none>
calico-node-ng8cr                           1/1     Running   0          8d    192.168.2.120   kubernetes-node-02   <none>           <none>
calico-node-ssg9n                           1/1     Running   1          8d    192.168.2.100   kubernetes-master    <none>           <none>
coredns-7f9c544f75-9bvjh                    1/1     Running   1          8d    10.100.237.4    kubernetes-master    <none>           <none>
coredns-7f9c544f75-p6qgt                    1/1     Running   1          8d    10.100.237.2    kubernetes-master    <none>           <none>
etcd-kubernetes-master                      1/1     Running   1          8d    192.168.2.100   kubernetes-master    <none>           <none>
kube-apiserver-kubernetes-master            1/1     Running   1          8d    192.168.2.100   kubernetes-master    <none>           <none>
kube-controller-manager-kubernetes-master   1/1     Running   1          8d    192.168.2.100   kubernetes-master    <none>           <none>
kube-proxy-5c5rx                            1/1     Running   0          8d    192.168.2.120   kubernetes-node-02   <none>           <none>
kube-proxy-9nvsb                            1/1     Running   0          8d    192.168.2.110   kubernetes-node-01   <none>           <none>
kube-proxy-xnmv9                            1/1     Running   1          8d    192.168.2.100   kubernetes-master    <none>           <none>
kube-scheduler-kubernetes-master            1/1     Running   1          8d    192.168.2.100   kubernetes-master    <none>           <none>
----------------------------------------------------------------------------------------------------------------------------------------------------
```

##### 4.  检查是否K8S 搭建成功

```bash
$ kubectl get nodes -o wide
----------------------------------------输出如下信息，当所有状态为Ready 代码K8s 搭建成功 ------------------------------------------------------------------------
NAME                 STATUS     ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION                CONTAINER-RUNTIME
kubernetes-master    Ready      master   8d    v1.18.4   192.168.2.100   <none>        CentOS Linux 7 (Core)   3.10.0-957.21.3.el7.x86_64    docker://19.3.8
kubernetes-node-01   Ready      <none>   8d    v1.18.4   192.168.2.110   <none>        CentOS Linux 7 (Core)   3.10.0-1062.el7.x86_64        docker://19.3.8
kubernetes-node-02   NotReady   <none>   8d    v1.18.4   192.168.2.120   <none>        CentOS Linux 7 (Core)   3.10.0-1127.el7.x86_64        docker://19.3.8
--------------------------------------------------------------------------------------------------------------------------------------------------------------
```

