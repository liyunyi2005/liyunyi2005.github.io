---
title: 循迹机器人：PID 控制与树莓派上位机通信
published: 2025-12-26
description: 以循迹机器人为例，介绍 PID 控制、传感器闭环与树莓派上位机通信设计。
tags: [嵌入式, 机器人, PID, 通信]
category: 项目实践
draft: false
---

循迹机器人是经典嵌入式项目，既能练控制算法，也能练通信协议。下面是一个以“下位机控制 + 树莓派上位机通信”为核心的设计拆解。

## 1. 系统架构

- **下位机**：电机驱动 + 传感器采集 + PID 控制
- **上位机（树莓派）**：路径参数下发、数据记录、调参界面
- **通信链路**：USB-UART 或 UART 直连

## 2. 传感器与控制

- 传感器：红外阵列或灰度传感器
- 误差：赛道中心线与当前偏差
- 控制：通过 PID 输出左右轮差速

```c
// 简化 PID
float pid_step(float err) {
    static float i = 0, last = 0;
    float p = 0.8f, ki = 0.02f, kd = 0.1f;
    i += err;
    float d = err - last;
    last = err;
    return p * err + ki * i + kd * d;
}
```

## 3. 上位机通信协议

建议自定义一套轻量帧格式，便于解析与校验。

```
帧头 0xAA 0x55 | LEN | CMD | DATA | CRC
CMD=0x01 设置 PID
CMD=0x02 回传传感器数据
```

树莓派端可以用 Python 快速搭建调参工具：

```python
import serial

ser = serial.Serial('/dev/ttyUSB0', 115200, timeout=0.1)

# 设置 PID 参数
frame = bytes([0xAA, 0x55, 0x04, 0x01, 0x08, 0x02, 0x01, 0x00])
ser.write(frame)
```

## 4. 调试与优化

- 用上位机实时绘制误差曲线
- 观察不同赛道下的参数稳定性
- 记录异常：漂移、抖动、饱和

## 5. 交付结果

一个可用的循迹机器人应当具备：

- 速度稳定、转弯不抖动
- 传感器异常时能自动降速
- 参数可在线调整并保存

这个项目的核心价值是“用完整链路跑通闭环控制”，对后续做更复杂的机器人项目非常有帮助。
