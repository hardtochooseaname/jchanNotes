# k8s集群安装

------





### 一些必要的系统设置(centos7)

设置hostname（如果需要的话还可以修改/etc/hosts文件使得节点之间可以通过主机名访问）

```bash
# hostname需要在每个主机上分别设置 
hostnamectl set-hostname k8s-master
```

下面的操作每个主机上都一样，所以写成bash脚本方便配置

```bash
#!/bin/bash

# 设置yum源
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
# 安装软件包
yum install -y conntrack ntpdate ntp ipvsadm ipset jq iptables sysstat libseccmp wget net-tools git vim

# 设置防火墙为iptables并设置空规则
systemctl stop firewalld && systemctl disable firewalld
yum -y install iptables-services && systemctl start iptables && systemctl enable iptables && iptables -F && service iptables save

# 永久关闭交换分区（k8s安装时会检查swap是否关闭，避免pod被放到swap中运行，降低运行效率）
swapoff -a && sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
# 关闭SELINUX（selinux可能会影响一些正常的网络访问）
setenforce 0 && sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

#调整k8s内核参数
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness = 0
EOF
# 启用配置
sysctl -p /etc/sysctl.d/k8s.conf

### 设置时区为中国上海（如果系统在安装时没有选择时区）

# 设置系统日志
mkdir /var/log/journal
mkdir /etc/systemd/journald.conf.d
at > /etc/systemd/journald.conf.d/99-prophet.conf<<EOF
[Journal]
Storage=persistent
Compress=yes
SyncIntervalSec=5m
RateLimitInterval=30s 
RateLimitBurst=1000
SystemMaxUse=10G
SystemMaxFileSize=200M
MaxRetentionSec=2week
ForwardToSyslog=no 
EOF
systemctl restart systemd-journald

### 升级系统内核（centos7旧的系统内核会使k8s和docker运行不稳定）
# 从ELRepo仓库安装一个长期支持内核
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install -y kernel-lt
# 设置默认引导内核为新安装的内核（BIOS系统）
DEFAULT_KERNEL=$(awk -F\' '/^menuentry / {print $2; exit}' /etc/grub2.cfg) && grub2-set-default "$DEFAULT_KERNEL"

#可以重启机器了(使用新的内核，重启后可以uname -r查看内核版本)
# reboot...
# reboot...
# reboot...


# kube-proxy开启ipvs的前置设置(可以提高代理的访问效率)
modprobe br_netfilter
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack
```



### 设置网络代理

```bash	
#!/bin/bash

cat >> ~/.bashrc << EOF
# Setup proxy
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7890

# Open proxy
on() {
    export https_proxy=http://127.0.0.1:7890
    export http_proxy=http://127.0.0.1:7890
    export all_proxy=socks5://127.0.0.1:7890
    echo "HTTP/HTTPS Proxy on"
}

# Close proxy
off() {
    unset http_proxy
    unset https_proxy
    unset all_proxy
    echo "HTTP/HTTPS Proxy off"
}
EOF
```



### docker安装

```bash
!#/bin/bash

# 安装所需的软件包
yum install -y yum-utils

# 设置docker的阿里云软件源
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast

# 安装docker
yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动docker & 设置docker为开机自启
systemctl start docker && systemctl enable docker

# 验证是否成功安装
docker run hello-world

# 设置docker的cgroup隔离由systemd负责（与kubernete一致）
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
	"exec-opts":["native.cgroupdriver=systemd"],
	"log-driver":"json-file",
	"log-opts":{
		"max-size":"100m"
	}
}
EOF
mkdir -p /etc/systemd/system/docker.service.d
systemctl daemon-reload && systemctl restart docker
```



### cri-docker安装

release页：https://github.com/Mirantis/cri-dockerd/releases/tag/v0.3.14

安装完成后，启动cri-docker服务时还需要同时启动cri-docker.socket，这是用于监听到cri-docker的连接的

```bash
# 下载并安装cri-docker
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.14/cri-dockerd-0.3.14-3.el7.x86_64.rpm
sudo rpm -ivh cri-dockerd-0.3.14-3.el7.x86_64.rpm

# 配置 cri-docker 以使用docker
sudo mkdir -p /etc/systemd/system/cri-docker.service.d
cat <<EOF | sudo tee /etc/systemd/system/cri-docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/cri-dockerd --container-runtime-endpoint unix:///var/run/docker.sock
EOF

# 启动 cri-docker 服务
systemctl daemon-reload && systemctl enable --now cri-docker cri-docker.socket
```

查看cri-docker的通信套接字（）

```bash
systemctl status cri-docker.socket

[root@k8s-master01 ~]# systemctl status cri-docker.socket
● cri-docker.socket - CRI Docker Socket for the API
   Loaded: loaded (/usr/lib/systemd/system/cri-docker.socket; enabled; vendor preset: disabled)
   Active: active (running) since 一 2024-06-03 12:31:50 CST; 14min ago
   Listen: /run/cri-dockerd.sock (Stream)

6月 03 12:31:50 k8s-master01 systemd[1]: Closed CRI Docker Socket for the API.
6月 03 12:31:50 k8s-master01 systemd[1]: Stopping CRI Docker Socket for the API.
6月 03 12:31:50 k8s-master01 systemd[1]: Starting CRI Docker Socket for the API.
6月 03 12:31:50 k8s-master01 systemd[1]: Listening on CRI Docker Socket for the API.
```

修改cri-docker的默认pause镜像

- 找到/usr/lib/systemd/system/cri-docker.service

- 在ExecStart最后添加--pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9

  ExecStart=/usr/bin/cri-dockerd --container-runtime-endpoint fd:// --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9

- 重启服务



### kubeadm安装

```bash
#!/bin/bash

# 设置kubeadm的阿里源
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

# 安装三件套
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# 设置kubelet服务开机自启，并立刻启用kubelet服务（--now）
systemctl enable --now kubelet

# 配置kubelet使用cri-docker(等同于在kubeadm init时--cri-socket=unix:///run/cri-dockerd.sock)
mkdir -p /etc/systemd/system/kubelet.service.d
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
[Service]
Environment="KUBELET_CONFIG_ARGS=--container-runtime-endpoint=unix:///var/run/cri-dockerd.sock"
EOF
systemctl daemon-reload && systemctl restart kubelet
```





## master节点初始化（control plane组件）

下面要master的ip和kubernete的版本要根据实际情况修改

```bash
kubeadm init \
  --kubernetes-version v1.28.2 \
  --apiserver-advertise-address=192.168.18.10 \
  --image-repository registry.aliyuncs.com/google_containers \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --node-name=k8s-master01 \
  --cri-socket=unix:///var/run/containerd/containerd.sock(unix:///var/run/cri-dockerd.sock)

# –apiserver-advertise-address # master的ip地址
# –image-repository # 由于默认拉取镜像地址k8s.gcr.io（需要代理），这里指定阿里云镜像仓库地址速度更快
# –kubernetes-version #K8s版本，安装的什么kubeadm版本这里填什么版本
# –service-cidr #集群内部虚拟网络，Pod统一访问入口，可以不用更改，直接用上面的参数
# –pod-network-cidr #Pod网络的子网网段，与flannel一致
```



## worker节点加入集群

master在`kubeadm init`成功执行后，按照打印的提示信息让其他worker加入集群

![image-20240604092611083](..\images\image-20240604092611083.png)

### 报错 & 解决

![image-20240604092849519](..\images\image-20240604092849519.png)

发现containerd有关CRI验证方面出了问题：删除containerd的配置文件并重启

```bash
rm -f /etc/containerd/config.toml
systemctl daemon-reload && systemctl restart containerd
```

然后再join就成功了

![image-20240604093118729](..\images\image-20240604093118729.png)



## P.S.

####  脚本中用`cat`写配置文件 

cat命令下面这种用法可以方便地在bash脚本中创建或追加配置文件

```bash
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
```

1. 第一个重定向符`>`：把cat的输出重定向到文件（**如果想要在文件中追加内容则用`>>`**）
2. `<<EOF ... EOF`：这是一种将多行字符串传递给命令的方法（Here Document）

