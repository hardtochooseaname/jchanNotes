# ps命令

### 查看特定进程拥有哪些线程

```sh
ps -Lf -p <进程PID>
```

- `-L`: 显示线程

- `-f`: 显示完整格式，包括父进程 ID (PPID)
- `-p`: 指定查询某个进程

> LWP: Light Weight Process

![image-20250507102643203](../images/image-20250507102643203.png)