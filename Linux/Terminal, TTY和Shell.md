# `Terminal` (终端) 

是抽象的"用户与系统交互界面"的统称.
例如串口界面, `Xterm`窗口, `SSH`连接, 都是`Terminal`.

## `Console` (控制台)

`Console`是`Terminal`的真子集, 是一种特殊的`Terminal`.
`Console`是`Linux`系统下, 用来显示内核输出信息的`Terminal`.

# `TTY` (TeleTYpewriter)

是提供`Terminal`的设备接口可能是物理层面或虚拟层面的.
例如`/dev/tty1`, `/dev/ttyS0`,`/dev/pts/0`等.

各种`TTY`类型与区别:

|     TTY名称     |   类型    |                                        说明                                        |
| :-----------: | :-----: | :------------------------------------------------------------------------------: |
|    `tty0`     | 显示设备的统称 |                                   通常是物理终端的总接口                                    |
| `tty1`~`tty6` |  虚拟终端   |                                   用于显示输出和键盘输入                                    |
|    `ttySx`    |  串口终端   |                                `x`代表串口号, 常见于嵌入式设备                                |
|   `ttyFIQ0`   | 特殊硬件串口  | `Rockchip`平台的"快速中断串口"<br>(Fast Interrupt Queue UART)<br>适合debug场景使用(默认`console`) |
|    `pts/x`    |   伪终端   |                              例如`SSH`,`Xterm`分配的伪终端                               |
可以使用如下命令判断当前使用的`tty`:
```bash
tty
```

可以使用如下命令切换`tty`:
```bash
chvt 
```

`tty`设备仅是一个通道, 其本身不负责数据的处理.

# `Shell`
是一种运行的程序, 又来接收用户的命令, 产生交互.
例如`/bin/sh`, `/bin/bash`, `zsh`等.

`shell`主要负责了以下工作:
- 读取输入: 从`tty`中读取
- 解析命令
- 启动进程: `fork()`出新的子进程用来执行命令
- 管理环境变量
- 提供脚本功能: 如`if`, `for`, 变量, 函数等, 能够编程
- 任务管理: 控制前台任务, 后台任务

如果输入`exit`命令, 会退出当前`shell`和`terminal`.

修改默认`shell`:
```bash
chsh -s $(which fish)
```
修改后重启`terminal`生效.

修改`fish`的路径显示风格为完整路径:

首先运行
```bash
funcsave fish_prompt
```

然后编辑
```bash
vim ~/.config/fish/functions/fish_prompt.fish
```

将`(prompt_pwd)`修改为`(pwd)`

撤回修改:
```bash
funcerase fish_prompt
exec fish
```
