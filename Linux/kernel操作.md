# 下载指定版本`kernel`
```bash
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.10.160.tar.xz
tar -xf linux-5.10.160.tar.xz
cd linux-5.10.160
```

编译`kernel`以及子模块

通过`make menuconfig`进行配置. 实际配置文件存储于`.config`
`Kconfig` (kernel config)决定某个选项出现在`menuconfig`的哪一层, 以及具体的描述和依赖等, 除非在开发新的驱动模块 (增加菜单), 否则不要调整`Kconfig`文件.

使用如下命令构建子模块:
```bash
make modules_prepare
make M=drivers/gpu/drm modules -j$(nproc)
```
- `M=路径`指定只编译该子目录下的模块
- 系统会读取该目录`Makefile`, 找到例如`- obj-$(CONFIG_TINYDRM_ST7735R) += st7735r.o`
- `modules`表示仅构建模块
- `-j$(nproc)`使用多线程

内核配置的三种状态:

| **表达式值** |    **含义**    |            **编译行为**            |
| :------: | :----------: | :----------------------------: |
|   `=y`   |  静态编译进内核镜像中  |        **不可卸载**, 启动时就加载        |
|   `=m`   | 编译为内核模块`.ko` | **可卸载**, 可用 insmod/modprobe 加载 |
|   `=n`   |     没启用      |        不会编译, 也不会出现在系统中         |

启用支持模块`=m`模式:
```bash
scripts/config --module DRM
```

`Makefile`可以加入如下描述, 匹配魔数:
```makefile
EXTRAVERSION = -36-rk356x
```

以下配置开启:
```bash
CONFIG_MODVERSIONS=y
```

将编译后得到的`*.ko`文件复制到如下目录:
```bash
/lib/modules/$(uname -r)/extra/
```

或者自动安装模块:
```bash
make M=drivers/gpu/drm modules_install
```

更新模块索引:
```bash
depmod -a
```

读取模块信息:
```bash
modinfo drm_mipi_dbi
```

尝试加载模块:
```bash
modprobe drm_mipi_dbi
```

查看已安装的外部模块:
```bash
lsmod
```