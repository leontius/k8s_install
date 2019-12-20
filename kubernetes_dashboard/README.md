# kubernetes-dashbaord 安装说明

执行如下命令，以安装 Kubernetes Dashboard

``` bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

或执行本地yaml文件

``` shell
kubectl apply -f v2.0.0-beta5.yaml
```



Kubernetes Dashboard 当前，只支持使用 Bearer Token登录。 由于 Kubernetes Dashboard 默认部署时，只配置了最低权限的 RBAC。因此，我们要创建一个名为 `admin-user` 的 ServiceAccount，再创建一个 ClusterRolebinding，将其绑定到 Kubernetes 集群中默认初始化的 `cluster-admin` 这个 ClusterRole。

> 更多关于权限管理的信息，请参考 [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

``` yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

获取Bearer Token

``` shell
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

以下命令作为反向代理的模式运行kubectl。 它处理对apiserver 的定位并进行认证。

``` shell
kubectl proxy
```

访问URL：

``` shell
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

将之前获取的token信息填入登陆界面即可。

