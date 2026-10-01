# 更新日志

本文件记录本项目（**ImmortalWrt-XG-040G**，Nokia XG-040G 系列）的重要变更。
ImmortalWrt 上游合并仅记录会影响本设备构建或运行行为的内容。

## 2026-10-01

### 首次成功构建并发布

- 完成 3 个 **UBI 布局**设备的固件构建：`nokia_xg-040g-md-ubi`、`nokia_xg-040g-tf-ubi`、`nokia_xg-040g-md-tcboot`。
- 固件名发行版前缀由 `ImmortalWrt naoki66 ...` 改为 `ImmortalWrt 040G-MD-immortwrt`（`scripts/set-build-version.sh`）。

### 设备阵容调整

- **移除未验证的标准 NAND 独立分区变体**（`nokia_xg-040g-md` / `nokia_xg-040g-tf`）：该布局（stock-parts）与真实硬件
  （tcboot U-Boot + `ubi.mtd=ubi`）不符，且上游（naoki66 / ponwrt / bingoguo93）均无该布局的编译验证记录。
- 保留 3 个 UBI 设备，统一复用 `Device/nokia_xg-040g-md-images` 镜像配方。

### 镜像布局（对齐 ponwrt）

- `KERNEL=kernel-bin|gzip` + 独立 `-recovery.itb`（initramfs）+ `sysupgrade.itb`（FIT：内核 + external rootfs + metadata）。
- 移除固定 8MB kernel 分区与 `factory-kernel.bin` / `factory-rootfs.bin` / `factory.bin`：
  `CONFIG_TARGET_ROOTFS_INITRAMFS=y` 使主内核携带 initramfs（约 51MB），与 8MB kernel 分区必然冲突。
- `nokia_xg-040g-md-tcboot` 设备树补充 `chosen/rootdisk = <&ubi_fit>`，使 `fitblk` 在 sysupgrade 时能定位 FIT 卷。

### 设备树修复

- 为 stock / tcboot 布局补齐 `pon_serial` 与 `pon_calibration` nvmem 单元，修复 DTC
  `phandle_references` 报错（`&xpon_mac` / `&en7572` 引用了仅 UBI 布局定义的标签）。

### 内核与驱动

- `NET_AIROHA` / `NET_AIROHA_NPU` 保持**内置（=y）**，确保 out-of-tree xPON 模块可链接 `airoha_eth` 的导出符号。
- NPU 固件缺失由 `-ENOENT` 映射为 `-EPROBE_DEFER`，允许探测重试。

### PON 与软件包

- PON 前端驱动 `kmod-airoha-en7572`、`kmod-airoha-paged-bosa` 改为 **`=y`**：
  原 `=m` 仅编译不打包进固件，`MODULE_DEFAULT` 无法加载不存在的 `.ko`，光模块前端将不可用。
- 对齐 bingoguo93 配置：`kmod-airoha-xpon=y`、`kmod-airoha-pon-frontend=y`。
- USB / 存储 / 文件系统包按设备需求启用（`kmod-usb3`、`kmod-usb-storage`、`kmod-fs-*` 等）。

### CI 修复

- verbose retry 步骤支持 `target/*` 失败路径：原实现会拼出不存在的 `package/target/linux/compile`，掩盖真实错误。
- `tf-ubi` 的引导产物改用 `nokia_xg-040g-md` U-Boot 变体（`uboot-airoha` 仅构建该变体）。
- 移除 Release body 中的上游捐赠图片，并清理仓库内残留的上游图片文件。
- `*.fip` 与 `*preloader.bin` 纳入 Actions artifact（不再只留在构建目录）。
