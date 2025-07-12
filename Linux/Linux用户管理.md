
在`Linux`系统中非管理员用户不能随意修改系统设置, 活其他用户的数据.
# 查看所有用户信息

使用如下命令查看全部用户的信息:
```bash
cat /etc/passwd
```

输出信息格式如下:
```bash
root:x:0:0:root:/root:/bin/bash
```
- `root`用户名
- `x`密码不可见
- `0`用户ID
- `0`主组ID
- `root`用户全名或描述
- `/root`用户主目录
- `/bin/bash`用户默认`shell`程序

或者使用如下命令仅查看仅输出用户名:
```bash
cut -d: -f1 /etc/passwd
```

使用如下命令查看所有组定义:
```bash
cat /etc/group
```

使用如下命令查看单个用户所属的所有组:
```bash
groups username
```

或者使用`id`命令获得更详细的信息:
```bash
id username
```

使用如下命令的时候可以看到一个文件的归属信息:
```bash
ls -lh /
```

输出信息样本如下:
```bash
lrwxrwxrwx   1 root root    7 Jul 24  2024 bin -> usr/bin
```
- `lrwxrwxrwx`代表文件类型和权限, `l`表示是符号链接
- `1`为硬链接数
- `root`表示所有者用户名
- `root`表示所有者所属组
- `7`表示文件大小, (此处为符号链接目标路径长度)
- `7 Jul 24  2024`为最后修改时间
- `bin -> usr/bin`为文件名和链接目标

# 创建新用户

执行如下命令创建新用户:
```bash
sudo useradd -m -s /bin/bash username
```

- `-m`命令表示在`/home`目录下创建用户名同名目录
- `-s /bin/bash`指定用户登录时使用的`shell`程序

为新用户设置密码:
```bash
sudo passwd username
```

输入两次密码进行确认

修改账户密码:
```bash
sudo passwd root
```

将用户添加到`sudo`组:
```bash
sudo usermod -aG sudo newuser
```

# 切换用户

使用如下命令切换用户:
```bash
su - username   # 切换到该用户的 shell 环境
```

或者如果有`sudo`权限, 可以切换到`root`用户再切换到其他用户:
```bash
sudo -i         # 切换到 root 用户（获得完整 shell）
sudo -u username -i   # 切换到指定用户
```

没有`sudo`权限的用户不能使用`sudo`命令, 尝试使用`sudo`命令会被系统记录.

换用账户登录后会启用该用户对应的`shell`程序.
可以使用如下命令修改用户默认的shell程序:
```bash
chsh -s /bin/zsh username
```

# 删除用户

使用如下命令删除用户 (但保留主目录):
```bash
sudo deluser username
```

使用如下命令删除用户 (同步删除主目录):
```bash
sudo deluser --remove-home username
```

从组中删除某个用户:
```bash
sudo deluser username groupname
```
