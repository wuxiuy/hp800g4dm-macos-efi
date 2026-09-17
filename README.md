<p align="center" style="font-size:3em; font-weight:bold; margin:0.3em 0;">⭐ 本 EFI 由豆包（Doubao AI）全程适配与调校</p>

<p align="center" style="font-size:1.4em; color:#444;">macOS Sequoia 15.6 · 稳定版</p>

---

# hp800g4dm-macos-efi — HP EliteDesk 800 65W G4 微型台式电脑 黑苹果 EFI

在 **HP EliteDesk 800 65W G4 微型台式电脑 / HP EliteDesk 800 65W G4 Mini Desktop PC**（官方中英文名，即常说的 HP EliteDesk 800 G4 Mini；i7-8700 / UHD630）上运行 **macOS Sequoia 15.6** 的 OpenCore 引导方案（与 Win11 双系统）。

## 硬件配置

| 部件 | 型号 | 备注 |
|---|---|---|
| 机型 | HP EliteDesk 800 65W G4 Mini | 标压版，90W 电源 |
| CPU | Intel Core i7-8700 | 6核12线程，3.2GHz / 睿频4.6GHz，65W TDP |
| 核显 | Intel UHD Graphics 630 | DP→HDMI 输出 |
| 内存 | 12GB DDR4 | 双通道 |
| 芯片组 | Intel Q370 | |
| 声卡 | Conexant CX20632 | layout-id = 0x14 |
| 有线网卡 | Intel I219-V | 千兆 |
| 无线网卡 | Intel AX200 | Wi-Fi 6，itlwm + HeliPort |
| 蓝牙 | AX200 内置蓝牙 | 不可用（软件层无解） |
| 磁盘0 | SAMSUNG MZVL2512HCJQ 512GB NVMe | Windows 11 |
| 磁盘1 | SAMSUNG MZVL2256HCHQ 256GB NVMe | macOS Sequoia 15.6 (Build 24G84) |

## 实现方式（怎么实现的）

| 组件 | 方案 |
|---|---|
| 引导 | OpenCore 1.0.7 · SMBIOS **Macmini8,1** |
| 核显 | WhateverGreen（UHD 630，DP→HDMI） |
| 声卡 | AppleALC（CX20632，layout-id 0x14） |
| 有线 | IntelMausiEthernet（I219-V，**千兆**） |
| 无线 | itlwm 2.3.0 + **HeliPort**（AX200 Wi-Fi） |
| 蓝牙 | **不可用**（AX200 蓝牙软件层无解；需要就买 USB 免驱棒） |
| 电源 | CPUFriend 功耗方案（见下） |

## CPU 功耗方案（重要：低功耗 CPU 用户必读）

> **本 EFI 内置 CPUFriend 功耗方案，按 i7-8700 标压 65W 平台调校**（800 G4 Mini 标压版，90W 电源）。

| 项 | 值 |
|---|---|
| 目标功耗口径 | PL1 = 65W / PL2 = 85W |
| 实现 | `CPUFriend.kext` + `CPUFriendDataProvider.kext` |
| LFM / HWP EPP / PerfBias | 0x08（800MHz）/ 0x40 / 0x04 |

**实测（i7-8700）**：全核满载 74-75W / 4.16-4.19GHz；单核 4.26-4.30GHz；空闲 **3.7-4.7W**（对齐 Win11 低负载 ~6W）。

**⚠️ 低功耗 CPU（T 系列 35W，如 i5-8500T / i7-8700T）用户**：本方案会让电源管理尝试拉高到 65W+，超出 35W 平台散热/供电能力。处理方式任选其一：
1. **推荐**：config → `Kernel > Add` 取消 `CPUFriend.kext` 与 `CPUFriendDataProvider.kext`（或删 `OC/Kexts/` 下对应文件）→ 回 macOS 原生电源管理，T 系列自动按 35W 调度；
2. 用 [CPUFriendFriend](https://github.com/corpnewt/CPUFriendFriend) 按自己 CPU 重新生成 provider（T 系列用更高 LFM、更省电 EPP）；
3. 只删 DataProvider（等同方案 1）。

移除 = 删两个 kext 即回退默认，无副作用。

## 备份与安装（细节见 01/02 文档）

**备份**：git 双仓推送（GitHub 公开仓库 `wuxiuy/hp800g4dm-macos-efi` + 本地私有镜像仓库）
```bash
git push github main && git push gitea main
```

**安装**：macOS Sequoia **15.6 老吴恢复版**（hpglw.com，~14GB）+ **R-Drive Image 7** 恢复到 APFS 容器，OpenCore 引导；原版备选 = 官方 InstallAssistant.pkg 做 U 盘（下载地址见 01/02 文档）。

## 文档导航

| 文档 | 内容 |
|---|---|
| `00-项目总览与状态.md` | 机器/EFI 配置/硬件功能全景/待办 |
| `01-驱动与软件指南.md` | 驱动清单 + HeliPort 用法 + 15.6 镜像下载 + BIOS |
| `02-装机与排错手册.md` | 装机流程 + 排错表 + 铁律 + 历史教训 |
| `03-升级与蓝牙记录.md` | 升级规划（15.8 → Tahoe 26）+ 蓝牙终局 + AirportItlwm 考证 |

## 铁律

1. `BOOTx64.efi` 必须保持原版 stub（严禁替换为 OpenCore 副本）
2. 修改 config 前先读 `02-装机与排错手册.md`（有据可依 / 一次解决）
3. EFI 备份不要放进 ESP（200MB 易爆），走 git / 本地目录
