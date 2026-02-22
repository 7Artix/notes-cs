# `du` `sort`

查看当前目录下所有对象的磁盘占用, 并从大到小排列

```shell
du -sh * | sort -rh
```

`du`:
- `-s` 等价于 `-d 0` 即不递归操作, 深度为0
- `-h` 人类可读大小
- `*` 通配符, 当前目录下所有对象, 否则计算当前目录总磁盘占用

`sort`:
- `-r` reverse, 从大到小 (默认从小到大).
- `-h` 根据人类可读大小排序 (分析M, G等缩写).


# `open`

当在系统目录直接open目录时, 有可能会识别为打开一个App而报错, 使用指定用于打开的App来打开目录:

```shell
open -a Finder ~/Library/Containers/com.tencent.xinWeChat
```

- `-a` 参数指定使用 `Finder` 打开目录.

