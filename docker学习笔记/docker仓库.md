# docker仓库

### docker镜像加速

```sh
#!/bin/bash

# 本脚本需要以root权限运行

# 定义镜像加速地址列表（截止到2024/8/28，下面三个镜像加速都是可用的）
mirrors=(
    "https://docker.chenby.cn"
    "https://docker.m.daocloud.io"
    "https://docker.nju.edu.cn"
)

# 函数：测试镜像加速地址是否可用
function test_mirror() {
    local mirror=$1
    curl -sL $mirror/v2/_catalog > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        echo "镜像加速地址 $mirror 可用"
        return 0
    else
        echo "镜像加速地址 $mirror 不可用"
        return 1
    fi
}

# 备份原始的daemon.json文件
if [ -e "/etc/docker/daemon.json" ]; then
    # 文件存在，进行备份
    cp /etc/docker/daemon.json /etc/docker/daemon.json.bak
fi

# 初始化daemon.json内容
cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
EOF

# 遍历镜像加速地址，测试并添加到daemon.json
for i in "${!mirrors[@]}"; do  # 使用索引遍历数组，方便判断是否是最后一个元素
    mirror="${mirrors[$i]}"
    if test_mirror "$mirror"; then
        if [ $i -ne $((${#mirrors[@]} - 1)) ]; then  # 判断是否为最后一个元素
            echo "    \"$mirror\"," >> /etc/docker/daemon.json
        else
            echo "    \"$mirror\"" >> /etc/docker/daemon.json
        fi
    fi
done

# 完善daemon.json内容，使得格式符合json要求
echo "  ]" >> /etc/docker/daemon.json
echo "}" >> /etc/docker/daemon.json

# 重启Docker服务
systemctl restart docker
```



### 阿里云个人镜像仓库服务

#### 创建个人仓库

https://cr.console.aliyun.com/cn-chengdu/instances

1. 选择希望仓库所处的地区
2. 创建仓库的个人实例
3. 设置仓库的登陆密码
4. 创建命名空间
5. 创建仓库

####  使用仓库

```sh
1. 登录阿里云Docker Registry
$ docker login --username=sea2five registry.cn-chengdu.aliyuncs.com
用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

2. 从Registry中拉取镜像
$ docker pull registry.cn-chengdu.aliyuncs.com/ndsl-czw/alpine:[镜像版本号]

3. 将镜像推送到Registry
$ docker login --username=sea2five registry.cn-chengdu.aliyuncs.com
$ docker tag [ImageId] registry.cn-chengdu.aliyuncs.com/ndsl-czw/alpine:[镜像版本号]
$ docker push registry.cn-chengdu.aliyuncs.com/ndsl-czw/alpine:[镜像版本号]
```

