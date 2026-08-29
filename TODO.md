# 今晚任务清单（回家继续）

> 目标：把"软件侧练手"正式启动 —— K2 Pro 编译 + 刷机跑通一轮。
> ✅ 2026-08-29 已完成首轮，完整操作归档见 [docs/05-macos-qmk-setup-flash-log.md](docs/05-macos-qmk-setup-flash-log.md)。

## 今晚必做（约 1–2 小时）

### 1. 环境搭建（注意：K2 Pro 在 bluetooth_playground 分支！）
- [x] macOS 用 venv + pip 装 qmk（`python3 -m venv ~/.qmk-venv && pip install qmk`，qmk 1.2.0）
- [x] `qmk setup -b bluetooth_playground keychron/qmk_firmware`（Keychron 官方分支 + 正确分支，默认分支里没有 K2 Pro）
- [x] 确认键盘变体：**ansi/rgb**（一字回车 + 彩色 RGB）
- [x] `qmk compile -kb keychron/k2_pro/ansi/rgb -km default` 编译通过 → `keychron_k2_pro_ansi_rgb_default.bin`（58944B，已存 build/）
- [x] 查实际路径：`keychron/k2_pro/ansi/rgb` ✓
- [x] 工具链：`arm-none-eabi-gcc@8` + `arm-none-eabi-binutils`（brew 已装，需加入 PATH，见 docs/05 §2.3）
- [ ] 补充：`avr-gcc` 未装（阶段 2 Pro Micro 需要，`brew tap osx-cross/avr && brew install avr-gcc@8`，源码编译 30min+）

### 2. 备份原厂固件（仓库里就有！）
- [x] 已下载 `keychron_k2_pro_ansi_rgb_via.bin` → `stock-firmware/`（58284B，SHA256: 830c4b49...）
- [x] 确认 .bin 已下载

### 3. 刷机第一轮（核心练习）
- [x] USB 连电脑 → 模式开关拨到 **Off** → **按住 Esc** → 开关拨到 **Cable** → 确认出现 DFU 设备（`dfu-util -l` 见 `0483:df11` STM32 BOOTLOADER）
- [x] `qmk flash -kb keychron/k2_pro/ansi/rgb -km default` 刷入成功（58928B，DFU 模式下载完成，自动重启回正常模式）
- [x] 验证能正常打字（2026-08-29 用户确认：刷机后正常打字，固件含蓝牙支持）
- [ ] 再练一次：重复"进 bootloader → 刷回默认固件 → 验证"（可选，下次补）

### 4. 小改一轮（可选，时间够就做）
- [ ] 编辑 `keyboards/keychron/k2_pro/ansi/rgb/keymaps/default/keymap.c`，改一个键位
- [ ] 重新 `make keychron/k2_pro/ansi/rgb:default` 编译 → 刷入验证
- [ ] （注意：QMK Configurator 网页版**没有** K2 Pro，因为它不在 QMK 主仓库）

## 今晚顺带（下单硬件，为阶段 2 做准备）

- [ ] 下单：**Pro Micro ×2**（~¥30/个，多买备用）
- [ ] 下单：**1N4148 二极管 ×100**（几块钱）
- [ ] 下单：**轴体 10–70 颗**（先买 10 颗练手，确定手感再补）
- [ ] 下单：**面包板或洞洞板**、飞线
- [ ] 检查家里是否有：恒温焊台/电烙铁（300–350℃）、焊锡、助焊剂（没有就一起下单）

## 遇到问题时的排查顺序

1. 编译报错 → 先 `qmk doctor` 检查环境，报错信息贴进搜索
2. 进不了 bootloader → 确认顺序：开关拨 **Off** → 按住 **Esc** 不放 → 再拨到 **Cable**；换一根数据线（不要充电线）
3. 刷完没反应 → 先刷回**原厂固件**验证硬件没坏，再重试
4. 都不行 → 记录现象，明天继续，别在深夜焊/刷

## 后续计划速览

- 下周：阶段 2 焊接闭环（Pro Micro + 面包板小矩阵）
- 之后：阶段 3 手搓 Alice（先定矩阵 → 焊接 → handwired 固件）
