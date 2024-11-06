# 公网ip部署kubernetes

如果在kubeadm init初始化集群时使用的是master的内网IP，那么即使在kubeadm join时填写master公网ip也不能将节点成功注册进集群。

而如果直接使用公网IP来kubeadm init，那么由于云服务器拥有的网卡绑定的是内网IP，那么会集群初始化失败。

### 1. 创建虚拟网卡

```sh
# 所有主机都要创建虚拟网卡，并绑定对应的公网 ip
# 临时生效，重启会失效
ifconfig eth0:1 1.95.82.68
```

### 2. 系统环境准备

### 3. 安装containerd && 设置代理

### 4. 安装kubernetes组件

### 5. 初始化集群

- #### 修改kubelet启动参数文件（每台机器都需要执行） 

```sh
# 此文件安装kubeadm后就存在了
vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

# 注意，这步很重要，如果不做，节点仍然会使用内网IP注册进集群
# 在KUBELET_KUBECONFIG_ARGS末尾添加参数 --node-ip=公网IP
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS --node-ip=<公网IP>
```

- #### 创建集群初始化配置文件

```sh
root@ecs-f195:~/install# kubeadm config print init-defaults > init.yaml
root@ecs-f195:~/install# vim init.yaml 
root@ecs-f195:~/install# cat init.yaml 
```

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.95.82.68 #
  bindPort: 6443
nodeRegistration:
  criSocket: unix:///var/run/containerd/containerd.sock #
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
  certSANs: 
  - 1.95.82.68 #
  - 192.168.0.32 #
  - 10.96.0.1 
  - ecs-f195 #
controlPlaneEndpoint: 1.95.82.68:6443 #
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers #
kind: ClusterConfiguration
kubernetesVersion: 1.28.13 # 
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16 #
scheduler: {}
--- 
apiVersion: kubeproxy-config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs
```

```sh
kubeadm init --config=init.yaml
```

- #### 修改kube-apiserver参数

```sh
# 修改两个信息，添加--bind-address和修改--advertise-address
vim /etc/kubernetes/manifests/kube-apiserver.yaml


spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=xx.xx.xx.xx # 修改为公网IP
    - --bind-address=0.0.0.0  # 新增参数

```

