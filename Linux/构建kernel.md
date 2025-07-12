首先配置`defconfig`, 可选的`defconfig`在`./arch/arm64/configs`, 可根据不同架构设置.
使用`make xxdefconfig`进行配置.
使用`make help`查找可用的`make`选项.

`make`后`Image`路径为: `./arch/arm64/boot/Image`
`dtb`路径为: `./arch/arm64/boot/dts`

```bash
dd if=rootfs.ext2 of=/dev/sdX2 bs=1M status=progress
```

使用tar将文件解压到目标分区
```bash
mkfs.ext4 /dev/mmcblk1p2
mount /dev/mmcblk1p2 /mnt/rootfs
tar -xpf rootfs.tar -C /mnt/rootfs
```
