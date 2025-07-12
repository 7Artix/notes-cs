使用`nmcli`工具

首先扫描列出WiFi列表
```bash
sudo nmcli dev wifi list
```

连接WiFi
```bash
sudo nmcli dev wifi connect "SSID" password "PASSWORD"
```

连接后使用`ping`命令检测连接是否成功
```bash
ping 8.8.8.8
```

客户端删除旧的`known_hosts`记录 (主机端变更或`DHCP`重新分配设备)
```bash
ssh-keygen -R 192.168.1.22
```

