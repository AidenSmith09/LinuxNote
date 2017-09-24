## 十四、SSH和DHCP {#ssh-dhcp}

```
RPM：openssh, openssh-clients, openssh-server
服务: sshd
监听端口: 22/tcp
服务器配置文件: /etc/ssh/sshd_config
服务的启动分类
运行状态启动：systemctl start service
开机自启动：systemctl enable service
```

sshd 服务配置选项

```
Port
Protocol
ListenAddress
PermitRootLogin
PermitEmptyPasswords
LoginGraceTime
MaxAuthTries
PasswordAuthentication
PubkeyAuthentication
AuthorizedKeysFile
Banner

应用配置
systemctl restart sshd

用户登录时, sshd 在用户和系统之间建立连接
但是, 在系统管理员未停止 sshd 服务前, 连接不会断开
查看linux下登陆用户的行为信息
w  who
找出连接进程信息
ps aux | grep sshd/pts
将进程结束
kill pid
May use w command to find out where user logged in
killall sshd

sftp host
sftp user@host
使用特定用户登录
sftp -C user@host
启用压缩

无密码, 但仍然安全的身份验证
使用ssh-keygen生成的两个密钥:
私钥保存在本地主机
公钥需要传递到对端主机，使用指令 ssh-copy-id
ssh-copy-id [ -i ~/.ssh/id_rsa.pub ] [user@]host

scp远程文件传输测试
rsync远程文件传输测试
```

基于SSH做文件的上传和下载（windows交互） yum install lrzsz rz+回车，表示是上传本地文件到Linux服务器 sz+文件路径+回车，表示是下载Linux服务器文件到本地

格式转换： yum install dos2unix 如果将文件从Linux服务器下载到windows中，可以使用dos2unix转换文件格式 如果将文件从windows上传到Linux服务器中，可以使用unix2dos转换文件格式

## DHCP 

```
yum install dhcp -y 
```

DHCP配置文件`/etc/dhcp/dhcpd.conf`

```
subnet网段声明
range   地址池划分
option subnet-mask  设置客户机子网掩码
option routers  设置客户机默认网关地址
host主机声明
hareware ethernet   指定主机MAC地址
fixed-address   为该主机保留的ip地址
```

启动DHCP

```
systemctl start dhcpd   查看端口号验证
```

```
subnet 10.0.0.0 netmask 255.255.255.0 {
range 10.0.0.100 10.0.0.200; 
option domain-name-servers a.simple.com; 
option domain-name "simple.com"; 
option routers 10.0.0.254; 
option broadcast-address 10.0.0.255; 
default-lease-time 6000; max-lease-time 72000; 
} 
host boss { 
hardware ethernet 00:1C:42:84:C0:C1; 
fixed-address 10.0.0.123; 
}
```

客户端获取IP 

方法1： 在网卡配置文件中设定为获取方式为dhcp，然后重启网络服务 

方法2： 客户端执行dhclient -d eth0获取ip

