# Deployment

这是一个控制器，主要用来维护和控制pod：

- 滚动更新 & 回滚
- 平滑扩容 / 缩容
- 暂停 & 恢复

同时deployment中还包含有**ReplicaSet**控制器：

- 维护pod副本数目

由此可见，deployment的功能主要分为两大块，其中维护副本数这个任务是由RS控制器来完成的。因此在创建deployment时会自动创建replicaset，然后由replicaset创建pod。而三者的联系是通过label和selector来绑定的：

![image-20240626150742966](..\images\image-20240626150742966.png)



## 一个deployment示例

### yaml文件

一个要注意的点：在pod的模板中不需要指定pod的名字，因为pod的名字会根据deployment的名字自动生成的

```yaml
apiVersion: apps/v1  # 要指出deployment资源的api组是apps
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deploy
  namespace: default
spec:
  replicas: 3  # 期望副本数
  revisionHistoryLimit: 10  # 更新后最多可以保留多少个版本（保留的版本在后面都可以回滚）
  selector: 
    matchLabels:  
      app: nginx-deploy
  strategy:  # 更新策略
    type: RollingUpdate   # 采用的更新类型
    rollingUpdate:  #滚动更新的配置
      maxSurge: 25%
      maxUnavailable: 25%
  template:  
    metadata:  
      labels:
        app: nginx-deploy #容器的标签不需要和deploy标签一致，只需要和选择器匹配，与deploy标签一致有助于资源的管理和维护
    spec:
      containers:
      - name: nginx-po 
        image: nginx:1.7.9
        imagePullPolicy: IfNotPresent
      restartPolicy: Always
```

### 扩容/缩容

- edit命令在配置文件中修改`replicas`

- scale命令指定新的副本数（实际上也是在修改配置文件中的replicas）

<img src="..\images\image-20240626153058678.png" alt="image-20240626153058678" style="zoom:80%;" />

![image-20240626153121764](..\images\image-20240626153121764.png)



### 滚动更新/回滚

<u>只要deployment的配置文件的`template`发生修改，就会触发更新操作</u>

- edit命令修改template字段
- set命令直接修改容器所用的镜像

#### edit修改容器的镜像版本

![image-20240626153641893](..\images\image-20240626153641893.png)

#### set命令直接修改镜像版本

![image-20240626154101633](..\images\image-20240626154101633.png)

**set的语法：**

```sh
kubectl set image <resource>/<name> <container-name>=<image>
```

![image-20240626160052534](..\images\image-20240626160052534.png)

![image-20240626160108787](..\images\image-20240626160108787.png)



#### rollout系列命令

##### **rollout status查看滚动更新的状态：**

```bash
kubectl rollout status <resource> <resource-name>
# 用斜杠隔开资源类型和资源名称也可以
kubectl rollout status <resource>/<resource-name>
```

![image-20240626160140060](..\images\image-20240626160140060.png)

##### **rollout history查看滚动更新历史：**

```bash
kubectl rollout history <resource> <resource-name>
kubectl rollout history <resource>/<resource-name>
```

![image-20240626161130632](..\images\image-20240626161130632.png)

注意：可以在`CHANGE-CAUSE`看到更新的原因，是因为之前在使用set命令修改镜像版本时，添加了`--record`选项（但是这个选项在后面好像会被弃用）。

##### **rollout history --revision查看指定版本的更新详情：**

版本号就是在`rollout history`命令的`REVISION`这一列的数字

![image-20240626161545786](..\images\image-20240626161545786.png)

##### rollout undo回退版本

回退到上一个版本

```sh
kubectl rollout undo deployment nginx-deployment
```

回退到指定版本

```sh
kubectl rollout undo deployment nginx-deployment --to-revision=1
```

**rollout pause/resume暂停与恢复：**

当需要对deployment做出较多修改时，可以把deployment暂停下来（避免修改一处就更新一次），等所有更新操作做完后，再恢复deployment，这时就会自动检测所有更新并一次完成。

```bash
kubectl rollout pause deployment nginx-deployment
kubectl rollout resume deployment nginx-deployment
```





#### 滚动更新的过程

滚动更新时会新建一个RS，新版本的pod都由这个新的RS来创建

<img src="..\images\image-20240626154524465.png" alt="image-20240626154524465" style="zoom:67%;" />

![image-20240626154817012](..\images\image-20240626154817012.png)

##### 关于保留的旧RS：

滚动更新创建新的RS后，旧的RS不会被删除，在之后如果需要回滚版本时，旧版本pod的创建就由对应的旧RS负责



##### describe查看具体过程

![image-20240626162627568](..\images\image-20240626162627568.png)





# StatefulSet

缩写：sts

StatefulSet是用于管理有状态服务的应用，有状态服务需要稳定的**网络配置**与**持久化存储**

![image-20240626174513540](..\images\image-20240626174513540.png)

### Headless Service

用于定义网络标志，将与域名与ip绑定。使得用户可以通过服务名获得访问路径（即域名），再通过域名得到ip

#### StatefulSet 中pod域名命名规则：

`statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local`

- serviceName 为 Headless Service 的名字
- 0..N-1 为 Pod 所在的序号，从 0 开始到 N-1
- statefulSetName 为 StatefulSet 的名字
- namespace 为服务所在的 namespace，Headless Servic 和 StatefulSet 必须在相同的 namespace
- .cluster.local 为 Cluster Domain

**注意：**后面四项`namespace.svc.cluster.local`可以省略

### volumeClaimTemplate

用于创建持久卷



## 一个StatefulSet的示例

### 配置文件

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:  # service中selector的写法少了matchLabels这一层
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"  # 使用哪个service来管理 dns
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80 # 容器内部要暴露的端口
          name: web  # 端口的名称
```

#### pod的命名

deployment创建的无状态服务的pod的名字后缀是随机的，而statefulset创建的有状态服务的pod名称是从0开始的有序数字

![image-20240626182548734](..\images\image-20240626182548734.png)

#### 服务访问路径

1. pod名称：`<sts-name>-<number>`

2. service路径：`<pod-name>.<service-name>`

##### nslookup查看dns

![image-20240626183841323](..\images\image-20240626183841323.png)



### 扩容和缩容

方法类似于deployment，不同的是sts中的扩容/缩容是有序进行的，扩容是升序，缩容是降序

![image-20240626184925094](..\images\image-20240626184925094.png)



### 镜像的更新 - patch

除了deployment中edit和set两种方式外，还通过patch操作来间接更新镜像：

```bash
# 使用具有位置数组的 json 补丁更新sts镜像
kubectl patch sts web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"nginx:1.9.1"}]'
```

从描述信息可以看到，实际采用的是**滚动更新**，删除一个，创建一个，并且是按**降序**顺序

![image-20240626213325810](..\images\image-20240626213325810.png)



### 金丝雀发布/灰度发布

当某个新版本需要测试/可能有严重bug时，为了避免新版本造成严重影响，不一次性更新完所有pod，而是只更新一部分。当确定新版本没问题后，再将剩下的更新为新版本；如果有问题，则回滚这一部分。甚至可以把这个过程分成更多步，逐步扩大新版本覆盖的范围。

<img src="..\images\image-20240626214318063.png" alt="image-20240626214318063" style="zoom:67%;" />

#### rollingUpate.partition字段

默认值0。意思是滚动更新时，一直更新到序号0的pod。因为是倒序更新，因此也就是更新所有的pod。

更改partition为n，那么就倒序更新到序号n的pod。可以通过多次修改该字段来逐步完成所有pod的更新。

#### 测试：

1. edit修改配置文件（镜像 & partion），保存退出后会自动更新。

<img src="..\images\image-20240626215342220.png" alt="image-20240626215342220" style="zoom:80%;" />

2. describe查看pod信息，发现web-3是更新后的1.7.9，web2就是旧的1.9.1
3. 查看statefulset信息，发现确实只更新了web-4和web-3

![image-20240626220008493](..\images\image-20240626220008493.png)

4. edit修改partition为0。更新剩下的所有的pod。



### onDelete更新

这种更新方式不是在修改配置文件后立刻更新，而是在delete删除掉pod后，新创建的pod才会被更新

配置文件只需要updataStrategy.type字段

![image-20240626221520619](..\images\image-20240626221520619.png)



# DaemonSet

在匹配的node机器上创建一个守护进程

使用场景：监控、日志收集等组件，通常需要到各个node上收集数据信息，这些进程通常就设置为守护进程在后台运行。

![image-20240626225403402](..\images\image-20240626225403402.png)

## 一个示例

注意两个选择器的层级位置

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:  # 这里的选择器决定的是daemonset与哪些pod关联
    matchLabels:
      app: logging
  template:
    metadata:
      labels:
        app: logging
        id: fluentd
      name: fluentd
    spec:
      nodeSelector: # 这里的选择器决定的是pod与哪些node关联
        type: microsvc
      containers:
      - name: fluentd-es
        image: agilestacks/fluentd-elasticsearch:v1.3.0
        env:
         - name: FLUENTD_ARGS
           value: -qq
        volumeMounts:
         - name: containers
           mountPath: /var/lib/docker/containers
         - name: varlog
           mountPath: /varlog
      volumes:
         - hostPath:
             path: /var/lib/docker/containers
           name: containers
         - hostPath:
             path: /var/log
           name: varlog
```

### 节点匹配

- 如果没有设置任何节点匹配的方法，那么守护进程就会被部署到除master外的所有节点

### 三种匹配方法

#### 节点选择器：nodeSelector

- 通过节点的标签来匹配node

- 如果有node新增了匹配的标签，那么这个node上就会被立刻部署上对应的守护进程

![image-20240626232355233](..\images\image-20240626232355233.png)

#### 节点亲和度

#### pod亲和度



### 更新策略

默认是滚动更新





# HPA

HPA也是一种资源，用于控制对pod进行自动的水平扩容/缩容

### 示例命令

```sh
kubectl autoscale deploy nginx-deploy --cpu-percent=20 --min=2 --max=5
```

这个命令会对名为 `nginx-deploy` 的 Deployment 创建一个 HPA，配置如下：

- **cpu-percent=20**：目标 CPU 使用率为 20%。如果实际使用率超过 20%，HPA 会**<u>尝试</u>**增加 Pod 的副本数量；如果实际使用率低于 20%，HPA 会尝试减少 Pod 的副本数量。
- **min=2**：最少保留 2 个 Pod。
- **max=5**：最多允许 5 个 Pod。

### 工作原理

1. **监控指标**：HPA 定期查询 Kubernetes Metrics Server（或其他自定义指标源）以获取当前的指标值。
2. **计算期望副本数**：HPA 根据当前的指标值和设定的目标值计算需要的副本数。例如，如果当前 CPU 使用率高于 20%，HPA 会增加 Pod 的副本数。
3. **调整副本数**：HPA 调整 Deployment 的副本数，以匹配计算出的期望副本数。

<u>HPA 并不是每次查询时都要进行扩容或缩容</u>。在实际操作中，HPA 只会在 CPU 使用率显著偏离目标值（例如 20%）时进行扩容或缩容。为了避免频繁调整 Pod 数量，HPA 有一些内置的机制和参数来平滑其行为。

### 主要机制和参数

1. **稳定窗口（Stabilization Window）**：
   - HPA 会在一段时间内（默认 ==5 分钟==）观察指标值的变化，确保 CPU 使用率的变化是持续的，而不是暂时的波动。
   - 如果在这个窗口内，CPU 使用率一直高于或低于目标值，HPA 才会做出调整。

2. **延迟（Cooldown Periods）**：
   - 扩容和缩容都有各自的冷却时间，默认是 3 分钟。在此期间，HPA 不会进行新的扩缩容操作，以避免频繁调整造成的系统不稳定。

3. **阈值范围（Thresholds）**：
   - HPA 可以设置上下限阈值，只有当 CPU 使用率超出这些阈值时才会触发扩容或缩容。
   - 例如，可能设置目标值为 20%，但只有当 CPU 使用率超过 25% 或低于 15% 时才会进行调整。

### 示例场景

假设 HPA 的目标 CPU 使用率为 20%：

- **CPU 使用率在 18%-22% 之间**：HPA 认为系统处于健康状态，不会进行任何操作。
- **CPU 使用率持续高于 22%**：如果超过稳定窗口时间（例如 ==5 分钟==），HPA 将触发扩容操作。
- **CPU 使用率持续低于 18%**：如果超过稳定窗口时间，HPA 将触发缩容操作。

通过这些机制，HPA 可以避免由于短暂的 CPU 使用率波动而频繁调整 Pod 数量，从而保持系统的稳定性和高效性。

### metrics-server监控组件

这是一个轻量级的集群监控组件，安装这个组件后，可以通过`kubectl top po`命令查看各个pod的资源使用情况，从而更好地观察HPA的行为。

#### 组件部署的配置文件

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
        image: registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-server:v0.6.2
        imagePullPolicy: IfNotPresent
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 10250
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
          seccompProfile:
            type: RuntimeDefault
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
```

然后直接`apply -f`就可以部署了



# CronJob

定时任务

### 配置文件

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

### 字段解释

#### `.spec.schedule`

Cron表达式，用于规定定时任务的执行周期

```
# ┌───────────── 分钟 (0 - 59)
# │ ┌───────────── 小时 (0 - 23)
# │ │ ┌───────────── 月的某天 (1 - 31)
# │ │ │ ┌───────────── 月份 (1 - 12)
# │ │ │ │ ┌───────────── 周的某天 (0 - 6)（周日到周六）
# │ │ │ │ │                          或者是 sun，mon，tue，web，thu，fri，sat
# │ │ │ │ │
# │ │ │ │ │
# * * * * *
```



#### `.spec.startingDeadlineSeconds`（optional）

**最大的延迟执行时间**。即如果因为某种原因任务没有按时执行，那么允许在多长时间内延后执行。

例如，如果将其设置为200，那么Job控制器允许在期望调度时间之后的200s内执行，如果超过200s则视为Job失败。



#### `.spec.concurrencyPolicy`（optional）

**并发性规则：**

- Allow（默认）：允许新旧job并发执行。
- Forbid：不允许并发执行。当新job该执行时如果旧job还没有执行完，那么会忽略新job的执行请求。但是新job的执行可以受startingDeadlineSeconds影响延后执行。
- Repace：当新job该执行时如果旧job还没有执行完，那么新job会直接取代旧job。



####  `.spec.suspend` （optional）

设置为`true`可以挂起CronJob调度的任务。当希望某段时间内CronJob不被执行，可以在这段时间将 `.spec.suspend` 置为`true`。

**挂起**，是指job会被按时调度，但是不会被执行，在挂起期间被调度的任务被统计为**错误的job**。当现有的 CronJob 将 `.spec.suspend` 从 `true` 改为 `false` 时， 且没有`startingDeadlineSeconds`，错过的 Job 会被立即调度。



#### 任务历史限制（optional）

当job被调度执行完成后，job的pod会被保留，可以通过`.spec.successfulJobsHistoryLimit`和`.spec.failedJobsHistoryLimit`指定保留多少已完成和失败的Job。两个字段都可以设置为0，成功Job默认保留3个，失败Job默认保留1个

<img src="..\images\image-20240801151830826.png" alt="image-20240801151830826" style="zoom:67%;" />

**当CronJob被删除时**，保留的所有pod都会被删除。
