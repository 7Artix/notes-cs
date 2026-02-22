# Deployment

through `http`

## Installation

```shell
apt install nginx libnginx-mod-http-dav-ext -y
```


## Set directory and permission

```shell
mkdir -p /var/www/webdav
chown -R www-data:www-data /var/www/webdav
```


## User Certification file

```shell
apt install apache2-utils -y
htpasswd -c /etc/nginx/webdav_passwd username
```


## Nginx Configuration

``/etc/nginx/conf.d/webdav.conf``

```shell
server {
    listen 8081; # 监听端口
    server_name localhost;

    # 日志
    access_log off;

    location / {
        root /mnt/webdav;

        # --- 核心 WebDAV 配置 ---
        dav_methods PUT DELETE MKCOL COPY MOVE;
        dav_ext_methods PROPFIND OPTIONS;
        
        # 权限控制参数解释：
        # create_full_put_path: 允许创建不存在的中间目录（比如上传 a/b/c.txt）
        create_full_put_path on;
        # dav_access: 设置文件创建后的权限。user:rw 表示运行 nginx 的用户可读写
        dav_access user:rw group:rw all:r;

        # --- 安全与兼容性优化 ---
        client_max_body_size 0; # 设置为 0 表示不限制上传文件的大小
        autoindex on;           # 在浏览器里访问时，显示文件列表

        # --- 账户认证 ---
        auth_basic "Welcome";
        auth_basic_user_file /etc/nginx/webdav_passwd;
    }
}
```


## Reload Nginx

```shell
nginx -t
systemctl restart nginx
```



# 使用Apache

安装Apache模块:

```shell
sudo apt update
sudo apt install apache2
# 启用 WebDAV 模块
sudo a2enmod dav
sudo a2enmod dav_fs
```

创建共享目录, 并确保Apache有访问权限:

```shell
sudo mkdir -p /var/www/webdav
sudo chown www-data:www-data /var/www/webdav
```

设置用户以及对应的密码文件:

```shell
# 创建用户密码文件
sudo htpasswd -c /etc/apache2/webdav.password <username>
```

<span style="color: red; font-size: 1.1em"> 注意: 后续添加新用户时一定不能加</span> `-c` <span style="color: red; font-size: 1.1em"> 参数.</span> 

创建Apache配置文件 `/etc/apache2/sites-available/webdav.conf` :

```Apache
Alias /dav /var/www/webdav

<Location /dav>
    DAV On
    AuthType Basic
    AuthName "WebDAV Storage"
    AuthUserFile /etc/apache2/webdav.password
    Require valid-user
</Location>
```

配置的 `Alias /dav /var/www/webdav` 表示访问 `http://IP/dav` 即可访问连接.

配置并重启:

```shell
sudo a2ensite webdav.conf
sudo systemctl restart apache2
```

Apache的主进程一直在监听 `80` 端口, 之后做URL解析, 当解析到 `/dav` 时, 会匹配到 `webdav.conf` 中的配置.

Apache和Nginx同时启用时可能会有端口冲突问题, 因为在TCP/IP协议中, 一个端口 (如80端口) 在同一个IP地址下, 同一时间只能被一个程序监听.

比较好的解决方案是使用Nginx做**反向代理**. 让Nginx守住 `80` 端口, 并根据URL路劲进行分流, 分流给Apache.

例如让Apache监听一个只有本地能访问的端口, 如 `8080`, 然后当Nginx检测到请求时 `/dav` 时, 将请求转给后端的 `http://127.0.0.1:8080/dav` 交给Apache处理.