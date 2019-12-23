# Linux环境设置

host设置

``` shell
10.6.208.11 zy-k8s-11
10.6.208.12 zy-k8s-12
10.6.208.13 zy-k8s-13
```

ntp设置

``` shell
yum install -y ntp wget
echo "server ntp1.aliyun.com" >> /etc/ntp.conf
```

修改内核参数

``` shell
cat <<EOF > /etc/sysctl.d/k8s.conf 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
EOF
```

应用内核参数

```
sysctl --system
```

关闭swap空间

``` shell
swapoff -a
yes | cp /etc/fstab /etc/fstab_bak
cat /etc/fstab_bak |grep -v swap > /etc/fstab
```

关闭SELinux

``` shell
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

安装完高版本Docker(1.13以后版本)切换防火墙默认转发策略为ACCEPT
``` shell
iptables -P FORWARD ACCEPT
```

至此设置完成。