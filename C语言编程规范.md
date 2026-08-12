# C 语言编程规范

> 版本：v1.0  
> 适用范围：嵌入式开发、通用C语言项目  
> 维护说明：本规范为团队强制遵循的编码标准，代码审查时以此为依据

---

## 目录

1. [文件结构规范](#1-文件结构规范)
2. [命名规范](#2-命名规范)
3. [代码格式](#3-代码格式)
4. [注释规范（Doxygen）](#4-注释规范doxygen)
5. [函数设计](#5-函数设计)
6. [变量与数据类型](#6-变量与数据类型)
7. [宏定义与常量](#7-宏定义与常量)
8. [指针与内存管理](#8-指针与内存管理)
9. [错误处理](#9-错误处理)
10. [头文件规范](#10-头文件规范)
11. [嵌入式专项规范](#11-嵌入式专项规范)
12. [代码审查清单](#12-代码审查清单)

---

## 1. 文件结构规范

### 1.1 文件命名

- 源文件：`模块名.c`，全小写，单词间用下划线分隔
- 头文件：`模块名.h`，与源文件同名
- 示例：`uart_driver.c` / `uart_driver.h`、`modbus_rtu.c` / `modbus_rtu.h`

### 1.2 源文件结构（自上而下）

```c
/**
 ******************************************************************************
 * @file    uart_driver.c
 * @author  作者姓名
 * @version V1.0.0
 * @date    2026-08-12
 * @brief   UART驱动模块实现
 ******************************************************************************
 */

/* 1. 本模块头文件 */
#include "uart_driver.h"

/* 2. 标准库头文件 */
#include <stdint.h>
#include <stddef.h>
#include <string.h>

/* 3. 第三方/平台头文件 */
#include "stm32f4xx_hal.h"

/* 4. 项目内其他模块头文件 */
#include "ring_buffer.h"

/* 5. 私有宏定义 */
#define UART_TX_BUF_SIZE    256
#define UART_RX_BUF_SIZE    512

/* 6. 私有类型定义 */
typedef struct {
    UART_HandleTypeDef *handle;
    ring_buffer_t       tx_buf;
    ring_buffer_t       rx_buf;
    volatile uint8_t    tx_busy;
} uart_ctx_t;

/* 7. 全局变量（尽量避免，必须用时加注释说明） */
static uart_ctx_t g_uart1_ctx;

/* 8. 私有函数前置声明 */
static void uart_irq_handler(uart_ctx_t *ctx);

/* 9. 公有函数实现 */
int uart_init(uart_port_t port, uint32_t baudrate)
{
    /* ... */
}

/* 10. 私有函数实现 */
static void uart_irq_handler(uart_ctx_t *ctx)
{
    /* ... */
}
```

### 1.3 头文件结构

```c
/**
 ******************************************************************************
 * @file    uart_driver.h
 * @author  作者姓名
 * @version V1.0.0
 * @date    2026-08-12
 * @brief   UART驱动模块对外接口声明
 ******************************************************************************
 */

/* 防止重复包含 */
#ifndef __UART_DRIVER_H
#define __UART_DRIVER_H

#ifdef __cplusplus
extern "C" {
#endif

/* 依赖的头文件（仅包含必要的类型定义） */
#include <stdint.h>

/* 导出宏定义 */
#define UART_PORT_1     0
#define UART_PORT_2     1

/* 导出类型定义 */
typedef enum {
    UART_OK = 0,
    UART_ERR_TIMEOUT = -1,
    UART_ERR_BUSY    = -2,
    UART_ERR_PARAM   = -3
} uart_status_t;

/* 导出函数声明 */
int uart_init(uart_port_t port, uint32_t baudrate);
int uart_send(uart_port_t port, const uint8_t *data, uint16_t len);
int uart_recv(uart_port_t port, uint8_t *buf, uint16_t len, uint32_t timeout);

#ifdef __cplusplus
}
#endif

#endif /* __UART_DRIVER_H */
```

---

## 2. 命名规范

### 2.1 总体原则

| 元素 | 规则 | 示例 |
|------|------|------|
| 宏定义 | 全大写 + 下划线 | `MAX_BUFFER_SIZE` |
| 枚举值 | 全大写 + 下划线 | `STATE_IDLE` |
| 全局变量 | `g_` 前缀 + 小驼峰/下划线 | `g_system_tick` |
| 静态变量 | `s_` 前缀 + 下划线 | `s_init_flag` |
| 函数名 | 模块前缀 + 下划线 + 动作 | `uart_send_data` |
| 局部变量 | 全小写 + 下划线 | `byte_count` |
| 结构体/类型 | 全小写 + `_t` 后缀 | `ring_buffer_t` |
| 指针变量 | 加 `p` 前缀或 `_ptr` 后缀 | `p_data` / `data_ptr` |
| 布尔变量 | `is_` / `has_` / `enable_` 前缀 | `is_ready` |

### 2.2 函数命名

- 格式：`模块名_动作_对象`
- 示例：
  - `uart_init()` — 初始化UART
  - `modbus_read_holding_registers()` — 读保持寄存器
  - `gpio_set_level()` — 设置GPIO电平
- 回调函数：`模块名_动作_callback` 或 `on_事件名`
  - `uart_rx_done_callback`
  - `on_timer_expire`

### 2.3 变量命名禁忌

- 禁止使用单字母变量（循环计数器 `i`、`j`、`k` 除外）
- 禁止使用拼音命名
- 禁止使用含糊缩写（`tmp`、`dat`、`buf` 在作用域清晰时可用）
- 变量名应体现用途而非类型（`int count;` 而非 `int int_count;`）

---

## 3. 代码格式

### 3.1 缩进与空格

- **缩进**：4个空格，禁止使用Tab（编辑器设置Tab为4空格）
- **大括号**：K&R风格（左括号不换行，右括号独占一行）

```c
/* 正确：K&R风格 */
if (condition) {
    do_something();
} else {
    do_other();
}

/* 错误：Allman风格（左括号换行） */
if (condition)
{
    do_something();
}
```

### 3.2 空格规则

```c
/* 关键字后加空格 */
if (x > 0) {
}

/* 函数名与括号之间不加空格 */
int result = calculate(a, b);

/* 运算符两侧加空格 */
int sum = a + b;
int val = (a * b) + c;

/* 逗号后加空格，逗号前不加 */
func(arg1, arg2, arg3);

/* 强制类型转换后加空格 */
uint32_t val = (uint32_t)ptr;
```

### 3.3 行宽与空行

- 单行不超过 **120字符**，超过时合理换行
- 函数之间空 **2行**
- 逻辑块之间空 **1行**
- 头文件包含区、宏定义区、类型定义区、函数区之间空2行

### 3.4 switch语句

```c
switch (state) {
    case STATE_IDLE:
        do_idle();
        break;

    case STATE_RUNNING:
        do_running();
        break;

    case STATE_ERROR:
        do_error();
        break;

    default:
        break;
}
```

### 3.5 长表达式换行

```c
/* 运算符在行尾，续行缩进对齐 */
if (condition_a && condition_b &&
    condition_c && condition_d) {
    do_something();
}

/* 函数参数过多时换行 */
ret = some_function(long_argument_name_1,
                    long_argument_name_2,
                    long_argument_name_3);
```

---

## 4. 注释规范（Doxygen）

### 4.1 文件头注释

每个文件顶部必须包含Doxygen格式的文件说明：

```c
/**
 ******************************************************************************
 * @file    modbus_rtu.c
 * @author  张三
 * @version V1.2.0
 * @date    2026-08-12
 * @brief   Modbus RTU协议栈实现
 *
 * @details 支持功能码 0x03(读保持寄存器)、0x06(写单寄存器)、
 *          0x10(写多寄存器)，采用状态机解析，非阻塞设计。
 *
 * @note    依赖 ring_buffer 模块和 systick 定时器
 ******************************************************************************
 */
```

### 4.2 函数注释

公有函数必须使用Doxygen注释，私有函数建议使用：

```c
/**
 * @brief   初始化UART端口
 *
 * @param   port      端口号，取值 UART_PORT_1 / UART_PORT_2
 * @param   baudrate  波特率，如 9600、115200
 *
 * @return  0  成功
 * @return  -1 参数错误
 * @return  -2 硬件初始化失败
 *
 * @note    调用前需确保GPIO时钟已使能
 * @warning 该函数不可在中断中调用
 */
int uart_init(uart_port_t port, uint32_t baudrate);
```

### 4.3 行内注释

```c
/* 单行注释用 /* */，不用 //（兼容C89） */
uint32_t timeout = 1000;  /* 超时时间，单位ms */

/* 多行注释 */
/*
 * 计算CRC16校验值
 * 多项式：0xA001
 * 初始值：0xFFFF
 */
uint16_t crc16_calc(const uint8_t *data, uint16_t len)
{
    /* ... */
}
```

### 4.4 注释原则

- 注释解释 **为什么**，而不是 **做什么**
- 代码本身能说明的不要重复注释
- 修改代码时同步更新注释
- 禁止注释掉的代码留在仓库中（用Git管理历史）
- TODO/FIXME标注格式：`/* TODO: 待优化的内容 */`

---

## 5. 函数设计

### 5.1 函数长度

- 单个函数不超过 **80行**（不含空行和注释）
- 超过时拆分为子函数
- 函数只做一件事，遵循单一职责原则

### 5.2 参数设计

- 参数不超过 **5个**，超过时用结构体传递
- 输入参数在前，输出参数在后
- 指针参数明确标注输入/输出方向

```c
/**
 * @brief  解析Modbus帧
 * @param  rx_buf  [IN]  接收缓冲区
 * @param  rx_len  [IN]  接收数据长度
 * @param  frame   [OUT] 解析后的帧结构
 * @return 0成功，非0错误码
 */
int modbus_parse(const uint8_t *rx_buf, uint16_t rx_len,
                 modbus_frame_t *frame);
```

### 5.3 返回值

- 统一返回 **0表示成功，负值表示错误**
- 错误码在头文件中用枚举定义
- 不要用返回值同时承载数据和状态（用输出参数返回数据）

```c
/* 正确：状态码返回，数据通过参数输出 */
int uart_recv(uint8_t *buf, uint16_t len, uint16_t *recv_len);

/* 错误：返回值既可能是数据又可能是错误码 */
int32_t read_adc(void);  /* -1表示错误？还是-1是合法值？ */
```

### 5.4 函数前置声明

- 私有函数（static）在文件顶部集中声明
- 公有函数在头文件中声明
- 禁止隐式声明（调用前必须有声明或定义）

---

## 6. 变量与数据类型

### 6.1 固定宽度类型

嵌入式开发必须使用 `stdint.h` 中的固定宽度类型，禁止使用 `int`、`short`、`long` 等基本类型：

| 类型 | 替代 | 说明 |
|------|------|------|
| char | `int8_t` / `uint8_t` | 明确有符号/无符号 |
| short | `int16_t` / `uint16_t` | |
| int | `int32_t` / `uint32_t` | 平台相关，禁止用于跨平台代码 |
| long | `int32_t` / `int64_t` | 32位和64位平台长度不同 |
| 布尔 | `bool`（stdbool.h） | `true` / `false` |

```c
/* 正确 */
uint32_t counter = 0;
int16_t  temperature = 0;
bool     is_enabled = false;

/* 错误 */
int counter = 0;
short temperature = 0;
unsigned char flag = 1;
```

### 6.2 变量声明

- 变量在 **使用处就近声明**（C99允许）
- 声明时必须初始化
- 一个声明只声明一个变量

```c
/* 正确 */
void process_data(const uint8_t *data, uint16_t len)
{
    if (len == 0) {
        return;
    }

    uint16_t checksum = 0;
    for (uint16_t i = 0; i < len; i++) {
        checksum += data[i];
    }
}

/* 错误：全部在函数顶部声明，且未初始化 */
void process_data(const uint8_t *data, uint16_t len)
{
    uint16_t i, checksum;  /* 多个变量一行，未初始化 */
    /* ... */
}
```

### 6.3 全局变量

- **尽量避免**全局变量，优先用函数参数传递
- 必须使用时：
  - 加 `static` 限制作用域（模块内私有）
  - 多线程/中断访问时加 `volatile`
  - 提供 getter/setter 函数封装访问
  - 命名加 `g_` 前缀

```c
/* 模块私有全局变量 */
static volatile uint32_t g_systick_count = 0;

/* 通过函数访问，禁止外部直接操作 */
uint32_t systick_get_count(void)
{
    return g_systick_count;
}
```

### 6.4 类型转换

- 禁止隐式类型转换，必须显式强制转换
- 注意符号扩展和截断问题

```c
/* 正确：显式转换 */
uint16_t len = 10;
uint32_t total = (uint32_t)len * sizeof(uint32_t);

/* 错误：隐式转换可能溢出 */
uint16_t len = 10;
uint32_t total = len * sizeof(uint32_t);  /* len*sizeof先按uint16计算 */
```

---

## 7. 宏定义与常量

### 7.1 宏定义规则

- 全大写命名，单词间下划线分隔
- 必须加括号保护参数和整体
- 多行宏用 `do { ... } while(0)` 包裹

```c
/* 正确：参数和整体都加括号 */
#define SQUARE(x)  ((x) * (x))

/* 正确：多行语句宏 */
#define SET_BIT(reg, bit)  do { \
    (reg) |= (bit); \
} while (0)

/* 错误：缺少括号 */
#define SQUARE(x)  x * x  /* SQUARE(a+b) 展开为 a+b*a+b，错误 */
```

### 7.2 宏 vs 内联函数 vs 枚举

| 场景 | 推荐方式 |
|------|----------|
| 整型常量集合 | `enum` |
| 单值常量 | `const` 变量或宏 |
| 简单表达式 | 宏（注意副作用） |
| 复杂逻辑 | `static inline` 函数 |

```c
/* 推荐：枚举定义状态码 */
typedef enum {
    STATE_IDLE = 0,
    STATE_RUNNING,
    STATE_DONE,
    STATE_ERROR
} state_t;

/* 推荐：const定义常量（有类型检查） */
static const uint32_t TIMEOUT_MS = 1000;

/* 推荐：内联函数替代复杂宏 */
static inline uint32_t min_u32(uint32_t a, uint32_t b)
{
    return (a < b) ? a : b;
}
```

### 7.3 魔术数字

- 代码中禁止出现未定义的魔术数字
- 所有常量必须用宏或枚举命名

```c
/* 正确 */
#define ADC_REF_VOLTAGE_MV   3300
#define ADC_RESOLUTION       4096
voltage_mv = (adc_raw * ADC_REF_VOLTAGE_MV) / ADC_RESOLUTION;

/* 错误：魔术数字 */
voltage_mv = (adc_raw * 3300) / 4096;
```

---

## 8. 指针与内存管理

### 8.1 指针使用

- 指针变量命名加 `p` 前缀或 `_ptr` 后缀
- 函数入口必须检查指针参数是否为 `NULL`
- 用完后置 `NULL`，避免野指针

```c
int data_copy(uint8_t *dst, const uint8_t *src, uint16_t len)
{
    /* 参数检查 */
    if ((dst == NULL) || (src == NULL) || (len == 0)) {
        return -1;
    }

    memcpy(dst, src, len);
    return 0;
}
```

### 8.2 const 使用

- 输入指针参数加 `const`
- 不修改的变量加 `const`
- 指针本身和指向内容都不变时双重const

```c
/* 指向const数据的指针（数据不可改，指针可改） */
int send_data(const uint8_t *data, uint16_t len);

/* const指针（指针不可改，数据可改） */
void init_buffer(uint8_t * const buf, uint16_t size);

/* 都不可改 */
uint32_t calc_crc(const uint8_t * const data, uint16_t len);
```

### 8.3 动态内存

- **嵌入式项目禁止使用 `malloc`/`free`**（避免内存碎片和不确定行为）
- 必须使用时：
  - 系统启动时一次性分配，运行期不释放
  - 使用内存池（memory pool）管理
  - 分配后必须检查返回值

```c
/* 推荐：静态分配 */
static uint8_t s_rx_buffer[512];

/* 禁止：运行时动态分配 */
uint8_t *buf = malloc(512);  /* 嵌入式不推荐 */
```

### 8.4 数组与指针

- 数组参数同时传长度，不要依赖 `\0` 结尾（二进制数据场景）
- 数组越界必须检查

```c
/* 正确：带长度参数 */
int buffer_append(uint8_t *buf, uint16_t buf_size,
                  const uint8_t *data, uint16_t data_len)
{
    if (data_len > (buf_size - current_len)) {
        return -1;  /* 防止越界 */
    }
    /* ... */
}
```

---

## 9. 错误处理

### 9.1 统一错误码

每个模块定义自己的错误码枚举：

```c
typedef enum {
    MODBUS_OK              = 0,
    MODBUS_ERR_TIMEOUT     = -1,
    MODBUS_ERR_CRC         = -2,
    MODBUS_ERR_FRAME       = -3,
    MODBUS_ERR_PARAM       = -4,
    MODBUS_ERR_BUSY        = -5,
    MODBUS_ERR_NO_RESPONSE = -6
} modbus_status_t;
```

### 9.2 错误检查

- 所有可能失败的函数调用都必须检查返回值
- 不要忽略返回值（明确不需要时用 `(void)` 标注）

```c
/* 正确：检查返回值 */
int ret = uart_send(port, data, len);
if (ret != 0) {
    log_error("uart send failed: %d", ret);
    return ret;
}

/* 正确：明确忽略返回值 */
(void)uart_flush(port);

/* 错误：忽略返回值 */
uart_send(port, data, len);  /* 失败了也不知道 */
```

### 9.3 断言使用

- 用 `assert()` 检查编程错误（不应发生的情况）
- 运行时可能发生的错误用返回值，不用断言
- 发布版本可关闭断言

```c
#include <assert.h>

void ring_buffer_write(ring_buffer_t *rb, uint8_t byte)
{
    /* 编程错误：指针不可能为NULL，用断言 */
    assert(rb != NULL);

    /* 运行时情况：缓冲区满，用状态处理 */
    if (ring_buffer_full(rb)) {
        return;
    }
    /* ... */
}
```

### 9.4 goto 使用

- 仅用于 **函数内统一错误处理/资源释放**
- 禁止用goto做正常流程跳转
- 标签命名：`err_xxx` 或 `cleanup`

```c
int process_file(const char *path)
{
    FILE *fp = fopen(path, "r");
    if (fp == NULL) {
        return -1;
    }

    uint8_t *buf = malloc(1024);
    if (buf == NULL) {
        goto err_close_file;
    }

    if (fread(buf, 1, 1024, fp) == 0) {
        goto err_free_buf;
    }

    /* 正常处理 */
    free(buf);
    fclose(fp);
    return 0;

err_free_buf:
    free(buf);
err_close_file:
    fclose(fp);
    return -1;
}
```

---

## 10. 头文件规范

### 10.1 包含保护

所有头文件必须有包含保护，格式：`__文件名_H`

```c
#ifndef __UART_DRIVER_H
#define __UART_DRIVER_H

/* 内容 */

#endif /* __UART_DRIVER_H */
```

### 10.2 包含原则

- 头文件只包含 **必要的** 其他头文件（仅为类型定义所需）
- 源文件中包含实现所需的头文件
- 禁止在头文件中包含 `.c` 文件
- 包含顺序：本模块头文件 → 标准库 → 第三方 → 项目内其他模块

### 10.3 接口最小化

- 头文件只暴露公有接口
- 私有类型、宏、函数声明放在 `.c` 文件中
- 不暴露内部实现细节

### 10.4 C++兼容

可能被C++引用的头文件加 `extern "C"` 包裹：

```c
#ifdef __cplusplus
extern "C" {
#endif

/* 函数声明 */

#ifdef __cplusplus
}
#endif
```

---

## 11. 嵌入式专项规范

### 11.1 中断服务函数

- ISR命名：`中断名_IRQHandler` 或 `模块名_irq_handler`
- ISR必须尽可能短，只做标志置位/数据搬运
- 复杂逻辑交给主循环或任务处理
- 共享变量加 `volatile`

```c
/**
 * @brief UART1中断服务函数
 * @note  仅处理数据接收和发送完成标志
 */
void USART1_IRQHandler(void)
{
    if (__HAL_UART_GET_FLAG(&huart1, UART_FLAG_RXNE)) {
        uint8_t byte = (uint8_t)(huart1.Instance->DR & 0xFF);
        ring_buffer_write(&g_uart1_ctx.rx_buf, byte);
    }

    if (__HAL_UART_GET_FLAG(&huart1, UART_FLAG_TC)) {
        g_uart1_ctx.tx_busy = 0;
        __HAL_UART_CLEAR_FLAG(&huart1, UART_FLAG_TC);
    }
}
```

### 11.2 寄存器操作

- 位操作使用宏封装，不直接写魔术数字
- 读-改-写操作注意原子性

```c
/* 正确：位操作宏 */
#define REG_SET_BIT(reg, bit)    ((reg) |=  (bit))
#define REG_CLR_BIT(reg, bit)    ((reg) &= ~(bit))
#define REG_TOG_BIT(reg, bit)    ((reg) ^=  (bit))
#define REG_CHK_BIT(reg, bit)    ((reg) &   (bit))

/* 正确：位域定义 */
typedef union {
    struct {
        uint32_t enable    : 1;
        uint32_t mode      : 2;
        uint32_t prescaler : 4;
        uint32_t reserved  : 25;
    } bits;
    uint32_t reg;
} ctrl_reg_t;
```

### 11.3 延时与超时

- 禁止使用空循环延时（`for(i=0;i<1000;i++);`）
- 使用硬件定时器或系统滴答
- 所有阻塞操作必须有超时

```c
/* 正确：基于systick的延时 */
void delay_ms(uint32_t ms)
{
    uint32_t start = g_systick_count;
    while ((g_systick_count - start) < ms) {
        ;
    }
}

/* 正确：带超时的等待 */
int wait_for_ready(uint32_t timeout_ms)
{
    uint32_t start = g_systick_count;
    while (!is_ready()) {
        if ((g_systick_count - start) >= timeout_ms) {
            return -1;  /* 超时 */
        }
    }
    return 0;
}
```

### 11.4 硬件抽象

- 驱动层与业务层分离
- 硬件相关代码集中在驱动模块
- 通过函数指针/结构体实现平台抽象

```c
/* 硬件抽象层接口 */
typedef struct {
    int (*init)(void *cfg);
    int (*read)(uint8_t *buf, uint16_t len);
    int (*write)(const uint8_t *buf, uint16_t len);
} hal_uart_ops_t;

/* 业务层只依赖抽象接口，不直接操作寄存器 */
int modbus_send(const hal_uart_ops_t *hal, const uint8_t *data, uint16_t len)
{
    return hal->write(data, len);
}
```

---

## 12. 代码审查清单

### 12.1 必查项（一票否决）

- [ ] 所有指针参数是否检查NULL
- [ ] 数组访问是否有越界保护
- [ ] 中断中是否有阻塞操作
- [ ] 共享变量是否加volatile
- [ ] 是否有未初始化的变量
- [ ] 返回值是否都被处理
- [ ] 内存分配是否有对应释放
- [ ] 是否存在死循环无退出条件

### 12.2 规范项

- [ ] 命名是否符合规范
- [ ] 缩进是否为4空格
- [ ] 大括号是否为K&R风格
- [ ] 函数是否超过80行
- [ ] 参数是否超过5个
- [ ] 是否有魔术数字
- [ ] 注释是否准确且不过度
- [ ] 头文件是否有包含保护

### 12.3 性能项（嵌入式重点）

- [ ] 高频路径是否有不必要的函数调用
- [ ] 是否有可以移到循环外的计算
- [ ] 浮点运算是否必要（优先定点）
- [ ] 缓冲区大小是否合理
- [ ] 是否有忙等待可以改为中断/事件驱动

---

## 附录：VS Code 配置建议

在项目根目录创建 `.vscode/settings.json`：

```json
{
    "editor.tabSize": 4,
    "editor.insertSpaces": true,
    "editor.formatOnSave": true,
    "editor.rulers": [120],
    "files.encoding": "utf8",
    "files.eol": "\n",
    "C_Cpp.clang_format_style": "{ BasedOnStyle: Google, IndentWidth: 4, ColumnLimit: 120, BreakBeforeBraces: Attach }"
}
```

---

> 本规范自发布之日起执行，新增代码必须严格遵循，存量代码逐步整改。
> 规范解释权归技术委员会所有，修订需提交评审。
