## 十四、SSH和DHCP

```
RPM包：openssh, openssh-clients, openssh-server
服务: sshd
监听端口: 22/tcp
服务器配置文件: /etc/ssh/sshd_config
服务的启动分类
运行状态启动：systemctl start service
开机自启动：systemctl enable service
```

sshd 服务配置选项

```
Port                    //端口号，默认为TCP的22号端口，修改端口时，取消注释，直接修改，或复制本行后修改
Protocol ssh            //安全传输协议，目前默认使用的是第二版协议
ListenAddress        //监听的地址段

PermitRootLogin        //是否允许用户在发起ssh请求的时候，以root的身份直连本地主机，yes表示启用，不允许root登陆
PermitEmptyPasswords    //是否允许空密码登陆

LoginGraceTime        //ssh登陆时，密码验证的超时时间，默认为2分钟
MaxAuthTries        //ssh输入密码的最大尝试次数，默认为6次

PasswordAuthentication    //是否启用密码身份验证
PubkeyAuthentication    //是否启用公私钥对进行身份认证

AuthorizedKeysFile    //指定ssh远程连接的公钥存放路径
Banner            //显示软件版本等相关信息

应用配置
systemctl restart sshd
```

用户登录时, sshd 在用户和系统之间建立连接。但是, 在系统管理员未停止 sshd 服务前, 连接不会断开。

```
查看linux下登陆用户的行为信息
w/who

找出连接进程信息
ps aux | grep sshd/pts

将进程结束
kill pid
May use w command to find out where user logged in
killall sshd
```

SFTP 文件传输协议

```
sftp host
sftp user@host

使用特定用户登录
sftp -C user@host

上传：put /path/filename(本地主机) /path/filename(远端主机)；
下载：get /path/filename(远端主机) /path/filename(本地主机)。
```

公私验证

无密码, 但仍然安全的身份验证

使用ssh-keygen生成的两个密钥:

```
[root@localhost ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):   //此处可自定义名称
Created directory '/root/.ssh'.                            //.ssh 目录则自动生成。若普通用户，则在 home 目录下生成。
Enter passphrase (empty for no passphrase):                //可不填写
Enter same passphrase again:                               //可不填写
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
bf:09:d4:0a:ed:fb:32:7b:74:8b:95:5d:fd:7d:35:fe root@localhost.localdomain
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|                .|
|       . .     .+|
|      . S .  o..=|
|       + o. + ..+|
|        +..+ .  o|
|        oooo.   E|
|        o*+      |
+-----------------+

[root@localhost ~]# cd .ssh/
[root@localhost .ssh]# ls
id_rsa  id_rsa.pub

.pub后缀的文件为公钥，
私钥保存在本地主机，重要私钥会进行加密


公钥需要传递到对端主机，使用指令 ssh-copy-id
ssh-copy-id id_rsa.pub [user]@host    //host 为 IP 地址。
或
ssh-copy-id [ -i ~/.ssh/id_rsa.pub ] [user@]host

若创建新的名称，则需要添加一下私钥，否则公钥就算传输至目标，也无法建立连接。
ssh-add [私钥]
ssh-copy-id -i [公钥] [user]@[host/IP]


scp远程文件传输测试
rsync远程文件传输测试
```

基于SSH做文件的上传和下载（windows交互） yum install lrzsz

上传本地文件到Linux服务器`rz+回车`

下载Linux服务器文件到本地`sz+文件路径+回车`

格式转换： `yum install dos2unix` 如果将文件从Linux服务器下载到windows中，可以使用dos2unix转换文件格式 如果将文件从windows上传到Linux服务器中，可以使用unix2dos转换文件格式

## DHCP

```
yum install dhcp -y
```

DNS服务器地址：

* DHCP Server：**UDP 67**
* DHCP Client：**UDP 68**

将模版信息追加到配置文件中

`cat /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example >> /etc/dhcp/dhcpd.conf`

DHCP配置文件`/etc/dhcp/dhcpd.conf`

```
subnet                网段声明
range                 地址池划分
option subnet-mask    设置客户机子网掩码
option routers        设置客户机默认网关地址
host                  主机声明
hareware ethernet     指定主机MAC地址
fixed-address         为该主机保留的ip地址
```

**启动DHCP**

```
systemctl start dhcpd      启动 dhcp 服务
systemctl enable dhcpd     添加开机启动
netstat -lntup | grep :67  查看端口号
```

```
vim /etc/dhcp/dhcpd.conf                        //可自己填写，可调用模板

subnet 10.0.0.0 netmask 255.255.255.0 {          //设置网段
range 10.0.0.100 10.0.0.200;                     //限制地址范围
option domain-name-servers a.simple.com;         //dns 地址
option domain-name "simple.com";                 //域名
option routers 10.0.0.254;                       //网管
option broadcast-address 10.0.0.255;             //广播地址
default-lease-time 6000;                         //默认租约时间，单位秒
max-lease-time 72000;                            //最大租约时间
} 
host boss { 
hardware ethernet 00:1C:42:84:C0:C1;             //MAC地址
fixed-address 10.0.0.123;                        //保留IP
}
```

客户端获取IP

方法1： 在网卡配置文件中设定为获取方式为dhcp，然后重启网络服务

方法2： 客户端执行dhclient -d eth0获取ip

