# 如何将 MBR 转换为 GPT 格式以启用 Secure Boot

这是一个非常关键的操作步骤！为了启用 Secure Boot，您的系统盘必须使用 **GPT（GUID 分区表）** 分区格式，并且您的 BIOS 必须设置为 **UEFI 模式**。

对于 Windows 10 或 Windows 11 系统，最安全、最推荐的**无损**转换方法是使用 Microsoft 官方提供的 **MBR2GPT** 工具。

## 🚀 步骤一：前置检查与准备

在进行转换之前，请务必完成以下检查和准备工作：

- **备份数据：** 尽管 MBR2GPT 工具被设计为无损转换，但任何涉及磁盘分区结构的更改都存在风险。**强烈建议您备份所有重要数据。**
- **确认 Windows 版本：** MBR2GPT 工具适用于 Windows 10 版本 1703 或更高版本，以及 Windows 11。
- **确认系统盘号：** 您需要知道要转换的系统盘的编号（例如 Disk 0）。
  - 按 Win+X，然后选择 **Disk Management（磁盘管理）**。
  - 找到您的系统盘（通常标记为 "Boot" 或 "System"），记下左侧显示的磁盘编号（如 Disk 0）。

## 🛠️ 步骤二：使用 MBR2GPT 进行转换（推荐无损）

您可以直接在运行中的 Windows 操作系统内使用此工具进行转换。

1. **以管理员身份打开命令提示符（CMD）或 PowerShell：**

   - 在搜索栏中输入 **cmd** 或 **PowerShell**。
   - 右键点击搜索结果，选择 **Run as administrator（以管理员身份运行）**。

2. **验证磁盘（可选但推荐）：**

   - 输入以下命令并按回车执行，将 `0` 替换为您的系统盘编号：

   mbr2gpt /validate /disk:0 /allowfullOS

   - 如果验证通过，您会看到类似 "Validation completed successfully"（验证成功完成）的提示。如果验证失败，请勿进行下一步转换，并检查您的系统是否符合要求（例如分区数量、分区类型）。

3. **开始转换：**

   - 如果验证成功，输入以下命令并按回车执行，将 `0` 替换为您的系统盘编号：

   mbr2gpt /convert /disk:0 /allowfullOS

   - 工具会执行转换过程。这包括在磁盘上创建必要的 UEFI 启动分区（EFI System Partition, ESP）并更新启动配置数据（BCD）。
   - 转换完成后，您会看到类似 "Conversion completed successfully"（转换成功完成）的提示。

## ⚙️ 步骤三：进入 BIOS 更改启动模式

转换 MBR 到 GPT 只是第一步。**要让系统启动，您必须将 BIOS 启动模式切换到 UEFI。**

1. **重启电脑并进入 BIOS/UEFI 设置。**
   - 重启时，快速按压 BIOS 快捷键（例如 F2、Del）。
2. **更改 Boot Mode（启动模式）：**
   - 导航到 **Boot（启动）** 或 **BIOS Features（BIOS 功能）** 选项卡。
   - 找到 **CSM（Compatibility Support Module，兼容性支持模块）** 或 **Legacy Support（传统支持）** 选项。
   - **禁用 (Disabled) CSM 或 Legacy Support**。
   - 确保 **Boot Mode（启动模式）** 更改为 **UEFI** 或 **Native UEFI**。
3. **启用 Secure Boot：**
   - 导航到 **Security（安全）** 或 **Boot（启动）** 选项卡。
   - 找到 **Secure Boot（安全启动）** 选项，将其设置为 **Enabled（启用）**。
4. **保存并退出。**
   - 保存更改并重启计算机。您的系统应该能够正常启动了。

------

> **⚠️ 注意：** 如果您使用的是 Windows 以外的操作系统或无法进入 Windows，则需要从 Windows PE 或 Windows 安装介质（如 USB 驱动器）启动，然后在恢复环境的命令提示符中使用 MBR2GPT 工具进行转换。