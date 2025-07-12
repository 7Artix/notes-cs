图形化调整设置
```bash
alsamixer
```

显示某个声卡可控参数
```bash
amixer -c 1
```

显示控件可控编号
```bash
amixer -c 1 controls
```

查看某个具体参数
```bash
amixer -c 1 cget numid=27
```

设置某个声卡的某个参数
```bash
amixer -c 1 cset numid=21 30,30
```

保存声音配置到文件:
```bash
alsactl store -f /etc/asound.state
```

从文件恢复配置:
```bash
alsactl restore -f /etc/asound.state
```

指定默认声卡:
在`/etc`下创建并编辑
```bash
# /etc/asound.conf
defaults.pcm.card 1
defaults.ctl.card 1
```

录音:
```bash
arecord -D hw:1,0 -f S16_LE -r 48000 -c 2 -d 5 test.wav
```