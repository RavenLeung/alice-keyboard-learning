# QMK 固件编译的"原材料"拆解（以 K2 Pro 为例）

> 2026-08-29 整理。理解 QMK 固件从源码到 .bin 的构成，是改键位、手搓键盘的
> 底层认知。本文以 Keychron K2 Pro（ansi/rgb）为实例拆解。

---

## 一、总览：4 层配方 + 2 个工具

```
┌─────────────────────────────────────────────┐
│ ① 键位图层 (Keymap)   ← 用户要改的地方       │
│ ② 键盘定义层 (Keyboard) ← 键盘厂商写的       │
│ ③ 芯片支持层 (MCU/ChibiOS) ← QMK 自带       │
│ ④ QMK 核心库 (QMK Core)   ← 框架本身        │
├─────────────────────────────────────────────┤
│ ⑤ 编译器 (arm-none-eabi-gcc)                │
│ ⑥ 链接/转换工具 (binutils → .bin)           │
└─────────────────────────────────────────────┘
```

编译 = 把 ①②③④ 用 ⑤⑥ 加工成一份 .bin 固件。

## 二、① 键位图层（Keymap）——"按键怎么排"

**唯一由用户编写的"原材料"**。K2 Pro 实例：

| 文件 | 路径 | 作用 |
|---|---|---|
| keymap.c | `keyboards/keychron/k2_pro/ansi/rgb/keymaps/default/keymap.c` | 定义各层键位（K2 Pro 有 MAC_BASE / MAC_FN / WIN_BASE / WIN_FN 四层，84 键） |

核心结构：

```c
const uint16_t PROGMEM keymaps[][MATRIX_ROWS][MATRIX_COLS] = {
[MAC_BASE] = LAYOUT_ansi_84(
     KC_ESC,   KC_BRID,  KC_BRIU, ...   // ← 每颗键放什么功能
     ...
```

**改键 = 改这个文件**。例：把 `KC_ESC` 换成 `KC_A`，Esc 就变成 A 键。

## 三、② 键盘定义层（Keyboard）——"这块板长什么样"

位于 `keyboards/keychron/k2_pro/`，核心文件：

| 文件 | 作用（K2 Pro 实例） |
|---|---|
| info.json | 声明 USB VID/PID（0x0220）、特性（rgb_matrix）、变体结构 |
| config.h | 引脚定义：`LED_CAPS_LOCK_PIN A7`、DIP 开关 `A8`、蓝牙复位 `A9` |
| rules.mk | 构建规则：`include keyboards/keychron/bluetooth/bluetooth.mk`（引入蓝牙） |
| matrix.c | 自定义矩阵扫描（K2 Pro 用 74HC595 移位寄存器） |
| k2_pro.h | `LAYOUT_ansi_84` 宏：84 键物理位置 ↔ 矩阵坐标映射 |
| ansi/rgb/ 子目录 | 变体层：rgb.c（RGB 灯效）、info.json、keymaps/ |

> 变体（ansi/iso/jis × rgb/white）是"键盘定义层"下的分支，选对变体才能编译出
> 匹配硬件的固件——这就是为什么刷机前要先确认变体。

## 四、③ 芯片支持层（MCU）——"跑在什么硬件上"

| 原材料 | K2 Pro 用的 |
|---|---|
| 芯片 | STM32L432（ARM Cortex-M4，170MHz，128KB Flash） |
| RTOS/底层 | ChibiOS（QMK 在 ARM 上的运行环境） |
| 芯片配置 | `halconf.h`、`mcuconf.h`（48MHz 时钟、低功耗 WFI、蓝牙共存） |
| 由谁提供 | QMK 仓库 `lib/chibios/` 子模块（qmk setup 时自动拉取） |

## 五、④ QMK 核心库——"框架本身"

`quantum/` 目录约 1500+ 源文件：按键处理、去抖、RGB 驱动、蓝牙协议、HID 报告、
层切换、组合键、宏……编译系统按需挑选编译，不需要全编。

## 六、⑤⑥ 工具链——"把源码变成固件"

| 工具 | 版本（本机） | 干什么 |
|---|---|---|
| arm-none-eabi-gcc | 8.5.0 | C 源码 → ARM 机器码（.elf） |
| arm-none-eabi-ar/ld | binutils 2.41 | 归档/链接 → 最终二进制 |
| dfu-util（刷机用） | 0.11 | 把 .bin 写入芯片 Flash |

## 七、组装流程（简化）

```
keymap.c（你的）  ─┐
k2_pro/*.c（板子） ├─→ arm-none-eabi-gcc ─→ .elf ─→ .bin（固件）
QMK core/*.c（框架）│     （编译+链接）              ↓
mcuconf.h（芯片）  ─┘                           刷入键盘
```

**输出**：`keychron_k2_pro_ansi_rgb_default.bin`（58944 字节）。

## 八、手搓 Alice 时哪些要自己写？

| 层 | 手搓 Alice 时 | 工作量 |
|---|---|---|
| ① keymap.c | 定义 Alice 配列键位 | 小（可抄现成 Alice 键盘的 LAYOUT） |
| ② 键盘定义 | `qmk new-keyboard` 生成 + 填矩阵引脚 | 中（核心学习点） |
| ③④⑤⑥ | **不用动** | 零 |

结论：手搓键盘 = 自己写"这块板长什么样"（②）+"键怎么排"（①），其余复用框架。
