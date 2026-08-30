# K2 Pro 蓝牙方案拆解（从固件源码推断）

> 2026-08-30 整理。纯从固件源码（`reference/k2-pro-source/`）推断 K2 Pro 的无线
> 方案，不依赖拆机。接 docs/01（无线方案调研）与 docs/07（固件原材料拆解）。

---

## 一、结论先行

K2 Pro 是**双芯片方案**，不是"主控里跑蓝牙协议栈"的单芯片方案：

```
┌─ 主控 STM32L432 ─────────────┐      UART (USART2)     ┌─ CKBT51 蓝牙模块 ──────┐
│ QMK：扫矩阵/处理键码/层/灯效   │  ◄──────────────────►  │ 蓝牙协议栈(封闭固件)    │
│ 打包 HID 报告 → 发给模块       │   命令/ACK/报告         │ 负责配对/连接/上报      │
└──────────────────────────────┘                        └───────────────────────┘
```

要点：

- 主控 **不碰蓝牙协议**，只通过串口命令控制一块 Keychron 自研模块 **CKBT51**
- 蓝牙逻辑全部被 `#ifdef KC_BLUETOOTH_ENABLE` 包裹 → **同一套板子，编译时决定带不带蓝牙**（这就是为什么同样有 ansi/white 有线版键位图）
- 代码全部是"模块控制面"（init / set_param / DFU / ACK），模块内部固件封闭

## 二、硬件接口：串口 + 专用引脚

| 引脚（`config.h`） | 作用 |
|---|---|
| `CKBT51_RESET_PIN A9` | 模块复位（注释：当前未使用） |
| `CKBT51_INT_INPUT_PIN A5` / `BLUETOOTH_INT_INPUT_PIN A6` | 模块中断 / 事件通知输入 |
| `USB_BT_MODE_SELECT_PIN A10` | 侧面三档拨杆：蓝牙 / USB 模式检测 |
| `USB_POWER_SENSE_PIN B1` | USB 是否插电检测 |
| `BAT_LOW_LED_PIN A4` | 低电量指示灯 |

芯片层配置：

- `mcuconf.h`：`STM32_SERIAL_USE_USART2 TRUE` → **与模块通信走 USART2 串口**
- `halconf.h`：蓝牙分支才开 `HAL_USE_SERIAL TRUE` + `PAL_USE_CALLBACKS`（中断回调）+ `HAL_USE_RTC`
- `rules.mk`：`include keyboards/keychron/bluetooth/bluetooth.mk` → Pro 系列共享同一套蓝牙框架
- 移植线索：`matrix.c` 里被注释掉的 `#include "q1_bluetooth.h"` → 代码源自 Q1 蓝牙版，Keychron 内部复用

## 三、模块协议：命令 / ACK 风格

`k2_pro.c` 里的完整模块控制面：

```c
ckbt51_init(false);            // 上电初始化
ckbt51_set_local_name(PRODUCT);// 蓝牙广播名 = "Keychron K2 Pro"
ckbt51_set_param(&param);      // 下发参数表
ckbt51_default_ack_handler()   // 模块 ACK 回调（0x45 应答时重设参数）
ckbt51_dfu_rx()                // 模块固件升级入口
```

`module_param_t` 参数表解读：

```c
.event_mode             = 0x02,   // 事件模式
.connected_idle_timeout = 7200,   // 连接空闲 2 小时 → 断连 / 低功耗
.pairing_timeout        = 180,    // 配对窗口 3 分钟
.reconnect_timeout      = 5,      // 断线 5 秒自动重连
.report_rate            = 90,     // HID 上报 90Hz（≈11ms 间隔）
.vendor_id_source       = 1,
.verndor_id             = 0,      // 0x3434
.product_id             = PRODUCT_ID
```

设计取舍：标准 BLE HID 常配成省电的低上报频率，Keychron 选了偏快的 **90Hz** 换手感——印证 docs/01 的结论：蓝牙延迟下界约 10ms+，追不上 2.4G。

## 四、三设备记忆与配对交互

```c
#define HOST_DEVICES_COUNT 3        // 记忆 3 台设备
// keymap：Fn1+1/2/3 = BT_HST1~3（k2_pro.h 定义，无蓝牙时退化为 KC_TRNS）
case BT_HST1 ... BT_HST3:
    if (get_transport() == TRANSPORT_BLUETOOTH) {   // 仅蓝牙模式生效
        if (record->event.pressed) {
            host_idx = keycode - BT_HST1 + 1;
            chVTSet(&pairing_key_timer, TIME_MS2I(2000), ...);  // 2 秒定时器
            bluetooth_connect_ex(host_idx, 0);       // 短按：直接切到该设备
        } else {
            chVTReset(&pairing_key_timer);           // 松开取消配对
        }
    }
```

交互逻辑：**短按 Fn+1/2/3 切换已配对设备，长按 2 秒进入该槽位配对模式**。这就是"3 设备一键切换"的固件实现。

## 五、模式拨杆与 USB 共存

```c
void bluetooth_pre_task(void) {          // 每帧扫描都跑
    if (readPin(USB_BT_MODE_SELECT_PIN) != mode) {
        if (readPin(USB_BT_MODE_SELECT_PIN) != mode) {   // 两次确认防抖
            mode = readPin(USB_BT_MODE_SELECT_PIN);
            set_transport(mode == 0 ? TRANSPORT_BLUETOOTH : TRANSPORT_USB);
        }
    }
}
```

- 侧面三档拨杆（Off / Cable / BT）只读 A10 一个引脚，**两次读取确认才切换**，防抖动误切
- `KEEP_USB_CONNECTION_IN_BLUETOOTH_MODE` + `USB_POWER_SENSE_PIN`：蓝牙模式下 USB 仍保持枚举，用于**充电 / 供电感知**（`usb_power_connected()`）
- 因此 `BAT_LVL` 键只在"蓝牙模式 && 未插 USB"时显示电量动画

## 六、电池管理：无电量计芯片的"电压估算法"

`battery_calculte_voltage()` 表明硬件上**没有库仑计 / 电量计 IC**，纯软件估算：

```c
uint16_t voltage = ((uint32_t)value) * 2246 / 1000;   // ADC 值 → 电压（含分压系数）
// 补偿 RGB 负载：大电流会把电池电压拉低，不补偿会误报低电量
if (rgb_matrix_is_enabled()) {
    totalBuf += g_pwm_buffer[i][j];     // 统计所有 LED 的 PWM 占空
    voltage += 60 * totalBuf / RGB_MATRIX_LED_COUNT / 255 / 3;  // 线性补偿
}
battery_set_voltage(voltage);           // → battery_get_percentage() → 电量灯效
```

- 电量显示：`BAT_LEVEL_LED_LIST {17..26}`（10 颗灯按电量亮）
- 低电指示：`BAT_LOW_LED_PIN A4`，开机点亮 3 秒（`POWER_ON_LED_DURATION 3000`）
- 充电检测：`USB_POWER_SENSE_PIN B1`，`USB_POWER_CONNECTED_LEVEL 0`

## 七、低功耗策略（无线键盘通用套路）

| 手段 | 代码位置 |
|---|---|
| 空闲进 WFI 睡眠 | `rules.mk`: `CORTEX_ENABLE_WFI_IDLE=TRUE` + `lpm.h` |
| 断连 40 秒关背光 | `DISCONNECTED_BACKLIGHT_DISABLE_TIMEOUT 40` |
| 连接 10 分钟关背光 | `CONNECTED_BACKLIGHT_DISABLE_TIMEOUT 600` |
| 关灯时 LED 驱动芯片整体 shutdown | `RGB_MATRIX_DRIVER_SHUTDOWN_ENABLE` + `LED_DRIVER_SHUTDOWN_PIN C14` |
| 低亮度直接熄灯 | `RGB_MATRIX_BRIGHTNESS_TURN_OFF_VAL 32` |
| 主频压到 48MHz | `mcuconf.h`（注释：USB 最低时钟与低功耗的折中，L4 支持动态变频后再优化） |
| USB 挂起时关 RGB | `RGB_DISABLE_WHEN_USB_SUSPENDED` |

## 八、开机时序与自动重连

```c
void bluetooth_enter_disconnected_kb(uint8_t host_idx) {
    /* CKBT51 bluetooth module boot time is slower, it enters disconnected after boot,
       so we place initialization here. */
    if (firstDisconnect && sync_timer_read32() < 1000 && ...) {
        ckbt51_param_init();
        bluetooth_connect();      // 开机 1 秒内的首次"断开"事件 = 模块刚启动
        firstDisconnect = false;  // → 趁机初始化参数并自动连回上次设备
    }
}
```

连"模块启动比主控慢"这种时序都写进了注释——CKBT51 是**独立固件的模块**，有自己的启动周期。主控流程：`keyboard_post_init_kb()` → `ckbt51_init(false)` → `bluetooth_init()`。

## 九、可升级性：模块固件能 OTA

`via_command_kb()` 里 `0xAA → ckbt51_dfu_rx()`：VIA 的 raw HID 通道（`RAW_HID_CMD 0xAA...0xAB`）可以给蓝牙模块刷固件。**键盘固件和模块固件是两套独立固件，都能升级**。

## 十、代码看不出来的（诚实边界）

1. **CKBT51 模块里具体是哪颗 SoC**（Realtek / Beken / Nordic 哪家）——代码只暴露命令协议。需要拆机或官方资料：[Keychron 官方拆解指南](https://keychron.de/de/blogs/archived/how-to-disassemble-the-k2-pro)、[deskthority BLE 模块替换帖](https://deskthority.net/viewtopic.php?t=28411)
2. BLE 版本（4.x / 5.x）、真实连接间隔、射频参数、天线设计
3. 实际续航 / 延迟的实测数字

## 十一、对 Alice 项目的意义

- **"主控 + 现成 BLE 模块 + UART 命令协议"是低成本做无线键盘的经典架构**：主控跑 QMK、模块管协议，比单芯片方案（如 nRF52840）实现简单得多。未来若做无线版 Alice，可照抄此思路
- 低功耗手段（WFI、背光超时、驱动 shutdown、主频折中）是任何电池键盘的必修课，本代码就是标准答案
- 当前 Alice 目标是**有线**，本章作为进阶储备；阶段 2/3 用不到

## 十二、相关文件速查

| 文件（reference/k2-pro-source/） | 蓝牙相关内容 |
|---|---|
| `k2_pro.c` | 全部蓝牙逻辑：模块控制、配对、transport、电池 |
| `config.h` | 蓝牙引脚 + `KEEP_USB_CONNECTION...` + `BLUETOOTH_NKRO_ENABLE` |
| `mcuconf.h` | USART2、48MHz 时钟 |
| `halconf.h` | SERIAL / RTC / 回调（蓝牙才开） |
| `rules.mk` | `include bluetooth.mk` |
| `k2_pro.h` | `BT_HST1~3` / `BAT_LVL` 自定义键码 |
| `ansi/rgb/keymaps/default/keymap.c` | Fn 层挂 BT_HST1~3、BAT_LVL 的位置 |
