## 十二、常规优化

```
帐号安全基本措施
[root@localhost ~]# vi /etc/profile
HISTSIZE=200
HISTFILESIZE=100
设置在命令行界面中超时自动注销
 [root@localhost ~]# vi ~/.bash_profile
export TMOUT=600

vi ~/.bash_logout
history -c
clear
vi ~/.bash_logout
rm -f  ~/.bash_history
[root@localhost ~]# chage -M 30 lisi
[root@localhost ~]# chage -d 0 zhangsan

[root@localhost ~]# chattr +i /etc/passwd /etc/shadow
[root@localhost ~]# chattr +a /var/log/messages
[root@localhost ~]# lsattr /etc/passwd /etc/shadow /var/log/messages
----i-------- /etc/passwd
----i-------- /etc/shadow
-----a------- /var/log/messages

启用pam_wheel认证模块
将允许使用su命令的用户加入wheel组
[root@localhost ~]# vi /etc/pam.d/su 
#%PAM-1.0
auth        sufficient  pam_rootok.so
auth        required    pam_wheel.so use_uid

[root@localhost ~]# usermod -G simple wheel
将用户“simple”加入到“wheel”组中

visudo 或者 vi  /etc/sudoers
记录格式：用户    主机名列表=命令程序列表

[root@localhost ~]# visudo
……
%wheel              ALL=NOPASSWD: ALL
jerry               localhost=/sbin/ifconfig
syrianer            localhost=/sbin/*,!/sbin/ifconfig,!/sbin/route
Cmnd_Alias          PKGTOOLS=/bin/rpm,/usr/bin/yum
mike                localhost=PKGTOOLS

[root@localhost ~]# visudo
…… 
Defaults logfile = "/var/log/sudo

查询授权的sudo操作
sudo –l

```