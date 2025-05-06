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

