VOP, VP, CRTC

VOP (Video Output Processor)
VP (Video Port)
CRTC (Cathode Ray Tube Controller)

VOP是SoC中负责图像处理和输出的核心模块, 可以看做是显示引擎的大脑, 一个SoC可能有一个或者多个VOP实例, 如RK3566仅有一个VOP.
VOP负责许多任务:
- 从内存中读取图像数据
- 对图像进行缩放, 旋转, 色彩空间转换, 混合(alpha blending)等
- 对处理后的图像数据进行格式化, 以符合不同显示接口, 如HDMI, MIPI DSI, eDP等
- 生成同步信号 (如 Hsync, Vsync) 控制显示器时序.

VP是VOP内部或与之关联的**视频输出端口**或**显示通道**. 它代表VOP的一个独立的输出路径, 可以连接到不同的显示接口.
一个VOP肯能有多个VP, 如RK3566有3个VP. 一个VOP可能会同时支持多种输出, 如HDMI和MIPI DSI, 此时需要多个VP来处理多路输出.

CRTC是抽象的概念, 代表显示管道中的时序控制器. 负责:
- 设置显示模式: 如分辨率, 刷新率, 时序参数.
- 扫描输出: 控制像素如何从framebuffer读取, 并发送到显示器.
- 管理Plane: 一个CRTC可以绑定一个或多个Plane(一个Primary Plane和可选的Overlay/Cursor Plane), 将这些Plane图像混合后输出.
CRTC是软件与硬件时序控制器交互的桥梁. DRM驱动会将CRTC实例映射到VOP硬件的具体显示通道, 即CRTC与VP绑定.

`Framebuffer (像素数据)` -> `Plane (图像层)` -> `CRTC (时序控制器)` -> `VP (VOP内部视频端口)` -> `Encoder (编码器)` -> `Connector (物理接口)` -> `Display (显示器)`

VOP和CRTC的关系: 硬件 vs 软件抽象
VOP 是实际的硬件单元, 负责物理层面的图像处理和输出. 而CRTC 是操作系统 (Linux 内核) 为了管理这个硬件而创建的软件抽象, 它提供了一个标准化的接口, 让应用程序和驱动程序能够控制 VOP 的行为.

VOP和GPU的区别: 功能侧重
GPU负责画图, VOP负责显示画好的图.

使用`modetest`查看全部显示硬件信息. 可用`-M`参数指定驱动程序, 如`modetest -M rockchip`
