进入如下目录:
```bash
cd /sys/class/leds
```

进入需要控制的`LED`目录, 如:
```bash
cd user-led1
```

使用`ls`命令可看到目录下文件:
```bash
brightness  device  invert  max_brightness  power  subsystem  trigger  uevent
```

在超级管理员交互模式(`sudo -i`)下, 使用如下命令改变`LED`亮度:
```bash
echo 1 > brightness
```

使用如下命令控制`LED`触发功能:
```bash
echo heartbeat > trigger
```