## WIN_PE离线注入驱动

在 PE 环境内使用 DISM 指令将驱动导入系统，通常分为两种常见场景：**场景 A** 是指把驱动注入到**已经安装好系统的离线 Windows 磁盘分区中**；**场景 B** 是指把驱动临时或永久注入到**正在运行的 PE 镜像（boot.wim）中**。

以下是这两种场景的具体操作步骤和指令：

### 1. 将驱动注入到“已安装好系统的目标分区”

如果你在 PE 下刚装完系统（或者现成的系统无法引导、缺少关键驱动），需要把驱动注入到目标系统的盘符中。

1. **查看并确认盘符**

   进入 PE 后，系统的盘符可能和平时不同（例如系统盘可能是 `D:` 或 `C:`）。可以通过 `diskpart` 或直接输入盘符确认系统根目录。假设你的目标系统在 `C:\`，驱动文件放在 `D:\Drivers`。  

2. **执行 DISM 注入指令**

   打开 CMD 命令行，输入以下命令：  

   ```powershell
   dism /Image:C:\ /Add-Driver /Driver:D:\Drivers /Recurse
   ```

   - `/Image:C:\`：指定目标系统的根目录（离线系统路径）。
   - `/Add-Driver`：执行添加驱动操作。
   - `/Driver:D:\Drivers`：指定驱动所在的文件夹（也可以精确指定到某一个 `.inf` 文件）。
   - `/Recurse`：**强烈建议加上**，它会递归遍历该文件夹下所有子目录中的 `.inf` 驱动文件。

3. **（可选）强制注入未签名驱动**

   如果遇到某些老旧或没有数字签名的驱动报错，可以在后面加上 `/ForceUnsigned` 参数：

   ```powershell
   dism /Image:C:\ /Add-Driver /Driver:D:\Drivers /Recurse /ForceUnsigned
   ```

### 2. 将驱动注入到“PE 系统自身（boot.wim）”中

如果你希望当前使用的 PE 在启动时就能识别某些特异的网卡、显卡或 NVMe 存储芯片，需要修改 PE 的镜像文件。

1. **准备工作**

   将 PE 镜像中的 `boot.wim` 提取出来（假设放在 `D:\pe\boot.wim`），并创建一个挂载文件夹 `D:\pe\mount`。

2. **挂载 WIM 镜像**

   ```powershell
   dism /Mount-Wim /WimFile:D:\pe\boot.wim /Index:1 /MountDir:D:\pe\mount
   ```

   *(注：如果是官方标准的 WinPE，通常有两个索引，`Index:1` 是 WinPE 核心，`Index:2` 是 Windows Setup 安装环境，根据需要挂载，一般修改 `Index:1`。)*

3. **注入驱动**  

   ```powershell
   dism /Image:D:\pe\mount /Add-Driver /Driver:D:\Drivers /Recurse
   ```

4. **保存并卸载镜像**

   修改完成后，提交改动并卸载：

   ```powershell
   dism /Unmount-Wim /MountDir:D:\pe\mount /Commit
   ```

   之后可以用修改好的 `boot.wim` 重新打包写入 U 盘启动。

### 3. PE 下临时加载驱动（免注入）

如果你只是在 PE 下临时需要某块网卡或 U 盘驱动能正常工作，**不需要**使用复杂的 DISM 挂载和注入指令，直接用 `drvload` 命令即可实时加载：

```powershell
drvload D:\Drivers\net.inf
```

这条命令会立即把驱动加载到当前运行的 PE 系统内存中，即插即用，非常适合临时救急。