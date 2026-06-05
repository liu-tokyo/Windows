## 使用 System 权限启动 PowerShell

在 Windows 中，`System`（即 `NT AUTHORITY\SYSTEM` 或 `本地系统`）是比管理员（Administrator）更高、最核心的系统权限。由于它是操作系统的核心账户，Windows 默认不提供直接“右键以 System 身份运行” underground 选项。

要以 System 权限启动 PowerShell，最经典、最稳定且安全的方法是使用微软官方 Sysinternals 工具包中的 **PsExec**，或者使用 PowerShell 自身的计划任务高级技巧。

以下是具体的实现方法：

### 方法一：使用微软官方 PsExec 工具（最推荐、最简单）

这是技术人员最常用的标准方法，无需修改注册表或使用第三方未知脚本。

1. **下载 PsExec：** 前往微软官网下载 [Sysinternals Suite 中的 PsExec](https://learn.microsoft.com/zh-cn/sysinternals/downloads/psexec)（或者直接下载 PsTools 压缩包）。

2. **解压工具：** 将下载好的 `PSTools.zip` 解压，比如解压到 `D:\Tools\` 目录中。

3. **以管理员身份打开普通的 PowerShell：** 按下 `Win + X`，选择 **“终端(管理员)”** 或 **“PowerShell (管理员)”**。

4. **运行 PsExec 命令：** 在打开的 PowerShell 中，切换到解压目录并执行以下命令（以 64 位系统为例）：

   ```PowerShell
   cd D:\Tools\
   .\psexec64.exe -i -s powershell.exe
   ```

   > **参数解析：**
   >
   > - `-i`：交互式模式（Interactive），让运行的程序显示图形界面。
   > - `-s`：以 System 权限（SYSTEM account）运行程序。

5. **验证权限：** 此时会弹出一个全新的 PowerShell 窗口。在新窗口中输入以下命令：

   ```PowerShell
   whoami
   ```

   如果输出为 `nt authority\system`，说明你已成功获取最高系统权限。

### 方法二：利用“计划任务”原生启动（免下载工具）

如果你处于无法连接外网、或者不能下载第三方工具的环境中，可以利用 Windows 自带的计划任务功能，因为计划任务允许以 `SYSTEM` 身份直接触发进程。

1. **以管理员身份打开 PowerShell**。

2. **直接运行以下复合命令：** 复制并执行以下代码，它会创建一个瞬间执行的系统任务，拉起一个 System 权限的 PowerShell，并在完成后自动删除该任务：

   ```PowerShell
   $Action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoExit -Command `"WindowTitle 'SYSTEM PowerShell' `""
   $Principal = New-ScheduledTaskPrincipal -UserId "NT AUTHORITY\SYSTEM" -LogonType ServiceAccount -RunLevel Highest
   $Task = New-ScheduledTask -Action $Action -Principal $Principal
   Register-ScheduledTask -TaskName "RunAsSystem" -InputObject $Task -Force
   Start-ScheduledTask -TaskName "RunAsSystem"
   Start-Sleep -Seconds 2
   Unregister-ScheduledTask -TaskName "RunAsSystem" -Confirm:$false
   ```

3. 随后桌面上会直接弹出一个标有 `SYSTEM PowerShell` 的新窗口。输入 `whoami` 即可验证。

### 方法三：使用高级第三方开源模块（如 Open-ProcessAsSystem）

如果你经常需要进行此类调试，可以通过 PowerShell 官方画廊（Gallery）安装社区维护的开源特权提升脚本。

1. 以管理员身份打开 PowerShell，运行以下命令安装模块：

   ```PowerShell
   Install-Module -Name ProcessAsUser -Force
   ```

2. 安装完成后，可以直接通过一行命令呼出 System 权限窗口：

   ```PowerShell
   Invoke-ProcessAsSystem powershell.exe
   ```

### ⚠️ 安全提示

- **不要保持长期运行：** `SYSTEM` 权限拥有修改核心系统文件（如 SAM 数据库、硬件驱动、受保护的注册表项）的绝对权力。一旦遭遇恶意脚本或误操作，可能会直接导致系统蓝屏或崩溃。
- **用完即关：** 完成特定的调试（例如修改某些连管理员都无法访问的注册表键值、测试底层服务行为）后，请立刻关闭该 PowerShell 窗口，切勿将其留在后台。