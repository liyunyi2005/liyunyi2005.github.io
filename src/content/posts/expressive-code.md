---
title: 嵌入式调试中的日志与代码规范
published: 2025-12-26
description: 用统一的日志与注释风格降低调试成本，提升固件可维护性。
tags: [嵌入式, 调试, 规范]
category: 工程实践
draft: false
---

在嵌入式项目里，问题往往不是“会不会写代码”，而是“能不能快速定位问题”。下面是一套轻量但实用的日志与代码规范，适合中小型固件团队落地。

## 日志设计原则

1. **固定格式**：让解析工具和人都能快速识别
2. **分级输出**：INFO/DBG/WARN/ERR
3. **可裁剪**：发布版本可以关闭冗余日志

```c
// 日志宏：编译期开关 + 统一格式
#define LOG_TAG "IMU"
#define LOGI(fmt, ...) printf("[I][%s] " fmt "\r\n", LOG_TAG, ##__VA_ARGS__)
#define LOGW(fmt, ...) printf("[W][%s] " fmt "\r\n", LOG_TAG, ##__VA_ARGS__)
#define LOGE(fmt, ...) printf("[E][%s] " fmt "\r\n", LOG_TAG, ##__VA_ARGS__)

void imu_init(void) {
    if (!imu_probe()) {
        LOGE("probe failed");
        return;
    }
    LOGI("init ok");
}
```

## 用状态机替代“到处 if”

复杂流程不要堆叠 if/else，用状态机能让行为可预测，日志也更清晰。

```c
typedef enum {
    STATE_IDLE,
    STATE_START,
    STATE_SAMPLE,
    STATE_ERROR,
} app_state_t;

static app_state_t state = STATE_IDLE;

void app_tick(void) {
    switch (state) {
    case STATE_IDLE:
        state = STATE_START;
        LOGI("start");
        break;
    case STATE_START:
        if (sensor_ready()) {
            state = STATE_SAMPLE;
        }
        break;
    case STATE_SAMPLE:
        if (!sensor_read()) {
            state = STATE_ERROR;
            LOGE("read failed");
        }
        break;
    case STATE_ERROR:
        // 保持在错误态，等待人工介入
        break;
    }
}
```

## 现场定位的三件事

- **串口日志**：第一手信息来源
- **逻辑分析**：用波形验证时序
- **统计计数**：发现偶发错误

```c
static uint32_t crc_err_count = 0;

if (!crc_ok) {
    crc_err_count++;
    LOGW("crc err=%lu", (unsigned long)crc_err_count);
}
```

## 交付建议

- 日志级别用宏统一控制
- 每个模块只暴露一个 `LOG_TAG`
- 保留一份“异常码表”用于现场定位

做到这些，调试效率会提升很多，项目交接也会更顺畅。
