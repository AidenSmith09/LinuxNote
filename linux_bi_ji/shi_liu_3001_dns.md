## 十六、DNS {#dns}

安装yum install bind 其中bind（Berkeley Internet Name Daemon）

* 主要配置文件
* 服务脚本 `/usr/lib/systemd/system/named.service` 端口 53 
* 主配置文件 `/etc/named.conf` 

主配置文件

```
直接在末尾追加即可

zone "Aiden.com" IN {
        type master;
        file "Aiden.com.zone";          //正向解析文件名
        allow-transfer {10.0.0.1; };    //DNS服务器地址
};

zone "0.0.10.in-addr.arpa" IN {
        type master;
        file "10.0.0.zone";                //反向解析文件名
        allow-transfer {10.0.0.1; };       //DNS 服务器地址
};
```

解析数据文件保存在 /var/named/

`Aiden.zone`  正向解析文件

```
$TTL 86400 
@ IN SOA Aiden.com. root.Aiden.com. ( 
                                        2017071401 ; serial    //时间一定要相同    
                                        1H ; refresh 
                                        10M ; retry 
                                        1W ; expire 
                                        1D ) ; minimum 
IN NS dns.Aiden.com. 
www IN A 10.0.0.234
```

`10.0.0.zone`   反向解析文件

```
$TTL 86400 @ IN SOA Aiden.com. root.Aiden.com. ( 
                                                2017071401 ; serial 
                                                1H ; refresh 
                                                10M ; retry 
                                                1W ; expire 
                                                1D ) ; minimum 
IN NS dns.Aiden.com. 
234 IN PTR www.Aiden.com.
```



