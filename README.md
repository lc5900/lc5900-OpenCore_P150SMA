# Clevo P150SMA OpenCore

Clevo P150SMA 的维护型 OpenCore 配置。2026-08 更新分支将引导器迁移到 OpenCore 1.0.7，并保留原机的 ACPI、HD4600、USB 和 ALC892 layout 55 定制。

> 这台机器已经不在维护者手中。当前更新通过静态检查，但没有真机启动、睡眠、音频、显示输出或升级测试。刷写前请备份可启动 EFI，并准备可恢复的 USB EFI。

## 当前组件

| 组件 | 版本 |
| --- | --- |
| OpenCore | 1.0.7 RELEASE |
| Lilu | 1.7.2 |
| WhateverGreen | 1.7.0 |
| VirtualSMC / plugins | 1.3.7 |
| VoodooPS2Controller | 2.3.7 |
| HibernationFixup | 1.5.4 |
| RealtekRTL8111 | 3.0.0 |
| AppleALC | 1.9.8（P150SMA layout 55 定制 RELEASE） |

OpenCore 1.0.7 的 `ocvalidate` 检查结果为零错误。新版配置只加载 `HFSPlus.efi` 与 `OpenRuntime.efi`；旧 `Bootstrap.efi` 及非必要的 AudioDxe、ExFat、NTFS、FireWire UEFI 驱动已移除。

## macOS 版本边界

- **推荐基线：macOS Monterey 12**。Dortania 将 Haswell 笔记本的原生支持上限列为 Monterey；配置已改用 `MacBookPro11,4`。
- **Big Sur 11** 理论上仍可用，但本次没有做降级验证。
- **Ventura 13 至 Sequoia 15**：HD4600 需要 OpenCore Legacy Patcher 的 root patch，不能只升级 OpenCore/kext。SIP、SecureBootModel、系统更新及补丁重装都需要按 OCLP 流程处理。
- **Tahoe 26**：不作为当前可用目标。除 Haswell 图形补丁外，Tahoe 移除了 AppleHDA；本机定制 AppleALC 还需要恢复 AppleHDA 或等价方案。老 Atheros Wi-Fi/蓝牙驱动也没有现代系统保证。
- **macOS 27 及以后**：Apple 已结束 Intel Mac 的系统支持，不列入本 EFI 的维护目标。

## 首次使用必须修改

`EFI/OC/config.plist` 中的 SMBIOS 身份已清空，避免继续公开旧机器的序列号。请用 GenSMBIOS 或 OpenCore 的 `macserial` 为 `MacBookPro11,4` 生成并填写：

- `PlatformInfo -> Generic -> SystemSerialNumber`
- `PlatformInfo -> Generic -> MLB`
- `PlatformInfo -> Generic -> SystemUUID`
- `PlatformInfo -> Generic -> ROM`

当前配置沿用了原 EFI 的 `csr-active-config=ff0f0000` 和 `SecureBootModel=Disabled`。这是宽松安全配置；仅跑 Monterey 时建议在真机验证后恢复 SIP，并重新评估 SecureBootModel。若使用 OCLP，则以对应版本的补丁要求为准。

## 定制 AppleALC 更新

声卡源码位于 [lc5900/AppleALC-P150sma-ALC892](https://github.com/lc5900/AppleALC-P150sma-ALC892)，P150SMA 使用 ALC892 layout 55。当前 `AppleALC.kext` 来自该 fork 的 `1.9.7-p150sma.3` Release，实际构建版本为 1.9.8；不要直接换成官方通用包，否则会丢失本机 layout 55。

## 真机回归清单

1. 先从 USB EFI 启动并重置 NVRAM。
2. 验证内屏、HDMI/DP/VGA、亮度调节和睡眠唤醒。
3. 验证扬声器、耳机自动切换、内置麦克风与 HDMI 音频。
4. 验证 USB 端口映射、键盘、触控板、电池、电源管理和有线网卡。
5. 单独验证 Atheros Wi-Fi/蓝牙；现代 macOS 更建议更换受支持的 Broadcom/Intel 网卡并采用对应方案。
6. 确认一切正常后再替换硬盘 EFI；系统大版本升级前保留旧 EFI。
