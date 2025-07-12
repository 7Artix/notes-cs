# 使用`FFmpeg`库

安装以下包:
- `libavformat-dev` 视频封装格式处理
- `libavcodec-dev` 音视频编解码
- `libswscale-dev` 图像缩放, 色彩空间, 像素格式转换
- `libavutil-dev` 常用工具函数, 时间戳, 内存管理

包含以下头文件:
```cpp
extern "C" {
#include <libavformat/avformat.h>
#include <libavcodec/avcodec.h>
#include <libswscale/swscale.h>
}
```

# `AVFormatContext` 结构体

是`FFmpeg`核心数据结构, 表示媒体文件或者媒体流的上下文, 用于控制输入输出格式, 流信息, 时序信息等. 在打开一个媒体文件后创建, 用于管理解封装 (Demuxing), 封装 (Muxing), 读写数据包 (AVPacket), 管理媒体流, 如音频, 视频, 字幕等.

成员后备注`Demuxing only`, 意味着只在读取现有文件 (解封装) 时有效, 在写文件 (封装) 时无效. 这类成员在读取播放视频时可用.

# 时间系统
|     **名称**     |     **类型/单位**     |       **所属对象**        | **作用 / 用途**                            |
| :------------: | :---------------: | :-------------------: | :------------------------------------- |
| `AV_TIME_BASE` | 整型常量<br>值为1000000 |       FFmpeg全局        | 用于统一表示"秒",单位是**微秒(us)**                |
|  `pts / dts`   |  帧级别时间戳`int64_t`  | 每个 `AVPacket/AVFrame` | 帧的"显示时间"(pts)<br>“解码顺序时间”(dts)         |
|  `time_base`   | 结构体 `AVRational`  |     每个 `AVStream`     | 表示该流的时间单位换算比例, 比如 (1/25) 表示1单位 = 1/25秒 |
### `AV_TIME_BASE`

```cpp
#define AV_TIME_BASE 1000000
```

全局时间单位, 用于定义1"秒", 用于所有流共用的同意时间尺度.
主要用途为: 和现实时间对齐, 获取当前系统时间, 进行seek等.

### `pts / dts`

`pts`: **Presentation Time Stamp**, 播放/显示时间
`dts`: **Decoding Time Stamp**, 解码时间

是单位为`int64_t`的时间戳, 配合`time_base`使用.
```cpp
real_time = pts * time_base
```

例如:
```cpp
frame->pts = 3000;
time_base = (1/1000); // 表示一单位是 1ms
// 实际时间为 3000 * (1/1000) = 3 秒
```

`pts` 用于决定画面什么时候显示, 视频播放器依据`pts`控制帧时间.
`dts` 用于决定画面什么时候解码, 视频解码器依据`dts`控制帧处理顺序 (如存在B帧的情况).

对于不含B帧的视频, `pts == dts`.

### `time_base` 时间基准

是一个`AVRational`类型的结构体, 含有`num` (Numerator 分子) 和`den` (Denominator 分母) 两个成员. 可以根据实际需求进行设置, 比如在`25fps`时, 设置为`1/25`.

举例说明
有
```cpp
frame->pts = 4200;
stream->time_base = (1/1000); // 每单位 = 1ms
```
则该帧的显示时间是`4200 × (1/1000) = 4.2 秒`

若想跳转到第10秒, 则:
```cpp
int64_t seek_ts = 10 / av_q2d(stream->time_base);
av_seek_frame(fmtCtx, stream_index, seek_ts, AVSEEK_FLAG_BACKWARD);
```

## 几个时间的相互转换

`μs`转`PTS`
```cpp
av_rescale_q(time_us, AV_TIME_BASE_Q, stream->time_base)
// or
(time_us / AV_TIME_BASE) / av_q2d(stream->time_base)
```

`pts`转`μs`
```cpp
av_rescale_q(pts, stream->time_base, AV_TIME_BASE_Q)
// or
pts * stream->time_base * AV_TIME_BASE
```

# Demux + Decode
```
fomatCtx
	↓ av_read_frame()
packet
	↓ avcodec_send_packet()
codecCtx
	↓ avcodec_receive_frame()
frame
```

# `AVFrame`

describes decoded (raw) audio or video data.

`uint8_t *data[AV_NUM_DATA_POINTERS]` 
指针数组, 最多支持`AV_NUM_DATA_POINTERS`个通道, 每个`data[i]`指向一块通道, 或平面的数据起始地址. 
对于packed格式, 只使用`data[i]`.
对于planar格式, `data[0]/[1]/[2]`对应`R/G/B`或`Y/U/V`.

`int linesize[AV_NUM_DATA_POINTERS]`
表示`data[i]`的每一行的字节跨度.
用于访问第`y`行时: `data[i] + y * linesize[i]`

使用`RGB565`举例 (packed)
如有`128 * 160`的帧, 格式为`AV_PIX_FMT_RGB565LE`
`data[0]`为图像数据起始地址.
`linesize[0]`正常是`256 (128 * 2byte)`, 可能是`256+padding`
访问第`y`行的时候使用:
```cpp
uint8_t* row = frame->data[0] + y * frame->linesize[0];
```
