## 一、Linux系统基本命令 {#linux}

查看Linux核心版本

* uname -a  
* uname -r

查看Linux不同版本的系统版本

* cat /etc/redhat-release //红帽
* cat /etc/CentOS-release //CentOS
* cat /etc/SuSE-release   //SuSE

关闭防火墙的运行

```
systemctl stop firewalld
```

开机禁用防火墙自运行

```
systemctl disable firewalld
```

查看防火墙当前的状态

```
systemctl status firewalld
```

关机

* systemctl poweroff
* poweroff  
* init 0 shutdown halt

重启

* systemctl reboot
* reboot init 6

查看内存`free -m`

查看CPU信息`cat /proc/cpuinfo`

