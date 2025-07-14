# Ingress

ingress是一个抽象的概念，它的具体实现可以帮助集群实现来自外部的访问。

它所充当的角色就类似于nginx反向代理，来自外部的流量统一访问ingress，然后由ingress进行响应。

ingress有很多实现，其中之一就是nginx。

![image-20240627192313961](..\images\image-20240627192313961.png)



## 安装helm & ingress-nginx

rnm给你一个小提示：看有关安装组件的视频的时候，可以把弹幕打开，有坑的话可能会有人提出来，cnm

```bash
# 首先安装helm
# 下载二进制文件
wget https://get.helm.sh/helm-v3.2.3-linux-amd64.tar.gz
tar -xzvf helm-v3.2.3-linux-amd64.tar.gz

# 将helm程序移动到 usr/local/bin/helm
mv linux-amd64/helm /usr/local/bin/helm

# 验证是否成功安装
helm version



# 添加ingress-nginx的helm仓库
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# 查看仓库列表
helm repo list

# 搜索 ingress-nginx
helm search repo ingress-nginx

# 下载安装包(可能会失败，隔两分钟试下可能就行了，是从GitHub上拉取的，玄学问题)
helm pull ingress-nginx/ingress-nginx --version=4.2.2

mkdir -p /opt/k8s/helm
mv ingress-nginx-4.2.2.tgz /opt/k8s/helm
cd /opt/k8s/helm
tar -xf ingress-nginx-4.2.2.tgz

# 解压后，进入解压完成的目录
cd ingress-nginx

# 修改 values.yaml
####################################
# 两处镜像及仓库需要修改成国内的源
registry: registry.cn-hangzhou.aliyuncs.com
image: google_containers/nginx-ingress-controller
image: google_containers/kube-webhook-certgen
# 同时修改tag
tag: v1.5.1
# 两处镜像附近的哈希校验需要注释掉（digest开头的，带sha256和长串字符的行）

hostNetwork: true

# gg到开头搜索DaemonSet，原本是Deployment的地方
修改部署配置的 kind: DaemonSet
# 增加这个DaemonSet的选择器
nodeSelector:
  # 原本这里有一行
  ingress: "true"
  
# 下面两处离得不远
将 admissionWebhooks.enabled 修改为 false
将 service 中的 type 由 LoadBalancer 修改为 ClusterIP，如果服务器是云平台才用 LoadBalancer
####################################

# 给ingress-nginx专门创建一个命名空间
kubectl create ns ingress-nginx
# 给需要安装ingress的机器打上标签（和之前修改values.yaml中的节点选择器有关)
kubectl label node k8s-node01 ingress=true
# 开始安装
helm install ingress-nginx -n ingress-nginx .
```



## 一个实例测试

### 配置文件

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress-test
  annotations:
    kubernetes.io/ingress.class: "nginx"
    #nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  #ingressClassName: nginx-example
  rules:
  - host: k8s.ingresstest.com	# 外部可以通过这个域名访问到ingress关联的服务
    http:
      paths:
      - path: /testIngress	# 访问路径
        pathType: Prefix	# 路径匹配的类型，前缀匹配
        backend:
          service:	# ingress后端关联到的服务
            name: nginx-svc	# 服务名
            port:			# 服务暴露的端口
              number: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  ports:
  - port: 80 # service自身暴露的端口，结合service的内网ip可以进行访问
    targetPort: 80  # service关联的pod使用的端口
    #nodePort: 32123 # 固定绑定到所有node的这个端口上
    name: web
  selector: # 选择器，所有匹配的pod都为与该service绑定，从而可以通过service访问到pod
    app: nginx-deploy
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deploy
  namespace: default
spec:
  replicas: 3
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - name: nginx-po
        image: nginx:1.7.9
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            cpu: 200m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 128Mi
      restartPolicy: Always
```

### 测试流程

1. 由上面配置文件创建ingress
2. 在windows主机上修改host，以便可以在浏览器中通过域名访问到ingress的服务。域名指向的ip必须是ingress-controller所在的节点ip

<img src="..\images\image-20240704183002768.png" alt="image-20240704183002768" style="zoom:60%;" />

3. 在主机浏览器中访问`k8s.ingresstest.com/testIngress`，发现404不能正常访问

4. 查看日志发现访问的后台路径是`/usr/share/nginx/html/testIngress`，也就把访问路径原封不动作为了要访问的后台资源，但实际上并没有这个资源
5. 修改配置文件，在`metadata.annotation`下增加`nginx.ingress.kubernetes.io/rewrite-target: /`，把访问的资源重定向到`/`根目录
