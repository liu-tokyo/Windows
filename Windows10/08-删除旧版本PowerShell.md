# 删除旧版本PowerShell

要使用 **Winget** 删除特定版本（例如 **7.4.12.0**）的应用程序，您需要知道该应用程序的 **名称 (Name)** 或 **ID**，并配合使用 **`--version`** 参数。

以下是操作步骤：



## 步骤 1: 查找应用程序 ID 或名称



首先，您需要确定要删除的应用程序在 Winget 中的 **ID** 或 **名称**。由于您使用的是版本号 **7.4.12.0**，这很可能是 **PowerShell** 的旧版本。

1. 在 PowerShell 或命令提示符中运行以下命令来列出所有已安装的包：

   PowerShell

   ```
   winget list
   ```

2. 在输出列表中找到您的目标应用程序，并记下它的 **名称** 或 **ID**。

   - 对于 **PowerShell**，常见的 ID 是 `Microsoft.PowerShell` 或类似的名字。
   - 确认列表里有多个版本的 PowerShell，并且其中包含您想删除的 **7.4.12.0** 版本。



## 步骤 2: 执行带版本号的卸载命令



使用 `winget uninstall` 命令，并通过 `--version` 参数指定要删除的确切版本号。

- 假设您要删除的应用程序的 ID 是 **`Microsoft.PowerShell`**，请运行以下命令：

  `PowerShell` 输入如下指令：

    ```powershell
    winget uninstall --id Microsoft.PowerShell --version 7.4.12.0
    ```

  ※ 多个版本存在的情况下，该指令无法找到正确的项目。

- 如果使用 **名称** 更方便，您也可以尝试：

  `PowerShell` 输入如下指令：

  ```powershell
  winget uninstall --name "PowerShell" --version 7.4.12.0
  ```

  (注意：如果名称包含空格，请使用引号。如果 Winget 提示不明确，请改用 **ID**。)



## ⚠️ 注意事项



- **管理员权限：** 某些应用程序可能需要您以**管理员身份**运行 PowerShell 或命令提示符才能成功卸载。
- **ID 优先：** 强烈建议使用 **`--id`** 参数和应用程序的 **确切 ID**，因为名称有时可能不唯一或不准确。
- **不保证多版本支持：** 尽管 Winget 支持指定版本卸载，但某些应用程序可能并不支持同一程序的多个版本并存（Side-by-Side），或者其卸载程序可能无法正确处理版本选择。如果遇到问题，您可能需要通过 **Windows 的“应用和功能”** 设置手动卸载旧版本。
- **版本号格式：** 确保您使用的版本号 **`7.4.12.0`** 与 `winget list` 输出中显示的版本号**完全一致**。