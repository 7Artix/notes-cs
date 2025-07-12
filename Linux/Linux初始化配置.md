# `Locales`

`locale`数据文件放置于`/usr/lib/locale/`, 由`locale-gen`命令生成.
`locale-gen`命令根据`/etc/locale.gen`里的条目生成对应的`locale`数据文件.

| **变量名**     | **含义**                           |
| ----------- | -------------------------------- |
| LANG        | 全局默认语言设置, 最主要的变量, 仅允许单一语言        |
| LANGUAGE    | 翻译语言的优先级列表（仅影响语言翻译）              |
| LC_CTYPE    | 字符分类（能否显示中文、是否支持中文输入）            |
| LC_NUMERIC  | 数字格式, 如小数点是.还是,                  |
| LC_TIME     | 时间格式, 如”2025-07-03”或”03/07/2025” |
| LC_COLLATE  | 排序规则, 比如 aB vs Ab 谁在前            |
| LC_MONETARY | 货币单位格式                           |
| LC_MESSAGES | 程序消息语言（比如错误提示语言）                 |
| LC_PAPER    | 纸张尺寸 A4 / Letter                 |
| LC_ALL      | 一键覆盖以上所有的设置, 优先级最高, 一般不建议设置      |
`update-locale`用于更新系统`locale`配置, 本身不生成任何文件, 仅修改`/etc/default/locale`的内容.

# `Time Zone`

在有`systemd`的系统下, 使用如下命令设置时区:
```bash
timedatectl set-timezone Asia/Shanghai
```

查看可用时区:
```bash
timedatectl list-timezones
```

设置时钟同步:
```bash
timedatectl set-ntp true
```

查看当前时间:
```bash
date
```
或
```bash
root@pedal:/etc/default# timedatectl
               Local time: Thu 2025-07-03 23:44:06 CST
           Universal time: Thu 2025-07-03 15:44:06 UTC
                 RTC time: Thu 2025-07-03 15:44:06
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

若没有`systemd`, 使用:
```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

设置本地consol默认字体:
```bash
# /etc/default/console-setup
# CONFIGURATION FILE FOR SETUPCON

# Consult the console-setup(5) manual page.

ACTIVE_CONSOLES="/dev/tty[1-6]"

CHARMAP="UTF-8"

CODESET="guess"
FONTFACE="Terminus"
FONTSIZE="6x12"

VIDEOMODE=

# The following is an example how to use a braille font
# FONT='lat9w-08.psf.gz brl-8x8.psf'
```
运行:
```bash
setupcon
```

# 包管理器

可以安装[`Flathub`](https://flathub.org/setup/Debian)包管理器
