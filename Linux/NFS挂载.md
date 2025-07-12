# 基本信息

使用`nfs`挂载网络文件系统, 以`Linux`开发板挂载`Mac`系统下目录举例.
待挂载`Mac`的`IP`以及目录:
```bash
192.168.1.181:/Volumes/case_sensitive/mnt/rock
```

挂载到的`Linux`开发板`IP`以及目录:
```bash
192.168.1.22:/mnt/mac_nfs
```

# 配置主机端`Mac`

创建如下文件, 并添加内容, 用以配置`NFS`共享:
```bash
artix@Organ / % cat /etc/exports
/Volumes/case_sensitive/mnt/rock 192.168.1.22 -alldirs -mapall=501:20
```

- `-alldirs`参数表明可以访问目录下全部子目录
- `-mapall`将所有 NFS 用户映射为你的本地用户, `501`为`UID`, `20`为`GID`, 可以使用`id`命令查看

使用如下命令开启并检查`NFS`服务:
```bash
sudo nfsd enable
sudo nfsd start
sudo nfsd status
```

# 配置客户端`Linux`开发板

若没有安装`nfs`客户端, 使用如下命令安装:
```bash
apt install nfs-common
```

使用如下命令挂载:
```bash
mount 192.168.1.181:/Volumes/case_sensitive/mnt/rock /mnt/mac_nfs
```

修改如下文件设置开机自动挂载:
```bash
root@rock-3c:/# cat /etc/fstab
UUID=56c08754-0d8e-4689-a1e4-1ece1ccf1f25 / ext4 defaults 0 1
UUID=1F82-01D1 /config vfat defaults,x-systemd.automount 0 2
UUID=2138-7A2C /boot/efi vfat defaults,x-systemd.automount 0 2
192.168.1.181:/Volumes/case_sensitive/mnt/rock /mnt/mac_nfs nfs defaults 0 0
```

防止开机阻塞版本:
```bash
192.168.1.181:/Volumes/case_sensitive/mnt/rock /mnt/mac_nfs nfs nofail,x-systemd.automount,_netdev,timeo=5,retrans=2,nfsvers=4.0 0 0
```

- 第一个参数`0`表示不需要`dump`程序备份此文件系统
- 第二个参数`0`表示不需要进行`fsck`检查 (文件系统检查)


## `sshfs`

mac开启SSH(远程登录, 允许所有文件访问)

板卡安装`sshfs`
测试是否能够挂载:
```bash
sshfs artix@192.168.1.181:/Volumes/case_sensitive/mnt/rock /mnt/mac_sshfs
```

生成秘钥开机自动登录
```bash
ssh-keygen -t ed25519 -C "rock-3c"
```

将公钥发送至主机, 实现免密登录:
```bash
ssh-copy-id artix@192.168.1.181
```

创建一个挂载单元在`/etc/systemd/system`目录下, 命名为`mnt-mac_sshfs.mount`.
其中`mac_sshfs`是实际的挂载点, 命名必须和实际一致, 将路径中的`/`替换成`-`.
在文件中写如下内容:
```bash
[Unit]
Description=Mount Mac via SSHFS
After=network-online.target
Wants=network-online.target

[Mount]
What=artix@192.168.1.181:/Volumes/case_sensitive/mnt/rock
Where=/mnt/mac_sshfs
Type=fuse.sshfs
Options=_netdev,IdentityFile=/home/radxa/.ssh/id_ed25519,allow_other,default_permissions,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3
TimeoutSec=30,uid=1000,gid=1000 ##需要修改id, 使用id username查看

[Install]
WantedBy=multi-user.target
```

开机自动启动:
```bash
systemctl enable mnt-mac_sshfs.mount
```


或者`.service`版本
```bash
[Unit]
Description=Mount Mac via SSHFS
After=network-online.target
Wants=network-online.target
Requires=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/sshfs artix@192.168.1.181:/Volumes/case_sensitive/mnt/rock /mnt/mac_sshfs \
    -o IdentityFile=/root/.ssh/id_ed25519 \
    -o allow_other \
    -o default_permissions \
    -o reconnect \
    -o ServerAliveInterval=15 \
    -o ServerAliveCountMax=3 \
    -o uid=0,gid=0,_netdev
ExecStop=/bin/fusermount -u /mnt/mac_sshfs
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```