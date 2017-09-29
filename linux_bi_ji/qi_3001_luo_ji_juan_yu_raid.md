## 七、逻辑卷与RAID

### 逻辑卷的创建步骤

```
fdisk 设备，增加磁盘分区，同时修改类型为8e
partprobe刷新磁盘分区后，将磁盘分区初始化为物理卷
pvcreate /dev/sdb6

将物理卷加入卷组
vgcreate  卷组名myvg  /dev/sdb6

在卷组中划分逻辑卷
lvcreate  -L 指定大小  -n  逻辑卷名称mylv   在哪个卷组创建myvg

创建后制作文件系统
mkfs.ext4  /dev/myvg/mylv

写分区文件，永久挂载，挂载设备为/dev/myvg/mylv，挂载路径为/mnt/lvm
vim /etc/fstab
/dev/myvg/mylv  /mnt/lvm  ext4  defaults 0 0

```

### 拉伸LVM

```
1.卷组空间能够满足拉伸需求时：
（1）拉伸LVM
   lvextend -L  +400M  /dev/myvg/mylv
（2）通知文件系统空间变化
   resize2fs  /dev/myvg/mylv
2.卷组空间不能满足拉伸需求时：
（1）创建新的磁盘分区
   fdisk 制作新分区
（2）将新分区初始化为物理卷
   pvcreate /dev/sdb7
（3）拉伸卷组
   vgextend  myvg  /dev/sdb7
（4）拉伸LVM
    lvextend -L  +400M  /dev/myvg/mylv
（5）通知文件系统空间变化
    resize2fs  /dev/myvg/mylv

    以上针对的是EXT文件系统
   如果文件系统格式为XFS，则拉伸时，resize2fs指令替换为xfs_growfs

```

### 缩小LVM {#lvm-0}

```
1.文件系统下线
umount /dev/lvm

2.强制检测磁盘是否有损坏
e2fsck -f /dev/myvg/mylv

3.通知文件系统新的空间大小
resize2fs  /dev/myvg/mylv  200M

4.缩小LVM
lvresize -L 200M /dev/myvg/mylv

```

### 删除LVM {#lvm-1}

```
1.修改分区文件，将LVM注释或删除
2.下线文件系统
3.lvremove
4.vgremove
5.pvremove
6.fdisk （d）

```

### 磁盘阵列 {#-0}

#### 软RAID的建立： {#raid-0}

```
1.利用3块硬盘组建RAID5
mdadm -C /dev/md0 -n3 -l5 /dev/sd[bcd]
-C 创建阵列存储设备
-n 添加磁盘的数量
-l RAID的等级

2.查看RAID状态
mdadm -D /dev/md0 查看uuid
cat /proc/mtstat

3.由于md0设备文件属于临时创建，重启系统后会失效，需要建立阵列的配置文件使其永久生效
vim /etc/mdadm.conf
ARRAY /dev/md0 UUID=xxxxxxxxxxxxxxxxxxxxxxx
可以通过pvcreate /dev/md0 建立物理卷卷组逻辑卷

```

#### RAID5的故障处理 {#raid5}

```
mdadm --manage /dev/md0 --fail /dev/sdb
--fail  将设备设定为出错状态
--remove    将设备从阵列中移除
--add   添加设备进入阵列

```

#### 停用阵列 {#-1}

```
umount /dev/md0 卸载设备
vim /etc/fstab
#/dev/vg_raid/lv_raid   /mnt/raid5  ext4    defaults    0 0 注释有效内容
vim /etc/mdadm.conf
# ARRAY /dev/md0 UUID=xxxxxxxxxxxxxxxxxxxxxxx   注释有效内容
mdadm -S /dev/md0   停止md0
cat /proc/mdstat    验证状态

```