
# Bluetooth的分类

Bluetooth大致可以分为 `BR/EDR` 和 `BLE`

`BR/EDR` 又称 `Classic Bluetooth` 是传统蓝牙协议, 传输速率较 `BLE` 高. 蓝牙音频 `A2DP` , 免提通话 `HFP` , 串口通信 `SPP` 等功能, 都必须通过 `BR/EDR` 蓝牙实现.
`BR/EDR` 通常使用 `Bluedroid` 协议栈

`BLE` , 即Bluetooth Low Energy, 是低功耗蓝牙, 传输速率较慢. `BLE` 对应的音频传输方案为 `Audio LE` 目前没有广泛应用, 仅有有限支持.
`BLE` 常用 `NimBLE` 协议栈.

