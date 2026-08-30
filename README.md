# Alice 键盘练手项目

从零开始学习客制化键盘：最终目标是**手搓一把 USB 连接的 Alice 配列键盘**。
当前阶段：利用闲置的 **Keychron K2 Pro** 练软件（QMK），并行练焊接，最后手搓 Alice。

---

## 项目路线图

```
阶段 0  搞懂原理（矩阵、二极管、物料）        ── docs/03
阶段 1  软件先行：QMK 编译/刷机跑通          ── docs/03 + docs/04
阶段 2  最小焊接闭环：Pro Micro + 面包板     ── docs/03
阶段 3  手搓 Alice：矩阵设计 + 焊接 + 固件   ── docs/03
阶段 4  进阶：VIA / RGB / 旋钮 / KiCad PCB  ── docs/03
```

当前进度：

- [x] 无线方案调研（蓝牙 vs 2.4G）→ docs/01
- [x] 分体键盘架构调研（单 dongle 星型 / 有线分体）→ docs/02
- [x] 阶段 1：K2 Pro 编译 + 刷机跑通（2026-08-29 第一轮刷机成功）→ docs/05 + TODO.md
- [ ] 阶段 2：焊接物料下单
- [ ] 阶段 3：手搓 Alice

## 文档目录

| 文档 | 内容 |
|---|---|
| [docs/01-wireless-bluetooth-vs-24g.md](docs/01-wireless-bluetooth-vs-24g.md) | 蓝牙 vs 2.4G 无线方案对比 |
| [docs/02-split-keyboard-architecture.md](docs/02-split-keyboard-architecture.md) | 分体键盘架构：无线星型 vs 有线分体 |
| [docs/03-beginner-roadmap.md](docs/03-beginner-roadmap.md) | 从零开始练手路线（含物料清单、资源链接） |
| [docs/04-keychron-k2pro-guide.md](docs/04-keychron-k2pro-guide.md) | Keychron K2 Pro 练手指南（软件侧） |
| [docs/05-macos-qmk-setup-flash-log.md](docs/05-macos-qmk-setup-flash-log.md) | macOS QMK 环境搭建 + K2 Pro 刷机操作归档（含踩坑记录） |
| [docs/06-stage2-prep-guide.md](docs/06-stage2-prep-guide.md) | 阶段 2 准备：二极管/鬼键原理 + RP2040 引脚规划 + 澳洲购物清单 |
| [docs/07-firmware-build-ingredients.md](docs/07-firmware-build-ingredients.md) | QMK 固件编译"原材料"拆解（4 层配方 + 工具链，以 K2 Pro 为例） |
| [docs/08-k2pro-bluetooth-architecture.md](docs/08-k2pro-bluetooth-architecture.md) | K2 Pro 蓝牙方案源码拆解（双芯片架构、配对/电池/低功耗） |
| [reference/k2-pro-source/](reference/k2-pro-source/REFERENCE.md) | K2 Pro 官方固件源码参考副本（matrix.c/config.h/keymap.c 等） |
| [TODO.md](TODO.md) | 今晚回家行动清单 |

## 关键结论速查

1. **无线方案**：游戏用 2.4G，办公/多设备用蓝牙，最优解是三模键盘
2. **分体键盘无线化**：左右两半无连接线 + 单个 2.4G dongle **可行**（星型拓扑，dongle 轮询合并），但固件复杂；**有线分体是工程上的明智选择**
3. **新手路径**：不需要嵌入式经验，QMK 让固件变配置；先软件后硬件
4. **K2 Pro**：官方支持 QMK/VIA，是完美的软件练手机；练不了焊接，需另买 Pro Micro
