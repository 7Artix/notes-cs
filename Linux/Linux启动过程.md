SoC Reset
  ↓
BootROM (BL1)
  ↓
idbloader.img（包含 SPL/DDR 初始化） (BL2)
  ↓
bl31.bin（ATF） (BL31, EL3)
  ↓
u-boot.itb → U-Boot proper (BL33, EL2)
  ↓
Linux Kernel (EL1)
  ↓
用户空间 (EL0)

| **名称** | **含义**                                 |
| ------ | -------------------------------------- |
| BL1    | Bootloader Level 1（BootROM）            |
| BL2    | Bootloader Level 2（SPL）                |
| BL31   | Bootloader at Exception Level 3（EL3）   |
| BL32   | Bootloader at Secure World（例如 OP-TEE）  |
| BL33   | Bootloader at Non-secure World（U-Boot） |

| **阶段** | **内容**               | **文件**                   | **需要构建** | **合并后生成**       |
| ------ | -------------------- | ------------------------ | -------- | --------------- |
| BL1    | SoC 固化 BootROM       | 无                        | ❌        | 无               |
| BL2    | SPL + TPL(DDR init)  | u-boot-spl.bin + ddr.bin | ✅        | ✅ idbloader.img |
| BL31   | ARM Trusted Firmware | bl31.bin                 | ✅        | 合并进 .itb        |
| BL33   | U-Boot Proper        | u-boot.bin               | ✅        | 合并进 .itb        |
| 最终镜像   | BL31 + BL33 + dtb    | u-boot.itb               | ✅        | ✅               |

ATF (ARM Trusted Firmware) 是运行在EL3特权级的bootloader framework, 也叫BL31.
它的职责是：
•	初始化系统运行环境（尤其是 Secure World 和 Non-Secure World 的切换）
•	设置异常向量表（EL3）
•	初始化 CPU 电源管理、热插拔等（PSCI）
•	在系统从 EL3 切换到 EL2/EL1 前负责安全检查和准备
•	有的 SoC 还把 DDR 初始化、串口调试也塞在这里（比如 Rockchip）

TPL程序和BL31可从芯片厂直接获取; 例如下载rockchip提供的rkbin.
rk3566_ddr_1056MHz_D4_LP4_4x_eyescan_v1.23.bin
rk3568_bl31_v1.44.elf / rk3568_bl31_ultra_v2.17.elf

TPL = Tertiary Program Loader(三级引导程序), 作用为初始化DDR控制器, 某些SoC不支持SPL直接初始化DRAM, 因此需要插入TPL.

u-boot-spl.bin由U-Boot编译生成
例如rk356x_ddr_1056MHz_v1.13.bin 从芯片厂仓库获取

设备树在BL33阶段开始解析, 最终会合并到.itb文件中. 构建U-Boot的时候需要准备好dtb文件.

构建U-Boot
1. 准备好上述两个rkbin, 设置变量
	- export ROCKCHIP_TPL=/path/to/rk3566_ddr_1056MHz_D4_LP4_4x_eyescan_v1.23.bin
	- export BL31=/path/to/rk3568_bl31_ultra_v2.17.elf
2. 初始化配置
	- make evb-rk3568_defconfig
3. 构建
	- make -j$(nproc)

最终输出文件: idbloader.img, u-boot.itb.

使用`mkimage`检查`.itb`文件是否正确.

烧写到磁盘, 首先确认disk位置, 如`/dev/disk4`
在`macOS`下使用如下命令烧写:
```bash
sudo dd if=idbloader.img of=/dev/disk4 bs=32k seek=1 conv=sync status=progress
sudo dd if=u-boot.itb of=/dev/disk4 bs=32k seek=512 conv=sync status=progress
```

查看是否烧写正确, 分别在offset 32KB和offset 16MB
```bash
sudo xxd -l 512 -s $((32*1024)) /dev/disk4
sudo xxd -l 512 -s $((32*1024*512)) /dev/disk4
```

# 整体流程概览

Linux启动流程如下:

|       嵌入式`Linux`启动流程       |          PC平台`Linux`启动流程          |
| :------------------------: | :-------------------------------: |
|          `SoC`上电           |                上电                 |
|         `ROM Code`         |            `UEFI/BIOS`            |
| `bootloader`加载 (如`U-Boot`) |     `bootloader`加载 (如`grub`)      |
|         `kernel`启动         |            `kernel`启动             |
|         挂载`rootfs`         | 挂载`rootfs`使用 (`initrd或initramfs`) |
|         执行`/init`          |             执行`/init`             |
|      读取`fstab`挂载其他分区       |          读取`fstab`挂载其他分区          |
|           用户空间运行           |              用户空间运行               |

## 上电阶段

`SoC`在`PMIC`的控制下依次上电, 通常需要多个电压轨, 并且上电顺序必须遵守规范, 否则会无法启动甚至烧毁芯片. 
上电顺序通常是: 
`VDD_IO -> VDD_CORE -> VDD_CPU -> VDD_DRAM -> VDD_GPU`

## `ROM Code`和`UEFI/BIOS`阶段

嵌入式平台主要使用`ROM Code`, 存储于`SoC`内部, 是简洁不可变的硬件引导方案; 
PC平台主要使用`UEFI`, `BIOS`是老旧标准, 现代较少使用, 均存储于`SPI Flash` (主板), 是复杂且可以更新的平台固件. 
`ROM Code`和`UEFI`都是上电后立即执行, 都是系统启动第一阶段的控制程序.

嵌入式平台可以通过`ROM Code`直接导引加载`bootloader`程序. 
PC平台需要通过`UEFI`程序完成加载`bootloader`前的一系列工作, 如: 识别硬盘, 读取分区表, 挂载`FAT`分区, 查找`UEFI`程序文件. 对于PC平台, `bootloader`例如`grubx64.efi`程序本身也是一个`UEFI`程序

## `bootloader`加载阶段

嵌入式平台常使用`U-Boot`, 拓展名为`.img`.
PC平台常使用`GRUB(GRand Unified Bootloader)`, 拓展名为`.efi`.

嵌入式系统在加载`bootloader`前要先经过`SPL(Secondary Program Loader)`阶段, 负责初始化`DRAM`, 然后引导到`bootloader`程序.
`SPL` 是一个精简版 `bootloader` (如 `u-boot-spl.bin`), 主要负责初始化内存控制器, 加载完整 U-Boot, 存在于无法直接运行完整 (`bootloader`) 的平台.

在`bootloader`程序加载过程中, 会进行如下工作:
1. 初始化基础硬件
	- 初始化`串口`, `DRAM`, `定时器`, `GPIO`等设备 .
2. 决定启动模式
	- 检查`按键` / `GPIO` / `环境变量`, 决定进入`正常启动` / `升级模式` / `boot shell`.
3. 挂载存储设备
	- 识别启动设备, 如`TF卡`, `eMMC`, `NAND Flash`等.
	- 挂载分区, 读取文件系统内容.
4. 加载关键文件
	- 加载`kernel`镜像 (`zImage`, `uImage`, `Image`, `vmlinuz`).
	- 加载设备树文件 (`*.dtb`).
	- 加载`initrd` / `initramfs`(嵌入式平台通常不需要).
5. 设置启动参数
	- 设置`bootargs`, 如 `console=ttyS0`, `root=UUID=0123abcd...`等
6. 调用启动命令

`bootloader`加载过程中, 会读取文件系统内的`extlinux.config`文件 (`U-Boot`), 来加载必要内容. 
`initrd`和`initramfs`作用是协助`kernel`挂载`rootfs`, 是一个mini环境, 二者二选一, 嵌入式平台通常不需要, 现代解决方案中通常仅使用`initramfs`.
`zImage`: `gzip`压缩的`Linux`内核镜像
`uImage`: `U-Boot`专用格式, 包含内核及头信息 (使用`mkimage`工具生成)
`vmlinuz`: 通常是压缩内核的别名,在 PC 上常见

## `kernel`启动阶段与挂载`rootfs`阶段

`kernel`启动阶段主要完成如下步骤:
1. `kernel`解压
2. 启动
3. 初始化
4. 挂载`rootfs`
5. 进入用户空间

`kernel`解压动作是由`kernel`自己完成的, `bootl`仅负责将压缩`kernel`镜像加载到内存中. 
`kernel`解压后会自动跳转到解压后的内核入口`start_kernel()`

在初始化阶段, `kernel`会进行如下动作:
1. 内存架构相关初始化
	- 建立页表 (`MMU`)
	- 配置缓存, 异常向量, `IRQ`中断向量表等
	- 初始化内核堆
2. `console`初始化
	- 根据`bootargs`设置串口输出, 如`console=ttyS0`
3. 设备树解析
	- 读取设备树`.dtb`
	- 注册平台信息, 地址映射, 中断控制器等信息
4. 驱动模型框架初始化
	- 初始化 `bus`, `class`, `device` 的`driver core` 框架
	- 注册 `subsystem`，如: `block`, `net`, `input`, `sound`等
5. 初始化内核子系统
	- 内存管理子系统 (`MM`), 进程管理
	- 文件系统
	- 块设备
6. 加载`initramfs`模块
	- 如果使用`initramfs`模块会加载驱动
7. 挂载`rootfs`
	- 根据`bootargs`设置寻找`root`路径, 并进行挂载.
8. 执行用户空间 `/init`
	- 挂载`rootfs`成功后执行.
	- `init`是用户空间的第一个程序, 存放在`/init`
9. 启动`init.d`或`systemd`
	- 控制服务启动.
	- `init.d`位于`/etc/init.d/`
	- `systemd`位于`/lib/systemd/`

自此, `Linux`系统正式启动.
挂载根文件系统后, `init`或 `systemd`会根据`/etc/fstab`内容挂载其他分区.
