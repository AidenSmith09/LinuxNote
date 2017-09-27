# Web Server {#toc_0}

可通过yum源码包安装该软件  
`yum install httpd`\(yum groupinstall 'web server'\) 组包安装时，会将ssl安全web站点，apache官方文档等安装到系统中，方便配置apache，也可以只安装httpd

主配置文件为  
`/etc/httpd/conf/httpd.conf`  
默认主页文件存放目录为  
`/var/www/html/`  
日志文件存放路径为  
`/var/log/httpd/`  
监听端口为  
`80/tcp (http), 443/tcp (https)`

apache站点权限控制：  
首先，2.4版本的apache配置与2.2版本不同

2.4版本的权限控制，需要配置`requireall`选项

如果所有人都允许，只禁止`10.0.0.164`，则设置为

* Require all granted
* Require not ip 10.0.0.16（not ip表示只有这台主机不能访问，not ip为保留字）

如果禁止所有人登陆，则设置为

\* Require all denied \(denied表示全拒绝\)

如果只允许10.0.0.123的访问，则设置为

* Require all granted
* require ip 10.0.0.123

**注：每修改一次主配置文件，都需要重启服务httpd**

### Linux 默认站点 {#toc_1}

Linux测试默认站点，需要先新建主站点文件index.html  
echo This is WebPage &gt; /var/www/html/index.html

apache用户登陆权限控制

1.新建apache用户数据库，第一次创建文件需要加‘-c’选项，创建出文件，以后加用户可以省略  
htpasswd -c /etc/httpd/conf/.htpasswd admin

2.增加本地用户验证的权限控制，将访问目录的开关由none修改为AuthConfig

```
vim /etc/httpd/conf/httpd.conf
<Directory “/var/www/html”>
    AllowOverride none/AuthConfig
</Directory>
```

在需要被控制的网站根目录中，创建隐藏文件‘.htaccess’,同时增加用户验证模块

```
vim /var/www/html/.htaccess
    AuthName “ACL Directory”    用户登陆时的提示框信息
    AuthType Basic                  用户验证模式，默认设置为basic，也支持MD5等
    AuthUserFile /etc/httpd/conf/.htpasswd  用户验证数据库的本地存放路径
    require valid-user          valid-user    表示所有数据库的用户都可以通过验证
    (也可以设置为require user admin，表示只有admin可以通过验证)
```

1. systemctl restart httpd win7
   [www.simpleware.com](http://www.simpleware.com)
   输入admin和设置密码

### 基于名称的虚拟主机 （虚拟主机开启，默认站点不生效） {#toc_2}

1. `vim /etc/httpd/conf/httpd.conf`在文件末尾添加

   ```
   <virtualhost *:80>
       servername www.aidenware.com
       documentroot /var/www/html/ware
   </virtualhost>
   <virtualhost *:80>
       servername www.aidenxue.com
       documentroot /var/www/html/xue
   </virtualhost>
   ```

2. 新建文件夹

   mkdir -p /var/www/html/{ware,xue}

3. 创建虚拟主机的默认站点  
   echo aidenxue &gt; /var/www/html/xue/index.html  
   echo aidenware &gt; /var/www/html/ware/index.html

4. 重启服务  
   systemctl restart httpd

5. 修改DNS解析 //基于 DNS 服务开启

   ```
   vim /etc/named.conf在文件中增加
   zone "aidenxue.com" IN {
       type master;
       file "aiden.com.zone";
       allow-transfer { 10.0.0.123; };
   };
   ```

6. 添加解析文件  
   cd /var/named/  
   cp -p aidenware.com.zone aidenxue.com.zone  
   将文件中的所有关键词‘aidenware’替换为‘aidenxue’

7. 重启named服务  
   systemctl restart named

8. windows测试  
   IE中分别访问：www.simpleware.com和www.simplexue.com

### 基于端口的虚拟主机 {#toc_3}

vim /etc/httpd/conf/httpd.conf在文件中修改自己的IP和端口

   servername[www.aidenware.com](http://www.simpleware.com)  
   documentRoot /var/www/html/ware

   servername[www.aidenxue.com](http://www.simplexue.com)  
   documentRoot /var/www/html/xue

同时增加一行信息`Listen 8080`保存后退出，并重启httpd服务

1. 防火墙为非标准端口放行  
   firewall-cmd --add-port=8080/tcp

2. windows测试  
   IE中分别访问：10.0.0.1 和 10.0.0.1:8080



