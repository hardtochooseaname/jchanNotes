

## 云端

```bash
# 拷贝并编辑配置文件, 安装 cloudcore
cp conf/config.cloud.example.sh config.sh
vim config.sh
KE_VERSION=1.9.1 ./install-ke-cloudcore.sh
```

```sh
# 云端配置
export AIEDGE_MASTER_IP=192.168.20.150
export AIEDGE_WORKER_IP=192.168.20.151
export AIEDGE_CLOUDCORE_IP=${AIEDGE_MASTER_IP}
export AIEDGE_NODEPORT_IP=${AIEDGE_WORKER_IP}
export AIEDGE_IMAGE_REGISTRY=registry.aiedge.lomot.top    # registry.aiedge.ndsl-lab.cn
export IMAGE_REGISTRY=${AIEDGE_IMAGE_REGISTRY}
```

### 环境准备脚本

```sh
#!/bin/bash

# 如果没有设置KE_VERSION则将其设置为1.8.2（相当于设置KE_VERSION的默认值）
KE_VERSION=${KE_VERSION:-1.8.2} 
# kubeedge相关安装包的位置
KE_PATH=./vendor/kubeedge/${KE_VERSION}
# 如果没有设置CPU_ARCH，则将其默认设置为amd64
CPU_ARCH=${CPU_ARCH:-amd64}

# mosquitto是一个消息代理软件，实现MQTT协议
apt install -y mosquitto docker.io

# 把keadm安装包解压到/usr/bin，并给keadm添加执行权限
########## 注意的是，1.9.1的安装包解压之后只有一个keadm，但新版keadm安装包的目录结构不一样
tar -C /usr/bin -zxf ${KE_PATH}/keadm-v${KE_VERSION}-linux-${CPU_ARCH}.tar.gz
chmod +x /usr/bin/keadm 

# 把kubeedge框架的安装包移动到/etc/kubeedge
mkdir -p /etc/kubeedge
cp ${KE_PATH}/kubeedge-v${KE_VERSION}-linux-${CPU_ARCH}.tar.gz /etc/kubeedge/
```

