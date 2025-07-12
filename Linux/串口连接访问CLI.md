# 首先查看设备信息:

```bash
ls /dev/
```

找到串口设备的设备名, 例如:
```bash
cu.wchusbserial59090476451
```

> 关于`cu`设备和`tty`设备的区别
在Linux系统中, 每个串行设备会在`/dev`目录下出现两次, 分别是`tty`和`cu`两类. `tty`设备用于接收来自UNIX系统的呼叫, 而`cu` (Call-Up) 设备则用于从系统拨出, 例如调制解调器. 当需要从Mac拨出时，应使用`/dev/cu.*`。主要区别在于`tty`设备等待DCD (数据载波检测) 信号，而`cu`设备不设置DCD, 因此会立即连接. 例如, 插入USB串口转换器后, 将列出多个串口设备, 包括`tty`和`cu`两种. 

# 使用`picocom`进行连接

```bash
picocom -b 1500000 -d 8 /dev/cu.wchusbserial59090476451
```

其中, `1500000`是波特率, `-d 8` 为默认参数, 可以选择不写.

退出:
`^A`进入转移模式, `^Q`退出.

# 使用`minicom`进行连接

`picocom`对长指令显示效果有问题, 可以使用`minicom`进行连接. 

输入如下指令进入`minicom`的配置页面:
```bash
sudo minicom -s
```

在`Serial port setup`选项菜单中进行修改, 注意修改如下内容:
- 串口设备名
- 波特率, 如`1500000 8N1 (8位数据位, 无校验, 1停止位)`
- 关闭硬件流控, 将`Hardware Flow Control`设置为`No`
- 关闭软件流控, 将`Software Flow Control`设置为`No`
保存设置`Save setup as dfl`
返回终端

使用如下命令启动`minicom`:
```bash
sudo minicom
```

若要退出`minicom`, 按`ESC+Z`
