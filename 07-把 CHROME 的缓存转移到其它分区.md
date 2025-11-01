# 把 CHROME 的缓存转移到其它分区

将 Google Chrome 的缓存转移到其他分区的方法与 Edge 浏览器类似，主要有两种有效的方法：使用**命令行符号链接**或修改**快捷方式启动参数**。

由于 Chrome（和 Edge 一样）没有内置的缓存路径修改选项，这些方法都是通过利用 Windows 的系统功能或启动机制来实现的。

### 方法一：使用命令行符号链接 (Symbolic Link) - **推荐**

符号链接可以让操作系统认为缓存文件夹仍在 C 盘的默认位置，但实际的读写操作却指向您指定的新分区。

#### 步骤：

1. **彻底关闭 Chrome 浏览器。** 务必确保所有 Chrome 窗口和后台进程都已关闭。

2. **创建新的缓存目录：** 在您想要存放缓存的分区（例如 D 盘）创建一个新的文件夹。

   - **示例：** 在 D 盘创建名为 `ChromeCache` 的文件夹，即路径为 `D:\ChromeCache`。

3. **定位并删除旧缓存文件夹：**

   - Chrome 的默认缓存路径通常在： `C:\Users\%username%\AppData\Local\Google\Chrome\User Data\Default\`
   - 找到该目录下的 **`Cache`** 文件夹，并将其**删除**（如果无法删除，说明 Chrome 没有完全关闭，请检查任务管理器）。
   - *(如果您使用多个 Chrome 配置文件，您可能需要对每个配置文件（如 Profile 1, Profile 2 等）下的 `Cache` 文件夹重复此操作。)*

4. **创建符号链接：**

   - 以**管理员身份**运行 **“命令提示符” (CMD)** 或 **“Windows PowerShell”**。
   - 输入以下命令并执行，注意将路径替换为您实际的用户名和新的缓存目录：

   DOS

   ```
   mklink /D "C:\Users\%username%\AppData\Local\Google\Chrome\User Data\Default\Cache" "D:\ChromeCache"
   ```

   - **示例：** `mklink /D "C:\Users\Admin\AppData\Local\Google\Chrome\User Data\Default\Cache" "D:\ChromeCache"`

5. **验证：** 命令执行成功后，您将在 C 盘的原路径看到一个新的 `Cache` 文件夹图标，上面带有一个**快捷方式的箭头**。重新打开 Chrome，缓存文件就会自动写入到您的新分区（D:\ChromeCache）。

### 方法二：修改 Chrome 快捷方式的启动参数

此方法是通过在 Chrome 启动时，利用命令行参数 `--disk-cache-dir` 强制指定缓存路径。

#### 步骤：

1. **找到 Chrome 快捷方式：**

   - 右键点击您用于启动 Chrome 的快捷方式（例如桌面上的图标或开始菜单中的图标），选择 **“属性”**。

2. **修改快捷方式属性：**

   - 切换到 **“快捷方式”** 标签页。
   - 在 **“目标”** 栏中，原有的内容是类似：`"C:\Program Files\Google\Chrome\Application\chrome.exe"`
   - 在现有内容**末尾**添加一个空格，然后加上以下参数，将路径替换为您想要的新缓存目录（例如 `D:\ChromeCache`）：

   ```
   --disk-cache-dir="D:\ChromeCache"
   ```

   - **完整示例：** `"C:\Program Files\Google\Chrome\Application\chrome.exe" --disk-cache-dir="D:\ChromeCache"`

3. **应用并重启 Chrome。** 之后只有通过**此快捷方式**启动的 Chrome 才会使用新的缓存目录。

## ⚠️ 注意事项

- **符号链接更可靠：** 如果您希望无论通过何种方式（如点击邮件链接、其他程序调用）启动 Chrome 都能使用新的缓存路径，**强烈建议使用方法一**（符号链接）。方法二仅对使用该特定快捷方式的启动有效。
- **权限：** `mklink` 命令需要在**管理员身份**的命令提示符或 PowerShell 中运行。
- **User Data 目录：** 如果您想移动**所有**用户数据（包括缓存、历史记录、书签等，以彻底释放 C 盘空间），您可以将参数更改为 `--user-data-dir="D:\ChromeData"` 或对整个 `User Data` 文件夹使用 `mklink` 命令进行链接。