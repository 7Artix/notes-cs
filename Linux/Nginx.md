安装`nginx`
```
apt install nginx
```

`systemd`开启服务
```
systemctl start nginx
systemctl enable nginx
```

更改`nginx`配置:
`/etc/nginx/sites-available/default`

建立软链接到`enabled`用以启动:
```
sudo ln -s /etc/nginx/sites-available/led-control /etc/nginx/sites-enabled/
```

重新加载:
```
sudo nginx -t # 检查配置文件语法
sudo systemctl reload nginx
```

访问日志:
```bash
tail -f /var/log/nginx/access.log
```
