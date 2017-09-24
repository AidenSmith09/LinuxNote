## 十六、DNS {#dns}

master: 安装yum install bind 其中bind（Berkeley Internet Name Daemon） 主要配置文件为 服务脚本 /usr/lib/systemd/system/named.service 端口 53 主配置文件 /etc/named.conf 解析数据文件保存在 /var/named/ 中

```
zone "simpleedu.com" IN {
        type master;
        file "simpleedu.com.zone";
        allow-transfer {10.0.0.123; };
};

zone "0.0.10.in-addr.arpa" IN {
        type master;
        file "10.0.0.zone";
        allow-transfer {10.0.0.123; };
};

```

simpleedu.com.zone $TTL 86400 @ IN SOA simpleedu.com. root.simpleedu.com. ( 2017071401 ; serial 1H ; refresh 10M ; retry 1W ; expire 1D ) ; minimum IN NS dns2.simpleedu.com. www IN A 10.0.0.234 dns IN A 10.0.0.1 dns2 IN A 10.0.0.123

10.0.0.zone $TTL 86400 @ IN SOA simpleedu.com. root.simpleedu.com. ( 2017071401 ; serial 1H ; refresh 10M ; retry 1W ; expire 1D ) ; minimum IN NS dns2.simpleedu.com. 234 IN PTR [www.simpleedu.com](http://www.simpleedu.com). 1 IN PTR dns.simpleedu.com. 123 IN PTR dns2.simpleedu.com.

slave: yum install bind 主配置文件 /etc/named.conf 解析数据文件保存在 /var/named/ 中

```
zone "simpleedu.com" IN {
        type slave;
        file "slaves/simpleedu.com.zone";
        masters {10.0.0.1; };
};

zone "0.0.10.in-addr.arpa" IN {
        type slave;
        file "slaves/10.0.0.zone";
        masters {10.0.0.1; };
};

```

为了保证解析主机的主机名显示，可以通过hostnamectl set-hostname设置主机名