安装预编译静态库 + 分模块头文件

下载release的deb包
```
https://github.com/CrowCpp/Crow/releases
```

使用`dpkg -i`安装
安装后使用`dpkg -s`查看是否安装成功
使用`dpkg -L`查看头文件路径

卸载:
```bash
dpkg -r crow
```

安装单头文件:
直接下载`https://github.com/CrowCpp/Crow/releases`中的`crow_all.h`文件