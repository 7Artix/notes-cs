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

