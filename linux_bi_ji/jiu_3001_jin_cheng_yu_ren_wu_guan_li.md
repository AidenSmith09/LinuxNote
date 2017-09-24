## 九、进程与任务管理

### 进程查看 {#-0}

#### ps {#ps}

```
ps aux 在静态页面查看系统进程信息
PID表示进程的运行ID表示，主要用来管理进程
选项：
S：当前进程处于sleeping状态
s：当前进程有多个子进程
l：当前进程会衍生并发多个进程
Z：僵尸进程
R：运行中的进程
+：在前台运行的进程
N：优先级比较低的进程
<：优先级比较高的进程

NICE优先级
-20～19，普通用户只能降低程序优先级，超级用户可以提升进程优先级
普通用户设置的范围：0-19
超级用户设置的范围：-20～19
nice  renice

```

#### top {#top}

```
top：动态查看进程信息（默认每3秒刷新一次，可以按q退出）
        M：按M表示按照内存使用量从大到小排序
        P：按P表示按照CPU使用量从大到小排序

```

#### pgrep {#pgrep}

```
pgrep -u named named
-u表示指定进程的发起用户（管理用户），第一个named针对的是passwd文件中的用户名，第二个named针对的是系统中的进程名称
pgrep -u named named=ps aux | grep named查看进程PID

pstree
以目录树结构查看进程之间的关系

```

#### 进程管理 {#-1}

```
Ctrl+z         //前台进程调入后台
Ctrl+c         //结束进程
jobs           //查看后台进程
fg #           //将后台第#个进程在前台运行
bg #           //将后台第#个进程在后台运行
&              //直接将进程放在后台运行
kill、killall  //结束进程
kill PID       //默认发送的信号为进程自尽
kill -9 PID    //发送的信号为强制结束.当-9也无法结束时，只能通过关机或重启解决进程
killall 进程名称   //通常用于结束多个相同名称的进程

```

#### 计划任务 {#-2}

```
at命令：一次性计划任务
服务脚本：/etc/init.d/atd
         /usr/lib/systemd/system/atd
设置格式
at [HH:MM] [yyyy-mm-dd]
at>ctrl+d 结束编辑

查询与删减
atq
atrm
at –c #（#表示at -l或atq查看到的数字） 查看指定计划任务内容

crontab命令
周期性计划任务，按照预先设置的时间周期执行用户制定命令的操作
服务脚本名称：/etc/init.d/crond
            /usr/lib/systemd/system/crond
主要配置文件
全局配置文件 /etc/crontab
系统默认的配置文件位于 /etc/cron.*/
用户定义的配置文件位于 /var/spool/cron/用户名

crontab -e [-u 用户名] 编辑计划任务
crontab -l [-u 用户名] 查看计划任务
crontab -r [-u 用户名] 删除计划任务

如果编辑时没有指定用户名表示为当前用户设定计划任务
cron.deny黑名单文件
cron.allow白名单文件
写用户名称时，只能以一个用户占用一行的方式，多个用户不可以用空格或‘，’隔开

```