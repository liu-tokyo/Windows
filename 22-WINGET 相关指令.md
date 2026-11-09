# WINGET 相关指令

## 禁止指定软件更新

在 Windows Package Manager (`winget`) 中，可以通过 **`pin`（固定）命令** 来禁止或阻止指定软件的自动升级与更新。

### 最常用的命令：完全禁止升级

打开 **PowerShell** 或 **命令提示符（cmd）**，运行以下命令：

```PowerShell
winget pin add --id <软件ID>
```

**示例：禁止 Google Chrome 更新**

```PowerShell
winget pin add --id Google.Chrome
```

运行后，即便执行 `winget upgrade --all`，该软件也会被自动跳过。

### 操作步骤与常用功能

#### 1. 查看软件的精确 ID

如果不确定软件的具体 ID，可以先搜索软件名称：

```PowerShell
winget search <软件名称>
```

#### 2. 限制更新到特定版本（版本锁）

如果只想停留在当前版本，但未来允许在特定的主版本范围内变动，可以指定版本：

```PowerShell
winget pin add --id Google.Chrome --version 120.0.*
```

#### 3. 查看所有已禁用的软件清单

查看当前系统中所有被锁定/禁止更新的软件：

```PowerShell
winget pin list
```

#### 4. 解除禁止（恢复允许更新）

如果以后想要恢复该软件的更新，执行删除 Pin 即可：

```PowerShell
winget pin remove --id <软件ID>
```

*验证方法：运行 `winget pin list` 确认该软件已从锁定列表中移除；或运行 `winget upgrade` 查看该软件已恢复在可升级列表中。*