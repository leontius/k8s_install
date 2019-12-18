centos安装keepalived

``` shell
yum install -y socat keepalived ipvsadm 
systemctl enable keepalived
```

各节点配置 keepalived
``` shell
cp /etc/keepalived/keepalived.conf /etc/keepalived/bak_keepalived.conf
:> /etc/keepalived/keepalived.conf
vim /etc/keepalived/keepalived.conf
```

``` shell
! Configuration File for keepalived

global_defs {
        router_id LVS_DEVEL
}

vrrp_script check_haproxy {
        script "killall -0 haproxy"
        interval 3
        weight -2
        fall 10
        rise 2
}

vrrp_instance VI_1 {
        state MASTER    #从节点是BACKUP
        interface ens192    #网卡名称，看准了
        virtual_router_id 51 #虚拟路由ID，不能变
        priority 250    #从节点依次减小这个值
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 35f18af7190d51c9f7f78f37300a0cbd
        }
        virtual_ipaddress {
            10.6.208.100/24 dev ens192
        }
        track_script {
           check_haproxy
        }
}
```

各节点启动keepalived
``` shell
systemctl enable keepalived.service 
systemctl start keepalived.service 
systemctl status keepalived.service
```

查看
``` shell
[root@zy-k8s-11 ~]#  ip address show ens192
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:8b:d5:23 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.5/24 brd 172.16.0.255 scope global noprefixroute ens192
       valid_lft forever preferred_lft forever
    inet 172.16.0.20/24 scope global secondary ens192
       valid_lft forever preferred_lft forever
```