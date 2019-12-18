
设置环境变量   
``` shell
# 只在 master 节点执行  # 替换 x.x.x.x 为 master 节点实际 IP（请使用内网 IP） 
# export 命令只在当前 shell 会话中有效，开启新的 shell 窗口后，如果要继续安装过程，请重新执行此处的 export 命令  
export MASTER_IP=x.x.x.x 
# 替换 apiserver.demo 为 您想要的 dnsName (不建议使用 master 的 hostname 作为 APISERVER_NAME)  
export APISERVER_NAME=apiserver.demo 
# Kubernetes 容器组所在的网段，该网段安装完成后，由 kubernetes 创建，事先并不存在于您的物理网络中  
export POD_SUBNET=10.100.0.1/20
echo "${MASTER_IP}    ${APISERVER_NAME}"  >>  /etc/hosts
```