## 十五、网络文件共享协议

samba： 一种用于windows与linux实现文件系统共享的服务 samba服务常用协议有以下两种 smb协议 server message block 服务消息快 cifs协议 common internet file system 通用互联网文件系统 在系统中可以通过yum install samba安装 samba服务主要有两个程序 smb：提供对服务器中文件的共享访问 nmb：提供基于netbios主机名称的解析

启动脚本 systemctl start smb nmb systemctl enable smb nmb Samba的配置文件 /etc/samba/smb.conf 该配置文件中#开通的内容表示注释内容，；开头的内容表示实例内容都可以通过过滤的方法将其去除

实验配置： server： yum install samba -y systemctl start smb nmb systemctl enable smb nmb firewall-cmd --permanent --add-rich-rule=&#039;rule family=ipv4 source address=10.0.0.0/24 service name=samba accept&#039; firewall-cmd --reload firewall-cmd --list-rich-rules mkdir /smbshare chcon -t samba_share_t /smbshare/ chmod 770 /smbshare/ chgrp sales /smbshare/ ls -ld /smbshare/ vim /etc/samba/smb.conf groupadd sales useradd -s /sbin/nologin -G sales john useradd -s /sbin/nologin -G sales james smbpasswd -a john smbpasswd -a james pdbedit -L systemctl restart smb

client： Windows访问需要使用unc路径\ Linux访问samba smbclient -L ip地址 #提示输入密码回车即可 smbclient -U 用户名 //ip地址/共享目录 使用mount挂载共享文件夹 mount –o username=用户名 //ip地址/共享目录 /挂载点

```
想要永久挂载多用户的samba服务，可以启用multiuser
//IP地址/dir   /mountpoint  cifs  username=john,password=redhat 0 0

```

NFS： Network File System，网络文件系统 主要用于linux与linux或者unix之间进行文件系统共享的服务 依赖于RPC服务（远端调用） 需要安装nfs-utils、rpcbind软件包 共享配置文件为/etc/exports

共享目录 主机(选项) /myshare *.simplexue.com.cn /myshare desktop[0-20].simplexue.com.cn /myshare 172.25.0.0/16 /myshare *.simplexue.com(rw) 172.25.0.0/16(ro) /myshare 172.25.0.100(rw,no_root_squash) no_root_squash选项作用，nfs默认会将root用户的身份转换为nfsnobody以提高安全性，如果需要使用root用户的身份挂载使用需要添加此配置

查看服务器共享内容 showmount -e 192.168.0.1（V4版NFS，此指令只能在服务器端查看） 手动挂载 mount -t nfs -o sync 192.168.0.1:/myshare /mountpoint 自动挂载 vim /etc/fstab ip地址:/myshare /mountpoint nfs sync,ro 0 0

FTP： FTP的工作模式 主动模式 开放20和21端口 被动模式 开放21以及一个随机端口 其中21端口固定用来做控制链接

FTP用户类型 匿名用户：anonymous 或者 ftp 本地用户：服务器本身的用户家目录为共享目录 虚拟用户：使用独立账户密码数据文件的用户 常见的FTP服务程序 IIS、Server-U Vsftpd（Very Secure FTP Daemon） 可以通过yum install vsftpd 安装 主要配置文件为 /etc/vsftpd/ftpusers 黑名单 /etc/vsftpd/user_list 黑白名单 /etc/vsftpd/vsftpd.conf 主配置文件

常用的匿名 FTP 配置项 anonymous_enable=YES：启用匿名访问 anon_umask=022：匿名用户所上传文件的权限掩码 anon_root=/var/ftp：匿名用户的 FTP 根目录 anon_upload_enable=YES：允许上传文件 anon_mkdir_write_enable=YES：允许创建目录 anon_other_write_enable=YES：开放其他写入权 anon_max_rate=50000：限制最大传输速率（字节/秒）

常用的本地用户 FTP 配置项 local_enable=YES：是否启用本地系统用户 local_umask=022：本地用户所上传文件的权限掩码 local_root=/var/ftp：设置本地用户的 FTP 根目录 chroot_local_user=YES：是否将用户禁锢在主目录 local_max_rate=0：限制最大传输速率（字节/秒）

Linux客户端推荐安装lftp yum install lftp ftp lftp ip地址 即可访问