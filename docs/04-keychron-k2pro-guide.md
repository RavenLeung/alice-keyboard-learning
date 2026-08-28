# Keychron K2 Pro 练手指南（软件侧）

**结论：K2 Pro 属于官方支持 QMK/VIA 的 Pro 系列，是绝佳的软件练手机。**
它是 75% 配列（非 Alice），但练的是"固件能力"，配列不影响。

## 型号分类速查

| 系列 | QMK/VIA 支持 | 适合练手吗 |
|---|---|---|
| Q 系列（Q1/Q2/Q3…）、Q Pro 系列 | ✅ 官方支持 | 非常适合 |
| V 系列（V1/V2/V3…） | ✅ 官方支持 | 非常适合 |
| C 系列（C1/C2，有线版） | ✅ 官方支持 | 适合 |
| K Pro 系列（K2 Pro/K3 Pro/K8 Pro…） | ✅ 官方支持 | 适合 |
| 老 K 系列（K2/K6/K8/K12 无线版） | ❌ 固件封闭 | 基本不行 |

判断方法：铭牌型号带 **Pro** 的 K 系列支持 QMK。老 K 系列只有社区移植（[K3v2 SonixQMK](https://github.com/timgdx/qmk_k3v2)、[K10 Max 社区固件](https://github.com/k0nker/qmk_firmware_k10max)），有风险，新手不碰。

## 第一步：环境搭建（注意：用 bluetooth_playground 分支！）

K2 Pro 的固件在 [Keychron 官方 QMK 分支](https://github.com/Keychron) 的 **`bluetooth_playground` 分支**上——默认分支（2025q3）里**没有** k2_pro，这是最容易踩的坑。不要用 QMK 主仓库：

```bash
qmk setup -b bluetooth_playground keychron/qmk_firmware
# 如果已经 setup 过：cd qmk_firmware && git fetch && git checkout bluetooth_playground

# 编译（变体按下面的表选）
make keychron/k2_pro/ansi/rgb:default
```

## 确认你的变体（先选对再编译）

`keyboards/keychron/k2_pro/` 下按 **配列 × 背光** 分 4 个变体（另有 jis 日文配列）：

| 变体 | 判断方法 | 编译目标 |
|---|---|---|
| ansi/rgb | 一字回车 + 彩色 RGB 背光 | `keychron/k2_pro/ansi/rgb` |
| ansi/white | 一字回车 + 单色白背光 | `keychron/k2_pro/ansi/white` |
| iso/rgb | 倒 L 形回车 + RGB | `keychron/k2_pro/iso/rgb` |
| iso/white | 倒 L 形回车 + 白背光 | `keychron/k2_pro/iso/white` |

判断方法：看**回车键形状**（一字 = ANSI，倒 L = ISO）；开背光看**颜色**（彩色 = RGB，单色 = White）。国内市售 K2 Pro 基本是 ansi/rgb。

## 第二步：刷机（练手核心环节）

官方 readme 里 K2 Pro 的进 bootloader 方法（K2 Pro 是无线键盘，**靠模式开关进，不是空格+B**）：

1. **备份原厂固件**：固件就在仓库里！`keyboards/keychron/k2_pro/firmware/` 下有对应变体的 `keychron_k2_pro_ansi_rgb_via.bin` 等文件（[在线查看](https://github.com/Keychron/qmk_firmware/tree/bluetooth_playground/keyboards/keychron/k2_pro/firmware)），下载和你键盘变体对应的 .bin 存好，刷坏能还原
2. **进 bootloader**：USB 线连电脑 → 模式开关拨到 **Off** → **按住 Esc**（或空格下方的复位小孔）→ 把模式开关拨到 **Cable** → 电脑出现 DFU 设备
3. **烧录**：`make keychron/k2_pro/ansi/rgb:default:flash`（或 `qmk flash -kb keychron/k2_pro/ansi/rgb -km default`，也可用 QMK Toolbox）
4. **验证**：能正常打字 = 闭环打通 ✅

**反复进出 bootloader 练几次** —— 以后焊错线救砖全靠这个肌肉记忆。

官方教程参考：[K2 Pro 恢复出厂与刷固件](https://keychron.de/de/blogs/archived/k2-pro-factory-reset-and-firmware-flash)（[其他语言版本](https://keychron.pt/blogs/archived/how-to-factory-reset-and-flash-firmware-for-your-k2-pro-keyboard)）

## 第三步：在它上面练什么

1. 改 keymap：层、组合键（combo）、宏，反复编译刷入
2. 用 VIA：原厂固件已内置 VIA（.bin 文件名带 _via），免编译改键；VIA 定义在 `keychron/k2_pro/via_json/`（如 `k2_pro_ansi_rgb.json`）
3. 读 `keychron/k2_pro/` 下的 `info.json`：看真实的行列引脚、二极管方向定义 —— **这就是以后手搓 Alice 要自己写的东西，先看标准答案**
4. 调 RGB 默认值等固件配置

## 注意事项

- **蓝牙**：Keychron 分支内置 BLE，刷 QMK 后蓝牙仍可用，但配对方式不同、有稳定性差异报告 → **练手阶段主用有线**
- **别刷错型号**：ANSI/ISO、RGB 版本要对上，刷错可能变砖（一般可救）
- **保修**：刷第三方固件理论上影响保修（闲置键盘无顾虑）
- **物尽其用**：练完后热插拔拆轴和键帽给 Alice 用（省 ¥100+）

## 它练不了什么（硬件侧）

- 焊接（PCB 出厂已装好）、矩阵接线（已印在板上）、换主控（MCU 焊死）

所以硬件侧仍需另买：Pro Micro + 二极管 + 洞洞板（见 docs/03 阶段 2）。
