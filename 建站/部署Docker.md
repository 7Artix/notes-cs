# 安装 `docker`

安装 `docker`

```shell
curl -fsSL https://get.docker.com | bash -s docker
systemctl start docker
systemctl enable docker
docker version
```


# `uptime-kuma` 状态监看

部署 `uptime-kuma` 用于状态监看

```shell
docker run -d \
  --name uptime-kuma \
  -p 3001:3001 \
  --restart always \
  -v /var/lib/uptime-kuma:/app/data \
  louislam/uptime-kuma:1
```

`-d` 后台运行
`-name` 容器名
`-p` 端口映射 Publish a container's port(s) to the host
`--restart` 意外崩溃后, docker会重新拉起
`-v` 和 `--mount` 类似, 将卷打包.

或者使用 `docker compose`

```shell
mkdir uptime-kuma
cd uptime-kuma
curl -o compose.yaml https://raw.githubusercontent.com/louislam/uptime-kuma/master/compose.yaml
docker compose up -d
```

作为参考, YAML内容为:

```shell
root@witch ~/uptime-kuma# cat compose.yaml 
services:
  uptime-kuma:
    image: louislam/uptime-kuma:2
    restart: unless-stopped
    volumes:
      - ./data:/app/data #宿主机:容器内部
    ports:
      # <Host Port>:<Container Port>
      - "3001:3001"
```

在服务器运行

```shell
cloudflared tunnel route dns organ status.7artix.com
```

创建CNAME, 将 `status.7artix.com` 指向隧道地址, 并通知 `cloudflare` 的边缘节点, 该域名由具体隧道接管.

更新 `cloudflared` 配置文件 `/etc/cloudflared/config.yml`

```yaml
tunnel: d9c7b61f-bcc2-47b7-a91d-b20a2721b3d2
credentials-file: /root/.cloudflared/d9c7b61f-bcc2-47b7-a91d-b20a2721b3d2.json

ingress:
  - hostname: ssh.7artix.com
    service: ssh://localhost:13579
  - hostname: status.7artix.com
    service: http://localhost:3001
  - service: http_status:404
```

重启 `cloudflared` 服务.

```shell
systemctl restart cloudflared
```
