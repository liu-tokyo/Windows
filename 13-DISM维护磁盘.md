# DISM维护磁盘

在 Windows 系统中，**DISM**（Deployment Image Servicing and Management，部署映像服务和管理）是一个极其强大的命令行工具。虽然它常被误认为只是修复系统的，但它的核心功能是**扫描、修复和准备 Windows 映像文件**。

当你的系统出现运行缓慢、蓝屏、文件损坏或更新报错时，DISM 是最稳妥的“底层医生”。

------

## 1. DISM 维护三部曲

建议按以下顺序执行操作，这能让你在不重装系统的情况下解决 90% 的系统完整性问题。

### 1.1 检查损坏情况 (CheckHealth)

- 这个命令用于快速检测系统映像是否已被标记为损坏。它不会进行修复，只是“挂个号”。

  ```
  DISM /Online /Cleanup-Image /CheckHealth
  ```

### 1.2 全面扫描 (ScanHealth)

- 如果第一步没发现问题，但你依然觉得系统不对劲，可以用这个。它会扫描映像是否存在组件存储损坏。

  ```
  DISM /Online /Cleanup-Image /ScanHealth
  ```

### 1.3 终极修复 (RestoreHealth)

- 这是最关键的一步。它会对比在线的 Windows 更新源，自动下载并替换损坏的系统文件。

  ```
  DISM /Online /Cleanup-Image /RestoreHealth
  ```

------

## 2. 进阶维护技巧

### 2.1 清理更新冗余

- Windows 更新后往往会留下大量的旧组件记录，占用数 GB 的磁盘空间。

  ```
  DISM /Online /Cleanup-Image /StartComponentCleanup /ResetBase
  ```

  **注意：** 执行此操作后，你将无法卸载当前的系统补丁，但能腾出大量空间。

### 2.2 搭配 SFC 使用 (黄金组合)

DISM 修复的是系统的“底座”（组件存储），而 `sfc /scannow` 修复的是具体的“砖块”（系统文件）。

- **最佳流程：** 先运行 `RestoreHealth` 修复底座，再运行 `sfc /scannow` 扫描修复。

------

### 2.3 注意事项

1. **管理员权限：** 必须右键点击“命令提示符”或“PowerShell”，选择**以管理员身份运行**。
2. **网络连接：** 执行 `/RestoreHealth` 时建议保持联网，因为它需要从云端下载健康的系统组件。
3. **看似“卡住”：** 进度条经常会停在 20% 或 80% 很久，这是正常现象，请耐心等待完成。



## 3. WinPE 离线维护系统

在 WinPE 离线维护系统时，由于 `DISM` 的 `/Online` 参数专门用于指定“正在运行的本机操作系统”，所以在纯净的 WinPE 环境下执行带 `/Online` 的命令会直接报错。  
针对离线系统（如安装在硬盘上的 Windows 系统的离线维护），需要使用 **`/Image:路径`** 参数来替代 `/Online`。  
针对离线系统的修复与维护指令：

### 3.1 离线系统组件健康检查与修复

- 如果硬盘里的系统出现蓝屏、系统文件损坏等问题，在 WinPE 下可以通过指定目标系统的路径来进行修复：

  ```powershell
  dism /Image:C:\ /Cleanup-Image /RestoreHealth
  ```

  - `/Image:C:\`：指定需要维护的目标系统盘符根目录。
  - `/Cleanup-Image /RestoreHealth`：扫描并修复镜像中的损坏组件。

### 3.2 指定健康的“源”进行修复

> （离线修复常用技巧）

在纯净的 WinPE 环境下，由于没有本地组件库，DISM 尝试联网去 Windows Update 获取文件往往会失败。因此，离线修复时通常需要准备一个标准官方系统的 `install.wim`（或 `install.esd`）作为健康的源。

- 假设你手头有一个包含标准系统的镜像，解压或挂载后其 `install.wim` 在 `D:\sources\install.wim`（或者直接解压出来的 `sources` 文件夹路径）：

  ```powershell
  dism /Image:C:\ /Cleanup-Image /RestoreHealth /Source:wim:D:\sources\install.wim:1 /LimitAccess
  ```

  - `/Source:wim:D:\sources\install.wim:1`：指定源镜像中对应的系统版本索引（通常 Windows 10/11 专业版是索引 1 或 2，可用 `/Get-WimInfo /WimFile:...` 查看）。
  - `/LimitAccess`：**非常重要**，它会阻止 DISM 尝试去连接互联网下载文件，强制它仅从你指定的离线源中提取干净的文件进行替换。

### 3.3 查看离线系统组件状态（预检）

- 在直接修复前，可以先检查离线系统是否存在损坏：

  ```powershell
  dism /Image:C:\ /Cleanup-Image /CheckHealth
  ```

- 或者执行更彻底的扫描（不修复，只报告）：

  ```powershell
  dism /Image:C:\ /Cleanup-Image /ScanHealth
  ```