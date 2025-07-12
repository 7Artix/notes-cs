使用如下命令构建Debian rootfs
```bash
mmdebstrap \
  --variant=standard \
  --mode=root \
  --aptopt='Apt::Install-Recommends "true"' \
  --components=main,contrib,non-free,non-free-firmware \
  --arch=arm64 \
  --include=openssh-server,network-manager \
  bookworm ./rootfs \
  http://mirrors.tuna.tsinghua.edu.cn/debian
```

删除`rootfs/dev`内的内容
使用`chroot`进入, 然后生成`initramfs`
```bash
apt install initramfs-tools
update-initramfs -c -k 6.1.115-artificial
```

使用`tar`转移`rootfs`:
```
tar --numeric-owner -cpf rootfs.tar rootfs/
```

`--numeric-owner`防止UID/GID被用户名干扰
`-p`保留权限

使用`cp`命令拷贝`.tar`文件

在目标处解包
```bash
tar --numeric-owner -xpf rootfs.tar -C rootfs_debian
```
`-C`指定解包目标路径