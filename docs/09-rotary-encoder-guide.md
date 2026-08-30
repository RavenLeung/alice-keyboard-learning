# DIY 键盘旋钮（Rotary Encoder）指南：音量 + 达芬奇/LR 滑块控制

> 2026-08-30 整理。给 Alice 项目加一颗旋转编码器旋钮的完整方案：硬件选型、
> QMK 配置、以及针对达芬奇 Resolve / Lightroom 的软件映射玩法。
> 对应路线图"阶段 4：进阶：VIA / RGB / 旋钮"。

---

## 一、结论先行

- **完全可行，且余量充足**：RP2040-Plus 剩 8 个 GPIO，一颗 EC11 只占 3 个（2 旋转 + 1 按键），放 1 颗剩 5 个，放 2 颗都够
- **旋钮是达芬奇 / LR 工作流里性价比最高的外设**——专业调色台（Loupedeck、TourBox）的核心就是一堆旋钮
- 核心玩法：**鼠标悬停在滑块上滚动滚轮即可调滑块**，LR 和达芬奇都原生支持，QMK 里模拟鼠标滚轮即可，不需要任何宿主软件配合
- QMK 新版 `ENCODER_MAP` 支持**按层自动切换旋钮功能**，正好实现"音量层 / 滑块层 / 微调层"多用途

## 二、硬件方案

### 2.1 GPIO 余量（对齐 docs/06 规划）

```
RP2040-Plus 可用 GPIO：26
Alice 矩阵 6×12：     18
剩余：                8
  ↳ EC11 旋钮 ×1 = 3（A + B + 按键）→ 剩 5
  ↳ EC11 旋钮 ×2 = 6 → 剩 2
```

### 2.2 EC11 是什么

最常见的**带按键旋转编码器**，5 个脚：

| 脚 | 接法 |
|---|---|
| A、B（正交信号） | 各接一个 GPIO |
| 公共端（中间脚） | GND |
| 按键两脚 | 一脚接 GPIO、另一脚接 GND |

旋转时 A/B 输出相位差 90° 的脉冲，固件数脉冲即可知道转了几格、往哪转。常见规格 **20 脉冲/圈**，旋钮帽配 6mm 圆轴。

### 2.3 接线（面包板即可先玩）

```
EC11 公共端 ── GND
EC11 A ── GPIO（如 GP0）
EC11 B ── GPIO（如 GP1）
EC11 按键 ── GPIO（如 GP2），另一脚接 GND
```

上拉建议：买**带板载上拉的模块版**（如 KY-040，CLK/DT/SW/+ 四脚）最省事；裸 EC11 用 QMK 内部上拉，稳妥做法是加 10k 外部上拉。

### 2.4 采购

- Jaycar 有 EC11（约 A$4–6）
- 淘宝/拼多多几块钱一颗
- 墨尔本本地客制化店（Daily Clack / Switchkeys）也可能有
- 建议买 2–3 颗：练手 + 备用

## 三、QMK 配置（新版 ENCODER_MAP，推荐）

旧写法是 `encoder_update_user()` 回调手写逻辑；新写法直接在 keymap 数组里声明，**自动按层生效**，正是多用途场景的正解。

### 3.1 rules.mk

```make
ENCODER_ENABLE = yes
ENCODER_MAP_ENABLE = yes
```

### 3.2 config.h（可选）

```c
#define ENCODER_RESOLUTION 4   // 每格检测 4 个脉冲状态，手感更跟手
```

### 3.3 keymap.c：encoder 映射表

```c
#if defined(ENCODER_MAP_ENABLE)
const uint16_t PROGMEM encoder_map[][NUM_ENCODERS][2] = {
    [MAC_BASE] = { ENC_CCW(KC_VOLD),         ENC_CW(KC_VOLU) },        // 音量
    [MAC_FN]   = { ENC_CCW(KC_MS_WH_DOWN),   ENC_CW(KC_MS_WH_UP) },    // 鼠标滚轮 → 滑块
    [WIN_BASE] = { ENC_CCW(KC_VOLD),         ENC_CW(KC_VOLU) },
    [WIN_FN]   = { ENC_CCW(KC_MS_WH_DOWN),   ENC_CW(KC_MS_WH_UP) },
};
#endif
```

编译进固件后，**旋钮在不同层自动切换功能**，与 keymap 里的 `MO()` 层机制无缝配合。多颗旋钮时按 `[层][旋钮编号][CW/CCW]` 顺序展开即可。

> 兼容写法（老版本 QMK 也可用）：
> ```c
> bool encoder_update_user(uint8_t index, bool clockwise) {
>     if (index == 0) {
>         if (clockwise) tap_code(KC_VOLU);
>         else tap_code(KC_VOLD);
>     }
>     return true;
> }
> ```

### 3.4 常用键码速查

| 功能 | 键码 |
|---|---|
| 音量减 / 加 | `KC_VOLD` / `KC_VOLU`（长名 `KC_AUDIO_VOL_DOWN/UP`） |
| 鼠标滚轮上下 | `KC_MS_WH_UP` / `KC_MS_WH_DOWN` |
| 鼠标滚轮左右 | `KC_MS_WH_LEFT` / `KC_MS_WH_RIGHT` |
| 方向键（微调用） | `KC_LEFT` / `KC_RIGHT` |

## 四、软件映射玩法（按实用度排序）

### ① 音量（任何软件通用）

`KC_VOLU / KC_VOLD` 是系统级媒体键，全局生效。旋钮的默认归宿。

### ② 滑块控制 = 模拟鼠标滚轮（达芬奇 / LR 核心玩法）

**把鼠标悬停在滑块上，滚动滚轮即可调滑块**，LR 和达芬奇都原生支持：

- **Lightroom**：基本面板（曝光/色温）、HSL 面板所有滑块都吃滚轮
- **达芬奇**：调色面板 Lift/Gamma/Gain 数值栏、Hue 等悬停滚动有效

`KC_MS_WH_UP / KC_MS_WH_DOWN` 模拟滚轮即可，无需任何软件配合。

### ③ 微调层（进阶但极好用）

LR 和达芬奇的滑块 / 数值栏**选中后支持方向键微调**（←/→ 一格）。加一层：

```c
[ADJ_FINE] = { ENC_CCW(KC_LEFT), ENC_CW(KC_RIGHT) },  // 精细微调
```

### ④ 交互设计建议

**用 EC11 按键做模式切换**：按一下在"音量 → 滚轮 → 微调"间循环；或 Fn 层挂微调。
这样一颗旋钮 = 粗调 + 细调 + 音量三合一。

## 五、进阶方向（先知道，初期别做）

1. **加速手感**：转得快时一次跳多格（媒体滚轮那种惯性感）。QMK 里手写：记录两次旋转时间间隔，间隔短则 `tap_code` 多次。非标配功能，代码量很小
2. **VIA 免编译改旋钮**：VIA 支持 encoder 重映射，刷 VIA 版固件后可像改普通键一样改旋钮功能
3. **达芬奇 Python API + raw HID**：达芬奇有 Python 脚本接口，可做到 Loupedeck 级别的精确控制（调色节点等）。工作量大，现阶段不做

## 六、落地路径（对齐现有阶段）

| 阶段 | 动作 |
|---|---|
| 阶段 2（面包板） | EC11 插面包板 + RP2040，先跑通音量，再试滚轮控制 LR/达芬奇滑块——顺带练接线 |
| 阶段 3（Alice） | 右上角预留 1 个旋钮位（可留 2 个位置），装 1 颗 |
| 阶段 4（进阶） | 上 VIA + 打板时把旋钮焊进 PCB |

## 七、参考资料

- [QMK 官方 Encoders 文档](https://docs.qmk.fm/features/encoders)（ENCODER_MAP / 配置项权威来源）
- [QMK Encoder 映射 PR #13286](https://git.pngu.org/qmk_firmware/commit/?h=0.18.4&id=8d5eacb7dd76bfd45444ceb1efa9a9f840173552)（ENCODER_MAP 引入历史）
- [QMK 固件旋钮编码器完全指南（中文）](https://blog.csdn.net/supershmily/article/details/147660721)
