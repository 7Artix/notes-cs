使用 Debian bookworm 或以上版本

```shell
export http_proxy="http://127.0.0.1:7890"
export https_proxy="http://127.0.0.1:7890"

curl -fsSL https://get.docker.com -o get-docker.sh 
sudo -E sh get-docker.sh
```

配置 `docker` 代理

```shell
mkdir -p /etc/systemd/system/docker.service.d
vim /etc/systemd/system/docker.service.d/http-proxy.conf
```

添加以下内容

```shell
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1,192.168.0.0/16"
```

```shell
systemctl daemon-reload
systemctl restart docker
```

验证代理配置成功:

```shell
docker info | grep Proxy

 HTTP **Proxy**: http://127.0.0.1:7890

 HTTPS **Proxy**: http://127.0.0.1:7890

 No **Proxy**: localhost,127.0.0.1,192.168.0.0/16
```


docker相关的进程: `docker` `docker.socket` `containerd`

`docker.socket` 是待机唤醒开关, 当运行 `docker` 命令, 就会唤醒 `docker.service`

`containerd` 是 `docker` 的依赖.