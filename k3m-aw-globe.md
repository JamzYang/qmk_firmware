# Keychron K3 Max Apple Globe 键实现文档

## 1. 背景 (Background)
Keychron 键盘默认的 `fn` 键仅用于键盘内部的层切换 (Layer Switch)，并不向操作系统发送任何键值信号。而 macOS 的 "Globe/Fn" 键实际上是一个特殊的 HID Consumer 键值 (`0x29D` - AC Next Keyboard Layout Select)，它能触发输入法切换或表情面板。

**核心问题**：要让 Keychron 的物理按键表现得像 Apple 的 Globe 键，必须修改键盘固件，使其能发送正确的 HID 信号。

## 2. 技术方案 (Technical Solution)

本次修改通过以下几个层面实现了功能：

1.  **定义新键值 (Keycode Definition)**：
    在 QMK 核心库中定义 `KC_GLOBE` (0x00C3)，并将其纳入 Consumer Keycode 范围。
    *   文件：[quantum/keycodes.h](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/quantum/keycodes.h)

2.  **映射 HID 信号 (HID Mapping)**：
    将 `KC_GLOBE` 映射到标准的 HID Usage ID `0x29D`。这是 macOS 识别该键的关键。
    *   文件：[tmk_core/protocol/report.h](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/tmk_core/protocol/report.h)

3.  **Keychron 适配层 (Keychron Wrapper)**：
    为了保持 Keychron 代码结构的完整性，我们没有直接使用 QMK 的 `KC_GLOBE`，而是定义了一个 Keychron 内部键值 `KC_APFN`。
    当检测到 `KC_APFN` 被按下时，触发 `KC_GLOBE` 的发送。
    *   文件：[keyboards/keychron/common/keychron_common.c](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/keyboards/keychron/common/keychron_common.c) & [.h](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/quantum/wpm.h)

4.  **VIA 支持 (VIA Support)**：
    在 VIA 的 JSON 定义文件中注册 "Apple Globe" 键值，使得可以在 VIA 软件界面中看到并分配该功能，而不需要每次都改代码。
    *   文件：[keyboards/keychron/k3_max/via_json/k3_max_ansi_white.json](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/keyboards/keychron/k3_max/via_json/k3_max_ansi_white.json)

5.  **macOS 兼容性 (Shared Endpoint)**：
    启用了 `KEYBOARD_SHARED_EP = yes`。这是为了确保 Globe 键可以作为修饰键使用（例如 Globe + Q 快速备忘录），否则系统会将按键视为来自两个不同的设备而无法组合。
    *   文件：[keyboards/keychron/k3_max/ansi/white/keymaps/yang/rules.mk](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/keyboards/keychron/k3_max/ansi/white/keymaps/yang/rules.mk)

## 3. 修改文件列表 (Modified Files)

本次修改共涉及 8 个文件：

1.  **Core QMK**:
    - [quantum/keycodes.h](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/quantum/keycodes.h): 添加 `KC_GLOBE`
    - [tmk_core/protocol/report.h](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/tmk_core/protocol/report.h): 添加 HID 映射

2.  **Keychron Common**:
    - [keyboards/keychron/common/keychron_common.h](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/keyboards/keychron/common/keychron_common.h): 定义 `KC_APFN`
    - [keyboards/keychron/common/keychron_common.c](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/keyboards/keychron/common/keychron_common.c): 实现 `KC_APFN` 逻辑
    - [keyboards/keychron/common/wireless/lpm_stm32f401.c](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/keyboards/keychron/common/wireless/lpm_stm32f401.c): (附带) 修复蓝牙唤醒延迟

3.  **K3 Max Specific**:
    - [keyboards/keychron/k3_max/via_json/k3_max_ansi_white.json](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/keyboards/keychron/k3_max/via_json/k3_max_ansi_white.json): 添加 VIA 支持
    - [keyboards/keychron/k3_max/ansi/white/keymaps/yang/keymap.c](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/keyboards/keychron/k3_max/ansi/white/keymaps/yang/keymap.c): 您的自定义键位
    - [keyboards/keychron/k3_max/ansi/white/keymaps/yang/rules.mk](file:///Users/yang/Dev/Projects/Learning/ai-config-test/qmk_firmware/keyboards/keychron/k3_max/ansi/white/keymaps/yang/rules.mk): 启用 Shared Endpoint

## 4. 编译与刷机指令 (Compilation & Flashing)

我们使用了您个人的 Keymap 目录 `yang`，以避免污染官方默认配置。

**编译指令 (Compile)**:
```bash
qmk compile -kb keychron/k3_max/ansi/white -km yang
```
*   这会在根目录下生成 `keychron_k3_max_ansi_white_yang.bin` 文件。

**刷机指令 (Flash)**:
```bash
qmk flash -kb keychron/k3_max/ansi/white -km yang
```
*   该指令会自动执行编译，并等待设备进入 Bootloader 模式后进行烧录。

**进入刷机模式 (Bootloader Mode)**:
1.  拔掉 USB 线。
2.  确保开关在 Cable 档位。
3.  按住 `Esc` 键。
4.  插入 USB 线（保持按住 Esc 约 3-5 秒）。

## 5. 附录：蓝牙唤醒修复 (Appendix: Bluetooth Wake Fix)

本次提交还包含了一个关键的蓝牙稳定性修复，解决了 Keychron K3 Max 在深度休眠后无法唤醒（“睡死”）的问题。

### 问题分析
用户反馈键盘进入深度休眠后无法唤醒，必须重新插拔 USB。
原因在于 `lpm_stm32f401.c` 中的 `enter_power_mode` 函数。系统从 `__WFI()` 醒来并恢复时钟后，操作系统的 SysTick 定时器可能尚未恢复。此时调用依赖定时器的 `wait_ms(10)` 会导致无限等待（死锁）。

### 解决方案
使用底层的“空循环”忙等待（Busy Wait）来替代操作系统提供的 `wait_ms`。这种方式不依赖任何中断或定时器。

### 修改文件
*   文件：`keyboards/keychron/common/wireless/lpm_stm32f401.c`

### 代码变更
```c
    writePinLow(BLUETOOTH_INT_OUTPUT_PIN);
    stm32_clock_init();
    
    // [FIX] 使用忙等待替代 wait_ms，避免因中断未恢复导致的死锁
    //STM32F401 运行在 ~84MHz，粗略估算 200,000 次循环约为 10-12ms
    for (volatile uint32_t i = 0; i < 200000; i++) {
        __asm__("nop");
    }
    
    writePinHigh(BLUETOOTH_INT_OUTPUT_PIN);
```


---

## 6. 蓝牙响应优化 (Bluetooth Response Optimization)

本次更新还包含了两个重要的蓝牙响应优化，解决了用户在日常使用中遇到的两个问题。

### 问题概述

| 问题       | 现象                             | 频率                     | 原因                            |
| ---------- | -------------------------------- | ------------------------ | ------------------------------- |
| **问题 1** | 按键需要4-5秒才能响应            | 偶尔（蓝牙信号不稳定时） | 蓝牙断连触发深度休眠            |
| **问题 2** | 第一次按键没反应，第二次才有反应 | 经常                     | `clear_keyboard()` 清除按键状态 |

### 问题 1：深度休眠延迟（4-5秒无响应）

#### 背景
用户习惯**关闭背光**使用键盘。在这种状态下，键盘容易进入深度休眠（PM_STOP 模式），导致按键需要4-5秒才能响应。如果背光开着，10分钟内不会出现这个问题。

#### 根本原因分析

在 `lpm.c` 的 `lpm_task()` 函数中，深度休眠需要满足多个条件，其中关键的一条是：

```c
if (!led_matrix_is_enabled() ||                              // 背光已关闭
    (led_matrix_is_enabled() && led_matrix_is_driver_shutdown()))  // 或背光超时后驱动关闭
```

这意味着：
- **背光关闭时**：`!led_matrix_is_enabled()` 返回 `true`，**立即满足**深度休眠条件
- **背光开启时**：需要等待背光超时（600秒）后驱动关闭，条件才满足

#### 为什么背光关闭后"感觉"信号更不稳定？

**实际上，蓝牙信号的稳定性在两种情况下是一样的**。但用户感觉背光关闭后更容易断连，原因如下：

**背光开启时**：
- 即使蓝牙信号不稳定，连接短暂断开
- 背光驱动不会立即关闭（因为背光是开着的）
- `led_matrix_is_driver_shutdown()` 返回 `false`
- 所以**不会进入深度休眠**
- 只会遇到"第一次按键没反应"的问题

**背光关闭时**：
- 蓝牙信号不稳定，连接短暂断开
- `!led_matrix_is_enabled()` 返回 `true`（背光已关闭）
- **立即满足深度休眠条件**
- 进入深度休眠，导致 4-5 秒无响应

**结论**：背光关闭不会影响蓝牙信号，但会让深度休眠条件更容易满足，从而放大了蓝牙信号不稳定的影响。

#### 解决方案

在用户的 keymap 中添加配置宏 `LPM_DISABLE_DEEP_SLEEP_ON_BACKLIGHT_OFF`，禁用"背光关闭时进入深度休眠"的行为。

**新增文件**：`keyboards/keychron/k3_max/ansi/white/keymaps/yang/config.h`
```c
#pragma once

// Disable deep sleep when backlight is off to prevent 4-5 second wake delay
#define LPM_DISABLE_DEEP_SLEEP_ON_BACKLIGHT_OFF
```

**修改文件**：`keyboards/keychron/common/wireless/lpm.c`

在 `lpm_task()` 函数中添加条件编译，当定义了该宏时，跳过背光状态检查：

```c
#if defined(LED_MATRIX_ENABLE) || defined(RGB_MATRIX_ENABLE)
#    ifndef LPM_DISABLE_DEEP_SLEEP_ON_BACKLIGHT_OFF
        if (
#        ifdef LED_MATRIX_ENABLE
            !led_matrix_is_enabled() ||
            (led_matrix_is_enabled() && led_matrix_is_driver_shutdown())
#        endif
            // ... RGB_MATRIX 部分省略
        )
#    endif
#endif
```

**效果**：当定义了 `LPM_DISABLE_DEEP_SLEEP_ON_BACKLIGHT_OFF` 时，无论背光是否关闭，都不会进入深度休眠。键盘将保持在轻度休眠状态，响应时间为毫秒级。

---

### 问题 2：第一次按键丢失

#### 背景
用户在短暂停顿后按键，第一次按键经常没有反应，需要按第二次才能生效。

#### 根本原因分析

在 `wireless.c` 的 `wireless_enter_connected()` 函数中：

```c
static void wireless_enter_connected(uint8_t host_idx) {
    wireless_state = WT_CONNECTED;
    indicator_set(wireless_state, host_idx);
    host_index = host_idx;

    clear_keyboard();  // <-- 这里清除了第一次按键！
    // ...
}
```

**问题**：蓝牙模块在某些情况下会重复发送 `EVT_CONNECTED` 事件（例如从低功耗状态恢复时），即使键盘已经处于连接状态。每次收到这个事件，`clear_keyboard()` 都会被调用，清除当前按下的所有按键。

**时间线**：
1. 键盘空闲，蓝牙连接可能进入低功耗状态
2. 用户按下第一个键
3. 蓝牙模块被唤醒，发送 `EVT_CONNECTED` 事件
4. `wireless_enter_connected()` 被调用
5. `clear_keyboard()` 清除所有按键状态 ← **第一次按键被丢弃**
6. 用户需要按第二次

#### 解决方案

修改 `wireless_enter_connected()` 函数，仅在状态真正从非连接变为连接时才调用 `clear_keyboard()`。

**修改文件**：`keyboards/keychron/common/wireless/wireless.c`

```c
static void wireless_enter_connected(uint8_t host_idx) {
    kc_printf("wireless_connected %d\n\r", host_idx);

    // Only clear keyboard when transitioning from non-connected to connected state
    bool was_connected = (wireless_state == WT_CONNECTED);

    wireless_state = WT_CONNECTED;
    indicator_set(wireless_state, host_idx);
    host_index = host_idx;

    // Only clear keyboard state on actual reconnection to prevent first keypress loss
    if (!was_connected) {
        clear_keyboard();
    }

    // ... 其余代码不变 ...
}
```

---

### 修改文件清单

| 文件                                                         | 操作 | 说明                   |
| ------------------------------------------------------------ | ---- | ---------------------- |
| `keyboards/keychron/k3_max/ansi/white/keymaps/yang/config.h` | 新增 | 添加配置宏禁用深度休眠 |
| `keyboards/keychron/common/wireless/lpm.c`                   | 修改 | 添加条件编译支持       |
| `keyboards/keychron/common/wireless/wireless.c`              | 修改 | 修复第一次按键丢失     |

### 风险评估

- **问题 1 修复**：低风险，仅影响用户自己的 keymap，不影响公共代码的默认行为
- **问题 2 修复**：中等风险，修改公共代码，但逻辑改动很小且合理

### 回滚方案

如果出现问题，可以：
1. 删除 `config.h` 中的宏定义
2. 恢复 `wireless.c` 中的原始代码
3. 重新编译刷入

---

## 7. 深度休眠问题修复 (Deep Sleep Bug Fix)

### 问题描述

在完成蓝牙响应优化（第 6 节）后，键盘出现了新问题：**无论背光开启还是关闭，都会进入深度休眠**。这比之前的问题更严重。

### 根本原因分析

#### RUN_MODE_PROCESS_TIME (1秒) 的真正作用

这个 1 秒**不是**"深度休眠前的等待时间"，而是一个**最小稳定间隔**：

```c
void lpm_task(void) {
    // 1秒后 lpm_time_up 变为 true，表示"系统已稳定，可以考虑进入休眠"
    if (!lpm_time_up && sync_timer_elapsed32(lpm_timer_buffer) > RUN_MODE_PROCESS_TIME) {
        lpm_time_up = true;
    }

    // 但进入深度休眠还需要满足其他条件！
    if (蓝牙模式 && lpm_time_up && !indicator_is_running() && lpm_is_kb_idle()) {
        if (背光关闭 || 背光驱动已超时关闭) {  // ← 这才是关键条件
            // 进入深度休眠
        }
    }
}
```

**真正控制深度休眠时间的是 `LED_MATRIX_TIMEOUT`**，而不是 `RUN_MODE_PROCESS_TIME`。

#### K3 Max 的配置

```c
// keyboards/keychron/k3_max/ansi/white/config.h
#define LED_MATRIX_TIMEOUT LED_MATRIX_TIMEOUT_INFINITE  // = UINT32_MAX
```

这意味着：**背光永远不会自动超时关闭**。

#### 深度休眠条件

```c
if (!led_matrix_is_enabled() ||                              // 条件A: 背光被手动关闭
    (led_matrix_is_enabled() && led_matrix_is_driver_shutdown()))  // 条件B: 背光开启但驱动超时关闭
```

由于 `LED_MATRIX_TIMEOUT = INFINITE`：
- **条件B 永远不会满足**（背光驱动永远不会超时关闭）
- **只有条件A 会触发深度休眠**（用户手动关闭背光时）

#### 原始行为 vs 错误修改后的行为

| 背光状态 | 原始行为 | 错误修改后 |
|---------|---------|-----------|
| 开启 | 不会进入深度休眠 | **会进入深度休眠** |
| 关闭 | 会进入深度休眠 | **会进入深度休眠** |

#### 错误的预处理器逻辑

我们之前的修改：
```c
#ifndef LPM_DISABLE_DEEP_SLEEP_ON_BACKLIGHT_OFF
    if (背光条件...)
#endif
    {
        // 深度休眠代码
    }
```

当定义了宏时，`if` 被删除，但 `{ }` 块**无条件执行**！这是 C 预处理器的经典陷阱。

### 修复方案

#### 步骤 1：修复 lpm.c 的预处理器逻辑

**修改前**（错误）：
```c
#if defined(LED_MATRIX_ENABLE) || defined(RGB_MATRIX_ENABLE)
#    ifndef LPM_DISABLE_DEEP_SLEEP_ON_BACKLIGHT_OFF
        if (
#        ifdef LED_MATRIX_ENABLE
            !led_matrix_is_enabled() ||
            (led_matrix_is_enabled() && led_matrix_is_driver_shutdown())
#        endif
#        ifdef RGB_MATRIX_ENABLE
                !rgb_matrix_is_enabled() ||
            (rgb_matrix_is_enabled() && rgb_matrix_is_driver_shutdown())
#        endif
        )
#    endif
#endif
```

**修改后**（正确）：
```c
#if defined(LED_MATRIX_ENABLE) || defined(RGB_MATRIX_ENABLE)
#    ifdef LPM_DISABLE_DEEP_SLEEP_ON_BACKLIGHT_OFF
        if (0)  // 禁用深度休眠
#    else
        if (
#        ifdef LED_MATRIX_ENABLE
            !led_matrix_is_enabled() ||
            (led_matrix_is_enabled() && led_matrix_is_driver_shutdown())
#        endif
#        ifdef RGB_MATRIX_ENABLE
                !rgb_matrix_is_enabled() ||
            (rgb_matrix_is_enabled() && rgb_matrix_is_driver_shutdown())
#        endif
        )
#    endif
#endif
```

#### 步骤 2：保留 config.h 中的宏定义

```c
// keyboards/keychron/k3_max/ansi/white/keymaps/yang/config.h
#pragma once

// 禁用"背光关闭时进入深度休眠"的行为
#define LPM_DISABLE_DEEP_SLEEP_ON_BACKLIGHT_OFF
```

### 修改文件清单

| 文件 | 操作 | 说明 |
|------|------|------|
| `keyboards/keychron/common/wireless/lpm.c` | 修改 | 修复预处理器逻辑，使用 `#ifdef` + `if(0)` |
| `keyboards/keychron/k3_max/ansi/white/keymaps/yang/config.h` | 保留 | 保留 `LPM_DISABLE_DEEP_SLEEP_ON_BACKLIGHT_OFF` 宏 |

### 验证方法

1. 编译固件：`qmk compile -kb keychron/k3_max/ansi/white -km yang`
2. 刷入键盘
3. 测试：
   - 背光关闭状态下，按键应该立即响应（不进入深度休眠）
   - 背光开启状态下，按键应该立即响应（不进入深度休眠）
   - 键盘会保持在轻度休眠状态，响应时间为毫秒级