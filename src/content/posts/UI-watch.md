---
title: 智能手表项目要点（嵌入式方向）
published: 2025-12-26
description: 从功能拆解到低功耗与固件架构，给出可落地的设计要点。
tags: [嵌入式, 智能手表, 低功耗]
category: 项目实践
draft: false
---

# 基于FreeRTOS的嵌入式智能可穿戴设备系统设计

## 项目概述

本项目设计并实现了一个基于STM32F4微控制器和FreeRTOS实时操作系统的嵌入式智能可穿戴设备。系统集成了多传感器数据采集、健康监测算法处理、低功耗电源管理以及嵌入式图形用户界面，提供心率监测、步数统计、睡眠分析和运动追踪等功能。系统采用模块化设计，具有良好的可扩展性和实时响应能力。

## 系统架构设计

### 硬件平台配置

#### 主控制器
- **微控制器**：STM32F411CEU6 ARM Cortex-M4
- **主频**：100MHz，配备512KB Flash，128KB RAM
- **功耗特性**：支持多种低功耗模式（Sleep，Stop，Standby）

#### 传感器模块
1. **运动传感器**：MPU9250 9轴IMU
   - 三轴加速度计：±2g/±4g/±8g/±16g量程
   - 三轴陀螺仪：±250/±500/±1000/±2000°/s量程
   - 三轴磁力计：±4800μT量程

2. **心率传感器**：MAX30102光电传感器
   - 集成红光（660nm）和红外光（880nm）LED
   - 18位ADC分辨率
   - 采样率：50-3200Hz可调

3. **环境传感器**：BMP280气压温度传感器
   - 气压测量范围：300-1100hPa
   - 温度测量精度：±0.5°C
   - 低功耗模式：<1μA

#### 显示模块
- **显示屏**：1.3英寸OLED，分辨率128×64
- **接口**：SPI接口，支持硬件加速
- **触摸控制**：电容式触摸面板

## FreeRTOS实时操作系统设计

### 任务划分与调度策略

#### 任务优先级设计
系统采用固定优先级抢占式调度策略，任务优先级分配如下：

| 任务名称 | 优先级 | 执行周期 | 功能描述 |
|---------|--------|----------|----------|
| Sensor_Acquisition | 6 | 10ms | 传感器数据采集 |
| Health_Algorithm | 5 | 20ms | 健康算法处理 |
| GUI_Refresh | 4 | 33ms (30fps) | 图形界面刷新 |
| Communication | 3 | 50ms | 蓝牙数据传输 |
| Power_Management | 2 | 100ms | 电源管理 |
| System_Monitor | 1 | 1000ms | 系统状态监控 |

#### 任务间通信机制

1. **消息队列**
```c
// 传感器数据消息队列
QueueHandle_t sensor_data_queue;
sensor_data_queue = xQueueCreate(10, sizeof(SensorData_t));

// GUI更新消息队列
QueueHandle_t gui_update_queue;
gui_update_queue = xQueueCreate(5, sizeof(GUI_Update_t));
```

2. **信号量与互斥锁**
```c
// SPI总线互斥锁
SemaphoreHandle_t spi_mutex;
spi_mutex = xSemaphoreCreateMutex();

// 数据就绪信号量
SemaphoreHandle_t data_ready_sem;
data_ready_sem = xSemaphoreCreateBinary();
```

3. **事件组**
```c
// 系统事件标志组
EventGroupHandle_t system_events;
system_events = xEventGroupCreate();

// 事件定义
#define SENSOR_DATA_READY_BIT (1 << 0)
#define GUI_REFRESH_BIT (1 << 1)
#define LOW_BATTERY_BIT (1 << 2)
```

### 内存管理策略

采用FreeRTOS Heap_4内存分配方案：
- **总堆大小**：64KB
- **最小块大小**：32字节
- **碎片整理**：自动合并空闲内存块

## 健康监测算法设计

### 心率检测算法

#### 1. 信号预处理
```c
// PPG信号滤波处理
typedef struct {
    float raw_signal;
    float filtered_signal;
    float dc_component;
    float ac_component;
} PPG_Signal_t;

// 自适应带通滤波器设计
void adaptive_bandpass_filter(PPG_Signal_t* signal, float heart_rate) {
    // 根据心率动态调整截止频率
    float low_cutoff = heart_rate * 0.8 / 60.0;  // Hz
    float high_cutoff = heart_rate * 1.2 / 60.0; // Hz
    
    // 实现二阶Butterworth滤波器
    // ...
}
```

#### 2. 心率计算算法
采用自相关函数法检测心率：
```c
float calculate_heart_rate(float* ppg_signal, int sample_count, float sample_rate) {
    float max_correlation = 0;
    int best_lag = 0;
    
    // 计算自相关函数
    for (int lag = MIN_LAG; lag <= MAX_LAG; lag++) {
        float correlation = 0;
        for (int i = 0; i < sample_count - lag; i++) {
            correlation += ppg_signal[i] * ppg_signal[i + lag];
        }
        
        if (correlation > max_correlation) {
            max_correlation = correlation;
            best_lag = lag;
        }
    }
    
    // 计算心率（次/分钟）
    return 60.0 * sample_rate / best_lag;
}
```

#### 3. 心率变异性分析
```c
// 计算SDNN（正常窦性心搏间期的标准差）
float calculate_sdnn(float* rr_intervals, int count) {
    float mean = 0;
    for (int i = 0; i < count; i++) {
        mean += rr_intervals[i];
    }
    mean /= count;
    
    float variance = 0;
    for (int i = 0; i < count; i++) {
        variance += (rr_intervals[i] - mean) * (rr_intervals[i] - mean);
    }
    
    return sqrt(variance / count);
}
```

### 步数统计算法

#### 1. 加速度数据预处理
```c
// 三轴加速度合成与重力去除
typedef struct {
    float x, y, z;
    float magnitude;
    float filtered;
} Acceleration_t;

void process_acceleration(Acceleration_t* accel) {
    // 计算合加速度
    accel->magnitude = sqrt(accel->x * accel->x + 
                           accel->y * accel->y + 
                           accel->z * accel->z);
    
    // 去除重力分量（高通滤波）
    static float gravity = 9.81;
    accel->filtered = accel->magnitude - gravity;
}
```

#### 2. 步数检测算法
基于峰值检测的步数统计算法：
```c
// 步数检测状态机
typedef enum {
    STEP_IDLE,
    STEP_POSITIVE_SLOPE,
    STEP_PEAK_DETECTED,
    STEP_NEGATIVE_SLOPE,
    STEP_VALLEY_DETECTED
} Step_State_t;

int detect_steps(float* acceleration, int sample_count, float threshold) {
    int step_count = 0;
    Step_State_t state = STEP_IDLE;
    float prev_value = acceleration[0];
    
    for (int i = 1; i < sample_count; i++) {
        float current = acceleration[i];
        
        switch (state) {
            case STEP_IDLE:
                if (current > threshold && prev_value <= threshold) {
                    state = STEP_POSITIVE_SLOPE;
                }
                break;
                
            case STEP_POSITIVE_SLOPE:
                if (current < prev_value) {
                    state = STEP_PEAK_DETECTED;
                }
                break;
                
            case STEP_PEAK_DETECTED:
                step_count++;
                state = STEP_NEGATIVE_SLOPE;
                break;
                
            case STEP_NEGATIVE_SLOPE:
                if (current > threshold) {
                    state = STEP_POSITIVE_SLOPE;
                } else if (current < -threshold) {
                    state = STEP_VALLEY_DETECTED;
                }
                break;
                
            case STEP_VALLEY_DETECTED:
                if (current > -threshold) {
                    state = STEP_IDLE;
                }
                break;
        }
        
        prev_value = current;
    }
    
    return step_count;
}
```

### 睡眠质量分析算法

#### 1. 活动水平计算
基于加速度数据的活动指数计算：
```c
typedef struct {
    float activity_level;      // 活动水平
    float body_movement;       // 身体移动强度
    float position_changes;    // 体位变化次数
    float sleep_score;         // 睡眠评分
} Sleep_Analysis_t;

void analyze_sleep_pattern(Acceleration_t* accel_data, int sample_count, 
                          Sleep_Analysis_t* analysis) {
    float total_activity = 0;
    int movement_count = 0;
    
    for (int i = 0; i < sample_count; i++) {
        // 计算活动水平
        float activity = fabs(accel_data[i].filtered);
        total_activity += activity;
        
        // 检测显著运动
        if (activity > MOVEMENT_THRESHOLD) {
            movement_count++;
        }
    }
    
    analysis->activity_level = total_activity / sample_count;
    analysis->body_movement = movement_count;
    analysis->position_changes = detect_position_changes(accel_data, sample_count);
    analysis->sleep_score = calculate_sleep_score(analysis);
}
```

#### 2. 睡眠阶段识别
基于活动水平和心率变异性：
```c
typedef enum {
    SLEEP_AWAKE,
    SLEEP_REM,
    SLEEP_LIGHT,
    SLEEP_DEEP
} Sleep_Stage_t;

Sleep_Stage_t classify_sleep_stage(float activity_level, float heart_rate_variability, 
                                  float time_since_sleep_start) {
    if (activity_level > 0.5) {
        return SLEEP_AWAKE;
    }
    
    if (time_since_sleep_start < 60) {  // 入睡初期
        if (heart_rate_variability < 30) {
            return SLEEP_LIGHT;
        } else {
            return SLEEP_REM;
        }
    } else {  // 深度睡眠期
        if (heart_rate_variability < 20 && activity_level < 0.1) {
            return SLEEP_DEEP;
        } else if (heart_rate_variability > 40) {
            return SLEEP_REM;
        } else {
            return SLEEP_LIGHT;
        }
    }
}
```

## 嵌入式GUI系统设计

### LVGL图形库集成

#### 1. 显示驱动配置
```c
// 显示缓冲配置
static lv_disp_draw_buf_t draw_buf;
static lv_color_t buf_1[DISP_HOR_RES * 20];  // 行缓冲
static lv_color_t buf_2[DISP_HOR_RES * 20];  // 双缓冲

// 显示驱动初始化
void lvgl_display_init(void) {
    lv_init();
    
    // 初始化显示缓冲
    lv_disp_draw_buf_init(&draw_buf, buf_1, buf_2, DISP_HOR_RES * 20);
    
    // 配置显示驱动
    static lv_disp_drv_t disp_drv;
    lv_disp_drv_init(&disp_drv);
    disp_drv.hor_res = DISP_HOR_RES;
    disp_drv.ver_res = DISP_VER_RES;
    disp_drv.draw_buf = &draw_buf;
    disp_drv.flush_cb = my_disp_flush;
    disp_drv.user_data = &spi_handle;
    
    lv_disp_drv_register(&disp_drv);
}
```

