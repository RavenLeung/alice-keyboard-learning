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

## 第一步：环境搭建（用 Keychron 官方 QMK 分支）

K2 Pro 固件在 [Keychron 官方 QMK 分支](https://github.com/Keychron)，内置自家 BLE 支持，**不要用 QMK 主仓库**：

```bash
qmk setup keychron/qmk_firmware -b master
qmk compile -kb keychron/k2_pro/ansi -km default   # 先编译默认固件验证环境
```

（目录名以 fork 实际结构为准，`keychron/k2_pro/` 下可能分 ansi/iso 或带 RGB 变体。）

## 第二步：刷机（练手核心环节）

官方教程：[K2 Pro 恢复出厂与刷固件](https://keychron.de/de/blogs/archived/k2-pro-factory-reset-and-firmware-flash)（[其他语言版本](https://keychron.pt/blogs/archived/how-to-factory-reset-and-flash-firmware-for-your-k2-pro-keyboard)）

1. **备份原厂固件**：去 Keychron 官网下载 K2 Pro 原厂固件存档，刷坏能还原
2. **进 bootloader**：拔 USB 线 → **按住空格+B** → 插 USB 线 → 电脑出现 DFU 设备
3. **烧录**：`qmk flash`（或 QMK Toolbox）
4. **验证**：能正常打字 = 闭环打通 ✅

**反复进出 bootloader 练几次** —— 以后焊错线救砖全靠这个肌肉记忆。

## 第三步：在它上面练什么

1. 改 keymap：层、组合键（combo）、宏，反复编译刷入
2. 用 VIA：刷带 VIA 的固件后免编译改键
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
