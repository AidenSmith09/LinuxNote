## 十三、selinux和防火墙 {#selinux}

### SElinux基本说明

selinux类型: Enforcing, Permissive, or Disabled

* Enforcing：既阻止用户的违规行为，同时对违规行为作日志记录
* Permissive：不对违规行为作阻止，只记录日志
* Disabled：SElinux不开启，不记录日志

配置文件： `/etc/sysconfig/selinux`（重启生效）

```
seLinux 文件内容
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled     #此处更改状态
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

图形工具

```
yum install policycoreutils-gui
system-config-selinux
```

在不重启状态下，如果当前的模式为enforcing或permissive，可以通过以下指令修改为另一种，但是**不可以**修改为disabled

```
获取当前SElinux工作状态
getenforce

运行时修改
setenforce 0 | 1
```

kernel参数的修改

当系统启动时，需要在kernel层对SElinux做控制时，可以使用以下参数：

```
selinux=0|1，如果设置为0时，开机后getenforce获取的状态为disabled
enforcing=0|1，如果设置为0时，开机后getenforce获取的状态为permissive
```



每个文件和进程都有上下文，查看的指令为：

ls -lZ 文件或目录（-d）

ps auxZ \| less

SElinux的用户类型分为三种：超级用户，系统用户，普通用户

文件或目录的SElinux权限控制主要设置的是第三列字段，type类型



修改文件标签的方法：

chcon -R -t public\_content\_t /srv/www

chcon表示运行状态下的修改，修改完立即生效，restorecon可以将之前的修改还原

在修复模式等特殊环境下，chcon做的修改会被重置还原

semanage fcontext -a -t httpd\_sys\_content\_t '/srv/www\(/.\*\)?'

semanage表示系统层的修改，修改完不会立即生效，需要通过重启或restorecon触发更新，更改之后的标签类型，restorecon无法还原，只能通过semanage设置新的标签类型

restorecon -Rv /srv/www

表示还原或触发系统默认标签类型

-R表示递归式还原，-v表示显示还原标签的变化信息



布尔值：

主要是通过应用规则，保护服务器的文件目录不被破坏

        获取布尔值的方法：

        getsebool -a

可以通过grep过滤需要设置的服务关键词，对布尔值作修改

	修改布尔值的方法：

	setsebool \[-P\] httpd\_enable\_cgi off/on

	-P表示规则永久生效，不加-P时，规则在不重启的情况下生效

端口SElinux标签查看方法：

	semanage port -l



SElinux日志机制：

如果SElinux的监听服务开启，setroubleshootd（RHEL5/6）/auditd（RHEL7）

SElinux相关的日志存放在/var/log/audit/audit.log文件中

如果SElinux的监听服务没有开启，则日志机制会被rsyslog代理监听

SElinux相关的日志会被存放在/var/log/messages文件中













iptables netfilter 位于Linux内核中的包过滤功能体系，基于内核控制，实现防火墙的相关策略

iptables 位于/sbin/iptables，用来管理防火墙规则的工具

#### iptables默认包括5种规则链

* INPUT：处理入站数据包 
* OUTPUT：处理出站数据包 
* FORWARD：处理转发数据包 
* POSTROUTING链：在进行路由选择后处理数据包 
* PREROUTING链：在进行路由选择前处理数据包

#### iptables默认包括4个规则表

* raw表：确定是否对该数据包进行状态跟踪 （kernel 2.6） 
* mangle表：为数据包设置标记 
* nat表：修改数据包中的源、目标IP地址或端口（网络地址转换） 
* filter表：确定是否放行该数据包（过滤）

#### 表与链的关联：

* raw：prerouting链 output链 
* mangle：prerouting链 input链 forward链 output链 postrouting链 
* nat：prerouting链 output链 postrouting链 
* filter：input链 forward链 output链

语法构成

```
iptables [-t 表名] 选项 [链名] [条件] [-j 控制类型] 
[root@localhost ~]# iptables -t filter -I INPUT -p icmp -j REJECT 

注意事项 
不指定表名时，默认指filter表 
不指定链名时，默认指表内的所有链 
除非设置链的默认策略，否则必须指定匹配条件 
选项、链名、控制类型使用大写字母，其余均为小写
```

数据包的常见控制类型

* ACCEPT：允许通过 
* DROP：直接丢弃，不给出任何回应 
* REJECT：拒绝通过，必要时会给出提示 
* LOG：记录日志信息，然后传给下一条规则继续匹配



#### 启用iptables的方法

首先需要安装iptables的服务包 `yum install iptables-services` 

因为RHEL7默认使用防火墙工具为firewalld

所以需要**关闭**firewalld才能开启iptables

```
systemctl stop firewalld.service     //仅停止，其他程序可调用
systemctl status firewalld.service   //查看 firewalld 状态
systemctl mask firewalld.service     //彻底关闭，其他程序无法调用
```

关闭firewalld后打开iptables

```
systemctl start iptables.service       //开启 iptables
systemctl enable iptables.service      //开启启动
systemctl start ip6tables.service      //开启 ip6tables
systemctl enable ip6tables.service     //开启启动
systemctl status iptables.service      //查看 ip6tables 状态
```

iptables规则管理

添加新的规则

```
-A：在链的末尾追加一条规则 
-I：在链的开头（或指定序号）插入一条规则 查看规则列表 
-L：列出所有的规则条目 
-n：以数字形式显示地址、端口等信息 
-v：以更详细的方式显示规则信息 
--line-numbers：查看规则时，显示规则的序号 删除、清空规则 
-D：删除链内指定序号（或内容）的一条规则 
-F：清空所有的规则 
-X：清空缓存信息
```

firewalld 动态链接防火墙，所做的配置可以实时生效

firewalld有两种配置状态：

运行时和永久配置 当规则被增加时，默认添加的位置为运行状态 firewall-cmd --add-service=mysql  
查看当前运行状态下，firewalld开启的服务 firewall-cmd --list-services

如果想要一个规则永久生效，需要增加选项“--permanent” firewall-cmd --permanent --add-service=mysql

查看配置文件中，firewalld开启的服务 firewall-cmd --list-services --permanent

获取当前主机中firewalld的激活区域 firewall-cmd --get-active-zones

firewalld默认的激活区域为public，所以在配置防火墙规则时，如果不做“--zone=”区域的制定，默认的防火墙规则都会被写入到public区域中

```
根据网络地址，协议，端口同时对firewalld设置，可以使用富规则
通过指定协议增加防火墙的限制
firewall-cmd --add-rich-rule='rule family=ipv4 source address=172.25.0.11/24 service name=ssh accept'
通过指定端口增加防火墙的限制
firewall-cmd --add-rich-rule='rule family=ipv4 source address=172.25.0.11/24 port protocol=tcp port=22 accept'
```

防火墙规则设置时，如果没有加--permanent，表示当前生效，重启主机或重启firewalld，规则失效（firewall-cmd --reload） 防火墙规则设置时，如果增加了--permanent，规则不会立即生效，可以通过firewall-cmd --reload，重新加载配置文件，使设置的规则生效

实验： server： firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=10.0.0.2/24 forward-port port=443 protocol=tcp to-port=22' client： ssh -p 443 主机名（连接失败） server： firewall-cmd --reload client： ssh -p 443 主机名（可以输入服务器密码，连接成功）

```
图形化工具
firewall-config
```



