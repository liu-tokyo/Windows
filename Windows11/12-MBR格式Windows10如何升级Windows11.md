## MBR 格式 Windows 10 升级 Windows 11 的步骤

将**MBR（主引导记录）格式的Windows 10**升级到Windows 11需要进行几个关键步骤，因为Windows 11的正式升级要求系统盘使用 **GPT（GUID 分区表）分区格式，并且电脑需要启用 UEFI 固件模式和安全启动（Secure Boot）**。

建议使用 Windows 10 内建的 **MBR2GPT** 工具进行转换，它可以**无损数据**地将系统盘从 MBR 转换为 GPT。

### 1. 确认和备份数据

- **检查系统兼容性：** 运行“电脑健康状况检查”应用，确保您的电脑满足 Windows 11 的其他硬件要求（如 TPM 2.0、CPU 等）。
- **备份数据：** 虽然 MBR2GPT 工具通常是无损的，但为了安全起见，强烈建议**完整备份**您的重要数据。
- **检查 Windows 10 版本：** MBR2GPT 工具要求您的 Windows 10 版本为 **1703 或更高版本**。

### 2. 将 MBR 转换为 GPT

使用 Windows 内建的 **MBR2GPT** 工具进行转换，这是最安全且无损数据的官方方法。

1. **进入高级启动：**

   - 打开 **设置** (Settings) $\rightarrow$ **更新和安全** (Update & Security) $\rightarrow$ **恢复** (Recovery)。
   - 在“**高级启动**” (Advanced startup) 部分，点击“**立即重新启动**” (Restart now)。

2. **打开命令提示符：**

   - 电脑重启后，依次选择：**疑难解答** (Troubleshoot) $\rightarrow$ **高级选项** (Advanced options) $\rightarrow$ **命令提示符** (Command Prompt)。

3. **运行 MBR2GPT：**

   - 在命令提示符中，输入以下命令并按 **Enter** 键：

     ```
     mbr2gpt /validate
     ```

     （该命令用于验证磁盘是否可以转换。如果验证通过，请继续下一步。）

   - 接着，输入以下命令并按 **Enter** 键执行转换：

     ```
     mbr2gpt /convert
     ```

     （如果您的系统盘不是 Disk 0，您可能需要使用 `/disk:n` 参数，其中 `n` 是系统盘的编号。您可以使用 `diskpart` $\rightarrow$ `list disk` 来查看磁盘编号。）

4. **转换完成：**

   - 看到“Conversion completed successfully”提示后，输入 `exit` 关闭命令提示符，然后点击“**继续**” (Continue) 或“**关闭电脑**” (Turn off your PC)。

### 3. 更改 BIOS/UEFI 固件设置

在完成磁盘格式转换后，您需要将主板固件的引导模式从传统 BIOS 模式更改为 **UEFI 模式**。

1. **进入 BIOS/UEFI 设置：**
   - 重新启动电脑，并在开机时根据屏幕提示（通常是 **F2**、**F10**、**F12** 或 **Del** 键）进入 **BIOS/UEFI 固件设置**。
2. **启用 UEFI 模式：**
   - 在 BIOS/UEFI 菜单中找到“启动”（Boot）或“启动模式”（Boot Mode）设置。
   - 将启动模式从 **Legacy**（传统）或 **CSM**（兼容性支持模块）更改为 **UEFI**。
3. **启用 TPM 2.0 和安全启动 (Secure Boot)：**
   - 在 BIOS/UEFI 设置中找到相关的安全（Security）或启动（Boot）选项。
   - 确保 **TPM 2.0** 处于**启用**（Enabled）状态。
   - 确保 **安全启动**（Secure Boot）处于**启用**（Enabled）状态。
4. **保存并退出：**
   - 保存您所做的更改并退出 BIOS/UEFI 设置。电脑将重新启动，此时应以 **UEFI 模式** 启动 Windows 10。

### 4. 升级到 Windows 11

在系统盘转换为 GPT、且启用 UEFI、TPM 2.0 和安全启动之后，您就可以通过 Windows Update 升级到 Windows 11 了。

1. **检查更新：**
   - 在 Windows 10 中，打开 **设置** $\rightarrow$ **更新和安全** $\rightarrow$ **Windows Update**。
   - 点击“**检查更新**” (Check for updates)。
2. **下载并安装：**
   - 如果您的电脑现在完全满足 Windows 11 要求，您将看到升级提示。
   - 点击“**下载并安装**” (Download and install) 按钮，按照屏幕指示完成 Windows 11 的升级过程。