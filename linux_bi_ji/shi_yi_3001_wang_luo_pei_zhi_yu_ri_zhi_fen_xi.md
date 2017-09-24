## 十一、网络配置与日志分析

### 网络配置与命令 {#-0}

```
ifconfig            //查看所有活动网络接口信息
ifconfig 网络接口名  //查看指定网络接口信息
ip address          //添加 IP地址
nmcli device show   //查看所有设备信息
hostname               //查看主机名

设置主机名通常使用三种方式：
1\. hostnamectl set-hostname 主机名
2\. vim /etc/hostname
3\. nmtui

route                //查看路由表
    route -n
    ip route

netstat        //查看接口统计信息等
    -l\-n\-t\-u\-p
    netstat -lntup 查看所有端口信息
    netstat -lntup | grep ：端口 只查看指定端口的信息

ping        //测试网络连通性

traceroute  测试主机到主机之间经过的网络节点

nslookup    测试DNS域名解析
    nslookup 主机名／ip

Linux网络接口表示方法
eth0 eth表示以太网卡0表示第一块网卡
lo0 表示本地回环网卡

设置网络接口ip以及子网掩码
ifconfig eth0 ip地址 [netmask 子网掩码]
ifconfig eth0 ip地址[/掩码长度]

禁用或者重新激活网卡
ifconfig eth0 up    或者  ifup eth0
ifconfig eth0 down  或者  ifdown eth0

设置虚拟网络接口
Ifconfig eth0:0 ip地址

删除路由表中默认网关记录
route del default gw IP地址
添加默认网关记录
route add default gw IP地址
添加指定网段的路由记录
route add -net 网段地址 gw IP地址
删除指定网段路由记录
route del -net 网段地址

第一块网卡的配置文件
/etc/sysconfig/network-scripts/ifcfg-eth0
网卡配置文件中的重要内容
DEVICE=eth0
ONBOOT=yes|no
BOOTPROTO=static|none|dhcp
IPADDR=
NETMASK=    或者  prefix=掩码位
GATEWAY=
网络服务重启使配置生效
service network restart

修改网卡名称为默认的eth设备
1.vim /etc/default/grub
    GRUB_CMDLINE_LINUX，在双引号中，参数的末尾追加net.ifnames=0 biosdevname=0
2.grub2-mkconfig -o /boot/grub2/grub.cfg
3.cd /etc/sysconfig/network-scripts/
    mv ifcfg-eno167* ifcfg-eth0
    mv ifcfg-eno344* ifcfg-eth1
4.reboot

```

RHEL7网卡配置 nmcli con show [--active] nmcli con show &quot;eth0&quot; nmcli dev status nmcli dev show eth0

```
nmcli con add con-name “Simple" ifname eth0 type ethernet ip4 172.25.0.X/24 gw4 172.25.X.254
nmcli con modify “Simple" ipv4.dns 172.25.254.254
nmcli con modify “Simple" ipv4.addresses 172.25.0.X+100/24
nmcli con modify “Simple" connection.autoconnect yes
nmcli con modify "System eth0" connection.autoconnect no
nmcli con modify "System eth0" +ipv4.addresses 10.0.0.X/24

/etc/hostname   主机名称配置文件
主机名
/etc/resolv.conf        DNS服务器地址保存位置
nameserver 8.8.8.8
/etc/hosts      主机与IP地址映射记录
ip地址    主机名

```

链路聚合技术 1\. 创建一个team设备，名称为team0，且使用主备模式 nmcli con add type team con-name st0 ifname team0 config &#039;{&quot;runner&quot;:{&quot;name&quot;: &quot;loadbalance/activebackup&quot;}}&#039; 2.设置 team0 组的 IP（V4、V6） 、网关等信息，同时设置ip 地址的获取方式 nmcli con mod st0 ipv4.addresses &#039;192.168.0.100+X/24&#039; nmcli con mod st0 ipv4.method manual 3.将指定以太网网卡设备加入team0组成网路组 nmcli con add type team-slave con-name team0-port1 ifname eth1 master team0 nmcli con add type team-slave con-name team0-port2 ifname eth2 master team0 4\. 查看 team0 设备连接是否已经创建 nmcli con up team0 teamdctl team0 state nmcli dev dis eth1 nmcli dev con eth1 nmcli con down team0-port1 nmcli con up team0-port1

```
teamdctl team0 state
ping test

```

### 日志管理 {#-1}

```
日志保存位置
默认位置位于：/var/log目录下
主要日志文件介绍
内核、公共消息日志
/var/log/messages
计划任务日志
/var/log/cron
系统引导日志
/var/log/dmesg
邮件系统日志
/var/log/maillog
用户登录日志
/var/log/secure

journalctl
journalctl -n 10
journalctl -p err
journalctl -f
journalctl --since today
journalctl --since "2017-02-10 20:30:00" --until "2017-02-13 12:00:00“

```