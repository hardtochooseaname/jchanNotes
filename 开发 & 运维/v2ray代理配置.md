### 安装

```sh
wget -qO - https://apt.v2raya.org/key/public-key.asc | sudo tee /etc/apt/keyrings/v2raya.asc

echo "deb [signed-by=/etc/apt/keyrings/v2raya.asc] https://apt.v2raya.org/ v2raya main" | sudo tee /etc/apt/sources.list.d/v2raya.list
sudo apt update

sudo apt install v2raya v2ray ## 也可以使用 xray 包

sudo systemctl enable --now v2raya.service
```

### 配置

![透明代理](https://v2raya.org/docs/prologue/quick-start/images/tproxy.png)