## 八、启动流程分析与服务控制

```
查看当前系统运行级别（模式）
systemctl get-default

设置当前系统运行级别（模式）
systemctl set-default multi-user.target/graphical.target

临时切换系统运行级别（模式）
systemctl isolate multi-user.target/graphical.target

CentOS 6 的常用方式
service 服务名称 控制类型

CentOS 7 的常用方式
systemctl 控制类型 服务名称 

常用的控制类型
   start 启动
   stop 停止
   restart 重新启动
   reload 重新加载
   status 查看服务状态

chkconfig工具（6版本）
不提供交互式、可视化界面
对单个服务管理效率更高

查看系统服务的启动状态
chkconfig --list [服务名称]

设置系统服务的启动状态
chkconfig [--level 级别列表] 服务名称 on|off

Systemctl工具（7版本）
不提供交互式、可视化界面
对单个服务或多个服务管理效率更高

查看系统服务的启动状态
systemctl list-units -t service -a
设置系统服务的启动状态
systemctl enable|disable 服务名称

```