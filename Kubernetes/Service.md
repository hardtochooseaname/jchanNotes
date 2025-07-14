# Service服务

service主要的功能是提供东西流量的转发，也就是集群内部的微服务（pods）之间的流量访问。同时提供负载均衡（四层负载-网络层）。

### 服务类型

#### 无状态服务

**特点**：

1. **无持久存储**：无状态服务不需要保存客户端会话数据或其他持久数据，对本地环境没有影响（通常需要依赖外部的存储服务）。
2. **可伸缩性**：很容易在集群中新增和减少应用实例，可以轻松地水平扩展。
3. **易于恢复**：单独的实例可以随时地重启或重新调度，不会影响整体的服务。

**使用场景**：

- Web 服务器
- API 服务
- 微服务架构中的小型独立组件

#### 有状态服务

**特点**：

1. **持久存储**：有状态服务需要保存客户端会话数据、数据库或其他持久数据。
2. **稳定的网络标识**：每个实例（Pod）都有一个稳定的网络标识（DNS 名称），以便于其他服务能够稳定地访问它们。
3. **顺序启动和停止**：有状态服务的实例通常需要按照特定顺序启动和停止。

**使用场景**：

- 数据库（如 MySQL、PostgreSQL）
- 分布式存储系统（如 Cassandra、Elasticsearch）
- 需要持久化数据的应用程序

#### 举例说明

![image-20240604204606318](..\images\image-20240604204606318.png)

##### 无状态的Nginx服务

1. 本身没有存储任何数据，只是提供一个反向代理服务
2. 可以随时新增（或减少）一个Nginx服务，不需要额外的设置或操作，这个新增的nginx就能在集群中正常工作

##### 有状态的redis服务

1. 本身提供的就是数据存储服务，在本地有持久化数据
2. 当新增一个redis服务的实例时，需要进行数据迁移和同步等操作之后，这个服务实例才能开始正常工作



### 工作原理

通过service访问其关联的应用一般有两种方式：

- `[service-name/ip]:[service-port]`：这是最常使用的访问方式，用于集群内部的相互访问。
- `NodeIP:NodePort`：这种方式可以从外网访问，但是不推荐（*当service类型是NodePort才可以*）。这里的节点ip和端口都是node主机的外网ip和端口，这是在创建service时，会在所有node上将一个`NodePort`关联到service服务上，从而实现从外网直接访问到节点上的service的方法。由于这种方法进行外部访问需要暴露节点ip和service对应的NodePort，这既不方便也不安全，因此不推荐。

#### 测试：

1. 从master/node主机上通过service-ip访问：可以访问 
   - 因为创建服务时同时修改了集群主机的iptables，因此可以从主机路由到内部的service-ip
2. 从master/node主机上通过service-name访问：不能访问 
   - 集群主机上并没有对该服务做dns映射
3. 从master上创建的临时pod中通过service-name：可以访问 
   - 集群内网中有服务的dns映射
4. 从windows上通过`NodeIP:NodePort`访问：可以访问 
   - NodePort和应用pod内的targetPort有映射关系
5. 从windows上通过service-ip访问：不能访问 
   - service-ip是集群的内网ip，windows既不在集群内网中，也没有对这个ip的路由信息

### service原理图

service有一个很重要的组件就是 EndPoint，每次创建service的时候都会创建配套的EndPoint，ep里面记录了service所关联的所有pod的ip，以及映射到pod内部的targetPort。EndPoint也是一种系统资源，简写是ep。

![image-20240627144251354](..\images\image-20240627144251354.png)

### 基础配置文件

```yaml
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
    name: web # 端口的名字，取名不重要，主要是方便辨识
  selector: # 选择器，所有匹配的pod都为与该service绑定，从而可以通过service访问到pod
    app: nginx-deploy
  type: NodePort
```



## service的类型

### NodePort

会在所有安装了 kube-proxy 的节点都绑定一个端口，此端口可以代理至对应的 Pod，集群外部可以使用任意节点 ip + NodePort 的端口号访问到集群中对应 Pod 中的服务。当类型设置为 NodePort 后，可以在 ports 配置中增加 nodePort 配置指定端口，需要在30000~32767端口范围内，如果不指定会随机指定端口（端口范围配置在 /usr/lib/systemd/system/kube-apiserver.service 文件中）。推荐使用随机的端口，可以避免端口冲突。

**使用场景：**

通常在开发环境使用，不在生产环境使用。因为service是四层负载，效率较低，同时不支持对应用层报文的解析。

### ClusterIP

不提供外部访问，service只能在集群内部访问。（默认type）

### External

service指向外部的域名。

### LoadBalancer

使用云服务商（阿里云、腾讯云等）提供的负载均衡器服务



## 通过service访问外部服务

### 使用场景

- 应用需要访问集群外部的服务
- 大型项目迁移，导致一部分项目在集群内，而一部分还在外面。同时还需要保持项目各部分之间的通信。

### 工作原理

通过service具体访问到的是什么服务，还是看与service绑定的endpoint关联了哪些应用。因此可以通过自定义endpoint，并把endpoint指向外部服务的方式，来使得service可以访问到外部服务。

### 配置文件1 - ClusterIP+EndPoint

```yaml 
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-external
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    targetPort: 80
    name: web
  type: ClusterIP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: nginx-svc-external # must be same as service's
  labels:
    app: nginx # same
  namespace: default
subsets:
- addresses:
  - ip: 103.41.167.234  # 这是知乎的ip
  ports:  # same
  - name: web
    port: 80
    protocol: TCP
```

可以看到，service中没有选择器，而service与endpoint本身是强绑定关系，这需要两者：

1. 名字与标签一致
2. 端口信息ports一致

### 配置文件2 - ExternalName

```yaml
apiVersion: v1
kind: Service
metadata:
  name: externalname-baidu
  labels:
    app: externalname-baidu
spec:
  type: ExternalName
  ExternalName: www.baidu.com
```

![image-20240627163413733](..\images\image-20240627163413733.png)

![image-20240627163604763](..\images\image-20240627163604763.png)