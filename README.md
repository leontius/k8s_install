# k8s_install

[centos安装keepalived](./centos/INSTALL_KEEPALIVED.md)

[centos安装HAProxy](./centos/INSTALL_HA.md)

kubeadm init
``` shell
kubeadm init \
--apiserver-advertise-address=10.6.208.11 \
--apiserver-bind-port=6443 \
--control-plane-endpoint=10.6.208.100:16443 \
--kubernetes-version v1.16.1 \
--image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
--service-cidr=10.96.0.0/16
```
kubeadm 输出信息
``` shell
You can now join any number of control-plane nodes by copying certificate authorities 
and service account keys on each node and then running the following as root:

  kubeadm join 10.6.208.100:16443 --token ncu01v.gv99eo6zwju93cqa \
    --discovery-token-ca-cert-hash sha256:79e9040911504bfb5a6c8fb8d64c2d06bff1c53983095688a97af49cdab21211 \
    --control-plane       

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.6.208.100:16443 --token ncu01v.gv99eo6zwju93cqa \
    --discovery-token-ca-cert-hash sha256:79e9040911504bfb5a6c8fb8d64c2d06bff1c53983095688a97af49cdab21211 
```

复制集群config，配置kubectl工具
``` shell
mkdir -p $HOME/.kube
/bin/cp -arf   /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

查看集群信息
``` shell
[root@zy-k8s-11 ~]# kubectl cluster-info
Kubernetes master is running at https://10.6.208.100:16443
KubeDNS is running at https://10.6.208.100:16443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
[root@zy-k8s-11 ~]# kubectl get nodes
NAME    STATUS     ROLES    AGE    VERSION
zy-k8s-11   NotReady   master   3m1s   v1.16.0
[root@zy-k8s-11 ~]# watch kubectl get pod -n kube-system -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
coredns-67c766df46-qsrxr            0/1     Pending   0          5m11s   <none>        <none>      <none>           <none>
coredns-67c766df46-xlzsq            0/1     Pending   0          5m11s   <none>        <none>      <none>           <none>
etcd-zy-k8s-11                      1/1     Running   0          4m9s    10.6.208.11   zy-k8s-11   <none>           <none>
kube-apiserver-zy-k8s-11            1/1     Running   0          4m      10.6.208.11   zy-k8s-11   <none>           <none>
kube-controller-manager-zy-k8s-11   1/1     Running   0          4m2s    10.6.208.11   zy-k8s-11   <none>           <none>
kube-proxy-dq6hx                    1/1     Running   0          5m10s   10.6.208.11   zy-k8s-11   <none>           <none>
kube-scheduler-zy-k8s-11            1/1     Running   0          4m3s    10.6.208.11   zy-k8s-11   <none>           <none>
```

设置calico网络  
master节点开启调度
``` shell
kubectl taint nodes --all node-role.kubernetes.io/master-
```
[calico官方文档](https://docs.projectcalico.org/v3.9/getting-started/kubernetes/)  
应用yaml
``` shell
kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
watch kubectl get pods --all-namespaces
```

calico容器初始化完成时即可：
``` shell
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-6bbf58546b-lw4dc   1/1     Running   0          2m
kube-system   calico-node-xnfnv                          1/1     Running   0          2m
```

处理多master  
传输证书文件到其他master
``` shell
ssh zy-k8s-12 rm -rf /etc/kubernetes
ssh zy-k8s-12 mkdir -p /etc/kubernetes/pki/etcd
scp /etc/kubernetes/admin.conf zy-k8s-12:/etc/kubernetes/
scp /etc/kubernetes/pki/{ca.*,sa.*,front-proxy-ca.*} zy-k8s-12:/etc/kubernetes/pki/
scp /etc/kubernetes/pki/etcd/ca.* zy-k8s-12:/etc/kubernetes/pki/etcd/


ssh zy-k8s-13 rm -rf /etc/kubernetes
ssh zy-k8s-13 mkdir -p /etc/kubernetes/pki/etcd
scp /etc/kubernetes/admin.conf zy-k8s-13:/etc/kubernetes/
scp /etc/kubernetes/pki/{ca.*,sa.*,front-proxy-ca.*} zy-k8s-13:/etc/kubernetes/pki/
scp /etc/kubernetes/pki/etcd/ca.* zy-k8s-13:/etc/kubernetes/pki/etcd/
```

其余master节点加入集群：
``` shell
  kubeadm join 10.6.208.100:16443 --token ncu01v.gv99eo6zwju93cqa \
    --discovery-token-ca-cert-hash sha256:79e9040911504bfb5a6c8fb8d64c2d06bff1c53983095688a97af49cdab21211 \
    --control-plane     
```

再在master zy-k8s-11上开启调度
``` shell
kubectl taint nodes --all node-role.kubernetes.io/master-
```

在其他master上配置下kubectl   

加入worker节点：  
**：token有效期是有限的，如果旧的token过期，可以在master节点上使用kubeadm token create --print-join-command重新创建一条token。 
``` shell
kubeadm join 10.6.208.100:16443 --token ncu01v.gv99eo6zwju93cqa \
    --discovery-token-ca-cert-hash sha256:79e9040911504bfb5a6c8fb8d64c2d06bff1c53983095688a97af49cdab21211 
```

确认所有节点Ready
``` shell
NAME        STATUS   ROLES    AGE     VERSION
zy-k8s-11   Ready    master   91m     v1.16.0
zy-k8s-12   Ready    master   56m     v1.16.0
zy-k8s-13   Ready    master   49m     v1.16.0
zy-k8s-14   Ready    <none>   5m11s   v1.16.0
zy-k8s-15   Ready    <none>   13m     v1.16.0
```

安装 Ingress Controller
``` shell
kubectl apply -f https://kuboard.cn/install-script/v1.16.0/nginx-ingress.yaml
```