# Nginx服务器

## 反向代理

一个简单的配置文件示例：

```nginx
# worker_processes指令定义了Nginx启动的进程数，通常设置为auto或CPU核心数。
worker_processes auto;

events {
    # 每个worker进程可以处理的最大连接数
    worker_connections 1024;
}

http {
    # 这是一个上游服务器组的定义，名为'app1_server'
    # 它指向了docker-compose中名为'app1'的服务，以及它在容器内监听的8000端口
    upstream app1_server {
        server app1:8000;
    }

    # 这是为app2定义的服务组
    upstream app2_server {
        server app2:8000;
    }

    # 这个server块负责将所有HTTP（80端口）的请求，强制重定向到HTTPS
    server {
        listen 80;
        # 替换成你的服务器公网IP地址
        server_name 192.168.20.247; 
        return 301 https://$host$request_uri;
    }

    # 这是处理所有HTTPS请求的主服务块
    server {
        listen 443 ssl;
        # 替换成你的服务器公网IP地址
        server_name 192.168.20.247; 

        # SSL证书配置，路径是容器内的路径
        ssl_certificate /etc/nginx/certs/cert.pem;
        ssl_certificate_key /etc/nginx/certs/key.pem;

        # 路由规则1：处理所有以 /app1/ 开头的请求
        location /api/app1/ {
            # proxy_pass指令将请求转发给名为'app1_server'的上游服务器组
            proxy_pass http://app1_server/;

            # 设置一些必要的HTTP头，将客户端的真实信息传递给后端服务
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # 路由规则2：处理所有以 /app2/ 开头的请求
        location /api/app2/ {
            proxy_pass http://app2_server/;
            # 同样的头部设置
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```



## 静态文件服务器

### **问题一：Nginx提供静态文件的具体工作原理**

您的理解**基本完全正确**。Nginx提供静态文件的核心原理，就是建立一个**“网络URL路径”**与**“服务器磁盘上的文件路径”**之间的映射关系。

这个映射主要由两个指令来控制：`root` 和 `location`。

#### **`root` 指令：定义网站的根目录**

`root`指令告诉Nginx：“这个网站所有的文件，都存放在服务器的这个基础文件夹里”。

**举个最简单的例子：**

- **Nginx配置 (`nginx.conf`)**:

  ```nginx
  server {
      listen 80;
      server_name example.com;
  
      # 定义网站的物理根目录
      root /var/www/my-website;
  }
  ```

- **服务器上的文件结构**:

  ```
  /var/www/my-website/
  ├── index.html
  ├── about.html
  └── images/
      └── logo.png
  ```

- **映射关系**: 当一个请求进来时，Nginx会把**URL中的路径**拼接到`root`指令指定的**物理路径**后面，从而找到对应的文件。

  - 用户访问 `http://example.com/about.html`
  - Nginx取下URL路径 `/about.html`
  - Nginx拼接路径：`root` + `URL路径` = `/var/www/my-website` + `/about.html`
  - 最终，Nginx读取并返回服务器上 `/var/www/my-website/about.html` 这个文件的内容。
  - 用户访问 `http://example.com/images/logo.png`
  - Nginx拼接路径：`/var/www/my-website` + `/images/logo.png`
  - 最终返回 `/var/www/my-website/images/logo.png` 文件。

所以，您说的“基于这个目录下文件的相对路径是什么，那么这个文件在网络上的访问路径就是什么”，这个理解是非常准确的。

---

### **问题二：如何区分和管理多个开发者提交的前端静态文件？**

您敏锐地指出了一个关键问题：如果简单地把所有同学的`dist`文件夹内容都混在一个`root`目录下，`index.html`、`favicon.ico`等同名文件就会互相覆盖，导致灾难。

**最佳实践是：在物理上和逻辑上都对它们进行隔离。**

我们通过“为每个应用分配一个URL前缀”和“为每个应用分配一个独立的物理目录”来实现。这里，Nginx的`location`指令和另一个强大的指令`alias`就派上了用场。

#### **`root` vs. `alias`指令的关键区别**

**两者的核心区别：**

- **`root`**: 将 `location` 匹配的完整URI路径，附加到 `root` 指定的路径后面。
- **`alias`**: 将 `location` 匹配的那部分路径**丢弃**，然后使用 `alias` 指定的路径作为新的根。这种方式其实就是给某个路由路径重新指定一个静态文件的根路径，会更具灵活性。

这个区别用一个例子就非常清楚了：

**场景：** 用户请求 `/app-a/style.css`

- **使用 `root`**:

  ```nginx
  location /app-a/ {
      root /var/www/html;
  }
  # Nginx会寻找: /var/www/html/app-a/style.css
  # 它把location的路径也作为了物理路径的一部分。
  ```

- **使用 `alias` (我们需要的！)**:

  ```nginx
  location /app-a/ {
      alias /var/www/html/alice-dist/;
  }
  # Nginx会寻找: /var/www/html/alice-dist/style.css
  # 它把location的路径 /app-a/ 替换成了 alias 的路径。
  ```

`alias`指令完美地实现了我们想要的“URL前缀”到“特定物理目录”的映射。



### **综合实践：在Docker中为多应用提供服务**

现在，我们将这个原理应用到我们最终的“Nginx为核心”的部署方案中。

#### **第一步：在Nginx镜像构建时，物理隔离前端文件**

在`nginx/Dockerfile`中，我们将不同同学构建好的`dist`文件夹，复制到Nginx镜像内部**不同的子目录**中。

```dockerfile
# nginx/Dockerfile

# --- STAGE 1: 构建Alice的前端 ---
FROM node:18-alpine AS alice-builder
WORKDIR /app
COPY ../alice-project/frontend/ ./
RUN npm install && npm run build

# --- STAGE 2: 构建Bob的前端 ---
FROM node:18-alpine AS bob-builder
WORKDIR /app
COPY ../bob-project/frontend/ ./
RUN npm install && npm run build

# --- STAGE 3: 最终的Nginx镜像 ---
FROM nginx:1.25-alpine
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/

# 将构建好的前端文件，按应用的“身份证”分别存放到不同的子目录
COPY --from=alice-builder /app/dist /usr/share/nginx/html/app-a
COPY --from=bob-builder /app/dist /usr/share/nginx/html/app-b
```

构建完成后，我们的Nginx镜像内部文件结构就变成了：

```
/usr/share/nginx/html/
├── app-a/
│   ├── index.html
│   └── assets/
└── app-b/
    ├── index.html
    └── assets/
```

这样就实现了物理上的完全隔离。



#### **第二步：在`nginx.conf`中，用`alias`实现逻辑隔离**

现在，我们用`alias`将URL路径和这些物理目录精确地对应起来。

```nginx
# nginx/nginx.conf
server {
    listen 443 ssl;
    # ... 其他配置 ...

    # --- Alice的应用路由 ---
    # 规则1: 静态文件服务
    location /app-a/ {
        # 当URL以/app-a/开头时，去容器内的/usr/share/nginx/html/app-a/目录找文件
        alias /usr/share/nginx/html/app-a/;
        # 对于单页应用，如果找不到文件，就返回它的主入口index.html
        try_files $uri $uri/ /app-a/index.html;
    }

    # 规则2: API反向代理
    location /api/app-a/ {
        proxy_pass http://app_a:8000/;
        # ... proxy headers ...
    }

    # --- Bob的应用路由 ---
    # 规则3: 静态文件服务
    location /app-b/ {
        alias /usr/share/nginx/html/app-b/;
        try_files $uri $uri/ /app-b/index.html;
    }

    # 规则4: API反向代理
    location /api/app-b/ {
        proxy_pass http://app_b:8000/;
        # ... proxy headers ...
    }
}
```

