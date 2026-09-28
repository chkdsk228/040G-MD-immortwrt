<img src="https://avatars.githubusercontent.com/u/53193414?s=200&v=4" alt="logo" width="200" height="200" align="right">

# ImmortalWrt-XG-040G-Enhanced（Nokia XG-040G 系列）

[![Build Status](https://img.shields.io/github/actions/workflow/status/chkdsk228/040G-MD-immortwrt/build-firmware.yml?branch=master&label=Build)](https://github.com/chkdsk228/040G-MD-immortwrt/actions/workflows/build-firmware.yml)
[![Sync Status](https://img.shields.io/github/actions/workflow/status/chkdsk228/040G-MD-immortwrt/sync-upstream.yml?branch=master&label=Sync)](https://github.com/chkdsk228/040G-MD-immortwrt/actions/workflows/sync-upstream.yml)
[![Upstream](https://img.shields.io/badge/upstream-immortalwrt%40master-blue)](https://github.com/immortalwrt/immortalwrt)
[![Kernel](https://img.shields.io/badge/kernel-6.18-red)](https://www.kernel.org/)
[![SoC](https://img.shields.io/badge/SoC-Airoha%20AN7581-orange)]()
[![License](https://img.shields.io/badge/license-GPL--2.0-green)](https://spdx.org/licenses/GPL-2.0-only.html)

基于 [naoki66/ImmortalWrt-for-Gemtek-brightspeed](https://github.com/naoki66/ImmortalWrt-for-Gemtek-brightspeed) 与
[ImmortalWrt](https://github.com/immortalwrt/immortalwrt) 为 **Nokia（贝尔）XG-040G 系列** Airoha AN7581
光猫设备维护的固件项目。

项目继承 naoki66 的稳定构建体系、补丁维护与 CI 流程，并针对 XG-040G 系列进行了设备适配、PON 支持与双 WAN 定制。

当前维护一个相互隔离的硬件配置（`040g.config`），启用以下设备：

- **XG-040G-MD**：标准 NAND 布局版
- **XG-040G-MD (UBI)**：UBI 布局版
- **XG-040G-TF**：标准 NAND 布局版
- **XG-040G-TF (UBI)**：TF 面板版（设备树借用于 ponwrt）

## 支持设备

| 设备 | 构建配置 | 当前定位 | 设备树/镜像 |
|------|----------|----------|------------|
| Nokia XG-040G-MD（标准 NAND） | [`040g.config`](040g.config) | 标准固件 | [`an7581-nokia_xg-040g-md.dts`](target/linux/airoha/dts/an7581-nokia_xg-040g-md.dts) |
| Nokia XG-040G-MD（OpenWrt U-Boot UBI 布局） | [`040g.config`](040g.config) | 整盘 UBI 引导方案 | [`an7581-nokia_xg-040g-md-ubi.dts`](target/linux/airoha/dts/an7581-nokia_xg-040g-md-ubi.dts) |
| Nokia XG-040G-TF（标准 NAND） | [`040g.config`](040g.config) | TF 面板标准版 | [`an7581-nokia_xg-040g-tf.dts`](target/linux/airoha/dts/an7581-nokia_xg-040g-tf.dts) |
| Nokia XG-040G-TF（UBI 布局，TF 面板） | [`040g.config`](040g.config) | TF 面板 UBI 版（借用于 [pbs05/ponwrt](https://github.com/pbs05/ponwrt)） | [`an7581-nokia_xg-040g-tf-ubi.dts`](target/linux/airoha/dts/an7581-nokia_xg-040g-tf-ubi.dts) |

### XG-040G-MD

默认管理地址：http://192.168.50.1 或 http://immortalwrt.lan，用户名：**root**，密码：*无*。

| 项目 | 参数 |
|------|------|
| **SoC** | Airoha AN7581（4 核 CPU + NPU） |
| **光接入** | EN7572 xPON 前端（原厂定位为 XG(S)-PON 网关） |
| **以太网** | 板载 1G 交换端口（lan2/lan3/lan4）+ 独立 gdm4 口（lan1） |
| **无线** | 本配置不启用无线驱动（光猫网关定位） |

## 固件特性

### 核心定制

- 基于 [naoki66/ImmortalWrt-for-Gemtek-brightspeed](https://github.com/naoki66/ImmortalWrt-for-Gemtek-brightspeed) 的设备树与内核补丁体系（`target/linux/airoha/patches-6.18/`）。
- 启用 Nokia XG-040G-MD / MD-UBI / TF-UBI 三个设备 profile（`040g.config` 使用 multi-profile 一次构建多份固件）。
- TF 面板设备树 `an7581-nokia_xg-040g-tf-*.dts*` 借用于 [pbs05/ponwrt](https://github.com/pbs05/ponwrt)。
- PON 支持：`CONFIG_AIROHA_PON_COMPAT=y`，驱动与用户态来自 [pbs05/openwrt-pon-drivers](https://github.com/pbs05/openwrt-pon-drivers) 与 [pbs05/openwrt-pon-userspace](https://github.com/pbs05/openwrt-pon-userspace)（`kmod-airoha-en7572`、`kmod-airoha-pon-frontend`、`kmod-airoha-xpon`、`luci-app-pon`）。
- CPUFreq/PM 域：`CONFIG_KERNEL_ARM_AIROHA_SOC_CPUFREQ`、`CONFIG_KERNEL_AIROHA_CPU_PM_DOMAIN`、`CONFIG_KERNEL_CPUFREQ_DT`。
- 内核配置导出：`CONFIG_KERNEL_IKCONFIG` / `_PROC`。

### 网络与默认行为

- **双 WAN 负载均衡布局**：
  - `lan1`（gdm4）+ `lan2`（gsw_port2）→ `br-lan`（LAN）
  - `lan3`（gsw_port3）→ `wan2`
  - `lan4`（gsw_port4）→ `wan`
- 默认 LAN 地址为 `192.168.50.1`（naoki66 默认）。
- 默认启用 firewall4（nftables）软件 flow offload。

### 预装 LuCI 应用

| 应用 | 来源 | 功能 |
|------|------|------|
| `luci-app-airoha` | 本仓库合并（继承 naoki66） | SoC/NPU 状态与加速开关、FlowSense |
| `luci-app-airoha-fancontrol` | 继承 naoki66 | 风扇速度/温度控制 |
| `luci-app-pon` | [pbs05/openwrt-pon-userspace](https://github.com/pbs05/openwrt-pon-userspace) | PON 管理界面 |
| `luci-app-iptv` | [pbs05/openwrt-pon-userspace](https://github.com/pbs05/openwrt-pon-userspace) | IPTV 组播 |
| `luci-app-openclash` | 社区 | 科学上网 |
| `luci-app-lucky` | [sirpdboy/luci-app-lucky](https://github.com/sirpdboy/luci-app-lucky) | Lucky（DDNS/反代/端口转发） |
| `luci-app-mwan3` | 社区 | 多 WAN 负载均衡 |
| `luci-app-autoreboot` | 社区 | 定时重启 |
| `luci-app-samba4` | 社区 | SMB 共享 |
| `luci-app-upnp` | 社区 | UPnP 自动端口转发 |
| `luci-app-wolplus` | 社区 | 网络唤醒 |

## GitHub Actions 工作流

| 工作流 | 触发方式 | 功能 |
|--------|---------|------|
| [build-firmware.yml](.github/workflows/build-firmware.yml) | 手动 (workflow_dispatch) | 构建固件并发布 Release |
| [sync-upstream.yml](.github/workflows/sync-upstream.yml) | 每 3 天定时 + 手动 | 同步 ImmortalWrt 上游 |

**构建配置**：仓库根目录的 [040g.config](040g.config) 对应 XG-040G 系列（默认）。`040g.config` 使用 multi-profile 一次构建 MD / MD-UBI / TF / TF-UBI 四份固件；Action 默认使用 `040g.config`，也可以在手动触发时选择 `1710.config` 或 `2010.config`（保留 naoki66 原设备）。
构建流程会执行 `cp <config> .config && bash scripts/set-build-version.sh .config && make defconfig`。

**Release 格式**：
- Tag：`YYYYMMDD-<short-hash>`
- 名称：`YYYYMMDD - Gemtek XG-040G Build (<short-hash>)`
- 选项：`release` / `prerelease` / `none`

## 下载

- [Releases 页面](https://github.com/chkdsk228/040G-MD-immortwrt/releases)
- XG-040G-MD 标准版：`immortalwrt-YYYYMMDD-<repo-hash>-airoha-an7581-nokia_xg-040g-md-*`
- XG-040G-MD UBI 版：`immortalwrt-YYYYMMDD-<repo-hash>-airoha-an7581-nokia_xg-040g-md-ubi-*`
- XG-040G-TF 标准版：`immortalwrt-YYYYMMDD-<repo-hash>-airoha-an7581-nokia_xg-040g-tf-*`
- XG-040G-TF UBI 版：`immortalwrt-YYYYMMDD-<repo-hash>-airoha-an7581-nokia_xg-040g-tf-ubi-*`
- 升级方法：LuCI → 系统 → 备份/升级 → 刷写固件

### 升级注意事项

> [!WARNING]
> LuCI 中的“保留配置”不会保留额外安装的软件包。升级前请备份配置并记录已安装的软件包；升级后需要
> 重新安装 OpenClash、PassWall 等非预装组件。请使用与新固件匹配的软件包，不要恢复旧固件的 `kmod-*` 内核模块。

## 本地构建（可选）

```bash
git clone https://github.com/chkdsk228/040G-MD-immortwrt.git
cd 040G-MD-immortwrt
./scripts/feeds update -a
./scripts/feeds install -a
bash scripts/fix-stale-golang-host.sh
cp 040g.config .config
bash scripts/set-build-version.sh .config
make defconfig
make -j$(nproc) world 2>&1 | tee build.log
bash scripts/summarize-build-errors.sh build.log
```

构建环境要求：GNU/Linux 系统（Debian 11+ 推荐），AMD64 架构，至少 4GB RAM 和 25GB 可用磁盘空间。详细依赖请参考 [ImmortalWrt 官方文档](https://openwrt.org/docs/guide-developer/build-system/install-buildsystem)。

## 致谢

### 上游固件
- [naoki66/ImmortalWrt-for-Gemtek-brightspeed](https://github.com/naoki66/ImmortalWrt-for-Gemtek-brightspeed) - 本项目基础（设备树、补丁、CI 流程）
- [immortalwrt/immortalwrt](https://github.com/immortalwrt/immortalwrt) - ImmortalWrt 主项目
- [immortalwrt/luci](https://github.com/immortalwrt/luci) - LuCI Web 界面
- [immortalwrt/packages](https://github.com/immortalwrt/packages) - 社区软件包仓库
- [openwrt/routing](https://github.com/openwrt/routing) - OpenWrt 路由相关包

### PON 驱动与用户态
- [pbs05/ponwrt](https://github.com/pbs05/ponwrt) - XG-040G-TF 设备树借用来源
- [pbs05/openwrt-pon-drivers](https://github.com/pbs05/openwrt-pon-drivers) - PON 内核驱动（`kmod-airoha-en7572`、`kmod-airoha-xpon`）
- [pbs05/openwrt-pon-userspace](https://github.com/pbs05/openwrt-pon-userspace) - `airoha-ponctl`、`airoha-pond` 和 LuCI PON 用户态

## 许可证

[GPL-2.0-only](https://spdx.org/licenses/GPL-2.0-only.html)（继承 ImmortalWrt）
