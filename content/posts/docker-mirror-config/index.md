---
title: Docker镜像源配置
date: "2025-10-09 10:15:42"
categories: ["技术"]
tags: ["Docker", "配置"]
---

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在拉取 Docker 镜像的时候，经常会遇到超时问题，这时我们需要配置 Docker 镜像源和 DNS 
## 配置 `/etc/docker/daemon.json` 文件
```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json > /dev/null <<'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://hub.uuuadc.top",
    "https://mirror.ccs.tencentyun.com"
  ],
  "dns": ["8.8.8.8", "1.1.1.1", "223.5.5.5", "114.114.114.114"],
  "max-concurrent-downloads": 1
}
EOF
```
设置镜像源是为了能够加速 Docker 镜像的下载，Docker镜像默认从 Docker Hub 拉取，中国大陆访问不稳定。设置 DNS 是为了解决域名解析问题，让 Docker 能够找到 IP 地址。上面写进文件里的 DNS 分别是 Google，Cloudflare，阿里和电信 DNS   
## 重启 Docker
```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 检查测试
```shell
sudo systemctl status docker -n 20  # 应输出 active(running)
```

```shell
docker run --rm hello-world  # Hello from Docker
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
