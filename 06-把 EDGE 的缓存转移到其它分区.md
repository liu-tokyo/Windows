# 把 EDGE 的缓存转移到其它分区

将 Microsoft Edge（基于 Chromium 内核的新版 Edge）的缓存转移到其他分区（如 D 盘或 E 盘），可以通过以下两种最常见且有效的方法实现：

### 方法一：使用命令行符号链接 (Symbolic Link)

这是最常用、最推荐的方法，因为它能让 Windows 误认为缓存仍然在默认位置，但实际数据却存放在您指定的新分区。

#### 步骤：

1. **彻底关闭 Edge 浏览器。** 确保所有 Edge 进程都在任务管理器中结束。

2. **创建新的缓存目录：** 在您想要存放缓存的分区（例如 D 盘）创建一个新的文件夹。

   - **示例：** 在 D 盘创建名为 `EdgeCache` 的文件夹，即路径为 `D:\EdgeCache`。

3. **定位并删除旧缓存文件夹：**

   - 打开文件资源管理器，导航到 Edge 的默认用户数据目录： `C:\Users\您的用户名\AppData\Local\Microsoft\Edge\User Data\Default\`
   - 在该目录下，找到并**删除**名为 **`Cache`** 的文件夹。
   - *(注意：如果您使用多个 Edge 配置文件，可能需要对每个配置文件（如 Profile 1, Profile 2 等）下的 `Cache` 文件夹重复此操作。)*

4. **创建符号链接：**

   - 以**管理员身份**运行 **“命令提示符” (CMD)** 或 **“Windows PowerShell”**。  
     ※ `Windows PowerShell` 无法正确识别 `mklink` 指令。
   - 输入以下命令并执行，注意将路径替换为您实际的用户名和新的缓存目录：

   DOS

   ```
   mklink /D "C:\Users\您的用户名\AppData\Local\Microsoft\Edge\User Data\Default\Cache" "D:\EdgeCache"
   ```

   - **示例：** `mklink /D "C:\Users\Admin\AppData\Local\Microsoft\Edge\User Data\Default\Cache" "D:\EdgeCache"`

5. **验证：** 如果成功，您将在默认位置看到一个新的 `Cache` 文件夹，但它带有一个**小小的快捷方式箭头图标**。打开 Edge 浏览器，它将开始在您的新分区（D:\EdgeCache）中存储缓存数据。

### 方法二：修改 Edge 快捷方式的启动参数

此方法是通过在启动 Edge 时强制指定缓存目录来实现的。

#### 步骤：

1. **找到 Edge 快捷方式：**

   - 在桌面上找到 Edge 快捷方式，或在开始菜单中右键点击 Edge -> **更多** -> **打开文件位置**。

2. **修改快捷方式属性：**

   - 右键点击 Edge 快捷方式，选择 **“属性”**。
   - 切换到 **“快捷方式”** 标签页。
   - 在 **“目标”** 栏中，原有的内容是类似：`"C:\Program Files\Microsoft\Edge\Application\msedge.exe"`
   - 在现有内容**末尾**添加一个空格，然后加上以下参数，将路径替换为您想要的新缓存目录（例如 `D:\EdgeCache`）：

   ```
   --disk-cache-dir="D:\EdgeCache"
   ```

   - **完整示例：** `"C:\Program Files\Microsoft\Edge\Application\msedge.exe" --disk-cache-dir="D:\EdgeCache"`

3. **应用并重启 Edge。** 之后通过此快捷方式启动的 Edge 就会使用新的缓存目录。

## ⚠️ 注意事项



- **方法一 (符号链接) 优先级更高，推荐使用。** 如果 Edge 不是通过您修改的快捷方式启动（例如点击邮件中的链接或设置了**“启动加速/启动提升”**功能），**方法二可能会失效**，导致缓存仍然创建在 C 盘。
- **启动加速功能：** 如果您使用方法二，请确保在 Edge 的设置中（`edge://settings/system`）**关闭**“启动加速”（Startup Boost）功能，否则 Edge 可能会在后台运行，从而忽略您的启动参数。
- **权限：** 无论是符号链接还是修改注册表（更高级的方法），都需要**管理员权限**才能执行命令或修改系统文件。
- **UserDataDir 参数：** 如果您想移动**所有**用户数据（包括缓存、历史记录、书签等），可以使用 `--user-data-dir="D:\EdgeData"` 参数，但这需要您先将整个 `User Data` 文件夹复制到新位置。一般只移动缓存就足够节省空间。