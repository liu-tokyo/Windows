# DISM维护磁盘

在 Windows 系统中，**DISM**（Deployment Image Servicing and Management，部署映像服务和管理）是一个极其强大的命令行工具。虽然它常被误认为只是修复系统的，但它的核心功能是**扫描、修复和准备 Windows 映像文件**。

当你的系统出现运行缓慢、蓝屏、文件损坏或更新报错时，DISM 是最稳妥的“底层医生”。

------

## 🛠️ DISM 维护三部曲

建议按以下顺序执行操作，这能让你在不重装系统的情况下解决 90% 的系统完整性问题。

### 1. 检查损坏情况 (CheckHealth)

这个命令用于快速检测系统映像是否已被标记为损坏。它不会进行修复，只是“挂个号”。

- **命令：** `DISM /Online /Cleanup-Image /CheckHealth`

### 2. 全面扫描 (ScanHealth)

如果第一步没发现问题，但你依然觉得系统不对劲，可以用这个。它会扫描映像是否存在组件存储损坏。

- **命令：** `DISM /Online /Cleanup-Image /ScanHealth`

### 3. 终极修复 (RestoreHealth)

这是最关键的一步。它会对比在线的 Windows 更新源，自动下载并替换损坏的系统文件。

- **命令：** `DISM /Online /Cleanup-Image /RestoreHealth`

------

## 💡 进阶维护技巧

### 🧹 清理更新冗余

Windows 更新后往往会留下大量的旧组件记录，占用数 GB 的磁盘空间。

- **深度清理：** `DISM /Online /Cleanup-Image /StartComponentCleanup /ResetBase`

  > **注意：** 执行此操作后，你将无法卸载当前的系统补丁，但能腾出大量空间。

### ⚙️ 搭配 SFC 使用 (黄金组合)

DISM 修复的是系统的“底座”（组件存储），而 `sfc /scannow` 修复的是具体的“砖块”（系统文件）。

- **最佳流程：** 先运行 `RestoreHealth` 修复底座，再运行 `sfc /scannow` 扫描修复。

------

## ⚠️ 注意事项

1. **管理员权限：** 必须右键点击“命令提示符”或“PowerShell”，选择**以管理员身份运行**。
2. **网络连接：** 执行 `/RestoreHealth` 时建议保持联网，因为它需要从云端下载健康的系统组件。
3. **看似“卡住”：** 进度条经常会停在 20% 或 80% 很久，这是正常现象，请耐心等待完成。