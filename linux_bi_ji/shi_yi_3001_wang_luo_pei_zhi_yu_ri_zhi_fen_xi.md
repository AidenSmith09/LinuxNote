## 十一、网络配置与日志分析

### 网络配置与命令

```
ifconfig            //查看所有活动网络接口信息
ifconfig 网络接口名   //查看指定网络接口信息
ip address          //添加 IP地址
nmcli device show   //查看所有设备信息
hostname            //查看主机名

设置主机名通常使用三种方式：
1.hostnamectl set-hostname 主机名
2.vim /etc/hostname
3.nmtui

route                //查看路由表
    route -n
    ip route

netstat              //查看接口统计信息等
    -l\-n\-t\-u\-p
    netstat -lntup   //查看所有端口信息
    netstat -lntup | grep ：端口  //只查看指定端口的信息

ping        //测试网络连通性

traceroute  //测试主机到主机之间经过的网络节点

nslookup    //测试DNS域名解析
nslookup 主机名／ip
```

### Linux网络接口表示方法

```
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
```

#### 第一块网卡的配置文件

网卡配置文件路径：`vim /etc/sysconfig/network-scripts/ifcfg-eth0`

```
DEVICE=eth0
ONBOOT=yes|no
BOOTPROTO=static|none|dhcp
IPADDR= 
NETMASK=    或者  prefix=掩码位
GATEWAY=
```

网络服务重启使配置生效

```
service network restart
```

修改网卡名称为默认的eth设备

```
1.vim /etc/default/grub
    GRUB_CMDLINE_LINUX，在双引号中，参数的末尾追加net.ifnames=0 biosdevname=0
2.grub2-mkconfig -o /boot/grub2/grub.cfg
3.cd /etc/sysconfig/network-scripts/
    mv ifcfg-eno167* ifcfg-eth0
    mv ifcfg-eno344* ifcfg-eth1
4.reboot
```

### 

### RHEL7网卡配置

* nmcli con show                  //查看所有网卡的链接信息
* nmcli con show eth0         //查看指定网卡链路信息
* nmcli dev status                //查看所有网卡的物理链路状态
* nmcli dev show eth0         //查看指定网卡的物理链路状态

```bash
添加新的网卡链接
nmcli con add con-name “xxxx" ifname eth0 type ethernet ip4 172.25.0.X/24 gw4 172.25.X.254
con   表示针对链接做相关操作
add   表示增加一个新的网卡链接
con-name 表示网卡的链接名称
ifname   表示本地的网卡设备名称
type     表示网卡的类型）

为现有的网卡链接设置DNS地址
nmcli con modify “xxxx" ipv4.dns 172.25.254.254

为现有的网卡链接设置新的IP地址（IP1
nmcli con modify “xxxx" ipv4.addresses 172.25.0.X+100/24

将新的链接作为网卡的默认加载文件（connection.autoconnect yes = 配置文件中的onboot=yes）
nmcli con modify “xxxx" connection.autoconnect yes

将不使用的链接禁用（connection.autoconnect no = 配置文件中的onboot=no），同时需要注意，同一块物理网卡，只允许一个链接启动
nmcli con modify "System eth0" connection.autoconnect no

为现有的网卡链接增加新的IP地址（IP2）
nmcli con modify "System eth0" +ipv4.addresses 10.0.0.X/24
```

### 网络信息更改

```bash
/etc/hostname   主机名称配置文件 
主机名

/etc/resolv.conf        DNS服务器地址保存位置
nameserver 8.8.8.8

/etc/hosts      主机与IP地址映射记录
ip地址    主机名
```

### 链路聚合技术

配置的网卡模式为team卡

team卡的工作模式常见的主要有3种：

1. roundrobin：轮询的方式，依次将数据流交给网卡进行数据传输
2. activebackup：热备模式，当主端口出现故障时，备用端口会基于监听模式自启动保证数据传输
3. loadbalance：根据哈希算法，将每个端口的数据流做统计，最大限度的保证每个端口的数据传输都是一样的

#### team卡配置命令

```
1. 创建一个team设备，名称为team0，且使用主备模式
nmcli con add type team con-name st0 ifname team0 config '{"runner":{"name": "loadbalance/activebackup"}}' 

2.设置 team0 组的 IP（V4、V6）、网关等信息，同时设置ip 地址的获取方式（manual表示为手动配置模式） 
nmcli con mod st0 ipv4.addresses '192.168.0.100+X/24' 
nmcli con mod st0 ipv4.method manual 

3.将指定以太网网卡设备加入team0组成网路组 
nmcli con add type team-slave con-name team0-port1 ifname eth1 master team0 
nmcli con add type team-slave con-name team0-port2 ifname eth2 master team0 

4. 查看 team0 设备连接是否已经创建 
nmcli con up team0
teamdctl team0 state


网卡链接状态控制
nmcli dev dis eth0 = nmcli con down team0-port1（关闭网卡链接）
nmcli dev con eth0 = nmcli con up team0-port1 （开启网卡链接）
```

---

### 日志管理 {#-1}

查看`journald`运行状态的方式：`systemctl status systemd-journald`

#### 日志保存位置

```
默认位置位于：/var/log目录下

主要日志文件介绍:
内核、公共消息日志: /var/log/messages

计划任务日志:/var/log/cron

系统引导日志:/var/log/dmesg

邮件系统日志:/var/log/maillog

用户登录日志:/var/log/secure


journalctl
    空格可以向下翻页，小写字母q表示退出当前日志的查看
journalctl -n 10
    表示查看journald记录日志的后十行
journalctl -p err
    表示查看对应消息级别的相关日志
journalctl -f
    表示实时监听系统日志
tail -f /var/log/messages（/var/log/secure）
    表示实时监听messages文件的日志变化
```

#### 日志消息的级别

* 0  EMERG（紧急）：     会导致主机系统不可用的情况
* 1  ALERT（警告）：            必须马上采取措施解决的问题
* 2  CRIT（严重）：        比较严重的情况
* 3  ERR（错误）：        运行出现错误
* 4  WARNING（提醒）：    可能会影响系统功能的事件
* 5  NOTICE（注意）：            不会影响系统但值得注意
* 6  INFO（信息）：            一般信息
* 7  DEBUG（调试）：            程序或系统调试信息等



