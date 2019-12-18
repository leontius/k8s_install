安装 HAProxy
``` shell
yum install -y socat  ipvsadm haproxy
```

各节点修改配置文件
``` shell
cp /etc/haproxy/haproxy.cfg /etc/haproxy/bak_haproxy.cfg
:> /etc/haproxy/haproxy.cfg
vi /etc/haproxy/haproxy.cfg
```

``` shell
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     40000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# kubernetes apiserver frontend which proxys to the backends
# --------------------------------------------------------------------
frontend kubernetes-apiserver
    mode                 tcp
    bind                 *:16443
    option               tcplog
    default_backend      kubernetes-apiserver
#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
#后端检测端口
backend kubernetes-apiserver
    mode        tcp
    balance     roundrobin
    server  zy-k8s-11  10.6.208.11:6443 check
    server  zy-k8s-12  10.6.208.12:6443 check
    server  zy-k8s-13  10.6.208.13:6443 check
#---------------------------------------------------------------------
# collection haproxy statistics message
#---------------------------------------------------------------------
listen stats
    bind               *:1080
    stats auth         admin:awesomePassword
    stats refresh      5s
    stats realm        HAProxy\ Statistics
    stats uri          /admin?stats

```

启动haproxy
``` shell
systemctl enable haproxy.service 
systemctl start haproxy.service 
systemctl status haproxy.service 
```

验证
``` shell
[root@zy-k8s-11 ~]# ss -lnt |grep -E "16443|1080"
LISTEN     0      128          *:1080                     *:*                  
LISTEN     0      128          *:16443                    *:* 
```