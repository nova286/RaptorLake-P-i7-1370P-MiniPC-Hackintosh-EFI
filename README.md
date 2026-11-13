# Raptor Lake-P (i7-1370P / Iris Xe `0xA7A0`) Mini-PC Hackintosh EFI — macOS Tahoe 26

搭载 **13 代酷睿 Intel Core i7-1370P（Iris Xe 96 EU `8086:A7A0`）** 迷你主机 / MoDT 主板的完整 OpenCore 黑苹果配置，已在 **macOS Tahoe 26.7.1 (`25G241`)** 下实机测试通过全部核心硬件模块。

Complete OpenCore EFI configuration for an **Intel 13th Gen Core i7-1370P (Raptor Lake-P, Iris Xe 96 EU `8086:A7A0` rev `0x04`)** Mini-PC / MoDT system, verified on **macOS Tahoe 26.7.1 (`25G241`)** with full Metal 2 4K60 HiDPI graphics acceleration, Wi-Fi, Bluetooth, Dual Ethernet, 14-port USB mapping, and onboard Conexant audio.

---

## 硬件配置与驱动状态 / Hardware & Status

| 模块 / Component | 硬件型号 / Hardware ID | 驱动方案 / Driver | 状态 / Status |
| :--- | :--- | :--- | :--- |
| **CPU** | Intel Core i7-1370P (6P + 8E, 14C/20T, `0x000B06A2`) | `Cpuid1Data` (`0x0A0655`) + `CpuTopologyRebuild 2.0.2` + `X86PlatformPlugin` | 正常 / Working |
| **核显 / iGPU** | Intel Iris Xe Graphics (`8086:A7A0` rev `0x04`, 96 EU, Display 13) | `WhateverXe 0.8.2` + `WhateverXe-Graphics 0.4.3` (`AppleIntelTGLCompat 0.8.1`) | **正常 / Working**（DP 4K@60Hz HiDPI、`1536 MB` VRAM、Metal 2 硬加速） |
| **有线网卡 1 / LAN 1** | Intel Ethernet I226-V 2.5GbE (`8086:125C`) | `AppleIGC 1.9d2` | 正常 / Working (`en0`, Built-in) |
| **有线网卡 2 / LAN 2** | Realtek RTL8168H/8111H 1GbE (`10EC:8168`) | `RealtekRTL8111 3.0.4` | 正常 / Working (`en1`, Built-in) |
| **无线网卡 / Wi-Fi** | Intel Wi-Fi 6 AX201 160MHz CNVi (`8086:51F1` / `0074:8086`) | `itlwm 2.3.0` + `HeliPort 1.5.0` | 正常 / Working |
| **蓝牙 / Bluetooth** | Intel AX201 USB Bluetooth (`8087:0026` on `HS10`) | `BlueToolFixup 2.7.3` → `IntelBluetoothFirmware 2.5.0` → `IntelBTPatcher 2.5.0` | 正常 / Working (`State: On`, FW `v13719`) |
| **声卡 / Audio** | Intel Raptor Lake-P HDA (`8086:51CA`) + Conexant (`14F1:1F87`) | `Extras/VoodooHDA.kext` (`3.6.8` in `/Library/Extensions` AuxKC) | 正常 / Working（3.5mm 耳机输出 + 麦克风输入） |
| **USB 定制 / USB** | Intel Alder/Raptor Lake PCH XHCI (`8086:51ED`) | `USBToolBox 1.2.0` + 定制 14 口 `UTBMap.kext` | 正常 / Working (`HS01..HS10` + `SS01..SS04`) |
| **存储 / Storage** | PCIe 4.0 NVMe SSD (`1E4B:1602`) | Native `IONVMeController` + `NVMeFix 1.1.3` | 正常 / Working (TRIM Enabled) |

---

## 使用前必读 / Setup Guide

### 1. BIOS 设置 / BIOS Settings
- **DVMT Pre-Allocated**: `96MB` 或 `128MB`（必须 ≥ 96MB；本机实测 `128MB`，`GGC = 0x04C1`）
- **IGD Aperture Size**: `256MB`
- **CSM (Compatibility Support Module)**: `Disabled`
- **Secure Boot**: `Disabled`
- **VT-d**: BIOS 可保持开启（`config.plist` 已开启 `DisableIoMapper = True`）

### 2. 显示器连接方式 / Display Connection
- 请使用机身上的 **标准 DisplayPort (DP) 接口**（物理路由为 `DDI D` / `TC1` Fixed Legacy DP）连接显示器。实测搭配 4K 显示器（Dell S2725QS）可直接开启原生 `3840x2160`（HiDPI `1920x1080 @ 60Hz`）及完整 Metal 2 硬件加速。

### 3. 生成你自己的三码 / Generate Your Own SMBIOS
仓库中的 `EFI/OC/config.plist` 已对个人设备序列号做脱敏处理。启动或登录 Apple ID 前，请使用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 或 OpenCore Configurator 为机型 **`iMac20,1`** 生成并填入：
- `PlatformInfo -> Generic -> SystemSerialNumber`
- `PlatformInfo -> Generic -> MLB`
- `PlatformInfo -> Generic -> SystemUUID`
- `PlatformInfo -> Generic -> ROM`

### 4. 进入系统后的配套组件安装 / Post-Install Components (macOS Tahoe 26)
本方案**不修改 macOS 密封系统根卷（`/` 保持 APFS Sealed）**，仅在 `/Library` 与应用层安装以下三项配套组件：

1. **核显用户态与 AuxKC 组件（WhateverXe Graphics）**
   - 前往 [b00t0x/WhateverXe-Graphics Releases](https://github.com/b00t0x/WhateverXe-Graphics/releases) 下载最新安装包并安装，在「系统设置 -> 隐私与安全性」批准第三方内核扩展后重启。
   - 注意：`EFI/OC/Kexts/AMFIPass.kext` 使用的是 WhateverXe 配套的 `AMFIPass 1.4.1 Mod`，且 `csr-active-config` 保持为 `03080000`。

2. **Wi-Fi 状态栏客户端（HeliPort）**
   - 下载安装 [HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases)（`v1.5.0+`），放入 `/Applications` 并开启开机自启。
   - 提示：`itlwm.kext` 将无线网卡模拟为以太网接口，系统自带「控制中心」的 Wi-Fi 开关无效，请点击顶部菜单栏中的 **HeliPort 图标**连接 Wi-Fi（macOS Tahoe 可在「系统设置 -> 菜单栏」底部确认允许 HeliPort 显示）。

3. **板载 Conexant 声卡驱动（VoodooHDA 3.6.8）**
   - macOS Tahoe 26 已移除原厂 `AppleHDA.kext`，且本机板载声卡为 Conexant `14F1:1F87`（非 Realtek ALC）。仓库 `Extras/VoodooHDA.kext` 已附带适配 Tahoe SDK 编译的 `v3.6.8` 版本：
   ```bash
   sudo cp -R Extras/VoodooHDA.kext /Library/Extensions/
   sudo chown -R root:wheel /Library/Extensions/VoodooHDA.kext
   sudo chmod -R 755 /Library/Extensions/VoodooHDA.kext
   sudo kmutil install --volume-root / --update-all --force
   ```
   - 在「系统设置 -> 隐私与安全性」点击允许并重启即可启用 3.5mm 音频输出与麦克风输入。

4. **关闭开机跑码（可选 / Optional）**
   - 默认 `boot-args` 包含 `-v` 便于排查问题；稳定使用后可在 `config.plist` 的 `boot-args` 中移除 `-v`（务必保留 `-no_compat_check -allow3d -disablegfxfirmware`）。

---

## 致谢 / Credits
- [Acidanthera](https://github.com/acidanthera) — OpenCorePkg, Lilu, VirtualSMC, NVMeFix, BrcmPatchRAM (`BlueToolFixup`)
- [b00t0x](https://github.com/b00t0x) — [WhateverXe](https://github.com/b00t0x/WhateverXe), [WhateverXe-Graphics](https://github.com/b00t0x/WhateverXe-Graphics), CpuTopologyRebuild
- [OpenIntelWireless](https://github.com/OpenIntelWireless) — itlwm, HeliPort, IntelBluetoothFirmware
- [CloverHackyColor](https://github.com/CloverHackyColor/VoodooHDA) — VoodooHDA
- [USBToolBox](https://github.com/USBToolBox/kext) — USBToolBox
