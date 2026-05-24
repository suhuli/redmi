# Xiaomi Kernel Action Build

基于 `suhuli/Action-Build` 的思路改造，但目标改为小米/红米旧版非 GKI 内核。

当前首个目标：

- Redmi Note 11 Pro / Pro+ 国行 MTK，代号 `pissarro`
- SoC: MT6877
- Kernel: 4.14
- 官方源码: `MiCode/Xiaomi_Kernel_OpenSource`
- 分支:
  - `pissarro-r-oss` -> Android 11 / R
  - `pissarro-s-oss` -> Android 12 / S

> 注意：国际版 Redmi Note 11 Pro 5G 常见代号是 `veux`，高通平台，不能使用 `pissarro` 内核。

## 使用方式

1. 把本项目推送到 GitHub。
2. 打开 `Actions`。
3. 运行 `Build Kernel Xiaomi`。
4. 推荐参数：

```text
FILE: redmi_note11pro_pissarro_r   # Android 11
# 或
FILE: redmi_note11pro_pissarro_s   # Android 12

MANAGER_SOURCE: RESUKISU
KSU_META: main//
FAST_BUILD: true
```

构建完成后下载 `AnyKernel3_*` artifact。

## 当前实现范围

已做：

- 拉取小米官方源码
- 拉取 Android clang/gcc 预编译工具链
- 注入 ReSukiSU
- 开启 `CONFIG_KSU`
- 开启 `CONFIG_SECCOMP` / `CONFIG_SECCOMP_FILTER`
- 编译 `Image.gz-dtb`
- 打包 AnyKernel3
- 下载 ReSukiSU 管理器 APK 到刷机包

未默认启用：

- SUSFS：4.14 非 GKI 内核需要单独适配，不直接沿用一加 GKI 补丁。
- KPM/KPN：旧 MTK 4.14 需要实机验证后再启用。

## 设备确认

刷入前请在手机上确认：

```sh
getprop ro.product.device
getprop ro.boot.hardware
uname -r
```

`pissarro` 机型才使用本 workflow 的 `redmi_note11pro_pissarro_*`。
