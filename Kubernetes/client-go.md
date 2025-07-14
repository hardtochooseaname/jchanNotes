# client-go

## API Server

### API组

- **核心 API 组 (`core` 或空 API 组)**:
  - 资源：Pod、Service、ConfigMap、Secret、PersistentVolume、PersistentVolumeClaim 等。
  - 版本：`v1`。
- **扩展 API 组**:
  - apps:
    - 资源：Deployment、ReplicaSet、StatefulSet、DaemonSet 等。
    - 版本：`v1`。
  - batch:
    - 资源：Job、CronJob 等。
    - 版本：`v1`。
  - networking.k8s.io:
    - 资源：Ingress、NetworkPolicy 等。
    - 版本：`v1`。

### URL结构

在 Kubernetes 中，当使用 `client-go` 或其他方式向 API Server 请求资源时，URL 的结构遵循 RESTful API 风格，并具有以下通用结构：

```
https://<api-server-host>:<api-server-port>/api(s)/<api-group>/<api-version>/namespaces/<namespace-name>/<resource>/<resource-name>
```

#### 具体说明：
1. `<api-server-host>`: 
   这是 API Server 的主机名或 IP 地址（通常在集群内部使用时是 `kubernetes.default.svc`）。

2. `<api-server-port>`: 
   API Server 的端口，通常是 `6443`（默认 HTTPS）。

3. `/api` 或 `/apis`:  

   - `/api`: 代表核心 API 组，例如处理像 Pod、Node 这样的资源。
   - `/apis`: 代表扩展 API 组（如自定义资源定义 CRD）。

4. `<api-group>`:  

   - 核心资源（如 Pod、Node）没有 API 组，因此直接省略。
   - 非核心资源则会有一个 API 组，比如 `apps`、`batch`、`extensions` 等。

5. `<api-version>`: 
   API 版本，例如 `v1`、`v1beta1`，版本根据资源和 API 组的定义而不同。

6. `namespaces/<namespace-name>`:

   指定命名空间

7. `<resource>`: 
   这是请求的资源类型，例如 `pods`、`nodes`、`deployments`。

8. `<resource-name>`: 
   资源的具体名称，如果是请求某个特定的资源则填写这个字段，例如某个 Pod 的名字。如果请求的是资源列表，则省略此字段。

#### 注意两处复数

1. 命名空间`namespaces`是复数
2. 资源类型是复数，例如 `pods`、`nodes`、`deployments`等

#### 实例：

1. **列出所有 Pods**：

   ```
   GET https://<api-server-host>:<api-server-port>/api/v1/pods
   ```

2. **获取特定 Namespace 中的某个 Pod**：

   注意：**namespaces是复数！**

   ```
   GET https://<api-server-host>:<api-server-port>/api/v1/namespaces/<namespace>/pods/<pod-name>
   ```

3. **列出所有 Deployments（非核心资源）**：
   ```
   GET https://<api-server-host>:<api-server-port>/apis/apps/v1/deployments
   ```

4. **获取某个 Deployment**：

   ```
   GET https://<api-server-host>:<api-server-port>/apis/apps/v1/namespaces/<namespace>/deployments/<deployment-name>
   ```

#### 关于 CRD：
如果你使用自定义资源定义 (CRD)，URL 结构类似：
```
https://<api-server-host>:<api-server-port>/apis/<custom-group>/<custom-version>/namespaces/<namespace>/<custom-resource>/<custom-resource-name>
```

这种 URL 结构是 Kubernetes 的核心设计，基于 REST API，方便操作和扩展不同的资源。



## 客户端Client

在 `client-go` 中，常见的四种客户端（Client）是：

1. **`RESTClient`**  
2. **`DynamicClient`**  
3. **`TypedClient`** (a.k.a. `ClientSet`)  
4. **`DiscoveryClient`**

每种客户端都有不同的用途和适用场景，以下是它们的详细区别和应用场景：

#### 1. `RESTClient`
- **定义**: `RESTClient` 是最底层的客户端，直接与 Kubernetes API Server 交互。它允许你对任何 Kubernetes API 资源执行 HTTP 请求（如 GET、POST、PUT、DELETE 等），通过指定 API 版本、资源类型、命名空间等参数进行操作。
- **使用场景**: 当需要直接使用 Kubernetes API、定制化 HTTP 请求时，或需要操作自定义资源（CRD）时，使用 `RESTClient`。它通常适用于高级用户，或者需要细粒度控制的场景。
- **灵活性**: 非常灵活，可以操作所有 API 资源，但需要开发者自行构建请求。
- **示例**: 通过 REST 调用自定义资源时可用：
  ```go
  restClient.Get().Namespace("default").Resource("pods").Name("pod-name").Do(ctx).Into(&pod)
  ```

#### 2. `DynamicClient`
- **定义**: `DynamicClient` 是一种不依赖具体类型的客户端，它不需要预先知道 API 资源的类型，可以动态处理 Kubernetes 资源对象。
- **使用场景**: 当操作的资源类型在编译时未知，或需要对多个不同的资源进行操作时，适合使用 `DynamicClient`。例如，处理自定义资源定义（CRD）或通用的资源管理任务。
- **灵活性**: 适用于那些无法确定具体类型的情况下（如 CRD），但缺乏类型安全（不能获得结构化的 Go 类型）。
- **示例**:
  ```go
  dynamicClient.Resource(gvr).Namespace("default").Get(ctx, "resource-name", metav1.GetOptions{})
  ```

#### 3. `TypedClient`（a.k.a. `ClientSet`）
- **定义**: `TypedClient` 是一种类型化的客户端，提供了对 Kubernetes 核心资源（如 Pod、Service、Deployment 等）以及扩展 API 资源的类型安全的访问。`ClientSet` 是最常见的 Kubernetes 客户端，按 API 版本和资源类型划分。
- **使用场景**: 当处理标准的 Kubernetes 资源时使用，支持代码补全和类型检查，开发者可以更轻松、安全地与 Kubernetes 交互。
- **灵活性**: 类型安全且友好。由于每个资源都有特定的客户端，可以直接操作（例如 Pods、Services、Deployments）。
- **示例**:
  ```go
  clientset.CoreV1().Pods("default").Get(ctx, "pod-name", metav1.GetOptions{})
  ```

#### 4. `DiscoveryClient`
- **定义**: `DiscoveryClient` 是一种特殊的客户端，主要用于发现 Kubernetes API 资源和服务器功能。它可以列出集群中可用的 API 组和资源，并检查服务器的版本等。
- **使用场景**: 通常用于客户端初始化时发现集群的 API 资源、版本信息，以及其他有关服务器能力的信息。
- **灵活性**: 只能用于发现 API，不用于操作资源。适合构建通用的工具或框架，以确保 API 的兼容性。
- **示例**:
  ```go
  discoveryClient.ServerResourcesForGroupVersion("v1")
  ```

---

#### 总结
- **`RESTClient`**: 灵活但底层，适合高级用户操作自定义资源。
- **`DynamicClient`**: 适合动态处理多个 API 资源，但不类型安全。
- **`TypedClient` (`ClientSet`)**: 类型安全的标准资源客户端，易于使用。
- **`DiscoveryClient`**: 用于发现集群的 API 资源和版本。