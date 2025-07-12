`GPIO`外设在`/sys/class/gpio`目录下.

```bash
root@rock-3c:/sys/class/gpio# ls
export  gpio97  gpiochip0  gpiochip128  gpiochip32  gpiochip511  gpiochip64  gpiochip96  unexport
```

- `export`: 只能写不能读, 用于导出`GPIO`, 向该文件写入编号`N`即可向内核深情将该编号`GPIO`导出到用户空间, 若内核本身并没有将该`GPIO`用于其他功能, 则会创建一个对应编号的目录, 例如`./gpio97`.
- `unexport`: 只能写不能读, 取消导出`GPIO`.
- `gpiochip*`: `GPIO`控制器. 描述一组`GPIO`引脚的管理者. 每个控制器管理32个, 64个或更多的`GPIO`引脚. 其下`ngpio`记录了管理的引脚数量
- `gpio*`: 具体`GPIO`引脚的控制目录, 包含有控制引脚的相应文件.

进入`gpio*`目录, 有如下文件:
```bash
active_low  device  direction  edge  power  subsystem  uevent  value
```

- `direction` 用于指定输入输出功能:
	- `in`输入模式
	- `out`输出模式, 默认电平不定
	- `high`输出模式, 默认输出高电平
	- `low`输出模式, 默认输出低电平
- `value`用于`GPIO`电平交互, 若为输出模式, 可修改该文件的值.
- `edge`用于配置`GPIO`的中断触发方式, 当GPIO被配置为中断时, 可以通过系统`poll`函数监听. 可取如下属性值:
	- `none`: 无中断模式.
	- `rising`: 中断输入模式, 上升沿触发.
	- `falling`: 中断输入模式, 下降沿触发.
	- `both`: 中断输入模式, 边沿触发.

一个使用`C++`编写的读取输入的案例如下:
```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <thread>
#include <chrono>

#define GPIO_PATH "/sys/class/gpio/gpio97/value"

int main() {
    while (true) {

		std::ifstream gpio_in;
		gpio_in.open(GPIO_PATH);

		if (!gpio_in.is_open()) {
			std::cerr << "Failed to open GPIO value file" << std::endl;
			return 1;
		}

		std::string value;
		gpio_in >> value;

		std::cout << "\rGPIO_97 Value: " << value << std::flush;

		std::this_thread::sleep_for(std::chrono::milliseconds(20));
	}

	return 0;
}
```

如上使用了`C++`的输入输出库.

# 使用`libgpiod`来控制`GPIO`

在新的`kernel`版本中, 推荐使用`gpiod`来控制`GPIO`.
