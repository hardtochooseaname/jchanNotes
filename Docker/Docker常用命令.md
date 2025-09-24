# Docker常用命令

Docker 常用命令可以分为以下几个主要类别，涵盖了镜像、容器、网络、数据卷等常见操作：

## 🐳 一、镜像相关

| 命令                            | 说明                                       |
| ------------------------------- | ------------------------------------------ |
| `docker pull <镜像>`            | 拉取镜像（如：`docker pull ubuntu`）       |
| `docker images`                 | 查看本地镜像列表                           |
| `docker rmi <镜像>`             | 删除镜像（如：`docker rmi ubuntu:latest`） |
| `docker build -t <名称:标签> .` | 使用 Dockerfile 构建镜像                   |
| `docker tag <源镜像> <新标签>`  | 给镜像打标签                               |

## 🧱 二、容器管理

| 命令                            | 说明                                                  |
| ------------------------------- | ----------------------------------------------------- |
| `docker run [选项] <镜像>`      | 创建并启动容器（如：`docker run -it ubuntu bash`）    |
| `docker ps`                     | 查看正在运行的容器                                    |
| `docker ps -a`                  | 查看所有容器（包括已停止）                            |
| `docker start <容器名/ID>`      | 启动已停止的容器                                      |
| `docker stop <容器名/ID>`       | 停止运行中的容器                                      |
| `docker restart <容器名/ID>`    | 重启容器                                              |
| `docker rm <容器名/ID>`         | 删除容器（需先停止）                                  |
| `docker exec -it <容器> <命令>` | 在运行中的容器中执行命令（如：进入容器）              |
| `docker attach <容器名/ID>`     | 连接到容器控制台（不推荐 detach 时使用 Ctrl + P + Q） |

## 📦 三、数据卷（Volumes）

| 命令                              | 说明                     |
| --------------------------------- | ------------------------ |
| `docker volume create <名称>`     | 创建数据卷               |
| `docker volume ls`                | 查看所有数据卷           |
| `docker volume inspect <名称>`    | 查看卷详情               |
| `docker volume rm <名称>`         | 删除卷                   |
| `docker run -v <卷名>:<容器路径>` | 使用卷挂载数据（持久化） |

## 🌐 四、网络管理

| 命令                                | 说明                 |
| ----------------------------------- | -------------------- |
| `docker network ls`                 | 查看网络列表         |
| `docker network create <名称>`      | 创建自定义网络       |
| `docker network inspect <名称>`     | 查看网络详情         |
| `docker run --network <名称> ...`   | 指定容器所连接的网络 |
| `docker network connect/disconnect` | 连接/断开容器与网络  |

## 🛠️ 五、系统信息与清理

| 命令                  | 说明                                 |
| --------------------- | ------------------------------------ |
| `docker info`         | 查看 Docker 系统信息                 |
| `docker version`      | 查看 Docker 版本                     |
| `docker stats`        | 查看容器资源使用情况                 |
| `docker system df`    | 查看磁盘空间使用                     |
| `docker system prune` | 清理无用的镜像、容器、网络等（慎用） |

------

## 📝 示例：启动一个带卷和端口映射的容器

```bash
docker run -d -p 8080:80 -v mydata:/data --name web nginx
```

说明：后台启动 nginx 容器，将主机 8080 映射到容器 80，挂载卷 `mydata` 到容器 `/data` 路径。



# docker run

### **1. 基础控制参数**

|     参数     |              说明              |                 示例                  |
| :----------: | :----------------------------: | :-----------------------------------: |
|   **`-d`**   | 后台运行容器（detached mode）  |         `docker run -d nginx`         |
| **`--name`** |          指定容器名称          |  `docker run --name my-nginx nginx`   |
|  **`--rm`**  | 容器退出后自动删除（测试常用） | `docker run --rm alpine echo "hello"` |
|  **`-it`**   |    交互式运行（分配伪终端）    |     `docker run -it ubuntu bash`      |

------

### **2. 网络与端口映射**

|      参数       |                   说明                   |                示例                 |
| :-------------: | :--------------------------------------: | :---------------------------------: |
|    **`-p`**     |    端口映射（`宿主机端口:容器端口`）     |    `docker run -p 8080:80 nginx`    |
| **`--network`** |               指定网络模式               |  `docker run --network=host nginx`  |
|  **`--link`**   | 连接其他容器（已过时，推荐用自定义网络） | `docker run --link redis:redis app` |

------

### **3. 环境变量与配置**

|       参数       |                说明                 |                示例                 |
| :--------------: | :---------------------------------: | :---------------------------------: |
|     **`-e`**     |            设置环境变量             |  `docker run -e MY_VAR=value app`   |
| **`--env-file`** |         从文件加载环境变量          |  `docker run --env-file=.env app`   |
|     **`-v`**     | 挂载数据卷（`宿主机路径:容器路径`） | `docker run -v /data:/app/data app` |

------

### **4. 资源限制**

|      参数       |          说明           |               示例                |
| :-------------: | :---------------------: | :-------------------------------: |
| **`--memory`**  | 限制内存（单位：`m/g`） |  `docker run --memory=512m app`   |
|  **`--cpus`**   |      限制CPU核心数      |    `docker run --cpus=1.5 app`    |
| **`--restart`** |  容器退出时的重启策略   | `docker run --restart=always app` |

------

### **5. 镜像与命令控制**

|        参数        |       说明       |                  示例                   |
| :----------------: | :--------------: | :-------------------------------------: |
| **`--entrypoint`** | 覆盖默认入口命令 | `docker run --entrypoint=/bin/bash app` |
| **`-v`** (镜像卷)  |  挂载镜像中的卷  |        `docker run -v /data app`        |
|    **`--pull`**    | 强制拉取最新镜像 |    `docker run --pull=always nginx`     |



# docker实战

## 镜像

### 镜像打包与加载

```sh
# 在源机器执行
docker save -o my-app.tar my-app:v1
scp my-app.tar ubuntu@192.168.1.100:/home/ubuntu/

# 在目标机器执行
ssh ubuntu@192.168.1.100
docker load -i /home/ubuntu/my-app.tar
docker run -d --name app my-app:v1
```



## 运维

**定期执行 `docker system prune -f`和 `docker builder prune -f`是 Docker 维护的最佳实践**，特别是在频繁构建镜像的开发环境中。

### 为什么应该经常执行：

1. 磁盘空间管理：
   - Docker 会积累大量悬空镜像、容器和缓存
   - 一个中型项目每月可积累 10-50GB 无用数据
   - 定期清理可节省 30-70% 的 Docker 磁盘空间
2. 性能优化：
   - 减少 Docker 守护进程的资源开销
   - 加速镜像构建（缓存更精简）
   - 改善 `docker` 命令响应速度
3. 系统稳定性：
   - 防止磁盘空间耗尽导致构建失败
   - 避免因缓存冲突导致的奇怪构建错误

#### 推荐执行频率：

| 环境类型     | 清理频率           | 建议命令                                            |
| ------------ | ------------------ | --------------------------------------------------- |
| 开发环境     | 每日或每次重启后   | `docker system prune -f`                            |
| CI/CD 流水线 | 每次构建任务完成后 | `docker builder prune -f --all`                     |
| 生产环境     | 每月（谨慎执行）   | `docker system prune -f --filter until=720h`        |
| 个人电脑     | 每周               | `docker system prune -f && docker builder prune -f` |

#### 安全清理策略：

```Bash
# 安全清理模板（保留最近7天的资源）
docker system prune -f --filter "until=168h"
docker builder prune -f --filter "until=168h"
```

------

### 一、`docker system prune -f` 深度解析

**删除的具体内容：**

1. **停止的容器（stopped containers）**
   - **产生原因：** `docker run` 后停止的容器，或 `docker-compose down` 后的容器
   - **作用：** 释放容器文件系统占用的空间（每个容器约 50-500MB）
2. **悬空镜像（dangling images）**
   - **产生原因：**
     - 构建失败时的中间层（显示为 `<none>`）
     - 重新构建后旧版本的镜像层
     - `docker build --no-cache` 产生的废弃层
   - **作用：** 清理镜像构建过程的"垃圾"，节省 50-80% 的镜像存储空间
3. **未使用的网络（unused networks）**
   - **产生原因：**
     - `docker network create` 创建但未使用的网络
     - 容器删除后遗留的自定义网络
   - **作用：** 减少网络命名空间占用，改善网络性能
4. **悬空构建缓存（部分）**
   - **产生原因：** 构建过程中生成的临时层
   - **作用：** 释放部分构建缓存空间
5. **未使用的卷（需加 `--volumes` 参数）**
   - **产生原因：** 容器删除后未指定 `-v` 删除的卷
   - **作用：** 清理持久化数据占用的空间

------

### 二、`docker builder prune -f` 深度解析

**专门针对构建系统的清理：**

1. **构建缓存层（build cache layers）**

   - **产生原因：**

     ```Dockerfile
     RUN apt-get update  # 这行产生的缓存层
     COPY . /app         # 文件变化会使后续缓存失效
     ```

   - **特点：** 每个 `RUN`/`COPY`/`ADD` 指令都会生成缓存层

   - **作用：** 清理无效缓存，避免缓存冲突导致的构建错误

2. **源上下文缓存（source context cache）**

   - **产生原因：** `docker build` 时复制的本地文件（如 `COPY . /app`）
   - **特点：** 文件变化会使整个缓存树失效
   - **作用：** 释放构建上下文占用的内存和磁盘空间

3. **多阶段构建中间层（multi-stage intermediates）**

   - **产生原因：**

     ```Dockerfile
     FROM node AS build  # 这个阶段产生的中间镜像
     FROM nginx          # 最终阶段
     ```

   - **作用：** 清理未被最终镜像引用的中间构建层

4. **未拉取的镜像层（unpulled image layers）**

   - **产生原因：** 构建过程中下载的基础镜像临时层
   - **作用：** 清理部分下载但未使用的镜像数据

------

### 三、资源产生的时间线

|                   | 成功                       | 失败                        |
| ----------------- | -------------------------- | --------------------------- |
| **开始构建**      | 拉取基础镜像               |                             |
|                   | 执行 `RUN` 命令            |                             |
|                   | 生成构建缓存               |                             |
|                   | 复制文件                   |                             |
|                   | 创建中间层镜像             |                             |
| **构建成功/失败** |                            |                             |
|                   | 运行容器                   | 产生悬空镜像                |
|                   | 停止容器                   |                             |
|                   | 产生停止的容器             |                             |
|                   | 产生构建缓存               | 产生构建缓存                |
|                   | 产生源上下文缓存           | 产生源上下文缓存            |
|                   | `docker system prune` 清理 | `docker builder prune` 清理 |

---

### 四、为什么需要分别使用两条命令

1. **职责分离：**

   - `system prune`：管理运行时资源（容器/镜像/网络）
   - `builder prune`：管理构建过程资源（缓存/中间层）

2. **存储位置不同：**

   - 构建缓存存储在 `/var/lib/docker/buildkit`（独立于镜像存储）
   - 镜像数据存储在 `/var/lib/docker/overlay2`

3. **清理粒度差异：**

   - `builder prune` 可精确控制缓存保留策略：

     ```Bash
     # 保留最近2天的缓存
     docker builder prune --filter until=48h
     ```

4. **性能影响：**

   - 构建缓存清理可加速后续构建（减少缓存验证时间）
   - 系统清理改善整体 Docker 守护进程性能
