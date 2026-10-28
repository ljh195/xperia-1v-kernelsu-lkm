# SukiSU LKM for Sony Xperia 1 V (pdx234 / XQ-DQ72)

为 **LineageOS 24.0 NIGHTLY** 编译的 **SukiSU Ultra `kernelsu.ko`**（LKM 模式），不替换内核文件。

## 目标设备

| 项目 | 值 |
|---|---|
| 机型 | Sony Xperia 1 V（XQ-DQ72 / pdx234） |
| 系统 | LineageOS 24.0 NIGHTLY（`24.0-20261025-NIGHTLY-pdx234`） |
| 内核 | 5.15.211-gdb39b3851f98（GKI 5.15） |
| 内核源码 | `LineageOS/android_kernel_sony_sm8550` @ `lineage-24.0` |
| 源码 commit | `db39b3851f981d5ef933437758e662cfa272a007`（与设备 `uname -r` 一致） |
| 工具链 | AOSP clang r596125（clang 22.0.2，与设备内核一致） |

## 产物

- `kernelsu-5.15.211-pdx234.ko` — SukiSU Ultra 内核模块（LKM）
- `SHA256SUMS.txt`

## 技术要点

- `CONFIG_KSU=m`（tristate，编成可加载模块）
- `CONFIG_KSU_MANUAL_SU=y`
- `CONFIG_MODULE_SIG_FORCE` 在设备内核中**未开启**，因此未签名模块可以加载
- `CONFIG_MODVERSIONS=y`，故需先完整编译 `vmlinux` 以生成 `Module.symvers`
- vermagic 与设备内核完全一致（同 commit + 同 clang 大版本）

## 使用（需要 USB + fastboot）

> ⚠️ 阶段说明：产物目前只是 `kernelsu.ko`。要真正生效还需要把它塞进 `init_boot` 并让 `init` 加载（SukiSU 的 LKM 安装流程），此步骤尚未自动化。

1. 从 Actions Artifacts 下载 `kernelsu-5.15.211-pdx234.ko`
2. 用 SukiSU Manager 的 LKM 安装功能包装 `init_boot`
3. 通过 fastboot 刷入 `init_boot`

若加载失败：刷回原始 `init_boot` 即可恢复，不影响系统启动。

## 回退

**先备份再操作：**

```bash
adb reboot bootloader
fastboot fetch init_boot_a init_boot_a.orig.img   # 保留原始镜像
```

恢复：

```bash
fastboot flash init_boot init_boot_a.orig.img
fastboot reboot
```

## 风险

- 不替换内核，最坏情况是模块加载失败、没有 root，**不会变砖**
- 但任何 boot 分区的写入都有风险，操作前务必确认备份到手
