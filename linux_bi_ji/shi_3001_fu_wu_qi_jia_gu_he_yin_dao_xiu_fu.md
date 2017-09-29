## 十、服务器加固和引导修复

#### 确认当前系统的内核版本

```
查看当前kernel: uname -r
查看可升级kernel: yum list kernel

```

#### 升级kernel: 

```
yum update kernel
rpm -ivh kernel-3.10.0-123.1.2.el7.x86_64.rpm
yum localinstall kernel-3.10.0-123.1.2.el7.x86_64.rpm
建议在升级完核心后立即重启，同时安装核心时，尽量不要做消耗资源的操作

```

#### 查看已安装kernel版本：

```
cat /boot/grub2/grub.cfg | grep -i 'red hat'
设置kernel启动版本
grub2-set-default number（数字按照上面文件的顺序设置，如果有3个核心，设置区间为0-2，如果设置为3的话则循环为0）

查看设置的启动版本
grub2-editenv list（/boot/grub2/grubenv）

修改grub菜单的超时时间
vim /etc/default/grub中GRUB_TIMEOUT=10
grub2-mkconfig -o /boot/grub2/grub.cfg

```

#### RHEL7密码破解

```
1\. 在kernel引导加载的参数后写入rd.break，按Ctrl+x启动
       rd.break

2\. 因为根分区默认是只读挂载的（mount | grep /dev/sda或mount | grep root），所以需要重新挂载
   mount -o remount,rw /sysroot

3\. 从修复模式的假根中切换到系统真正的根分区
   chroot /sysroot

4\. 修改密码
   passwd root

5\. 因为密码破解是，selinux是不工作的，所以修改密码后，文件属性会发生变化，系统再重启后，会导致密码修改失败
   新建文件，通知系统，不要做覆盖
   touch /.autorelabel (then 2 exit)

```

#### grub2引导菜单加密

```
通过命令grub2-mkpasswd-pbkdf2生成加密密码（复制生成的字符串）
修改/etc/grub.d/00_header，在末尾追加
cat << EOF
set superusers="user1"
password_pbkdf2 user1 粘贴生成的字符串
EOF
更新配置文件以生效
grub2-mkconfig -o /boot/grub2/grub.cfg

```

#### grub引导故障 {#grub}

```
故障原因 grub.cfg文件丢失
故障现象 grub>     

解决办法一：
进行手动引导
set root=（hd0，msdos1） 设定根为第一个硬盘的第一个分区
linux /vmlinuz-*.el7.x86_64 root=/dev/mapper/xxx（/dev/sda1） 指定内核将根的位置交给文件系统
Initrd /initramfs-*.el7.x86_64.img 加载初始化镜像盘为硬件加载驱动
Boot 引导进入系统

解决办法二：
在硬盘sda上安装grub2，重新覆盖原来的引导
grub2-install /dev/sda
让grub2自己识别不同的系统，然后按照脚本自己创建引导，并更新配置文件/boot/grub2/grub.cfg
grub2-mkconfig -o /boot/grub2/grub.cfg

diff 源文件 目标文件  对文件信息做每一行的对比

```

#### 救援模式的使用 {#-1}

```
当系统出现故障无法进入时，可以通过光盘中的救援系统进行数据的提取（类似于winpe）
通常需要将光盘或网卡作为默认引导设备，然后选择troubshooting中的rescue模式，修复系统
也可以在安装菜单单机键盘的tab键，在参数后面增加 rescue

修复模式中，正根被挂载到了/mnt/sysimage目录下
chroot /mnt/sysimage可以切换到真根
测试时，可以新建两个分区，分别格式化为EXT文件系统和XFS文件系统
磁盘检查的指令
EXT：fsck （fsck -f ／dev/sda1）
XFS：xfs_repire  （xfs_repire -L ／dev/sda2）

```