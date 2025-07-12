# 从TF卡转移到EMMC

创建bootloader部分以及两个分区: 
- FAT32分区用来存储Linux内核(vmlinuz)以及设备树等文件
- EXT4分区作为主分区
- 另外要留前8MB的裸空间用于存放bootloader

---
## 步骤1: 创建eMMC分区

**注意: `1 block = 1 sector = 512B`**

首先用 `lsblk`来确认eMMC的序号, 确定后续要操作的对象

使用`fdisk`交互式命令创建分区

进入工具:
```bash
sudo fdisk /dev/mmcblk0
```

创建新分区表:
```bash
Command (m for help): g  #输入'g'标识创建GPT分区表
```

此处输出例如: 
```bash
Created a new GPT disklabel (GUID: F6129424-0EA9-9F4C-BE0F-E23818FF387A).
```

开始创建分区, 依次输入以下指令: 
```bash
g        # 创建新的 GPT 分区表
n        # 新建分区
1        # 默认分区编号 1
32768    # 起始扇区空出32768 sector, 为bootloader空余空间(16MB)
+300M    # 设置分区大小为 300MB
t        # 设置类型
1        # 选择分区 1
1        # 设为 EFI System
n        # 创建第二个分区
[Enter]  # 默认编号 2
[Enter]  # 默认起始扇区
[Enter]  # 使用默认到末尾 (占满剩余空间) 
p        # 打印查看分区情况
w        # 写入保存
```

当打印分区时, 应当输出类似:
```bash
Welcome to fdisk (util-linux 2.36.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
  
Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xabf51291.
  
Command (m for help): g
Created a new GPT disklabel (GUID: B60A0A8E-7361-8D49-8DF6-A78B099AEA29).
  
Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-122224606, default 2048): 32768
Last sector, +/-sectors or +/-size{K,M,G,T,P} (32768-122224606, default 122224606): +300M
  
Created a new partition 1 of type 'Linux filesystem' and of size 300 MiB.
  
Command (m for help): t
Selected partition 1
Partition type or alias (type L to list all): 1
Changed type of partition 'Linux filesystem' to 'EFI System'.
  
Command (m for help): n
Partition number (2-128, default 2): 2
First sector (2048-122224606, default 647168): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (647168-122224606, default 122224606): 
  
Created a new partition 2 of type 'Linux filesystem' and of size 58 GiB.
  
Command (m for help): p
**Disk /dev/mmcblk0: 58.28 GiB, 62579015680 bytes, 122224640 sectors**
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: B60A0A8E-7361-8D49-8DF6-A78B099AEA29
  
**Device**          **Start**       **End**   **Sectors**  **Size** **Type**
/dev/mmcblk0p1  32768    647167    614400  300M EFI System
/dev/mmcblk0p2 647168 122224606 121577439   58G Linux filesystem
  
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

随后可以使用`lsblk`指令来查看分块状况, 应当输出类似: 
```bash
NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
mtdblock0     31:0    0    16M  0 disk 
mmcblk1      179:0    0 238.8G  0 disk 
├─mmcblk1p1  179:1    0    16M  0 part /config
├─mmcblk1p2  179:2    0   300M  0 part /mnt
└─mmcblk1p3  179:3    0 238.4G  0 part /
mmcblk0      179:32   0  58.3G  0 disk 
├─mmcblk0p1  179:33   0   300M  0 part 
└─mmcblk0p2  179:34   0    58G  0 part 
mmcblk0boot0 179:64   0     4M  1 disk 
mmcblk0boot1 179:96   0     4M  1 disk 
zram0        254:0    0   1.8G  0 disk [SWAP]
```

此外, 也可以使用`fdisk`指令来查看:
```bash
fdisk -l
```


**如果需要重新创建分区表以及分区, 可以使用下述方案**:
### 重新创建分区: 全部写`0`

使用如下命令进行全盘逐字节写`0`:
```bash
sudo dd if=/dev/zero of=/dev/mmcblk0 bs=1M
```

### 重新创建分区: 删除头尾分区表

使用如下命令删除头部分区表, 以及尾部备份分区表:
```bash
# 清前 16MB，覆盖 GPT, bootloader, MBR 等
sudo dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=16

# 获取总扇区数
sectors=$(sudo blockdev --getsz /dev/mmcblk0)

# 清除尾部 33 扇区（备份 GPT header + entries）
sudo dd if=/dev/zero of=/dev/mmcblk0 bs=512 seek=$((sectors - 33)) count=33
```

### 重新扫描分区信息

在删除分区表后, 使用如下命令重新扫描分区信息:
```bash
sudo partprobe
```

---
## 步骤2: 格式化分区并挂载

格式化新建的分区. 

首先安装`mkfs.vfat`工具:
```bash
apt update
apt install dosfstools
```

开始格式化:
```bash
sudo mkfs.vfat -F 32 /dev/mmcblk0p1
sudo mkfs.ext4 /dev/mmcblk0p2
```

可以使用如下命令确认格式化是否正确:
```bash
lsblk -f
```

> **内核在启动时根据启动参数决定挂载分区, 例如在`extlinux.config`通过UUID来决定挂载分区**

创建临时的挂载点, 用来将现有系统文件复制到eMMC新建的分区内:
```bash
sudo mkdir -p /mnt/boot
sudo mkdir -p /mnt/rootfs
```

挂载分区:
```bash
sudo mount /dev/mmcblk0p1 /mnt/boot
sudo mount /dev/mmcblk0p2 /mnt/rootfs
```

如果需要卸载分区使用:
```bash
sudo umount /mnt/boot
sudo umount /mnt/rootfs
```

验证挂载结果:
```bash
df -hT
```

查看分区的`UUID`:
```bash
blkid
```

或通过如下命令查看`UUID`:
```bash
ls -l /dev/disk/by-uuid/
```

可以为分区添加`label`便于识别:
```bash
# 设置 LABEL
e2label /dev/mmcblk0p2 rootfs
fatlabel /dev/mmcblk0p1 boot  # 如果系统支持

# 设置 PARTLABEL (可选)
parted /dev/mmcblk2
(parted) name 1 boot
(parted) name 2 rootfs
```

---
## 步骤3: 复制现有系统文件到目标分区

复制`rootfs`(排除虚拟目录):
```bash
rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /mnt/rootfs
```

关于`rsync`命令的参数解释如下:
- `-a`: 归档模式, 保留权限/ 符号链接/ 时间戳等
- `-A`: 保留 ACL 权限 (Access Control Lists)
- `-X`: 保留扩展属性 (如 SELinux)
- `-v`: 显示详细过程
- `--exclude`: 排除指定目录, 注意 /* 是为了不排除空目录

复制`boot`分区内容:
```bash
rsync -aAXv /boot/ /mnt/boot
```

复制完成后进行同步, 将数据确实写入到`eMMC`中, 避免保存在`cache`内:
```bash
sync
```

创建虚拟挂载点, 为后续系统服务:
```bash
mkdir -p /mnt/rootfs/{dev,proc,sys,tmp,run}
chmod 1777 /mnt/rootfs/tmp
```

`chmod 1777`作用为:
- `1( sticky bit)` : 只允许文件的拥有者或`root`删除文件. 防止用户乱删文件.
- `7`: 读写执行权限 (owner) 
- `7`: 读写执行权限 (group) 
- `7`: 读写执行权限 (other) 

修改`extlinux.conf`文件, 使系统能够正常挂载分区:
将`append root=UUID=`后的分区UUID设置为新创建的`EXT4`分区的`UUID`
修改`kernel`以及`initrd`的路径, 由于有`EFI`分区, 因此需要用相对路径进行索引, 基于`extlinux/`

文件类似于:
```bash
## /boot/extlinux/extlinux.conf
##
## IMPORTANT WARNING
##
## The configuration of this file is generated automatically.
## Do not edit this file manually, use: u-boot-update
  
default l0
menu title U-Boot menu
prompt 1
timeout 10
  
  
label l0
        menu label Debian GNU/Linux 11 (bullseye) 5.10.160-36-rk356x
        linux ../vmlinuz-5.10.160-36-rk356x
        initrd ../initrd.img-5.10.160-36-rk356x
        fdtdir /usr/lib/linux-image-5.10.160-36-rk356x/
  
        append root=UUID=c0bdea87-bd3b-480a-a7ad-1d4e3239d0a0 console=ttyFIQ0,1500000n8 quiet splash loglevel=4 rw earlycon consoleblank=0 consol1...
  
label l0r
        menu label Debian GNU/Linux 11 (bullseye) 5.10.160-36-rk356x (rescue target)
        linux /boot/vmlinuz-5.10.160-36-rk356x
        initrd /boot/initrd.img-5.10.160-36-rk356x
        fdtdir /usr/lib/linux-image-5.10.160-36-rk356x/
        append root=UUID=c0bdea87-bd3b-480a-a7ad-1d4e3239d0a0 console=ttyFIQ0,1500000n8 splash loglevel=4 rw earlycon consoleblank=0 console=tty1e...
```

修改`/etc/fstab`中`UUID`的值.

可以使用如下命令查看文件夹整体空间占用: 
```bash
du -sh /mnt
```

完成后可以使用`umount`进行卸载.

---
## 步骤4: 复制`bootloader`

从`TF卡`中将`bootloader`复制到`eMMC`的裸设备内.
首先确认好分区与设备名称, 防止搞反顺序:
```bash
lsblk
```

输出如下:
```bash
root@Marimba:/# lsblk
NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
mtdblock0     31:0    0    16M  0 disk 
mmcblk1      179:0    0 238.8G  0 disk 
├─mmcblk1p1  179:1    0    16M  0 part /config
├─mmcblk1p2  179:2    0   300M  0 part /boot/efi
└─mmcblk1p3  179:3    0 238.4G  0 part /
mmcblk0      179:32   0  58.3G  0 disk 
├─mmcblk0p1  179:33   0   300M  0 part 
└─mmcblk0p2  179:34   0    58G  0 part 
mmcblk0boot0 179:64   0     4M  1 disk 
mmcblk0boot1 179:96   0     4M  1 disk 
zram0        254:0    0   1.8G  0 disk [SWAP]
```

如上`mmcblk0`是`eMMC`设备, `mmcblk1`是`TF卡`设备.
执行如下命令:
```bash
dd if=/dev/mmcblk1 of=/dev/mmcblk0 bs=512 skip=64 seek=64 count=32704 conv=fsync
```

dd命令是`Linux`中低级数据复制工具, 可以在块设备上进行原始数据写入.
参数解释如下:

|      **参数**       |                 **含义**                  |
| :---------------: | :-------------------------------------: |
| `if=/dev/mmcblk1` |            输入设备, 即 TF 卡整块设备             |
| `of=/dev/mmcblk0` |             输出设备, 即目标 eMMC              |
|     `bs=512`      |           块大小为 512 字节 (标准块大小)           |
|     `skip=64`     | 从`TF卡`第 64 块 (即32KB) 开始读, `rockchip`的规范 |
|     `seek=64`     |       从`eMMC`第 64 块 (即 32KB) 开始写        |
|   `count=32704`   |           复制 32704 个块, 需要实际考虑           |
|   `conv=fsync`    |      写入后强制刷新缓存, 确保数据写入到`eMMC`实体硬件       |

---
## 知识点记录
### GPT分区表的结构
**GPT（GUID Partition Table）的基本结构如下：**
```bash
| Protective MBR (LBA 0) |
| Primary GPT Header (LBA 1) |
| GPT Partition Entries (LBA 2–33 or more) |
| ... 分区内容 ... |
| Backup GPT Partition Entries (倒数第 33 LBA 开始) |
| Backup GPT Header (最后一个 LBA) |
```
除了头部的分区表外, 在尾部也有分区表备份:
- **Primary GPT Header** 位于磁盘开头 (LBA 1)
- **Backup GPT Header** 位于磁盘结尾 (最后 1 个扇区)
- **GPT Partition Entries** 也有一个备份版本, 通常倒数第 33 扇区开始 (默认 entries 占 32 个扇区)




---
# Q&A记录

> **Q: 系统加载过程中如何找到`extlinux.conf`的位置? `extlinux.conf`在`/boot`目录下, 但`/boot`是挂载点, 挂载是由`extlinux.conf`指引的, 挂载前如何读取?**  
> 
> **A: 系统启动时分阶段进行的, 不同阶段看文件系统的方式不同.  **
> 1. `bootloader`阶段, 底层启动阶段, 不依赖挂载机制, 可以直接读取FAT分区, 例如`/dev/mmcblk0p1`. 会主动读取FAT分区的指定路径, 例如`/extlinux/extlinux.conf`. 这个过程不在乎Linux系统的`/boot`是否已经挂载. 
> 2. `kernel`加载阶段, 此阶段会读取`/extlinux/extlinux.conf`, 从中找到  
> - LINUX —— `kernel` 镜像路径 (相对 FAT 分区根目录) 
> - FDT —— `dtb` 文件路径
> - INITRD —— `initrd` (如果有) 
> - APPEND —— 启动参数, **指定 `rootfs` 分区 `UUID` 或 `/dev/mmcblkXpY`**
> 3. `kernel`启动 + 挂载`rootfs`
> - Kernel 启动之后会读取 `APPEND` 里的内容: `root=UUID=xxxx 或 root=/dev/mmcblk0p2`.
> - 使用ext4 驱动挂载这个分区为 /, 成为新的根目录.
> - 后续 `init` 脚本会挂载 /proc, /dev, /sys 等内容.
