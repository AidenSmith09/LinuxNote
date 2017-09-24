## 十三、selinux和防火墙 {#selinux}

selinux模式 类型: Enforcing, Permissive, or Disabled 配置文件 /etc/sysconfig/selinux 图形工具 yum install policycoreutils-gui system-config-selinux 运行时修改 getenforce and setenforce 0 | 1 Kernel arguments: selinux=0 | 1 or enforcing=0 | 1

```
修改文件的上下文
chcon -R -t public_content_t /srv/www
semanage fcontext -a -t httpd_sys_content_t '/srv/www(/.*)?'
还原（触发）系统默认规则
restorecon -Rv /srv/www

布尔决定是否在运行时应用某些规则
管理指令:
getsebool -a
setsebool [-P] httpd_enable_cgi off

查看端口的SElinux
semanage port -l

```

iptables netfilter 位于Linux内核中的包过滤功能体系，基于内核控制，实现防火墙的相关策略 iptables 位于/sbin/iptables，用来管理防火墙规则的工具

iptables默认包括5种规则链 INPUT：处理入站数据包 OUTPUT：处理出站数据包 FORWARD：处理转发数据包 POSTROUTING链：在进行路由选择后处理数据包 PREROUTING链：在进行路由选择前处理数据包

iptables默认包括4个规则表 raw表：确定是否对该数据包进行状态跟踪 （kernel 2.6） mangle表：为数据包设置标记 nat表：修改数据包中的源、目标IP地址或端口（网络地址转换） filter表：确定是否放行该数据包（过滤）

表与链的关联： raw：prerouting链 output链 mangle：prerouting链 input链 forward链 output链 postrouting链 nat：prerouting链 output链 postrouting链 filter：input链 forward链 output链

语法构成 iptables [-t 表名] 选项 [链名] [条件] [-j 控制类型] [root@localhost ~]# iptables -t filter -I INPUT -p icmp -j REJECT 几个注意事项 不指定表名时，默认指filter表 不指定链名时，默认指表内的所有链 除非设置链的默认策略，否则必须指定匹配条件 选项、链名、控制类型使用大写字母，其余均为小写

数据包的常见控制类型 ACCEPT：允许通过 DROP：直接丢弃，不给出任何回应 REJECT：拒绝通过，必要时会给出提示 LOG：记录日志信息，然后传给下一条规则继续匹配

启用iptables的方法 首先需要安装iptables的服务包 yum install iptables-services 因为RHEL7默认使用防火墙工具为firewalld，所以需要关闭firewalld才能开启iptables systemctl stop firewalld.service systemctl mask firewalld.service 关闭firewalld后打开iptables systemctl start iptables.service systemctl enable iptables.service systemctl start ip6tables.service systemctl enable ip6tables.service 查看iptables状态 systemctl status iptables.service 确认firewalld关闭 systemctl status firewalld.service

iptables规则管理 添加新的规则 -A：在链的末尾追加一条规则 -I：在链的开头（或指定序号）插入一条规则 查看规则列表 -L：列出所有的规则条目 -n：以数字形式显示地址、端口等信息 -v：以更详细的方式显示规则信息 --line-numbers：查看规则时，显示规则的序号 删除、清空规则 -D：删除链内指定序号（或内容）的一条规则 -F：清空所有的规则 -X：清空缓存信息

firewalld 动态链接防火墙，所做的配置可以实时生效，firewalld有两种配置状态：运行时和永久配置 当规则被增加时，默认添加的位置为运行状态 firewall-cmd --add-service=mysql 查看当前运行状态下，firewalld开启的服务 firewall-cmd --list-services 如果想要一个规则永久生效，需要增加选项“--permanent” firewall-cmd --permanent --add-service=mysql 查看配置文件中，firewalld开启的服务 firewall-cmd --list-services --permanent 获取当前主机中firewalld的激活区域 firewall-cmd --get-active-zones firewalld默认的激活区域为public，所以在配置防火墙规则时，如果不做“--zone=”区域的制定，默认的防火墙规则都会被写入到public区域中

```
根据网络地址，协议，端口同时对firewalld设置，可以使用富规则
通过指定协议增加防火墙的限制
firewall-cmd --add-rich-rule='rule family=ipv4 source address=172.25.0.11/24 service name=ssh accept'
通过指定端口增加防火墙的限制
 firewall-cmd --add-rich-rule='rule family=ipv4 source address=172.25.0.11/24 port protocol=tcp port=22 accept'

```

防火墙规则设置时，如果没有加--permanent，表示当前生效，重启主机或重启firewalld，规则失效（firewall-cmd --reload） 防火墙规则设置时，如果增加了--permanent，规则不会立即生效，可以通过firewall-cmd --reload，重新加载配置文件，使设置的规则生效

实验： server： firewall-cmd --permanent --add-rich-rule=&#039;rule family=ipv4 source address=10.0.0.2/24 forward-port port=443 protocol=tcp to-port=22&#039; client： ssh -p 443 主机名（连接失败） server： firewall-cmd --reload client： ssh -p 443 主机名（可以输入服务器密码，连接成功）

```
图形化工具
firewall-config

```