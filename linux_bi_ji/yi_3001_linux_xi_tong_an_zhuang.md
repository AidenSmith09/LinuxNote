## 一、Linux系统安装 {#linux}

查看Linux核心版本 uname -a／uname -r 查看Linux系统版本 cat /etc/redhat-release CentOS-release SuSE-release 关闭防火墙的运行状态 systemctl stop firewalld 开机禁用防火墙自运行 systemctl disable firewalld 查看防火墙当前的状态 systemctl status firewalld 关机 systemctl poweroff/poweroff init 0 shutdown halt 重启 systemctl reboot/reboot init 6 查看内存 free -m 查看CPU信息 cat /proc/cpuinfo