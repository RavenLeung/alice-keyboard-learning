# 分体键盘架构：无线星型 vs 有线分体

需求背景：设计 Alice 配列分体键盘，左右两块电路板，**两片之间无连接线**，只用一个 2.4G 接收器（dongle）。

## 方案一：无线星型（两半都无线，单 dongle）

**结论：可行**，已有成熟先例。dongle 合并两路数据，以单个 HID 键盘设备上报给电脑，系统无感知。

**先例：**
- [keyplus](https://geekhack.org/index.php?topic=90726.msg2493527#msg2493527)：模块化有线/无线分体键盘项目，nRF24L01+，支持多板连一个 dongle
- [Mitosis 无线分体键盘](https://global.v2ex.co/t/372555)：经典 DIY 项目，左右各带 nRF24L01+，无连接线，单 dongle
- [NocFree](https://www.kickstarter.com/projects/solar-nocfree/nocfree-and-the-wireless-split-keyboard)：量产无线分体（蓝牙/2.4G/有线三模）
- ZMK 生态的 [2.4G Dongle RF 架构](https://mechanicalkbd.com/zmk-split-keyboard-3-connection/)

**核心难点：单射频同一时刻只能在一个信道收。**
解决方案 = **TDMA 轮询**：

```
dongle ──poll→ 左半片 ──数据→ dongle ──poll→ 右半片 ──数据→ dongle ──循环
```

时序预算（nRF24L01+，2Mbps，Enhanced ShockBurst）：
- 每轮 poll+响应 ≈ 0.3–0.5ms，一轮覆盖左右 ≈ 1ms
- 每半片约 1000Hz 轮询率，有重传时 ~500Hz，仍远好于蓝牙
- 两半片晶振会漂移 → dongle 需周期性发 **beacon 同步时隙**

**推荐架构：把"大脑"放 dongle 里**
- 半片：只做矩阵扫描 + 去抖 + 上报原始矩阵状态（"傻瓜采集器"）
- dongle：持有完整 keymap，统一做层计算、合并、NKRO
- 原因：避免左右半片层/修饰键状态不一致的经典坑
- 代价：改键位要刷 dongle

**硬件选型：**

| 位置 | 方案 |
|---|---|
| 两个半片 | MCU（STM32 / nRF52840 / ATmega32u4）+ nRF24L01+ 模块 |
| dongle | nRF24LU1+（带 USB 的 nRF24 专用芯片）或 nRF52840 USB dongle 跑 ZMK |
| 固件 | keyplus（现成）；或 ZMK 2.4G 分支/OEM 方案 |

**工程注意点：**
1. 电源：左右各一块电池、各一个 USB-C 充电口，对称设计
2. 抗干扰：2.4G 与 WiFi/蓝牙/USB3.0 共存，两个发射端碰撞面大 → 必须自动重传 + 退避
3. 合并正确性：左右同时按键时 NKRO 和组合键要正确
4. 延迟：打字毫无压力；竞技游戏每半片 500–1000Hz 够用，但不如单板 2.4G 干净

## 方案二：有线分体（两半用线连，主半连 dongle）

**结论：简单非常多，是绝大多数量产分体和 DIY 的标准答案**（Corne、Lily58、ErgoDash 等都是）。

```
dongle ←→ 主半片（唯一射频）
             ↑ 有线（串口/I2C，TRRS 或 USB-C 对连线）
           从半片（无射频）
```

**对比：**

| 维度 | 无线星型 | 有线分体 |
|---|---|---|
| 射频链路 | 2 条 | 1 条 |
| 时隙同步/防碰撞 | 必须 TDMA + beacon | 不需要 |
| dongle 固件 | 自研合并逻辑 | 不用写 |
| 半片间协议 | 自己定 | QMK/ZMK 现成分体支持（自动同步矩阵） |
| 电源 | 两套电池 + 两套充电 | 可单电池，从半片走线供电 |
| 延迟 | 两半共享空口 | 从半→主半亚毫秒，射频轮询率不受影响 |

**简化点：**
1. QMK split / ZMK 有线分体模式都是成熟特性，几乎不用写协议代码
2. 从半片可以是"哑设备"：甚至可用 I/O 扩展芯片（MCP23017）或移位寄存器，无需 MCU
3. dongle 端无感，跟普通单板无线键盘一样

**代价与注意：**
- 桌面多一根线（TRRS 或 USB-C 对连线，可理线）
- **TRRS 热插拔风险**：带电插拔可能短路引脚烧 MCU → 用 USB-C 接口或断电再插拔

## 结论

- 做产品/练手 → **先做有线分体**：成本、可靠性、开发周期都友好
- 追求"两片完全无线" → 无线星型（keyplus 那种），炫技但复杂
