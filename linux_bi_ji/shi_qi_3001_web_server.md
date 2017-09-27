## 十七、web server {#web-server}

可通过yum源码包安装该软件

yum install httpd \(yum groupinstall 'web server'\)

主要配置文件

* 启动脚本为 `/usr/lib/systemd/system/httpd.service` \(/etc/init.d/httpd\)
* 主配置文件为 `/etc/httpd/conf/httpd.conf`
* 默认主页文件存放目录为 `/var/www/html/`
* 日志文件存放路径为 `/var/log/httpd/`
* 监听端口为 80/tcp \(http\)、 443/tcp \(https\)

主配置文件中重要选项

```
ServerRoot “/etc/httpd“     服务主目录
Listen 80                   监听端口
Include conf.d/*.conf       扩展配置文件
User apache                 启动用户
Group apache                启动组
ServerAdmin root@localhost  管理员邮箱
ServerName www.example.com:80        域名主机名
DocumentRoot “/var/www/html“         默认主页存放目录
DirectoryIndex index.html            索引文件
ErrorLog logs/error_log              错误日志
CustomLog logs/access_log combined   访问日志
```

主机访问权限控制

仅允许IP：192.168.0.1 访问

* Require all granted
* Require ip 192.168.0.1

仅禁止IP：192.168.0.1访问

* Require all granted
* Require not ip 192.168.0.1
* Require not ip 172.16 192.168.100

用户验证权限控制

1. 创建认证用户的数据库 htpasswd -c /etc/httpd/conf/.htpasswd 用户名
2. 启用用户授权配置
   * AllowOverride AuthConfig
3. 创建授权文件.htaccess文件

   * AuthName “ACL Directory”     显示信息
   * AuthType Basic                       认证类型
   * AuthUserFile /etc/httpd/conf/.htpasswd 账号文件
   * require valid-user 有效用户可登录

4. 重启服务后验证

基于域名的虚拟主机配置  
servername [www.aidenedu.com](http://www.simpleedu.com)  
documentroot /var/www/html/edu

servername [www.aidenware.com](http://www.simpleware.com)  
documentroot /var/www/html/ware  
需要将DNS的解析增加为两个域名，编辑named.conf

同时需要在/var/named/创建另一个域名的解析 测试端根据不同的域名访问站点

基于相同IP不同端口的虚拟主机配置** **  
servername [www.aideneedu.com](http://www.simpleedu.com)  
documentroot /var/www/html/edu

servername [www.aideneware.com](http://www.simpleware.com)  
documentroot /var/www/html/ware  
在主配置文件中还需要开启Listen端口监听

如果是非标准端口，还需要将selinux的标签增加到指定端口

比如8090:semanage port -a -t http\_port\_t -p tcp 8090

同时需要对非标准端口配置防火墙规则

firewall-cmd --permanent --add-port=8080/tcp 客户端根据相同的IP，不同的端口做最终的访问测试

