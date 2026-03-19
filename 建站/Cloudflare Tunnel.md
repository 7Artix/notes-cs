# 临时域名

由于 SSH 端口流量受限, 可以使用 Cloudflare 进行流量转发.
相当于数据通过 Cloudflare 进行传输. 然后连接到对应的 SSH 端口.

```shell
# 1. 下载 deb 安装包 (针对 64 位系统)
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb

# 2. 使用 dpkg 安装
dpkg -i cloudflared-linux-amd64.deb

# 3. 验证是否安装成功
cloudflared --version
```

随后执行:

```shell
cloudflared tunnel --url ssh://localhost:13579
```

之后, 会输出一个以 `.trycloudflare.com` 结尾的临时域名, 可以用于远程访问.

在 mac 上安装:

```shell
brew install cloudflared
```

通过 ssh 连接:

```shell
ssh -o ProxyCommand='/opt/homebrew/bin/cloudflared access ssh --hostname sky-harvey-satisfaction-isolated.trycloudflare.com' root@localhost
```

使 `ssh` 通过 `Cloudflare` 的域名连接服务器.

在本机通过 `nping` 检测与服务器的延迟

```shell
sudo nping --tcp -p 443 period-observer-referenced-wallace.trycloudflare.com
```

```shell
Starting Nping 0.7.98 ( https://nmap.org/nping ) at 2026-01-28 11:34 CST

# SENT (0.0168s) nping发出数据包的时刻
# 192.168.0.102:1704 本机的内网IP和临时端口
# 104.16.230.132:443 Cloudflare节点的IP和HTTPS端口
# S(SYN) Synchronize 请求建立链接
# IP层参数
# ttl Time To Live 生存时间 数据包在被丢弃钱允许经过的最大路由器跳数 初始值通常为64/128
# id Identification 标识符 数据包的标签 用于分片重组 将大数据包拆分和重组
# iplen Total Length IP总长度 整个IP数据包的大小 报头+数据
# TCP层参数
# seq Sequence Number 序列号 TCP是流式传输 用于保证接受数据不乱序
# win Window Size 窗口大小 用于流量控制 通知本机有多少内存空间可以用于接收数据
SENT (0.0168s) TCP 192.168.0.102:1704 > 104.16.230.132:443 S ttl=64 id=59664 iplen=40  seq=3949870531 win=1480 

# RECEIVED (0.2458s) 本机网卡收到回应的时刻
# SA(SYNC/ACK) 代表Acknowledge 服务器回复准备就绪
# ttl=51 数据包从Cloudflare到本机经过了64-51=13个路由器
# 单次延迟约为: 0.2458s-0.0168s=0.229s=229ms
RCVD (0.2458s) TCP 104.16.230.132:443 > 192.168.0.102:1704 SA ttl=51 id=0 iplen=44  seq=638643521 win=65535 <mss 1400>

SENT (1.0170s) TCP 192.168.0.102:1704 > 104.16.230.132:443 S ttl=64 id=59664 iplen=40  seq=3949870531 win=1480 

RCVD (1.2868s) TCP 104.16.230.132:443 > 192.168.0.102:1704 SA ttl=51 id=0 iplen=44  seq=654268712 win=65535 <mss 1400>

SENT (2.0174s) TCP 192.168.0.102:1704 > 104.16.230.132:443 S ttl=64 id=59664 iplen=40  seq=3949870531 win=1480 

RCVD (2.2416s) TCP 104.16.230.132:443 > 192.168.0.102:1704 SA ttl=51 id=0 iplen=44  seq=669900795 win=65535 <mss 1400>

SENT (3.0180s) TCP 192.168.0.102:1704 > 104.16.230.132:443 S ttl=64 id=59664 iplen=40  seq=3949870531 win=1480 

RCVD (3.2511s) TCP 104.16.230.132:443 > 192.168.0.102:1704 SA ttl=51 id=0 iplen=44  seq=685527439 win=65535 <mss 1400>

SENT (4.0185s) TCP 192.168.0.102:1704 > 104.16.230.132:443 S ttl=64 id=59664 iplen=40  seq=3949870531 win=1480 

RCVD (4.2591s) TCP 104.16.230.132:443 > 192.168.0.102:1704 SA ttl=51 id=0 iplen=44  seq=701199405 win=65535 <mss 1400>

Max rtt: 269.686ms | Min rtt: 223.967ms | Avg rtt: 239.219ms

Raw packets sent: 5 (270B) | Rcvd: 5 (220B) | Lost: 0 (0.00%)

Nping done: 1 IP address pinged in 4.26 seconds
```

分析完整延迟:

```shell
artix@Organ ~> curl -o /dev/null -s -w "DNS: %{time_namelookup}s | TCP: %{time_connect}s | TLS: %{time_appconnect}s | Server_Response: %{time_starttransfer}s | Total: %{time_total}s\n" https://period-observer-referenced-wallace.trycloudflare.com

DNS: 0.002622s | TCP: 0.101580s | TLS: 0.206999s | Server_Response: 0.510716s | Total: 0.510824s
```


# 注册域名

当注册域名后, 可以登录 `cloudflared tunnel` 服务, 获得稳定连接.

```shell
root@witch ~# cloudflared tunnel login
Please open the following URL and log in with your Cloudflare account:

https://dash.cloudflare.com/argotunnel?aud=&callback=httpsxxxxxxxxxxxxxxxxxxxxxx

Leave cloudflared running to download the cert automatically.
2026-01-28T09:03:51Z INF Waiting for login...
2026-01-28T09:04:39Z INF You have successfully logged in.
If you wish to copy your credentials to a server, they have been saved to:
/root/.cloudflared/cert.pem
```

在浏览器上访问链接, 完成授权.

在服务器上创建隧道:

```shell
cloudflared tunnel create organ(隧道名)
```

配置域名指向, 将子域名交于隧道处理, 通知Cloudflare将域名指向这个隧道:

```shell
cloudflared tunnel route dns organ(隧道名) ssh.7artix.com


cloudflared tunnel route dns witch artixzhang.com
cloudflared tunnel route dns witch www.artixzhang.com
cloudflared tunnel route dns witch ssh.artixzhang.com
cloudflared tunnel route dns witch status.artixzhang.com
```

编写配置文件 `/etc/cloudflared/config.yml` , 配置隧道请求进入后应该如何转发:

```yaml
tunnel: d9c7b61f-bcc2-47b7-a91d-b20a27abcdd2(之前的隧道id)
credentials-file: /root/.cloudflared/d9c7b61f-bcc2-47b7-a91d-b20a27abcdd2.json

ingress:
  - hostname: ssh.7artix.com
    service: ssh://localhost:13579
  - service: http_status:404
```

```yaml
tunnel: 53e9605c-36d0-407d-86a5-832b20551632
credentials-file: /root/.cloudflared/53e9605c-36d0-407d-86a5-832b20551632.json

ingress:
  - hostname: ssh.artixzhang.com
    service: ssh://localhost:13579
  - hostname: status.artixzhang.com
    service: http://localhost:3001
  - hostname: artixzhang.com
    service: http://localhost:5555
  - hostname: www.artixzhang.com
    service: http://localhost:5555
  - service: http_status:404
```

试运行, 测试隧道配置是否正确:

```shell
cloudflared tunnel --config /etc/cloudflared/config.yml run <隧道名>
```

注册 `systemd` 服务:

```shell
cloudflared service install
systemctl start cloudflared
systemctl enable cloudflared # 开机启动
```

在 Mac 上安装 `cloudflared` 并添加 SSH 配置 `~/.ssh/config` .
因为本质上, SSH流量已经被封装进 `cloudflared` 的特殊协议, 普通的 SSH 客户端无法直连, 需要通过本地的 `cloudflared` 进行中转. SSH 握手相当于在本地完成.

```yaml
Host witch
    HostName ssh.7artix.com
    User root
    # 调用 cloudflared 建立连接
    ProxyCommand /opt/homebrew/bin/cloudflared access ssh --hostname %h
```


## 传输文件

```shell
scp -v -P 13579 \
  -o "ProxyCommand=cloudflared access ssh --hostname ssh.7artix.com" \
  site_20260130v0.tar root@ssh.7artix.com:/root/
```

拦截 `SSH` 的原生连接动作, 而使用 `cloudflared` 程序建立连接.

或使用 `rsync`

```shell
artix@Organ ~/P/7artix.com> rsync -avzP -e "ssh -p 13579 -o 'ProxyCommand=cloudflared access ssh --hostname ssh.7artix.com'" site_260130v0.tar root@ssh.7artix.com:/root/
```

`-a` 归档模式, 保留文件权限和时间戳
`-v` 显示更多内容
`-z` 压缩传输, 节省带宽
`-P` 断点续传
`-e` 使用特殊建立连接方式

