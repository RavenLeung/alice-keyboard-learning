# K2 Pro 固件源码参考（reference copy）

> 本目录是从 QMK 环境复制的 **Keychron K2 Pro 官方源码**，仅供学习参考。
> 源位置：`~/qmk_firmware/keyboards/keychron/k2_pro/`（分支 `bluetooth_playground`）
> 上游仓库：https://github.com/Keychron/qmk_firmware （GPL 协议）

## 目录说明

| 路径 | 内容 |
|---|---|
| `config.h` / `rules.mk` / `info.json` | 键盘定义层核心：引脚、构建规则、USB 信息 |
| `matrix.c` | 自定义矩阵扫描（K2 Pro 用 74HC595 移位寄存器） |
| `k2_pro.c` / `k2_pro.h` | 键盘逻辑 + `LAYOUT_ansi_84` 配列宏 |
| `halconf.h` / `mcuconf.h` | 芯片层配置（STM32L432 + ChibiOS） |
| `ansi/rgb/` 等变体目录 | 各变体的 config.h / info.json / rgb.c / white.c |
| `ansi/rgb/keymaps/default/keymap.c` | **默认键位图（我们编译的就是它）** |
| `ansi/rgb/keymaps/via/` | VIA 键位图（支持免编译改键） |
| `firmware/` | 官方原厂固件备份（全部 6 变体，比 stock-firmware/ 全） |
| `via_json/` | VIA 配置 JSON |

## 使用提示

- 想看"某层原材料长什么样"，对照 docs/07 的拆解在这里找文件
- 想改键位：编辑 `ansi/rgb/keymaps/default/keymap.c` 后
  `qmk compile -kb keychron/k2_pro/ansi/rgb -km default`
- 注意：这里只是**参考副本**；真正的编译源在 `~/qmk_firmware/`，
  改文件要去那边改（或改完拷回去）
- `readme.md` 为 Keychron 官方原版说明，保留未动
