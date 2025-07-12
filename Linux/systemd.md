# 现有服务启动

启动服务 (立刻启动), 以`SSH`为例:
```bash
systemctl start ssh
```

设置开机启动服务 (不会立刻启动), 以`SSH`为例:
```bash
systemctl enable ssh
```

取消开机启动服务 (不会立刻失能), 以`SSH`为例:
```bash
systemctl disable ssh
```

查询开机自启动状态, 以`SSH`为例:
```bash
systemctl is-enabled ssh
```

# 自定义服务启动

如设计一个`shell`脚本控制`LED`的开关:
```bash
root@rock-3c:/usr/local/bin# cat start_led.sh 
#!/bin/bash

LED_PATH="/sys/class/leds/user-led1"

echo "Booting LED blink..."

echo none > $LED_PATH/trigger

for i in {1..3}; do
	echo 255 > $LED_PATH/brightness
	sleep 1
	echo 0 > $LED_PATH/brightness
	sleep 0.2
done
```

需要注意, 首行的`#!/bin/bash`为`shebang`, 用于告知内核使用什么解释器执行脚本, 不能删除.

赋予执行权限:
```bash
chmod +x start_led.sh
```

创建`systemd`服务文件:
```bash
root@rock-3c:/etc/systemd/system# cat start_led.service 

[Unit]
Description=Blink LED on boot
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/start_led.sh
RemainAfterExit=true 

[Install]
WantedBy=multi-user.target
```

启用服务:
```bash
sudo systemctl enable start_led.service
```

查看服务状态:
```bash
sudo systemctl status start_led.service
```

# 自定义关机服务

设计`shell`脚本:
```bash
#!/bin/bash
LED_PATH="/sys/class/leds/user-led1"

echo none > $LED_PATH/trigger

while true; do
	echo 255 > $LED_PATH/brightness
	sleep 0.05
	echo 0 > $LED_PATH/brightness
	sleep 0.05
done
```

创建`systemd`服务文件:
```bash
[Unit]
Description=Blink LED before shutdown
DefaultDependencies=no
Before=poweroff.target halt.target reboot.target
Conflicts=shutdown.target
  
[Service]
Type=simple
ExecStart=/usr/local/bin/halt_led.sh
Restart=always
  
[Install]
WantedBy=halt.target reboot.target poweroff.target
```

重新加载`systemd`配置:
```bash
sudo systemctl daemon-reload
```

清除熔断状态:
```bash
systemctl reset-failed poweroff_led.service
```

测试脚本服务是否正常工作:
```bash
systemctl start halt_led.service
systemctl stop halt_led.service
```

启动服务:
```bash
systemctl enable halt_led.service
```

查看全部masked的服务
```bash
systemctl list-unit-files --state=masked
```

查看全部已加载服务
```bash
systemctl list-units --type=service
```

查看全部服务(包含未启动)
```bash
systemctl list-unit-files --type=service
```