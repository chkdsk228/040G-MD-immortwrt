<img src="https://avatars.githubusercontent.com/u/53193414?s=200&v=4" alt="logo" width="200" height="200" align="right">

# ImmortalWrt-XG-040G-Enhanced (forked from naoki66/ImmortalWrt-for-Gemtek-brightspeed)

[![Build Status](https://img.shields.io/github/actions/workflow/status/chkdsk228/040G-MD-immortwrt/build-firmware.yml?branch=master&label=Build)](https://github.com/chkdsk228/040G-MD-immortwrt/actions/workflows/build-firmware.yml)
[![Sync Status](https://img.shields.io/github/actions/workflow/status/chkdsk228/040G-MD-immortwrt/sync-upstream.yml?branch=master&label=Sync)](https://github.com/chkdsk228/040G-MD-immortwrt/actions/workflows/sync-upstream.yml)
[![SoC](https://img.shields.io/badge/SoC-Airoha%20AN7581-orange)]()
[![License](https://img.shields.io/badge/license-GPL--2.0-green)](https://spdx.org/licenses/GPL-2.0-only.html)

基于 [naoki66/ImmortalWrt-for-Gemtek-brightspeed](https://github.com/naoki66/ImmortalWrt-for-Gemtek-brightspeed)
与 [ImmortalWrt](https://github.com/immortalwrt/immortalwrt) 为 **Nokia (贝尔) XG-040G** 系列 Airoha AN7581
光猫设备维护的固件项目。

## 支持的设备

| 设备 | 配置 | 说明 |
|---|---|---|
| **XG-040G-MD** | `040g.config` | 标准 NAND 版 |
| **XG-040G-MD (UBI)** | `040g.config` | UBI 布局版 |
| **XG-040G-TF (UBI)** | `040g.config` | TF 面板版 (DTS 借用于 ponwrt) |

> 仅启用以上 3 个 XG-040G 变体编译；XR1710G/XG2010G 保持可用（1710.config/2010.config）。

## 特性

- **双 WAN 负载均衡**: lan1+lan2 → LAN, lan3 → wan2, lan4 → wan（自研布局）
- **PON 支持**: `CONFIG_AIROHA_PON_COMPAT=y`
- **NPU 面板**: `luci-app-airoha`（naoki66 新版）
- **CPUFreq/PM 域**: ARM AIROHA SoC CPUFreq + CPU PM Domain
- **内核配置导出**: IKCONFIG
- **默认后台**: 192.168.1.1

## 编译流程

手动触发 Actions 工作流 **Build Firmware**:

1. **config_seed** 选择 `040g.config`
2. **release_type** 选择 `none` / `release` / `prerelease`
3. 运行后产物输出到 `bin/targets/airoha/an7581/`

自动同步: **Sync Upstream ImmortalWrt** 每 3 天合并上游更新。

## 设备树来源

- `an7581-nokia_xg-040g-md*.dts*`: naoki66 上游
- `an7581-nokia_xg-040g-tf*.dts*`: 借用于 [pbs05/ponwrt](https://github.com/pbs05/ponwrt)

## 参考

- [naoki66/ImmortalWrt-for-Gemtek-brightspeed](https://github.com/naoki66/ImmortalWrt-for-Gemtek-brightspeed)
- [pbs05/ponwrt](https://github.com/pbs05/ponwrt)
