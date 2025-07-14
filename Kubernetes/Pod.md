# Pod

### 配置文件（基础）

```yaml
apiVersion: v1 # api 文档版本
kind: Pod  # 资源对象类型，也可以配置为像Deployment、StatefulSet这一类的对象
metadata: # Pod 相关的元数据，用于描述 Pod 的数据
  name: nginx-demo # Pod 的名称
  labels: # 定义 Pod 的标签
    type: app # 自定义 label 标签，名字为 type，值为 app
    test: 1.0.0 # 自定义 label 标签，描述 Pod 版本号
  namespace: 'default' # 命名空间的配置
spec: # 期望 Pod 按照这里面的描述进行创建
  containers: # 对于 Pod 中的容器描述
  - name: nginx # 容器的名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 镜像拉取策略，指定如果本地有就用本地的，如果没有就拉取远程的
    command: # 指定容器启动时执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 定义容器启动后的工作目录
    ports:
    - name: http # 端口名称
      containerPort: 80 # 描述容器内要暴露什么端口
      protocol: TCP # 描述该端口是基于哪种协议通信的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量的值
    resources:
      requests: # 最少需要多少资源
        cpu: 100m # 限制 cpu 最少使用 0.1 个核心
        memory: 128Mi # 限制内存最少使用 128兆
      limits: # 最多可以用多少资源
        cpu: 200m # 限制 cpu 最多使用 0.2 个核心
        memory: 256Mi # 限制 最多使用 256兆
  restartPolicy: OnFailure # 重启策略，只有失败的情况才会重启
```

#### 配置文件编写的坑

##### 数字 or 字符串

`1.1`这种会被识别为数字而不是字符，`1.1.1`就会被识别为字符串。

如果想要数字被识别为字符串，可以加上引号`'1.1'`。

例如下面我想要标识版本号，但是被识别为数字，导致在创建pod时报错：

<img src="..\images\image-20240613220912389.png" alt="image-20240613220912389" style="zoom: 67%;" />



### 重启策略

- `Always`：只要容器终止就自动重启容器。（默认设置）
- `OnFailure`：只有在容器错误退出（退出状态非零）时才重新启动容器。
- `Never`：不会自动重启已终止的容器。



## 探针

### 探针的种类

**Liveness Probe**：

- **作用**：用于检查容器是否处于正常运行状态。如果 Liveness Probe 失败，<u>Kubernetes 会杀死该容器，并根据容器的重启策略决定是否重启容器</u>。
- **使用场景**：适用于检测容器是否进入无法正常工作的状态。

**Readiness Probe**：

- **作用**：用于检查容器是否准备好接受请求。如果 Readiness Probe 失败，Kubernetes 会<u>将该 Pod 从服务的负载均衡器中移除</u>（即，暂时不再向该 Pod 发送流量）。
- **使用场景**：适用于检测容器是否已经初始化完成并准备好处理请求。或者当服务暂时不可用的时候，例如进行内部更新或维护，告知集群这个pod暂时不能提供服务。

**Startup Probe**：

- **作用**：是v1.16 版本新增的探针，用于检查容器的启动状态。Startup Probe 在容器启动时运行，替代 Liveness Probe。当 Startup Probe 成功时，Liveness Probe 开始生效。如果 Startup Probe 失败，Kubernetes 会<u>杀死并重启容器</u>。
- **产生的背景**：在出现StartupProbe之前，要检测容器的运行状态只能通过LivenessProbe。而LivenessProbe有一个字段`initialDelaySeconds`，指定的时容器启动后经过多少秒才开始执行LivenessProbe的检测。但是有些应用本身初始化的时间较长，但是开发者并不知道这个时间是多少。假如我认为这个应用初始化需要10s，于是我设置`initialDelaySeconds: 15`，于是15s后LivenessProbe开始工作。但是实际上这个应用的初始化需要20s，20s才能响应LivenessProbe的检测。那么在15s的时候LivenessProbe就会检测容器没有正常运行，并且尝试重启容器。而重启容器后又会发生同样的问题，进入一个死循环。这个问题的关键就是我们不能预测应用的初始化到底需要多长时间，也就难以确定一个合适的`initialDelaySeconds`值。引入StartupProbe后，LivenessProbe和ReadinessProbe都会在StartupProbe成功后才开始工作，从而避免了先前的问题。
- **使用场景**：适用于启动时间较长（应用需要较长时间来进行初始化后，才能开始提供服务）的容器，以避免在启动过程中被 Liveness Probe 错误地认为容器失败。
- **Attention**：StartupProbe只需要成功一次，成功之后就会激活另外两种探针。确认失败后就不会再尝试。同时`successThreshold`只能是1，不能是其他值。

### 探测的方式

**HTTP GET 探针**：

- 通过发送 HTTP GET 请求到容器内的某个端点来检测容器的健康状况或就绪状态。如果返回的状态码是 200-399 范围内的任何值，则探测成功。

**TCP Socket 探针**：

- 通过尝试打开一个 TCP 连接到容器的指定端口来检测容器的健康状况。如果TCP连接能够建立（某个端口能建被访问），则探测成功。

**Exec 探针**：

- 通过在容器内执行一个指定的命令来检测容器的健康状况或就绪状态。如果命令的退出状态码为 0（命令执行成功），则探测成功。例如：ls一个不存在的目录、cat 一个不存在的文件，就会被判定为探测失败。





### 实践测试

`successThreshold`默认是1

注意配置文件中探针所在的层级：

​	<img src="..\images\image-20240614102442693.png" alt="image-20240614102442693" style="zoom: 50%;" />

#### StartupProbe

```yaml
    startupProbe:
      httpGet:
        path: /probe/test
        port: 80
      successThreshold: 1  # 成功1次就算成功
      failureThreshold: 3  # 失败3次才算彻底失败
      periodSeconds: 10   # 每两次检测需要间隔10s（不管成功与否）
      timeoutSeconds: 5   # 超时5s没有响应就算失败
```

显然上面这个path是错误的，所以：

![image-20240613222040895](..\images\image-20240613222040895.png)

<img src="..\images\image-20240613222330161.png" alt="image-20240613222330161" style="zoom:67%;" />

#### LivenessProbe

```yaml
    livenessProbe:
      exec:
        command:
          - cat
          - /tmp/probe.test
      failureThreshold: 3
      periodSeconds: 10
```

能够成功执行`cat /tmp/probe.test`命令的时候，容器被判定为正常运行。显然，一开始没有这个文件：

![image-20240614000417881](..\images\image-20240614000417881.png)

<img src="..\images\image-20240614000534347.png" alt="image-20240614000534347" style="zoom:80%;" />

在失败3次之前，pod还被认为在正常运行，但是超过3次后：

<img src="..\images\image-20240614000707680.png" alt="image-20240614000707680" style="zoom:80%;" />

但是如果在超过3次失败前，写入这个文件，那么就会被检测成功：

![image-20240614000819040](..\images\image-20240614000819040.png)



## 容器生命周期

![image-20240614100928629](..\images\image-20240614100928629.png)

### 容器的两个回调函数 

**PostStart：**

这个回调在容器被创建之后立即被执行。 但是，不能保证回调会在容器入口点（ENTRYPOINT）之前执行。 没有参数传递给处理程序。

**PreStop：**

在容器因 API 请求或者管理事件（诸如存活态探针、启动探针失败、资源抢占、资源竞争等） 而**被终止之前**，此回调会被调用。 如果容器已经处于已终止或者已完成状态，则对 preStop 回调的调用将失败。 在用来停止容器的 TERM 信号被发出之前，回调必须执行结束。 Pod 的终止宽限周期在 `PreStop` 回调被执行之前即开始计数， 所以无论回调函数的执行结果如何，容器最终都会在 Pod 的终止宽限期内被终止。 没有参数会被传递给处理程序。



### pod退出流程（delete pod）

容器在整个生命周期中只会被调度一次。一旦pod被assigned到某个节点运行，那么这个pod会一直运行直到主动停止或者错误终止。如果某个节点挂了，这个节点上的pods不会被重新调度，而会被安排delete。

1. Endpoint删除pod的ip地址
2. Pod变成Terminating状态，等待宽限时间`terminationGracePeriodSeconds`给用户做`PreStop`操作。宽限时间默认30s，如果超过这个时间就会立即终止。（宽限时间被用于优雅地gracefully退出）
3. 执行`PreStop`钩子函数

#### 强制删除出错的pod

```sh
kubectl delete pod <pod-name> --grace-period=0 --force
```



### PostStart与PreStop实践

- 生命周期有关配置放在`lifecycle`字段下
- 注意`lifecycle`的层级

**pod配置文件：**

```yaml
apiVersion: v1 # api 文档版本
kind: Pod  # 资源对象类型，也可以配置为像Deployment、StatefulSet这一类的对象
metadata: # Pod 相关的元数据，用于描述 Pod 的数据
  name: nginx-demo # Pod 的名称
  labels: # 定义 Pod 的标签
    type: app # 自定义 label 标签，名字为 type，值为 app
    test: 1.0.0 # 自定义 label 标签，描述 Pod 版本号
  namespace: 'default' # 命名空间的配置
spec: # 期望 Pod 按照这里面的描述进行创建
  containers: # 对于 Pod 中的容器描述
  - name: nginx # 容器的名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 镜像拉取策略，指定如果本地有就用本地的，如果没有就拉取远程的
    lifecycle:
      postStart:
        exec:
          command:
          - sh
          - -c
          - "echo '<h1> here is postStart </h1>' > /usr/share/nginx/html/postStart.html"
      preStop:
        exec:
          command:
          - sh
          - -c
          - "sleep 40; echo 'finished sleep...' > /usr/share/nginx/html/preStop.html"
    command: # 指定容器启动时执行的命令
    - nginx
    - -g
    - 'daemon off;' # nginx -g 'daemon off;'
    workingDir: /usr/share/nginx/html # 定义容器启动后的工作目录
    ports:
    - name: http # 端口名称
      containerPort: 80 # 描述容器内要暴露什么端口
      protocol: TCP # 描述该端口是基于哪种协议通信的
    env: # 环境变量
    - name: JVM_OPTS # 环境变量名称
      value: '-Xms128m -Xmx128m' # 环境变量的值
    resources:
      requests: # 最少需要多少资源
        cpu: 100m # 限制 cpu 最少使用 0.1 个核心
        memory: 128Mi # 限制内存最少使用 128兆
      limits: # 最多可以用多少资源
        cpu: 200m # 限制 cpu 最多使用 0.2 个核心
        memory: 256Mi # 限制 最多使用 256兆
  restartPolicy: OnFailure # 重启策略，只有失败的情况才会重启
```

#### **测试postStart:**

![image-20240626101656047](..\images\image-20240626101656047.png)

#### **测试preStop1：**

**使用`-w`选项持续观察pod的状态**

![image-20240626102125629](..\images\image-20240626102125629.png)

**在另一个终端delete该pod  >>> 观察到pod状态进入`Terminating`**

![image-20240626102303040](..\images\image-20240626102303040.png)

![image-20240626102325244](..\images\image-20240626102325244.png)

![image-20240626102430729](..\images\image-20240626102430729.png)

##### **结论：**

如果测试`kubectl delete`命令执行时间，会发现这个时间是在30s多一点

- 默认宽限时间`terminationGracePeriodSeconds`是30s，30s没有完成preStop操作就会被强制终止
- 因此配置文件中等待40s的操作没有做完就终止了

#### 测试preStop2：

修改配置文件

- 宽限时间改成40s（**注意在yaml的层级和位置**）

- preStop只睡眠10s

配置文件片段：

```yaml
###
spec: # 期望 Pod 按照这里面的描述进行创建
  terminationGracePeriodSeconds: 40
  containers: # 对于 Pod 中的容器描述
  - name: nginx # 容器的名称
    image: nginx:1.7.9 # 指定容器的镜像
    imagePullPolicy: IfNotPresent # 镜像拉取策略，指定如果本地有就用本地的，如果没有就拉取远程的
    lifecycle:
      postStart:
        exec:
          command:
          - sh
          - -c
          - "echo '<h1> here is postStart </h1>' > /usr/share/nginx/html/postStart.html"
      preStop:
        exec:
          command:
          - sh
          - -c
          - "sleep 10; echo 'finished sleep...' > /usr/share/nginx/html/preStop.html"
###
```

![image-20240626103841444](..\images\image-20240626103841444.png)

![image-20240626103905081](..\images\image-20240626103905081.png)

##### 结论：

preStop操作如果在宽限时间之前做完，那么pod就会立刻完成终止。



### Pod Phase

A Pod's `status` field is a [PodStatus](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.30/#podstatus-v1-core) object, which has a `phase` field.

| Value       | Description                                                  |
| :---------- | :----------------------------------------------------------- |
| `Pending`   | The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been <span style="color:tomato">set up and made ready to run</span>. This includes time a Pod spends waiting to be <span style="color:tomato">scheduled </span>as well as the time spent <span style="color:tomato">downloading container images</span> over the network. |
| `Running`   | The Pod  has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting. |
| `Succeeded` | All containers in the Pod have terminated in success, and <span style="color:tomato">will not be restarted</span>. |
| `Failed`    | All containers in the Pod have terminated, and at least one container has terminated in failure. That is, the container either exited with non-zero status or was terminated by the system, and is not set for automatic restarting. |
| `Unknown`   | For some reason the state of the Pod could not be obtained. This phase typically occurs due to an error in communicating with the node where the Pod should be running. |



## Init容器

在容器生命周期的[初始化阶段](##容器生命周期)运行的容器，主要做一些初始化的准备工作。

- 如果定义了多个Init容器，那么Init容器串行执行，前一个成功执行结束，后一个再开始执行。
- 如果某个Init容器执行失败，那么整个Pod会被视为失败。Init容器也可以设置重启策略。

```yaml
#下面的例子定义了一个具有 2 个 Init 容器的简单 Pod。 第一个等待 myservice 启动， 第二个等待 mydb 启动。 一旦这两个 Init 容器都启动完成，Pod 将启动 spec 节中的应用容器。

apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

### 使用示例

下面是一些如何使用 Init 容器的想法：

- 等待一个 Service 完成创建，通过类似如下 Shell 命令：

  ```shell
  for i in {1..100}; do sleep 1; if nslookup myservice; then exit 0; fi; done; exit 1
  ```

- 注册这个 Pod 到远程服务器，通过在命令中调用 API，类似如下：

  ```shell
  curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'
  ```

- 在启动应用容器之前等一段时间，使用类似命令：

  ```shell
  sleep 60
  ```

- 克隆 Git 仓库到[卷](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/)中。
- 将配置值放到配置文件中，运行模板工具为主应用容器动态地生成配置文件。 例如，在配置文件中存放 `POD_IP` 值，并使用 Jinja 生成主应用配置文件。

### 



##  相关kubectl命令

### 创建pod

```bash
kubectl create -f <pod-yaml-file>
```



### 查看pod状态

**查看详细信息 & 运行状态 - describe** 

```bash
kubectl describe po -n <namespace> <pod-name>
```

**持续监听pod的状态 - get -w**

```bash
kubectl get po -n <namespace> -w <pod-name>
```



### 编辑配置文件 - edit

```bash
kubectl edit po -n <namespace> <pod-name>
```

edit会获取对应的配置信息（**由API Server统一保存在etcd中**），并使用vi打开，修改完后保存。kubectl 会通过 API Server自动将新的配置应用到集群中。



### 输出某个资源的配置文件 - `-o yaml`

`kubectl get po nginx-demo -o yaml`



