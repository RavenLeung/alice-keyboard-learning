# 2026-08-29 操作归档：macOS 搭建 QMK 环境 + K2 Pro 第一轮刷机

> 本次在 macOS（Intel）上从零搭建 QMK 环境，完成 Keychron K2 Pro（ansi/rgb 变体）
> 的固件编译与第一轮刷机，验证打字正常。本文记录完整操作与踩坑，供日后重装/换机复用。

---

## 一、环境信息

| 项目 | 值 |
|---|---|
| 系统 | macOS 15.7.4（Intel x86_64） |
| Homebrew | 6.0.20（/usr/local，Intel 前缀） |
| Python | 3.9.6（系统自带）+ venv |
| QMK CLI | 1.2.0（pip 安装） |
| 固件分支 | `bluetooth_playground`（Keychron 官方仓库） |
| 键盘 | Keychron K2 Pro，变体 **ansi/rgb**（一字回车 + 彩色 RGB） |
| 主控 | STM32L432（ARM Cortex-M4） |

## 二、环境搭建（踩坑记录）

### 2.1 为什么不用 brew 装 qmk，改用 pip

**尝试过的路径（失败/放弃）**：

```bash
# 第一次尝试：brew 安装
brew install qmk/qmk/qmk
# 报错：Refusing to load formula from untrusted tap
# → 需先信任 tap
brew trust qmk/qmk
```

信任后再次安装，报新错误：

```
Warning: No available formula with the name "osx-cross/arm/arm-none-eabi-gcc@8".
Warning: No available formula with the name "osx-cross/avr/avr-gcc@8".
```

需要补齐两个工具链 tap：

```bash
brew tap osx-cross/arm && brew trust osx-cross/arm
brew tap osx-cross/avr && brew trust osx-cross/avr
```

⚠️ **关键坑**：`osx-cross/avr` 的 `avr-gcc@8` 在 Intel Mac 上**没有预编译 bottle，需要从源码编译**，耗时极长（30 分钟以上）。而 K2 Pro 主控是 **ARM（STM32L432）**，编译只需要 `arm-none-eabi-gcc`，**根本不需要 AVR 工具链**。于是杀掉 brew 任务，改用 pip 快速安装。

### 2.2 最终采用的安装方式（pip + venv）

```bash
# 1. 建虚拟环境（避免污染系统 Python）
python3 -m venv ~/.qmk-venv
source ~/.qmk-venv/bin/activate

# 2. 装 qmk CLI
pip install --upgrade pip
pip install qmk        # 得到 qmk 1.2.0

# 3. 拉取 Keychron 官方分支 + bluetooth_playground 分支（默认分支没有 K2 Pro！）
qmk setup -b bluetooth_playground keychron/qmk_firmware -y

# 4. 补装仓库 Python 依赖（Keychron 分支需要 appdirs 等额外模块）
pip install -r ~/qmk_firmware/requirements.txt
```

### 2.3 ARM 工具链（brew 安装，已装好）

```bash
brew install arm-none-eabi-gcc@8 arm-none-eabi-binutils
```

⚠️ **PATH 坑**：brew 安装后工具不在默认 PATH，编译会报 `arm-none-eabi-ar: command not found`（`ar` 在 binutils 包里）。每次编译前需：

```bash
export PATH="/usr/local/opt/arm-none-eabi-gcc@8/bin:/usr/local/opt/arm-none-eabi-binutils/bin:$PATH"
```

**AVR 工具链（未装，阶段 2 用 Pro Micro 时再补）**：

```bash
brew tap osx-cross/avr && brew trust osx-cross/avr
brew install avr-gcc@8    # 源码编译，耗时 30min+，提前规划
```

## 三、编译

```bash
# 确认键盘路径
qmk list-keyboards | grep k2_pro
# → keychron/k2_pro/ansi/rgb（正确变体）

# 编译 default 固件
cd ~/qmk_firmware
qmk compile -kb keychron/k2_pro/ansi/rgb -km default
# 产物：keychron_k2_pro_ansi_rgb_default.bin（58944 字节）
# 输出：Size after: text=0 data=58894 → 固件大小约 58.9KB
```

产物已复制到项目 `build/keychron_k2_pro_ansi_rgb_default.bin`（SHA256 `6774e34b...`）。

## 四、原厂固件备份

原厂固件就在 Keychron 仓库里（无需导出）：

```bash
curl -sL -o stock-firmware/keychron_k2_pro_ansi_rgb_via.bin \
  "https://raw.githubusercontent.com/Keychron/qmk_firmware/bluetooth_playground/keyboards/keychron/k2_pro/firmware/keychron_k2_pro_ansi_rgb_via.bin"
# 58284 字节，SHA256 830c4b49...
```

刷坏可随时刷回（见第六节）。

## 五、第一轮刷机（核心练习）

### 5.1 进入 DFU 刷机模式（官方方法，练肌肉记忆）

```
1. USB 线连电脑
2. 模式开关拨到 Off
3. 按住 Esc（或空格下方复位小孔）
4. 保持按住，开关拨到 Cable
5. 松开 Esc
```

验证进入成功：`dfu-util -l` 应看到 `STM32 BOOTLOADER`：

```
Found DFU: [0483:df11] ... name="@Internal Flash /0x08000000/0128*0002Kg"
```

（macOS 系统报告里显示 `STM32  BOOTLOADER`，Vendor 0483 = STMicroelectronics）

### 5.2 刷入

```bash
source ~/.qmk-venv/bin/activate
export PATH="/usr/local/opt/arm-none-eabi-gcc@8/bin:/usr/local/opt/arm-none-eabi-binutils/bin:$PATH"
cd ~/qmk_firmware
qmk flash -kb keychron/k2_pro/ansi/rgb -km default
```

刷机日志关键行：

```
Found DFU: [0483:df11]
Downloading element to address = 0x08000000, size = 58928
Erase done. / Download done.
File downloaded successfully
Transitioning to dfuMANIFEST state
```

### 5.3 验证

- 键盘自动退出 DFU，重新以 `Keychron K2 Pro`（Product ID 0x0220）出现
- **正常打字验证通过**（用户实际用该键盘输入确认）

## 六、回退原厂固件（备用）

```bash
dfu-util -d 0483:df11 -a 0 -s 0x08000000:leave -D stock-firmware/keychron_k2_pro_ansi_rgb_via.bin
```

## 七、蓝牙结论（重要）

- 本次编译的 QMK 固件**包含完整蓝牙支持**（`KC_BLUETOOTH_ENABLE`，CKBT51 驱动已链接进固件）
- 刷 QMK 后蓝牙仍可用，但配对方式可能不同、稳定性有差异报告 → **练手期主用有线**
- 原厂固件已备份，蓝牙体验不佳可随时刷回

## 八、环境备忘（下次继续）

```bash
# 每次新终端使用前：
source ~/.qmk-venv/bin/activate
export PATH="/usr/local/opt/arm-none-eabi-gcc@8/bin:/usr/local/opt/arm-none-eabi-binutils/bin:$PATH"
# QMK_HOME 默认 = ~/qmk_firmware（分支 bluetooth_playground）
```

**待办**：
- [ ] avr-gcc 工具链（阶段 2 Pro Micro 需要，源码编译 30min+，提前装）
- [ ] 第二次刷机练习（TODO.md）
- [ ] 阶段 2：焊接物料下单（Pro Micro、1N4148、轴体、面包板、焊台）
