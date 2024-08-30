# AIEdge



### 1. 基础框架模块

内容：K8s + KubeEdge框架，也就是平台安装脚本

路径：`source/platform-install/install-*`

部署方式：脚本

#### TO-DO

- 尝试手动部署并测试kubeedge框架

### 2. 边缘子网模块

内容：边缘 CNI + Kubeproxy，用于实现边缘集群，使得边缘Pod之间可以互通

路径：`source/platform-install/vendor/flannel`

部署方式：脚本

#### TO-DO

- 为什么要实现边缘集群
- 如何安装CNI与flannel实现Pod网络互通

### 3.  认证鉴权模块

内容：鉴权中心 Auth，用于实现用户的认证与授权，向用户发放token

路径：`source/aiedge-auth-go`

部署方式：k8s yaml配置文件

#### TO-DO

- 如何部署
- 这里的认证鉴权用于何处
- 阅读源码，弄清楚认证鉴权的工作原理

#### Question：

- `/k8s/aiedge-auth-deploy.yaml`里面的configmap的键`config.json`，存的json数据用在哪的

### 4. 边缘名字服务模块

内容：coredns，实现在边缘的域名服务

路径：`source/coredns`

部署方式：k8s yaml配置文件

#### TO-DO

- 如何部署
- 深入了解coredns的作用与原理
- 搞清coredns在边缘子网的作用

### 5. 通用数据通道模块

内容：通用数据通道（CEC）是用于云边通用数据通信的消息队列组件，提供了一个稳定、可靠的通用数据通信机制。也就是用于发送控制指令的数据通道。

路径：`source/cec-server`

部署方式：k8s yaml配置文件

#### TO-DO

- 如何部署
- 该模块的作用与原理

### 6. 设备抽象模块

内容：Mapper组件，用于实现边缘设备的接入

路径：`source/rtmp-mapper`

部署方式：k8s yaml配置文件

#### TO-DO

- 如何部署
- 阅读源码，弄清楚源码逻辑

### 7. 流数据通道模块

内容：用于传输摄像头视频流的组件

路径：  `source/stream-transfer`   `source/aiedge-stream-scheduler  `

部署方式：k8s yaml配置文件

#### TO-DO

- 如何部署
- 阅读源码，弄清楚源码逻辑

### 8. AIEdge控制中心模块

内容：提供给用户的具有良好交互性的web应用，用于实现对集群的控制和管理

路径：`source/acc-frontend`, `source/control-center`

部署方式：k8s yaml配置文件

#### TO-DO

- 如何部署
- 阅读control-center的代码，也就是web控制中心的后端代码

