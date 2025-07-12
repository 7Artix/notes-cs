# `buildroot`配置

首先`git clone`:
```bash
git clone -b 2025.02.x https://github.com/buildroot/buildroot.git
```

进入`buildroot`目录, 开始配置:
```bash
cd buildroot
make menuconfig
```

## `Target Option`

```bash
Target Architecture (AArch64 (little endian))
Target Architecture Variant (cortex-A55)
Floating point strategy (FP-ARMv8)
MMU Page Size (4KB)
Target Binary Format (ELF)
```

## `Toolchain`

选择`Install glibc utilities`.

选择`Copy gconv libraries`支持多语言字符.
在`Gconv libraries to copy`填写`*`以支持全部的字符集.

## `Build Options`
可以选择`Enable compiler cache`, 相同模块可以不重复构建.
`cache`目录为: `$(HOME)/.buildroot-ccache`

## `System Configurations`

`Init system (systemd)`选择`systemd`作为初始化系统.

`(/usr/bin:/usr/sbin:/bin:/sbin:/usr/local/bin) Set the system's default PATH`配置`PATH`防止找不到路径.

添加多语言支持:
```bash
(C en_US zh_CN ja_JP) Locales to keep                                  (en_US.UTF-8 zh_CN.UTF-8 ja_JP.UTF-8) Generate locale data
```

## `Linux Kernel`

