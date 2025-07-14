# containerd代理设置

使用阿里云新加坡服务器作为代理服务器

### 0. 将本机ssh密钥加入到代理服务器

```sh
# 本机
ssh-keygen
cat ~/.ssh/id_rsa.pub

# 代理服务器
vim ~/.ssh/authorized_keys
```

### 1. 建立到代理服务器的ssh隧道

```sh
ssh -v -TND 9876 root@8.222.184.209
```

#### 测试代理是否成功：

```sh
curl -x socks5h://localhost:9876 -I https://www.google.com
```

### 2. 设置containerd的镜像代理

```sh
# 下面的操作需要以root权限进行
sudo su

mkdir -p /etc/systemd/system/containerd.service.d
touch /etc/systemd/system/containerd.service.d/http-proxy.conf
tee /etc/systemd/system/containerd.service.d/http-proxy.conf << EOF
[Service]
Environment="HTTP_PROXY=socks5://localhost:9876"
Environment="HTTPS_PROXY=socks5://localhost:9876" 
Environment="NO_PROXY=localhost,127.0.0.1,containerd,10.96.0.0/12,192.168.0.0/24"
EOF
systemctl daemon-reload
systemctl restart containerd
```

#### 注意：`NO_PROXY`的设置很重要！！！

这是选择在访问这些集群内ip、本机ip时不使用代理。

如果没有设置这一行，或者没有设置正确，可能会导致pod创建时一直处于ContainerCreating状态。原因就是在使用集群内地址访问API Server时通过代理访问，导致到API Server的请求转发到了代理服务器，而代理服务访问不了集群中的API Server。

### 3. 测试能否拉取镜像

```sh
crictl pull alpine
```



## 把隧道代理设置为后台服务

可以将你的 SSH 隧道设置为一个系统服务，以便它在开机时自动启动并在断开连接时重新连接。以下是一个简单的步骤指南：

### 1. 创建服务文件

首先，在 `/etc/systemd/system/` 目录下创建一个服务文件，例如 `ssh-tunnel.service`：

```bash
sudo vim /etc/systemd/system/aliyun-proxy.service
```

### 2. 添加服务配置

在打开的文件中，添加以下内容：

```ini
[Unit]
Description=SSH Tunnel to remote server
After=network.target

[Service]
User=root
ExecStart=/usr/bin/ssh -N -D 9876 root@8.222.184.209
Restart=always
RestartSec=10
Environment=NO_PROXY="127.0.0.1,localhost,10.96.0.0/12,192.168.0.0/24"

[Install]
WantedBy=multi-user.target
```

#### 需要修改的部分：

- User：注意这里的用户，要填代理服务能够识别的用户，也就是前面创建sshkey的用户
- NO_PROXY：最后两个是集群的内部地址和内网地址

### 3. 重新加载 systemd

添加完服务文件后，运行以下命令重新加载 systemd 管理器配置：

```bash
sudo systemctl daemon-reload
```

### 4. 启用并启动服务

启用并启动 SSH 隧道服务：

```bash
sudo systemctl enable aliyun-proxy
sudo systemctl start aliyun-proxy
```

### 5. 检查服务状态

你可以使用以下命令检查服务的状态：

```bash
sudo systemctl status aliyun-proxy
```

### 6. 日志查看

要查看服务的日志，可以使用以下命令：

```bash
journalctl -u aliyun-proxy -f
```

这样设置后，SSH 隧道将会在开机时自动启动，并且在网络波动导致连接断开时会尝试重新连接。确保 `NO_PROXY` 环境变量根据你的需求进行适当设置。
