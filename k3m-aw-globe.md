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
该修复已包含在本次编译的固件中。
