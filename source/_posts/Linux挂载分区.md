---
title: Linux挂载分区
---

## 查看硬盘
### 已挂载
``` bash
[root@VM_0_9_centos ~]# df -h 
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        50G   18G   29G  39% /
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           3.9G  696K  3.9G   1% /run
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
tmpfs           783M     0  783M   0% /run/user/0
```
### 所有
``` bash
[root@VM_0_9_centos ~]# fdisk -l

Disk /dev/vda: 53.7 GB, 53687091200 bytes, 104857600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0009ac89

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048   104857599    52427776   83  Linux

Disk /dev/vdb: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
需要挂载的是/dev/vdb
## 找到硬盘分区格式化
### 磁盘分区
``` bash
[root@VM_0_9_centos ~]# fdisk /dev/vdb       //进入磁盘
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x5e70f310.

Command (m for help): p	                    //查看已有分区

Disk /dev/vdb: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x5e70f310

   Device Boot      Start         End      Blocks   Id  System

Command (m for help): n                    //创建分区
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p                      //主分区
Partition number (1-4, default 1): 
First sector (2048-209715199, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-209715199, default 209715199): 
Using default value 209715199
Partition 1 of type Linux and of size 100 GiB is set

Command (m for help): p                   //查看确认主分区

Disk /dev/vdb: 107.4 GB, 107374182400 bytes, 209715200 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x5e70f310

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048   209715199   104856576   83  Linux

Command (m for help): w                    //保存修改
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.

```
### 分区格式化
必须格式化，不然拿不到UUID
``` bash
[root@VM_0_9_centos ~]# mkfs.xfs /dev/vdb1 
meta-data=/dev/vdb1              isize=512    agcount=4, agsize=6553536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=26214144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=12799, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
### 查看检查分区
``` bash
[root@VM_0_9_centos ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 37.7M  0 rom  
vda    253:0    0   50G  0 disk 
└─vda1 253:1    0   50G  0 part /
vdb    253:16   0  100G  0 disk 
└─vdb1 253:17   0  100G  0 part 
```
## 挂载分区到"/home"
### 查看UUID，写入fstab
``` bash
[root@VM_0_9_centos ~]# blkid /dev/vdb1 
/dev/vdb1: UUID="0e1577b6-52df-49c7-8ea3-1d1a83e71809" TYPE="xfs" 
```
### 编辑fstab
``` bash
$ vim /etc/fstab
```
尾行添加`UUID=0e1577b6-52df-49c7-8ea3-1d1a83e71809            /home xfs defaults    0 0 `
### 重新读取配置文件
``` bash
$ mount -a 
```
### 查看结果
``` bash
[root@VM_0_9_centos home]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        50G   18G   29G  39% /
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           3.9G  724K  3.9G   1% /run
tmpfs           3.9G     0  3.9G   0% /sys/fs/cgroup
tmpfs           783M     0  783M   0% /run/user/0
/dev/vdb1       100G   33M  100G   1% /home
```