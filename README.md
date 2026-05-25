# Xiaomi Kernel Action Build

基于 `suhuli/Action-Build` 的思路改造，目标是用 GitHub Actions 在线编译小米/红米旧版 4.14 非 GKI 内核，并集成 ReSukiSU、SUSFS、KPM、seccomp 等功能。

## 样板机型：Redmi 10X 5G

当前已验证可成功编译的样板机型：

| 项目 | 值 |
| --- | --- |
| 机型 | Redmi 10X 5G |
| 代号 | `atom` |
| SoC | MT6873 |
| Android | 11 / R |
| Kernel | 4.14 |
| 类型 | Non-GKI |
| MiCode 分支 | `bomb-q-oss` |
| defconfig | `atom_user_defconfig` |
| 编译目标 | `Image.gz-dtb` |
| workflow 选项 | `redmi_10x5g_atom_r` |

推荐构建参数：

```text
FILE: redmi_10x5g_atom_r
MANAGER_SOURCE: RESUKISU
KSU_META: main//
FAST_BUILD: true
ENABLE_KPM: true
ENABLE_SUSFS: true
SUSFS_META: kernel-4.14//
ZERO_WIDTH_FIX: true
```

最近一次成功构建：

- Run: https://github.com/suhuli/redmi/actions/runs/26370235104
- Artifact: `AnyKernel3_ReSukiSU_4220_redmi_10x5g_atom_r_resukisu-susfs-kpm-zerowidth-real-fts-v16.zip`
- Artifact ID: `7187696403`

注意：这个 run 已验证内核本体可成功编译，但对应 AK3 包仍带有 AnyKernel3 示例脚本，不建议直接作为刷机包使用。应基于原厂 `boot.img` 替换其中的 `Image.gz-dtb` 生成新的 `boot.img` 后测试；后续新 run 会使用修正后的 Redmi 10X 5G AK3 脚本。

已确认配置：

```text
CONFIG_KSU=y
CONFIG_KPM=y
CONFIG_KSU_SUSFS=y
CONFIG_SECCOMP=y
CONFIG_SECCOMP_FILTER=y
SUSFS_VERSION=v1.5.5
ReSukiSU: using SuSFS Inline hook
```

## Redmi 10X 5G 的 FocalTech 处理

`bomb-q-oss` 源码缺少 FocalTech 触摸驱动编译期 include 文件：

```text
drivers/input/touchscreen/focaltech_touch/include/firmware/fw_ft3518_j7.i
drivers/input/touchscreen/focaltech_touch/include/firmware/fw_sample.i
drivers/input/touchscreen/focaltech_touch/include/pramboot/FT8719_Pramboot_V0.5_20171221.i
```

项目已改为从 MiCode 官方 `cannon-q-oss` 分支补全这些文件，并校验大小和 SHA256。`bomb-q-oss` 与 `cannon-q-oss` 的 FocalTech 驱动源码已比对一致，因此这里不再使用 `0x00` 占位文件。

当前校验值：

| 文件 | 大小 | SHA256 |
| --- | ---: | --- |
| `fw_ft3518_j7.i` | `279409` | `c063733f44d1cdef4d3df9d5168c63b21f609b1297dfb7c1cdb9e2456da646b2` |
| `fw_sample.i` | `0` | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| `FT8719_Pramboot_V0.5_20171221.i` | `17347` | `2a1513c55730ec453846f10b0c93ecd0bf7af737b408a11e7025f6b4dac685f4` |

构建日志中应出现：

```text
fw_ft3518_j7.i.tmp: OK
fw_sample.i.tmp: OK
FT8719_Pramboot_V0.5_20171221.i.tmp: OK
```

## 已接入机型

| workflow 选项 | 机型 | Android | MiCode 分支 | defconfig | 状态 |
| --- | --- | --- | --- | --- | --- |
| `redmi_10x5g_atom_r` | Redmi 10X 5G / atom | 11 / R | `bomb-q-oss` | `atom_user_defconfig` | 已成功编译，作为样板 |
| `redmi_note11pro_pissarro_r` | Redmi Note 11 Pro / Pro+ 国行 MTK / pissarro | 11 / R | `pissarro-r-oss` | `pissarro_user_defconfig` | 已接入，需按机型验证 |
| `redmi_note11pro_pissarro_s` | Redmi Note 11 Pro / Pro+ 国行 MTK / pissarro | 12 / S | `pissarro-s-oss` | `pissarro_user_defconfig` | 已接入，需按机型验证 |

注意：国际版 Redmi Note 11 Pro 5G 常见代号是 `veux`，高通平台，不能使用 `pissarro` 内核。

## 项目运转逻辑

工作流文件：

```text
.github/workflows/Build Kernel Xiaomi.yml
```

整体流程：

```text
GitHub Actions 手动触发
  -> 解析机型配置
  -> 拉取 MiCode 小米官方内核源码
  -> 修复 MTK Python2 DCT 脚本兼容问题
  -> 拉取 Android clang/gcc 工具链
  -> 注入 ReSukiSU
  -> 可选打 SUSFS kernel-4.14 补丁
  -> 可选启用零宽修复
  -> 手动补 ReSukiSU / SUSFS hook
  -> 按机型做源码兼容修复
  -> 生成 .config
  -> 编译 Image.gz-dtb
  -> 打包 AnyKernel3
  -> 下载 ReSukiSU Manager APK
  -> 上传 AnyKernel3 artifact
```

## 功能支持情况

| 功能 | 状态 | 说明 |
| --- | --- | --- |
| ReSukiSU | 支持 | 默认从 `ReSukiSU/ReSukiSU` 拉取 |
| KPM | 支持 | 通过 `ENABLE_KPM` 控制 |
| SUSFS | 支持 | 使用 `susfs4ksu` 的 `kernel-4.14` 分支 |
| 零宽修复 | 支持 | 通过 `ZERO_WIDTH_FIX` 控制 |
| seccomp | 支持 | 启用 `CONFIG_SECCOMP` / `CONFIG_SECCOMP_FILTER` |
| AnyKernel3 打包 | 支持 | 产物为 `AnyKernel3_*` artifact |
| ReSukiSU Manager | 支持 | 自动下载并放入刷机包 |

## 使用方式

1. 打开仓库的 `Actions` 页面。
2. 选择 `Build Kernel Xiaomi`。
3. 点击 `Run workflow`。
4. 选择机型和功能开关。
5. 构建完成后下载 `AnyKernel3_*` artifact。

对 Redmi 10X 5G，推荐优先使用基于原厂 `boot.img` 合成的新 boot 镜像做真机测试。如果使用 AK3 zip，请确认 `anykernel.sh` 中至少满足：

```text
device.name1=atom
device.name2=bomb
BLOCK=boot
IS_SLOT_DEVICE=0
split_boot;
flash_boot;
```

Redmi 10X 5G 推荐直接使用样板参数：

```text
FILE: redmi_10x5g_atom_r
MANAGER_SOURCE: RESUKISU
KSU_META: main//
FAST_BUILD: true
ENABLE_KPM: true
ENABLE_SUSFS: true
SUSFS_META: kernel-4.14//
ZERO_WIDTH_FIX: true
```

## 设备确认

刷入前请在手机上确认设备代号和内核版本：

```sh
getprop ro.product.device
getprop ro.boot.hardware
uname -r
```

Redmi 10X 5G 应重点确认：

```text
ro.product.device: atom
Kernel: 4.14.x
Android: 11 / R
```

## 说明

本项目目前的样板目标是让 Redmi 10X 5G 的非 GKI 4.14 内核在 GitHub Actions 上可重复编译，并保留清晰的适配逻辑。后续新增机型时，建议按 Redmi 10X 5G 的方式处理：

1. 明确 MiCode 分支、defconfig、内核产物路径。
2. 先保证原厂源码可编译。
3. 再逐步加入 ReSukiSU、SUSFS、KPM、零宽修复。
4. 对缺失源码文件必须找到可信来源或禁用相关逻辑，不建议使用伪造占位文件。
