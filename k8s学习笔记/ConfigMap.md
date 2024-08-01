# ConfigMap



## 基本用法

```sh
Examples:
  # --from-file=<目录>：把一个目录中的所有文件都加载到configmap中
  kubectl create configmap my-config --from-file=path/to/bar
  
  # --from-file=<文件>：只加载指定的文件
  kubectl create configmap my-config --from-file=key1=/path/to/bar/file1.txt --from-file=key2=/path/to/bar/file2.txt
  
  # --from-file=<新文件名>=<文件>：在configmap中把配置文件重命名
  kubectl create configmap my-config --from-file=key1=/path/to/bar/file1.txt --from-file=key2=/path/to/bar/file2.txt
  
  
  
  # 下面的是不常用的
  
  # 直接用key-value对创建configmap，这样就没有文件名
  kubectl create configmap my-config --from-literal=key1=config1 --from-literal=key2=config2
  
  # Create a new config map named my-config from the key=value pairs in the file
  kubectl create configmap my-config --from-file=path/to/bar
  
  # Create a new config map named my-config from an env file
  kubectl create configmap my-config --from-env-file=path/to/foo.env --from-env-file=path/to/bar.env
```

- 文件/目录的路径可以是绝对路径也可以是相对路径



### 用法1：从ConfigMap加载env环境变量

- 创建ConfigMap

![image-20240722104628049](..\images\image-20240722104628049.png)

- 创建一个pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-env-config
  namespace: default
spec:
  containers:
  - name: busybox
    image: busybox:1.28.4
    command: ["/bin/sh", "-c", "env; sleep 3600"]
    env:
    - name: COLLEGE
      valueFrom:
        configMapKeyRef:
          name: test-env-config
          key: SCHOOL
    - name: CITY
      valueFrom:
        configMapKeyRef:
          name: test-env-config
          key: LOCATION
    imagePullPolicy: IfNotPresent
  restartPolicy: Never
```

- 查看pod日志

<img src="..\images\image-20240722104732710.png" alt="image-20240722104732710" style="zoom:50%;" />



### 用法2：把ConfigMap的文件作为数据卷加载

- ConfigMap中可能包含多个配置文件，而这些配置文件可以作为一个数据卷被挂载到容器中

<img src="..\images\image-20240722143934229.png" alt="image-20240722143934229" style="zoom:80%;" />

- pod的配置文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-cm-vol-pod
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx:1.9.1
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "sleep 3600"]
    volumeMounts:	# 需要挂载的卷
    - name: name-of-vol	# 卷的名字（来自下面volumes属性）
      mountPath: "/usr/local/mysql/conf"	# 挂载点
      readOnly: true	# 是否只读
  volumes:  # 卷的配置
    - name: name-of-vol  # 卷的名字，自己取
      configMap:   # 卷的内容来自configMap
        name: test-cm-vol  # configMap的名字
        items:		# 指定需要加载configMap中的哪些文件
        - key: "file2.test"  # configMap中的文件名
          path: "mytest.conf"  # 文件挂载进入后映射为哪个文件，其实就是提供对该文件重命名的机会
  restartPolicy: Never
```

- 创建pod，并且进入容器查看对应目录的文件，发现成功把configMap中的文件挂载到了容器

![image-20240722150150425](..\images\image-20240722150150425.png)

- 去掉`spec.volumes.name.configMap.items`属性，也就是不具体指定加载的文件，这样直接把configMap中所有文件都加载到数据卷

```yaml
...
  volumes:
    - name: name-of-vol
      configMap: 
        name: test-cm-vol
...
```

![image-20240722151511914](..\images\image-20240722151511914.png)



## subPath挂载数据卷中的单一文件

​	上面从configMap加载数据卷有一个问题，就是configMap中的文件会把挂载点所在的目录给覆盖掉。也就是说，如果原先挂载点本身有文件，那么这样挂载会导致挂载点的原有文件丢失（被覆盖）。

​	考虑这样一种情况，容器的某个目录中存放的是一系列的配置文件，其中某个文件可能需要经常修改，那么我们希望可以把这个文件单独拿出来作为configMap加载进去。这样当我们更新configMap时，容器中的该配置文件就会自动更新。这就需要使用到subPath属性。

#### 测试用例

- 以nginx镜像中`/etc/nginx`目录下的`nginx.conf`文件为例（该nginx运行在一个deployment中）

​	<img src="..\images\image-20240722153856916.png" alt="image-20240722153856916" style="zoom:67%;" />



- 把`nginx.conf`文件的内容复制，用以在外面单独创建一个`nginx.conf`文件，并且使用该文件创建一个configMap

- edit修改配置，把configMap挂载进去
  1. sepc.volume属性
  2. spec.containers.volumeMounts属性
  3. 增加一个command（不加这个容器重启会出错，不知道为什么）

​	<img src="..\images\image-20240722161157625.png" alt="image-20240722161157625" style="zoom:67%;" />



- 进入容器发现挂载点的原有文件都被configMap的文件覆盖了

​	<img src="..\images\image-20240722161534767.png" alt="image-20240722161534767" style="zoom:67%;" />

挂载之前👆    ->   挂载之后👇

​	<img src="..\images\image-20240722161421117.png" alt="image-20240722161421117" style="zoom:67%;" />



- edit修改配置，增加subpath属性

  1. moutPath需要修改为文件的路径（之前是挂载点的路径）
  2. 增加subPath属性，指定具体的文件（configMap中可以有很多文件，只要在这里指定清楚就可以）

  

  subPath属性的解释：

  ```sh
  [root@k8s-master config]# kubectl explain pods.spec.containers.volumeMounts.subPath
  KIND:     Pod
  VERSION:  v1
  
  FIELD:    subPath <string>
  
  DESCRIPTION:
       Path within the volume from which the container's volume should be mounted.
       Defaults to "" (volume's root).
       
       subPath意思是子路径，“子”是相对mountPath挂载卷的路径而言，也就是指数据卷内部的路径
       设置subPath属性，就是要在数据卷中指定一个文件，然后单独把数据卷中的这个文件挂载到挂载点
  ```

  

​	<img src="..\images\image-20240722163359366.png" alt="image-20240722163359366" style="zoom:67%;" />

- 再进入容器查看，发现`/etc/nginx`目录下的其他文件没有被覆盖



## configMap的更新

### 更新configMap的方式

#### 方法1：使用更新过的配置文件重新创建configMap

`kubectl create cm test-cm-subpath --from-file=nginx.conf --dry-run=client -o yaml | kubectl apply -f -`

- `--dry-run=client`: 这个参数表示模拟执行命令，但不实际提交到集群。它在客户端进行验证，并生成相应的配置文件。
- `-o yaml`: 这个参数用于指定输出格式为 YAML。
- `kubectl apply -f -`: 这个命令接收通过管道传输过来的 YAML 文件内容，并应用到集群中。

#### 方法2：edit直接修改configMap



### configMap更新后容器中文件内容的更新

#### 不使用subPath属性时

当configMap发生修改时，容器挂载点中的对应文件，也会自动更新，不过会有十几秒的延迟

```sh
### 修改configMap
[root@k8s-master config]# kubectl create cm test-cm-subpath --from-file=nginx.conf --dry-run=client -o yaml | kubectl apply -f -
Warning: resource configmaps/test-cm-subpath is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
configmap/test-cm-subpath configured

### 检查configMap是否修改成功
[root@k8s-master config]# kubectl describe cm test-cm-subpath
Name:         test-cm-subpath
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx.conf:
----
change:true


BinaryData
====

Events:  <none>

### 进入容器查看挂载的文件是否被修改
[root@k8s-master config]# kubectl get po
NAME                                READY   STATUS      RESTARTS   AGE
nginx-deployment-5bb7f4d69d-592jr   1/1     Running     0          10m
nginx-deployment-5bb7f4d69d-hjrr9   1/1     Running     0          10m
test-cm-vol-pod                     0/1     Completed   0          114m
[root@k8s-master config]# kubectl exec -it nginx-deployment-5bb7f4d69d-592jr -- sh
# cat /etc/nginx/nginx.conf
change:true
# exit
```

#### 使用subPath时

configMap修改，挂载点文件不会自动更新

解决办法：

- 曲线救国，不使用subPath属性把文件直接挂载到挂载点，而是新建一个中转目录，把文件加载到该目录（不使用subPath属性），再把该文件软链接到挂载点
  - 这需要在初始化阶段删除掉挂载点的原文件

​	<img src="..\images\image-20240722173223894.png" alt="image-20240722173223894" style="zoom:67%;" />



### immuable属性限定configMap不能更新

```yaml
apiVersion: v1
data:
  file1.test: |
    good:morning
    name:jchan
  file2.test: |
    hh:haha
    ha:hhh
immutable: true		# 注意它的层级
kind: ConfigMap
metadata:
  creationTimestamp: "2024-07-22T06:38:40Z"
  name: test-cm-vol
  namespace: default
  resourceVersion: "445687"
  uid: e692c046-e38e-48ec-be7d-ca543b9d03ce
```





## secret

secret就是加密的配置文件，用法和configmap类似

#### 基本用法

```sh
[root@k8s-master config]# kubectl create secret -h
Create a secret using specified subcommand.

Available Commands:
  docker-registry Create a secret for use with a Docker registry
  generic         Create a secret from a local file, directory, or literal value
  tls             Create a TLS secret

Usage:
  kubectl create secret [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

#### 用法1：generic - 一般的文本加密

![image-20240722111333737](..\images\image-20240722111333737.png)

#### 用法2：docker-registry - 配置docker仓库的登陆信息

当pod的镜像需要从私有仓库拉取时，需要设置私有仓库的登陆信息

**这是实际使用secret最多的地方**

![image-20240722112902739](..\images\image-20240722112902739.png)

```sh
Usage:
  kubectl create secret docker-registry NAME --docker-username=user --docker-password=password --docker-email=email
[--docker-server=string] [--from-file=[key=]source] [--dry-run=server|client|none] [options]
```



# 存储卷

### hostPath

使得容器可以直接访问主机的文件系统，把主机的目录或文件挂载到容器。（有点像类似虚拟机的共享文件夹）

因此，使用这种数据卷类型，就需要注意pod具体运行在哪个node，想要挂载点目录/文件是否在这个node上。

另外，pod中不同容器也可以挂载同一个hostPath，这样可以在主机和pod多个容器之间实现存储共享。

> 由于hostPath会让容器能够直接访问到主机文件系统，这是一种相当不安全的方式，应该尽量避免使用hostPath

#### 测试用例/配置文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-vol-pod
  namespace: default
spec:
  containers:
  - name: nginx-test-vol
    image: nginx
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: test-hostpath
      mountPath: /test-vol  # 挂载点目录不存在会自动创建目录
  volumes:
  - name: test-hostpath
    hostPath:  # 数据卷类型为hostPath
      path: /root/test
      type: DirectoryOrCreate
  restartPolicy: Never
```

#### 类型： `volumes.hostPath.type`属性

- 空字符串：默认类型，不做任何检查
- DirectoryOrCreate：如果给定的 path 不存在，就创建一个 755 的空目录
- Directory：这个目录必须存在
- FileOrCreate：如果给定的文件不存在，则创建一个空文件，权限为 644
- File：这个文件必须存在
- Socket：UNIX 套接字，必须存在
- CharDevice：字符设备，必须存在
- BlockDevice：块设备，必须存在

#### 警告warning

使用 `hostPath` 类型的卷存在许多安全风险。如果可以，你应该尽量避免使用 `hostPath` 卷。 例如，你可以改为定义并使用 [`local` PersistentVolume](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#local)。

如果你通过准入时的验证来限制对节点上特定目录的访问，这种限制只有在你额外要求所有 `hostPath` 卷的挂载都是**只读**的情况下才有效。如果你允许不受信任的 Pod 以读写方式挂载任意主机路径， 则该 Pod 中的容器可能会破坏可读写主机挂载卷的安全性。

------

无论 `hostPath` 卷是以只读还是读写方式挂载，使用时都需要小心，这是因为：

- 访问主机文件系统可能会暴露特权系统凭证（例如 kubelet 的凭证）或特权 API（例如容器运行时套接字）， 这些可以被用于容器逃逸或攻击集群的其他部分。
- 具有相同配置的 Pod（例如基于 PodTemplate 创建的 Pod）可能会由于节点上的文件不同而在不同节点上表现出不同的行为。
- `hostPath` 卷的用量不会被视为临时存储用量。 你需要自己监控磁盘使用情况，因为过多的 `hostPath` 磁盘使用量会导致节点上的磁盘压力。



### emptyDir

实现pod中不同容器之间的存储共享。原理就是创建一个epmtyDir即空目录，并将该空目录同时挂载到pod中的不同容器中，这样任何容器对该目录的修改，都会反馈到所有容器中。

需要注意的是，emptyDir没有持久化存储的特性，emptyDir目录会随着pod的删除被删除。

#### 测试用例/配置文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-emptydir-pod
  namespace: default
spec:
  containers:
  - name: test-emptydir1
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "sleep 1000"]
    volumeMounts:
    - name: test-emptydir
      mountPath: /test-vol
  - name: test-emptydir2 # 两个容器名字一定要不同
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "sleep 1000"]
    volumeMounts:
    - name: test-emptydir
      mountPath: /test-emptydir  # 两个容器emptyDir的挂载点可以不同
  volumes:
  - name: test-emptydir
    emptyDir: {}	# 数据卷类型为emptyDir
  restartPolicy: Never
```

`emptyDir` 的一些用途：

- 缓存空间，例如基于磁盘的归并排序。
- 为耗时较长的计算任务提供检查点，以便任务能方便地从崩溃前状态恢复执行。
- 在 Web 服务器容器服务数据时，保存内容管理器容器获取的文件。

`emptyDir.medium` 字段用来控制 `emptyDir` 卷的存储位置。 默认情况下，`emptyDir` 卷存储在该节点所使用的介质上； 此处的介质可以是磁盘、SSD 或网络存储，这取决于你的环境。 你可以将 `emptyDir.medium` 字段设置为 `"Memory"`， 以告诉 Kubernetes 为你挂载 tmpfs（基于 RAM 的文件系统）。 虽然 tmpfs 速度非常快，但是要注意它与磁盘不同， 并且你所写入的所有文件都会计入容器的内存消耗，受容器内存限制约束。

你可以通过为默认介质指定大小限制，来限制 `emptyDir` 卷的存储容量。 此存储是从[节点临时存储](https://kubernetes.io/zh-cn/docs/concepts/configuration/manage-resources-containers/#setting-requests-and-limits-for-local-ephemeral-storage)中分配的。 如果来自其他来源（如日志文件或镜像分层数据）的数据占满了存储，`emptyDir` 可能会在达到此限制之前发生存储容量不足的问题。





### NFS

#### 安装NFS

```sh
# 安装 nfs(在要使用nfs的所有节点)
yum install nfs-utils -y

# 启动 nfs(在要使用nfs的所有节点)
systemctl start nfs-server

# 查看 nfs 版本
cat /proc/fs/nfsd/versions

# 创建共享目录
mkdir -p /data/nfs
cd /data/nfs
mkdir rw
mkdir ro

# 设置共享目录 export
vim /etc/exports
/data/nfs/rw 192.168.18.0/24(rw,sync,no_subtree_check,no_root_squash)
/data/nfs/ro 192.168.18.0/24(ro,sync,no_subtree_check,no_root_squash)

# 重新加载
exportfs -f
systemctl reload nfs-server
```

1. 安装nfs并启动nfs服务
2. 在node01上创建两个目录作为网络文件系统挂载
3. `/etc/exports` 文件在 NFS 服务器上是一个关键配置文件，用于定义哪些目录将被共享给哪些客户端，以及这些共享目录的访问权限和行为控制。其中最关键的是网段的设置，规定了这个nfs目录可以被哪个网段的终端使用。
4. 重新加载配置

#### 测试NFS

在master中mount设置好的nfs目录

```sh
mkdir -p /mnt/nfs/rw
mount -t nfs k8s-node01:/data/nfs/rw /mnt/nfs/rw
mount -t nfs k8s-node01:/data/nfs/ro /mnt/nfs/ro

# 测试完后umount卸载nfs
umount /mnt/nfs/rw
umount /mnt/nfs/ro
```

#### 测试nfs类型volume

其实和hostPath类型数据卷差不多，只不过hostPath只支持挂载本地目录，而nfs支持挂载远程机器上的目录

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-nfs-pod
spec:
  containers:
  - image: nginx
    name: test-nfs-nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: nfs-vol-ro
  volumes:
  - name: nfs-vol-ro
    nfs:
      server: k8s-node01 # 网络存储服务地址
      path: /data/nfs/ro # 网络存储路径
      readOnly: true # 是否只读
```

- 在node01的/data/nfs/ro创建index.html
- 再curl这个pod，可以看到获取的是nfs中的文件

