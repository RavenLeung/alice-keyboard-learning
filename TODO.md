# 今晚任务清单（回家继续）

> 目标：把"软件侧练手"正式启动 —— K2 Pro 编译 + 刷机跑通一轮。

## 今晚必做（约 1–2 小时）

### 1. 环境搭建（注意：K2 Pro 在 bluetooth_playground 分支！）
- [ ] Windows 装 **QMK MSYS**（官网 qmk.fm 下载安装包）；或 Linux/macOS 用 pip 装 qmk
- [ ] `qmk setup -b bluetooth_playground keychron/qmk_firmware`（Keychron 官方分支 + 正确分支，默认分支里没有 K2 Pro）
- [ ] 确认键盘变体：回车键形状（一字=ANSI / 倒 L=ISO）+ 背光颜色（彩色=RGB / 单色=White）→ 国内市售基本是 **ansi/rgb**
- [ ] `make keychron/k2_pro/ansi/rgb:default` 编译通过 ✅（变体不符就换 ansi/white、iso/rgb 等）
- [ ] 查实际路径：`qmk list-keyboards | grep -i k2_pro`

### 2. 备份原厂固件（仓库里就有！）
- [ ] 从 [keyboards/keychron/k2_pro/firmware/](https://github.com/Keychron/qmk_firmware/tree/bluetooth_playground/keyboards/keychron/k2_pro/firmware) 下载对应变体的 `keychron_k2_pro_ansi_rgb_via.bin`，存好备用
- [ ] 先**不要刷**，只确认 .bin 已下载

### 3. 刷机第一轮（核心练习）
- [ ] USB 连电脑 → 模式开关拨到 **Off** → **按住 Esc**（或空格下方复位孔）→ 开关拨到 **Cable** → 确认电脑出现 DFU 设备
- [ ] `make keychron/k2_pro/ansi/rgb:default:flash`（或 `qmk flash -kb keychron/k2_pro/ansi/rgb -km default`，也可用 QMK Toolbox）
- [ ] 验证能正常打字
- [ ] 再练一次：重复"进 bootloader → 刷回默认固件 → 验证"

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
