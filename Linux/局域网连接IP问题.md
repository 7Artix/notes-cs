
使用 `mdns` 解析 `.local` 域名, 用于连接局域网.

对应Linux安装包:

```shell
sudo apt install avahi-daemon
sudo apt install libnss-mdns
```

修改 `nsswitch` 文件 `/etc/nsswitch.conf`

```shell
hosts: files mdns4_minimal [NOTFOUND=return] dns
```

使用 `mdns4_minimal` 解析域名只会解析 `.local` 域名, 对于解析不到的 `.local` 域名直接返回, 因为后续的递归 `DNS` 也一定解析不到, 因为 `.local` 域名只会出现在局域网内.

# NSS

即 **N**ame **S**ervice **S**witch. 是Linux中负责查询各类信息的模块, 负责告诉系统, 去哪里查找对应信息, 以及寻找顺序如何.

```shell
hosts: files mdns4_minimal dns
```

代表, 在查找 `hosts` 时:
1. 查看本地的记录 `files` , 即 `/etc/hosts` 文件, 若有记录, 则直接使用.
2. 使用 `mdns4_minimal` 解析 `mdns` 
3. `DNS` 解析


