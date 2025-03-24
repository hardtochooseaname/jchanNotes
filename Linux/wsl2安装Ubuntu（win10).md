# wsl2安装Ubuntu（win10)

[知乎]: https://zhuanlan.zhihu.com/p/584398122	"win10下wsl2安装图形化界面Ubuntu"

# 首先需要升级wsl2

### 0. 检查系统是否支持wsl2

win+R打开运行，然后输入winver检查windows版本，版本需要大于1903

![image-20250318192008992](C:\Users\10133\AppData\Roaming\Typora\typora-user-images\image-20250318192008992.png)

### 1. 管理员打开powershell

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

### 2. 控制面板--程序--启用或关闭Windows功能

打开：

- Windows虚拟机监控程序平台
- 虚拟机平台

![img](https://pica.zhimg.com/v2-4021b5ad8a82214202af9b7d22cf3526_1440w.jpg)

### 3. 重启

如果后面设置wsl默认版本为wsl2或者安装Ubuntu时报错：**请启用虚拟机平台 Windows 功能并确保在 BIOS 中启用虚拟化。**

需要在重启时进入BIOS打开cpu虚拟化相关选项。

### 4. 安装wsl2升级包

[下载链接]:https://link.zhihu.com/?target=https%3A//wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi

### 5. 设置WSL默认版本为WSL2

powershell执行：
```powershell
wsl --set-default-version 2
```

### 6. 微软商店下载Ubuntu

这一步微软商店打开报错：初始化失败

出错原因：网络代理设置的问题

1. win搜索：internet选项
2. 连接 ->局域网设置
3. 取消勾选`代理服务器`

### 7. 打开Ubuntu

它会自动安装，成功后设置用户名密码就可以

Windows的所有磁盘都挂载到了/mnt目录下，可以`ls /mnt`查看



# 限制wsl内存占用

不然的话wsl内存占用相当大（8GB+)

在Windows的用户家目录(C:\users\username)下创建`.wslconfig`文件：

```
[wsl2]
memory=4GB
swap=5GB
localhostForwarding=true
```

