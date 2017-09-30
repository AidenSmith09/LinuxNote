## 六、文件系统建立与磁盘配额

### 创建并使用磁盘分区

建议按照以下7步操作：


1.添加硬件设备，并且让系统识别新设备
fdisk -l查看识别的新设备

2.创建磁盘分区，并识别新分区,以下是整体执行过程。
    
    fdisk ／dev/sdb     #执行命令 sdb 可根据实际盘符更改
    Command (m for help):n        #输入“m”查看帮助信息 输入“n”创建新的磁盘分区
    Partition type:               #默认为 P 主分区
    p   primary (0 primary, 0 extended, 4 free)
    e   extended
    Select (default p)                      #p 主分区，e 扩展分区
    Using default response p
    Partition number (1-4, default 1):      #分区号：默认4个主分区
    First sector (2048-20971519, default 2048):   #上一个扇区结束位置，因新磁盘默认
    Using default value 2048
    Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519):   #大小
    Using default value 20971519
    Partition 1 of type Linux and of size 10 GiB is set
    Command (m for help):w
    The partition table has been altered!
    Calling ioctl() to re-read partition table.
    Syncing disks.
 
d删除指定的磁盘分区
p查看已创建磁盘分区的信息
w保存分区设置并推出
q不保存直接推出

如果磁盘是**首次创建分区**，fdisk划分磁盘分区之后，系统会**自动更新磁盘分区变化**
如果磁盘不是第一次创建分区，fdisk划分磁盘分区之后，需要执行`partprobe /dev/sdb`强制刷新磁盘信息
磁盘创建后，系统无法识别，需要通过partprobe 设备路径，刷新磁盘的分区信息，通知文件系统
（rhel5，刷新磁盘的指令为`partprobe`，rhel6，刷新磁盘的指令为`partx`）

3.制作文件系统
`mkfs -t ext4 /dev/sdb1`

4.制作磁盘标签（option）
`e2label /dev/sdb1  newpart`
查看已制作的磁盘分区信息，可以使用blkid获取

5.创建挂载点
`mkdir /mnt/newpart`

6.写系统启动的分区加载文件
```
vim /etc/fstab
/dev/sdb1  /mnt/newpart  ext4  defaults 0 0
```

fstab文件分为六个字段
* 第一个字段表示待挂载（待使用）的设备路径（/dev/sdb1,LABEL,UUID）
* 第二个字段表示设备挂载的本地路径
* 第三个字段表示设备的类型
* 第四个字段表示设备使用的权限控制，defaults表示可读写
* 第五个字段表示此设备是否做dump备份
* 第六个字段表示此设备加载前是否做磁盘检查


7.挂载并使用磁盘分区 
`mount -a`的行为是读取fstab文件，将文件中未挂载的设备挂载上

8.验证磁盘分区及分区信息查看
`df -Th`df查看磁盘挂载信息
* -T选项表示显示磁盘的设备类型
* -h表示以人性化方式显示磁盘空间

使用光盘永久挂载
```
vim /etc/fstab写入
/dev/sr0    /mnt/cdrom      iso9660  defaults 0 0
```

### 修改系统语言的三种方式

```
1.点击系统登录用户，找到设置，找到区域和语言，将language修改为简体中文或美式英文
2.yum install system-config-language ，然后调用指令system-config-language，设置默认的语言格式
3.编辑文件，/etc/locale.conf，修改为相应的语言格式，英文为LANG=en_US.UTF-8，中文为LANG=zh_CN.UTF-8
修改后建议立即重启使修改生效
```

### 虚拟内存使用

虚拟内存格式化使用`mkswap` 挂载并使用已制作的磁盘分区为虚拟内存，使用`swapon -a` 查看虚拟内存是由哪些设备提供的空间，使用`swapon -s`

### 磁盘配额的创建与使用

1. 挂载的同时需要为文件系统添加支持配额的选项

   ```
   vim /etc/fstab
   /dev/sdb1  /mnt/ext4  ext4   defaults,usrquota,grpquota    0 0
   保存退出后，可以执行mount -o remount /mnt/ext4 
   (umount /dev/sdb1,mount -a)
   mount | grep sdb1   发现属性中增加了磁盘配额
   ```

2. 在分区中生成配额文件quota.user和quota.group

   ```
    quotacheck -augcv
   -a 扫描所有支持配额的分区
   -u 扫描磁盘计算用户所占用的文件数
   -g 扫描磁盘计算组所占用的文件数
   -c 创建配额文件aquota.user和aquota.group
   -v 显示详细信息
   ```

3. 为用户建立配额信息 edquota -u user1 编辑用户user1的配额信息

   ```
   Disk quotas for user tom (uid 1001):
   Filesystem    blocks  soft    hard   inodes    soft     hard
   /dev/sdb1     0      81920   （80M）  102400  （100M）     0   8   10

   blocks表示tom用户在sdb1中已使用的空间大小（默认不做修改，系统自识别）
   soft表示tom用户在sdb1中创建文件时，空间的警告阈值
   hard表示tom用户在sdb1中创建文件时，空间的最大使用大小
   inodes表示tom用户在sdb1中已创建的文件数量
   soft表示tom用户在sdb1中创建文件时，文件数量的警告阈值
   hard表示tom用户在sdb1中创建文件时，文件数量的最大限制
   ```

4. 开启/关闭配额功能

   ```
   quotaon/quotaoff
   quotaon -a     //-a 表示开启支持配额功能所有分区
   ```

5. 切换用户，做空间和文件数量的限制测试

   ```
   空间测试可以使用dd指令
   dd if=/dev/zero of=tom1 bs=1M count=30
   文件数量测试可以使用touch指令
   touch file{1..10}
   ```

6. 检测系统中磁盘配额的使用情况

   ```
   按照用户查看的方式
   quota -u username

   按照磁盘分区查看的方式
   repquota /dev/sdb1
   ```



