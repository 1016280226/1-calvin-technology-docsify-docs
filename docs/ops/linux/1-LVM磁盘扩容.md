# 笔记一 LVM 磁盘扩容

## 一、LVM 简介

- **`LVM`** 是 **逻辑卷管理**(**`Logical Volume Manager`**)。

- **`LVM`** 是 Linux 环境下对**磁盘分区**进行管理的一种机制。

- **`LVM`** 将一个或多个**`PV`磁盘分区**虚拟为**`VG`一个卷组**，相当于一个大的硬盘，我们可以在上面划分一些**`LV `逻辑卷**。

  >- 当卷组的空间不够使用时，可以将新的磁盘分区加入进来。
  >
  >- 我们还可以从卷组剩余空间上划分一些空间给空间不够用的逻辑卷使用。

  

## 二、LVM 结构模型

LVM 结构模型，如下图：

<img src="../../../../statics/images/linux/lvm_disk_extend.png" style="zoom:100%;" alt="LVM结构模型图"/>                                         

### LVM 的基本概念

- **`PV`** (**Physical Volume**) **物理卷：** 可以在上面进建立卷组的媒介，可以是硬盘分区，也可以是硬盘本身或者回环（loopback file）。

  物理卷包括一个特殊的header，其余部分被切割为一块块物理区域（physical extents）

- **`VG`** (**Volume group**) **卷组：** 将一组物理卷收集为一个单元。

- **`LV`**(**Logical volume**) **逻辑卷：** 虚拟分区，由物理区域（physical extents）组成。

- **`PE` (Physical extent) 物理区域：** 硬盘可共指派给逻辑卷的最小单位 （通常为 4MB）。

## 三、LVM 扩容步骤

### 1. 查看磁盘被使用大小。

```bash
$ df -h

------------------------------------- 输出如下信息 ----------------------------------------------
/dev/mapper/centos-root  50G   31G  20G    62% /
devtmpfs                 5.7G     0  5.7G    0% /dev
tmpfs                    5.7G     0  5.7G    0% /dev/shm
tmpfs                    5.7G   36M  5.7G    1% /run
tmpfs                    5.7G     0  5.7G    0% /sys/fs/cgroup
/dev/sdb1               1014M  329M  686M   33% /boot
# 这里是我存储东西放在这里
/dev/mapper/centos-home  411G  227G  185G   56% /home
```



### 2. 查询当前<font color="#ff6702">LV </font>逻辑卷位置。

```bash
$ lvdisplay

------------------------------------- 输出如下信息 ----------------------------------------------
--- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                hshKov-UucB-aAXb-SwAF-3hjU-38jl-pUkvp0
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-03-12 09:03:53 +0800
  LV Status              available
  # open                 0
  LV Size                3.62 GiB
  Current LE             928
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/centos/home
  LV Name                home
  VG Name                centos
  LV UUID                5h1kGH-VO3t-er3x-AmnB-JCm9-yncC-Ck1uA5
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-03-12 09:03:53 +0800
  LV Status              available
  # open                 1
  LV Size                <411.13 GB
  Current LE             130849
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                a4rIYp-AdeV-JplX-Cux4-wKt1-eC4k-fKLgFz
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-03-12 09:03:56 +0800
  LV Status              available
  # open                 1
  LV Size                50.00 GiB
  Current LE             12800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

```



### 3. 查看<font color="#ff6702">VG</font>卷组。

```bash
$ vgdisplay

------------------------------------- 输出如下信息 ----------------------------------------------
--- Volume group ---
  # 当前卷组名称为 centos
  VG Name               centos  
  System ID             
  Format                lvm2
  Metadata Areas        1   
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <464.76 GiB
  PE Size               4.00 MiB
  Total PE              595909
  Alloc PE / Size       118977 / 464.75 GiB
  Free  PE / Size       1 / 4.00 MiB  
  VG UUID               Dpz2WX-Qo2X-jVWc-bm0Z-kIpo-x39a-7JdPGN
```



### 4. 查看 <font color="#ff6702">PV </font>物理卷。

```bash
$ pvdisplay

------------------------------------- 输出如下信息 ----------------------------------------------

 # 现在只有一块物理硬盘为 500 GiB 
  --- Physical volume ---
  PV Name               /dev/sdb2
  VG Name               centos
  PV Size               464.76 GiB / not usable 3.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              118978
  Free PE               0
  Allocated PE          118978
  PV UUID               LYxeai-UBFG-y1Rw-DHXM-g1An-Ankk-RKWWnT

```



### 5. <font color=red><b> 在服务器或电脑上安装新的物理硬盘（SAS、SATA、SSD...）。</b></font>



### 6. 安装新的物理硬盘后，使用命令查看新的磁盘是否被识别出来。

```bash
# 查看所有存储设备
$ fdisk -l

------------------------------------- 输出如下信息 ----------------------------------------------
# 识别成功，这是一个新的磁盘 2TiB, 新的磁盘分区以 /dev/sda
磁盘 /dev/sda：2000.4 GB, 2000398934016 字节，3907029168 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x0b0ab6df

# 这是旧的，500 GiB
磁盘 /dev/sdb：500.1 GB, 500107862016 字节，976773168 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节
磁盘标签类型：dos
磁盘标识符：0x0001ccc2

   设备 Boot      Start         End      Blocks   Id  System
/dev/sdb1   *        2048     2099199     1048576   83  Linux
/dev/sdb2         2099200   976773119   487336960   8e  Linux LVM

磁盘 /dev/mapper/centos-root：50.00 GB, 53687091200 字节，104857600 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节


磁盘 /dev/mapper/centos-swap：3.62 GB, 3755758592 字节，28573696 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节


磁盘 /dev/mapper/centos-home：411 GB, 41135766784 字节，3699679232 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节

```



### 7. 给新的磁盘分区。按照界面的提示，依次输入 :

- **`n`**(新建分区) ； 
- **`p`**(查看现有分区信息) 
- **`1`**(使用第1个主分区) ；
- **`Enter`** 两次回车(使用默认配置) ；
- 输入 **`w`** (保存分区表)，开始分区。

```bash
$ fdisk /dev/sda

------------------------------------- 输出如下信息 ----------------------------------------------
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。
命令(输入 m 获取帮助)：n
Partition type:
   p   primary (1 primary, 0 extended, 0 free)
   e   extended
Select (default p): p
分区号 (1,4，默认 1)：1
起始 扇区 (3907029167-3907029167，默认为 3907020800)：
将使用默认值 3907020800
Last 扇区, +扇区 or +size{K,M,G} (3907029167-3907029167，默认为 3907029167)：
将使用默认值 3907029167
分区 1 已设置为 Linux 类型，大小设为 2 TiB
The partition table has been altered!

Calling ioctl() to re-read partition table.
正在同步磁盘。
```



### 8. 查看Linux 文件系统类型(EXT3、EXT4、 XFS）

```bash
$ lsblk -f 

------------------------------------- 输出如下信息 ----------------------------------------------
# 这里文件 xfs
sdb               isw_raid_member                                              
└─md126                                                                        
  ├─md126p1       xfs                   e7bde924-1cfd-42f2-a14b-73a2358a906d   /boot
  └─md126p2       LVM2_member           v3eJwD-dIyb-8Tak-aI8i-0c1f-xqYh-26dZ06 
    ├─centos-root xfs                   91a73b33-36f0-4c7a-8166-79d47caf8036   /
    ├─centos-swap swap                  e7857228-d9e5-498a-8f7b-2bcd1cd4bd84   
    └─centos-home xfs                   49179c4a-c34c-4d92-b85b-8aa604d38f3f   /home
```



### 9. 格式化磁盘为 XFS 系统文件

```bash
$  mkfs -t xfs /dev/sda1

--------------------------------- 输出如下信息 --------------------------------------------------
meta-data=/dev/sda1                 isize=512  agcount=4, agsize=122094598 blks
         =                          sectsz=512 attr=2, projid32bit=1
         =                          crc=1      finobt=0, sparse=0
data     =                          bsize=4096 blocks=488378390, imaxpct=5
         =                          sunit=0    swidth=0 blks
naming   =version 2                 bsize=4096 ascii-ci=0 ftype=1
log      =internal log              bsize=4096 blocks=238466, version=2
         =                          sectsz=512 sunit=0 blks, lazy-count=1
realtime =none                      extsz=4096 blocks=0, rtextents=0
```



### 10. 创建一个<font color="#ff6702">PV </font>物理卷

```bash
$ pvcreate /dev/sda1

--------------------------------- 输出如下信息 --------------------------------------------------
WARNING: xfs signature detected on /dev/sda1 at offset 1080. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/sda1.
  Physical volume "/dev/sda1" successfully created.
```



### 11. 扩容 <font color="#ff6702">VG</font> 卷组

```bash
$ vgextend centos /dev/sda1

--------------------------------- 输出如下信息 --------------------------------------------------
Volume group "centos" successfully extended
```



### 12. 查看 <font color="#ff6702">VG</font> 卷组，是否多了一块 <font color="#ff6702">PV</font> 物理卷

```bash
$ pvscan
--------------------------------- 输出如下信息 --------------------------------------------------
PV /dev/sdb2   VG centos          lvm2 [<464.76 GiB / 0    free]
PV /dev/sda1   VG centos          lvm2 [<1.82 TiB / 1.82 TiB    free]
Total: 2 [2.27 TiB] / in use: 2 [2.27 TiB] / in no VG: 0 [0   ]
```



### 13. 查看 <font color="#ff6702">VG</font> 卷组

```bash
$ vgdisplay

--------------------------------- 输出如下信息 --------------------------------------------------
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  11
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               2.27 TiB   # VG 变成了 2.27 TiB (464.76 GiB + 1.82 TiB)
  PE Size               4.00 MiB
  Total PE              595909
  Alloc PE / Size       118977 / 464.75 GiB
  Free  PE / Size       3907029167 / 1.82 TiB    # 空闲 1.82 TiB
  VG UUID               Dpz2WX-Qo2X-jVWc-bm0Z-kIpo-x39a-7JdPGN

```



### 14. 扩容 <font color="#ff6702">LV</font> 逻辑卷

```bash
# 按固定大小追加
$ lvextend -L +500G /dev/centos/root

# 按百分比追加
$ lvextend -l +100%FREE /dev/centos/home

Size of logical volume centos/home changed from < 50.00 GiB to 550 GiB
Logical volume centos/home successfully resized

Size of logical volume centos/home changed from < 464.75 GiB to 1.72 TiB
Logical volume centos/home successfully resized
```



### 15. 刷新扩容后的分区

```bash
# xfs 如下 (ext4 使用命令 resize2fs)
$ xfs_growfs /dev/centos/root && xfs_growfs /dev/centos/home
```



### 16. 验证是否成功

```bash
 $ lvdisplay
 
 --------------------------------- 输出如下信息 -------------------------------------------------
 
 --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                hshKov-UucB-aAXb-SwAF-3hjU-38jl-pUkvp0
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-03-12 09:03:53 +0800
  LV Status              available
  # open                 0
  LV Size                13.62 GiB
  Current LE             3488
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/centos/home
  LV Name                home
  VG Name                centos
  LV UUID                5h1kGH-VO3t-er3x-AmnB-JCm9-yncC-Ck1uA5
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-03-12 09:03:53 +0800
  LV Status              available
  # open                 1
  LV Size                1.72 TiB  # 扩容后的磁盘空间
  Current LE             451621
  Segments               5
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                a4rIYp-AdeV-JplX-Cux4-wKt1-eC4k-fKLgFz
  LV Write Access        read/write
  LV Creation host, time localhost, 2019-03-12 09:03:56 +0800
  LV Status              available
  # open                 1
  LV Size                550.00 GiB  # 扩容后的磁盘空间
  Current LE             140800
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```



```bash
$ df -h

--------------------------------- 输出如下信息 -------------------------------------------------
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root  550G   31G  520G    6% /       # 扩容后，可以用 520G
devtmpfs                 5.7G     0  5.7G    0% /dev
tmpfs                    5.7G     0  5.7G    0% /dev/shm
tmpfs                    5.7G   44M  5.6G    1% /run
tmpfs                    5.7G     0  5.7G    0% /sys/fs/cgroup
/dev/sdb1               1014M  329M  686M   33% /boot
/dev/mapper/centos-home  1.8T  227G  1.6T   13% /home   # 扩容后，可以用1.6T

```

