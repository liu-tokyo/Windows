## 如何更新新安装的Windows7

在 Windows 7 上无法进行更新，通常是因为缺少必要的更新组件或者系统缺少一些关键的支持文件。要解决这个问题，可以按照以下步骤进行操作：

**恢复 Windows Update 的 5 个核心补丁**

| 核心补丁  | 简介                          | 备注                                                         |
| --------- | ----------------------------- | ------------------------------------------------------------ |
| KB3020369 | 服务堆栈更新/SSU              | 它是后面所有大补丁的“地基”，不装它，后面的汇总包会提示“此更新不适用于您的计算机”。 |
| KB3125574 | 便利汇总包/Convenience Rollup | 也就是传说中的 **“SP2”**。它包含了从 2011 年到 2016 年的所有补丁，能帮你省掉几百次重启。 |
| KB4490628 | 关键 SSU 更新                 | 引入了对 **SHA-2 签名**的初步支持。没有它，系统无法识别现代安全证书。 |
| KB4474419 | SHA-2 签名支持更新            | **最核心的补丁！** 2019 年后微软所有的更新都只用 SHA-2 签名。不装这个，Windows Update 会报 `80072EFE` 错误。 |
| KB4534310 | 2020 年 1 月最终月度汇总      | 这是微软正式停止免费支持前的最后一个官方汇总包。装完它，你的 Win7 就达到了 2020 年的官方最高水位线。 |



### 1. **安装 Windows 7 更新助手 (Windows Update Agent)**

- Windows 7 的更新机制可能需要使用更新代理，确保系统能够正常与微软服务器进行通信。你可以下载并安装最新的 **Windows Update Agent**，它能够帮助修复与更新相关的问题。
- 下载地址：[Windows Update Agent 7.6.7600.320](https://support.microsoft.com/en-us/help/949104)

安装之后，重新启动计算机，再尝试进行更新。

### 2. **安装 .NET Framework 更新**

- 有些 Windows 更新依赖于 **.NET Framework**。如果你的系统没有安装最新的 .NET Framework 版本，可能会导致更新失败。
- 推荐安装最新版本的 **.NET Framework 4.8**：
  - 下载 [.NET Framework 4.8](https://dotnet.microsoft.com/download/dotnet-framework)
  - 安装后，重启计算机并再次尝试更新。

### 3. **安装 Windows Update 服务依赖组件**

- 确保 Windows 更新服务的必要组件已安装。你可以手动安装以下组件：
  - **Windows Update KB3020369**（该更新帮助 Windows 7 兼容更新）：
    - 下载地址：[KB3020369](https://support.microsoft.com/en-us/kb/3020369)
  - **Windows Update KB3172605**（该更新修复 Windows 更新问题）：
    - 下载地址：[KB3172605](https://support.microsoft.com/en-us/kb/3172605)

### 4. **手动下载和安装更新**

- 如果 Windows 更新中心无法自动获取更新，你可以手动下载并安装更新。
- 访问 [Microsoft Update Catalog](https://www.catalog.update.microsoft.com/Home.aspx)，搜索并下载适合你版本的更新包，然后手动安装。

### 5. **检查并重置 Windows 更新组件**

如果上述方法依然无法解决问题，你可以尝试重置 Windows 更新组件。步骤如下：

1. 打开 **命令提示符**（管理员权限）。

2. 输入以下命令来停止 Windows 更新服务：

   ```
   net stop wuauserv
   ```

3. 删除更新文件夹：

   ```
   del /f /s /q %windir%\SoftwareDistribution\*
   ```

4. 重新启动 Windows 更新服务：

   ```
   net start wuauserv
   ```

5. 然后再次尝试运行 Windows 更新。

### 6. **Windows 7 需要 SP1（Service Pack 1）**

如果你的 Windows 7 版本没有安装 **SP1**，系统可能无法进行正常更新。你可以通过以下链接下载并安装 **SP1**：

- 下载 [Windows 7 SP1](https://www.microsoft.com/en-us/download/details.aspx?id=5842)

安装了上述组件和更新之后，重新启动计算机并尝试更新，通常可以解决问题。如果问题依然存在，可以尝试使用 **Microsoft Fixit** 工具修复 Windows 更新问题。