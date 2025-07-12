# `Makefile`测试

编写如下`Makefile`:
```makefile
#Makefile格式
#目标:依赖的文件或其它目标
#Tab 命令1
#Tab 命令2
#第一个目标, 是最终目标及make的默认目标
#目标a, 依赖于目标targetc和targetb
#目标要执行的shell命令 ls -lh, 列出目录下的内容
targeta: targetc targetb
ls -lh
  
#目标b, 无依赖
#目标要执行的shell命令, 使用touch创建test.txt文件
targetb:
touch test.txt
  
#目标c, 无依赖
#目标要执行的shell命令, pwd显示当前路径
targetc:
pwd
  
#目标d, 无依赖
#由于abc目标都不依赖于目标d, 所以直接make时目标d不会被执行
#可以使用make targetd命令执行
targetd:
rm -f test.txt
```

其中, 目标必须顶格写, 不能有空格或缩进, 而命令必须用`Tab`缩进.

在`Makefile`目录下运行`make`命令时, 会自动索引当前目录下的`Makefile`或`makefile`文件, 并根据文件内容进行解析执行. 如果要指定其他文件, 可以使用`make -f 文件名`, `-f`参数指定文件.

或者可以使用`-f`参数显式指定`makefile`文件名.

上述文件中, `targeta`是首个即默认目标, 会默认执行.
由于`targeta`依赖`targetc`和`targetb`目标, 所以在执行前会先执行`targetc`和`targetb`.

`targetd`不是默认目标, 也没有被依赖, 因此不会执行, 可以使用如下命令执行`targetd`:
```bash
make targetd
```

# `make`和`shell`脚本的区别

|     **特性**      |          Makefile          |       **Shell 脚本**       |
| :-------------: | :------------------------: | :----------------------: |
|   **避免重复工作**    |    ✅ 只重建需要更新的部分 (依赖检查)     |       ❌ 每次都全部重新执行        |
|   **自动依赖管理**    |        ✅ 有依赖图和规则机制         |        ❌ 你得手写所有逻辑        |
|   **按需执行目标**    | ✅ 只执行你请求的目标 (如 make clean) |       ❌ 通常是一整段顺序执行       |
| **模块化构建 (多目标)** |         ✅ 可以分目标构建          |     ❌ shell 脚本通常顺序化      |
|  **并行构建 (多核)**  |       ✅ make -j 自动并行       | ❌ shell 没有并行执行逻辑 (除非多进程) |

`make`的核心优势是增量构建, 通过检查依赖和依赖的时间戳来判断哪些项目需要重新构建.

以如下`Makefile`为例:
```makefile
#Makefile格式
#目标:依赖
#Tab 命令1
#Tab 命令2
#默认目标
#hello_main依赖于hello_main.c和hello_func.c文件
hello_main: hello_main.c hello_func.c
	gcc -o hello_main hello_main.c hello_func.c -I .
	
#clean目标, 用来删除编译生成的文件
clean:
	rm -f *.o hello_main
```

在`make`的时候, 会检查构建依赖`hello_main.c`和`hello_func.c`的时间戳, 如果有更新, 则会重新构建, 如果没有更新, 则不会重新构建.

在`Makefile`中编写的目标, 对于`make`来说实际是一个文件. 如果目标目录真的存在和`target`同名的文件, 且文件和依赖都是新的, 那么就会出现`make`不会再次构建的误会.
比如目录中真的存在一个叫`clean`的文件, 且其是新的, 那么`make clean`并不会执行本该执行的动作.
为了防止以上问题发生, 在这种不产生实际文件的`target`前, 需要先声明为`.PHONY`, 例如:
```bash
.PHONY:clean
clean:
	rm -f *.o hello_main
```

# 使用`make`和`gcc`进行编译

![[make_gcc_flowchart.png]]
如上图所示, 构建最终的`hello`文件实际依赖的是`*.o`文件, 并不是`*.c`文件, 将`*.o`文件通过链接最终得到`hello`文件.
但`make`有一条默认规则, 如果需要链接的`*.o`文件不存在, 则会编译同名的`*.c`文件.
以下是一个示例:
```makefile
hello_main: hello_main.o hello_func.o
	gcc -o hello_main hello_main.o hello_func.o
	
#以下是make的默认规则, 下面两行可以不写
#hello_main.o: hello_main.c
	#gcc -c hello_main.c
	
#以下是make的默认规则, 下面两行可以不写
#hello_func.o: hello_func.c
	#gcc -c hello_func.c
```

# 使用变量

在自动编译的过程中有如下缺陷:
- 没有指定头文件的依赖关系, 因此如果修改了头文件, 并不会更新`*.o`文件
- 默认规则使用`cc`编译, 可能和预期的编译工具链不符.

## 为变量赋值

在`make`中, 可以使用变量进行值替换, 变量的命名可以包含字符, 数字, 下划线, 变量区分大小写, 可以通过以下4种形式进行赋值:
- "=" : 延时赋值, 只有在调用的时候, 变量才会被赋值
- ":=" : 直接赋值, 变量的值在定义时就确定
- "?=" : 若变量值为空, 则进行赋值, 通常用于设置默认值
- "+=" : 追加赋值, 可以往变量后面增加新的内容

使用变量时, 用法为:
```makefile
$(VAR_NAME)
```

测试如下`makefile`:
```makefile
VAR_A = FILEA
VAR_B = $(VAR_A)
VAR_C := $(VAR_A)
VAR_A += FILEB
VAR_D ?= FILED
.PHONY:check
check:
   @echo "VAR_A:"$(VAR_A)
   @echo "VAR_B:"$(VAR_B)
   @echo "VAR_C:"$(VAR_C)
   @echo "VAR_D:"$(VAR_D)
```

以上例子运行后输出为:
```bash
VAR A:FILEA FILEB
VAR_B: FILEA FILEB
VAR C: FILEA
VAR_D: FILED
```

重点关注`VAR_B`的值, 由于其采用延时赋值, 因此, 只有在调用时, 在真正的赋值为`$(VAR_A)`, 此时值为`FILEA FILEB`.

在命令前加`@`修饰符可以是命令本身不出现在`shell`中.

现在使用变量修改之前的`gcc`编译的`makefile`:
```makefile
#定义变量
CC=gcc
CFLAGS=-I .
DEPS=hello_func.h

#目标文件
hello_main: hello_main.o hello_func.o
	$(CC) -o hello_main hello_main.o hello_func.o

#*.o文件的生成规则
%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

#伪目标
.PHONY: clean
clean:
	rm -f *.o hello_main
```

上述`makefile`中, `%.o`和`%.c`使用了通配符, `%`代表任意字符. 编译`%.o`时, 还依赖`DEPS`的变量值.

`$@`和`$<`是`makefile`的保留关键字, 是系统保留的自动化变量. 
`$@`代表目标文件, `$<`代表第一个依赖文件.
即`$@`等价于`%.o`, 而`$<`等价于`%.c`.


## 一些自动化变量

`makefile`中还有如下的一些自动化变量:

|  符号  |             意义             |
| :--: | :------------------------: |
| `$@` |           匹配目标文件           |
| `$%` | 与`$@`类似, 但`$%`仅匹配"库"类型的目标文件 |
| `$<` |        依赖中的第一个目标文件         |
| `$^` |  所有的依赖目标, 如果依赖中有重复的, 只保留一份   |
| `$+` |   所有的依赖目标, 即使依赖中有重复的也原样保留   |
| `$?` |        所有比目标要新的依赖目标        |

# 使用分支

在`makefile`中, 可以使用条件分支:
```makefile
ifeq(arg1, arg2)
分支1
else
分支2
endif
```

判断`arg1`和`arg2`的值是否相同, 若相同, 则进入分支1, 若不同, 则进入分支2.
`arg1`和`arg2`可以是常量或者变量.

如下案例使用分支判断平台:
```makefile
#定义变量
#ARCH默认为x86, 使用gcc编译器
#否则使用arm编译器
ARCH ?= x86
TARGET = hello_main
CFLAGS = -I .
DEPS = hello_func.h
OBJS = hello_main.o hello_func.o

#根据输入的ARCH变量来选择编译器
#ARCH=x86, 使用gcc
#ARCH=arm, 使用arm-gcc
ifeq ($(ARCH), x86)
CC = gcc
else
CC = arm-linux-gnueabihf-gcc
endif

#目标文件
$(TARGET): $(OBJS)
   $(CC) -o $@ $^ $(CFLAGS)

#*.o文件的生成规则
%.o: %.c $(DEPS)
   $(CC) -c -o $@ $< $(CFLAGS)

#伪目标
.PHONY: clean
clean:
   rm -f *.o hello_main
```

在实际使用时, 需要使用如下命令:
```bash
#清除编译输出, 确保不受之前的编译输出影响
make clean
#使用ARM平台
make ARCH=arm
#清除编译输出
make clean
#默认是x86平台
make
```


# 使用函数

在复杂工程中, 头文件, 源文件可能放在二级目录中, 编译生成的`*.o`或可执行文件也可能放到专门目录便于整理. 例如, `*.h`头文件放在`include`目录下, `*.c`文件放在`sources`目录下, 不同平台的编译输出分别存放在`build_x86`和`build_arm`中.
实现这些功能需要`makefile`函数.

在`makefile`中使用函数, 格式如下:
```makefile
$(函数名 参数)
#或者使用花括号
${函数名 参数}
```

下面为几个函数的介绍.

## `notdir`函数

用于去除文件路径中的目录部分. 

格式如下:
```makefile
$(notdir 文件名)
```

例如输入:
```makefile
$(notdir ./sources/hello_func.c)
```

则得到的变量值实际为`hello_func.c`.

## `wildcard`函数

用于获取文件列表, 并使用空格分隔开.

格式如下:
```makefile
$(wildcard 匹配规则)
```

例如输入:
```makefile
$(wildcard sources/*.c)
```

函数值为: `sources/hello_func.c sources/hello_main.c sources/test.c`

## `patsubst`函数

功能为模式字符串替换.

格式如下:
```makefile
$(patsubst 匹配规则, 替换规则, 输入的字符串)
```

例如:
```makefile
$(patsubst %.c, build_dir/%.o, hello_main.c )
#函数输出为:
build_dir/hello_main.o

$(patsubst %.c, build_dir/%.o, hello_main.xxx )
#由于hello_main.xxx不符合匹配规则"%.c", 所以函数没有输出
```

## 使用上述函数编写的`makefile`案例

```makefile
#定义变量
#ARCH默认为x86, 使用gcc编译器
#否则使用arm编译器
ARCH ?= x86
TARGET = hello_main

#存放中间文件的路径
BUILD_DIR = build_$(ARCH)
#存放源文件的文件夹
SRC_DIR = sources
#存放头文件的文件夹
INC_DIR = includes

#源文件
SRCS = $(wildcard $(SRC_DIR)/*.c)
#目标文件（*.o）
OBJS = $(patsubst %.c, $(BUILD_DIR)/%.o, $(notdir $(SRCS)))
#头文件
DEPS = $(wildcard $(INC_DIR)/*.h)

#指定头文件的路径
CFLAGS = $(patsubst %, -I %, $(INC_DIR))

#根据输入的ARCH变量来选择编译器
#ARCH=x86, 使用gcc
#ARCH=arm, 使用arm-gcc
ifeq ($(ARCH), x86)
CC = gcc
else
CC = arm-linux-gnueabihf-gcc
endif

#目标文件
$(BUILD_DIR)/$(TARGET): $(OBJS)
	$(CC) -o $@ $^ $(CFLAGS)

#*.o文件的生成规则
$(BUILD_DIR)/%.o: $(SRC_DIR)/%.c $(DEPS)
#创建一个编译目录, 用于存放过程文件
#命令前带"@", 表示不在终端上输出
@mkdir -p $(BUILD_DIR)
$(CC) -c -o $@ $< $(CFLAGS)

#伪目标
.PHONY: clean cleanall
#按架构删除
clean:
	rm -rf $(BUILD_DIR)

#全部删除
cleanall:
	rm -rf build_x86 build_arm
```

