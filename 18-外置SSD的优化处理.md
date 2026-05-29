# 如何针对 USB 连接的 SSD 进行 Trim

在 Windows 11 中，针对通过 USB 外接的固态硬盘（SSD）进行 **Trim（修剪，用于保持 SSD 长期读写不掉速）**，其核心逻辑和内置的 SATA 或 NVMe 固态硬盘是一样的，都是通过系统自带的“驱动器优化”工具来实现。

但是，**USB 外接 SSD 能否成功 Trim，取决于一个致命的前置硬件条件**。请按照以下步骤进行操作和排查：

## 🛠️ 第一步：使用系统自带工具执行 Trim

Windows 11 的系统底层已经把 Trim 整合进了“碎片整理和优化驱动器”工具中。对 SSD 而言，执行这个操作就是发送 Trim 指令。

1. 点击任务栏的**开始菜单/搜索框**，输入 **“优化”** 或 **“Defrag”**，然后点击打开 **“碎片整理和优化驱动器” (Defragment and Optimize Drives)**。
2. 在弹出的窗口中，你会看到电脑上连接的所有驱动器列表。
3. 找到你通过 USB 连接的那个外接 SSD。
   - **关键检查：** 观察它的 **“媒体类型” (Media type)** 一栏。如果显示为 **“固态驱动器” (Solid state drive)**，说明系统已正确识别。
4. 鼠标点击选中该外接 SSD，然后点击下方的 **“优化” (Optimize)** 按钮。
5. 此时“当前状态”会闪过“正在运行...”，随后显示“已完成”，这就代表 Windows 11 已经成功向这块 USB SSD 发送了 Trim 指令。

## 🚨 第二步：硬核排查 —— 为什么我的 USB SSD 无法优化或显示为“未知”？

很多时候，你会发现外接 SSD 在这个工具里显示为 **“硬磁盘驱动器” (Hard disk drive)**、**“未知”**，或者 **“优化”按钮是灰色的无法点击**。

这是因为：**传统的 USB 协议是不支持传输 Trim 指令的。** 要让 Trim 穿透 USB 接口到达里面的 SSD 颗粒，你的外接设备必须完美满足以下两点硬件要求：

### 1. 检查你的外接硬盘盒/移动硬盘是否支持 UASP 协议

- **UASP (USB Attached SCSI Protocol)** 是一种高效的传输协议，只有支持 UASP 的硬盘盒，才能把 Windows 的 Trim 指令（SCSI UNMAP 命令）翻译并传递给里面的 SSD。
- **排查方法：** 右键点击开始按钮 -> 选择 **“设备管理器”** -> 展开 **“存储控制器”**。 如果连上外接硬盘后，列表里出现 **“USB Attached SCSI (UAS) 质存设备”**（英文通常带有 `UAS Bus Driver`），说明硬件支持；如果显示的仅仅是老旧的 `USB Mass Storage`，说明这个硬盘盒/线材不支持 Trim，无解，必须更换支持 UASP 的硬盘盒。

### 2. 用命令检测系统底层是否对该 USB 盘开启了 Trim

即使硬件支持，Windows 有时也会保守地对 USB 设备禁用 Trim。我们可以用 PowerShell 强制检测：

1. 右键点击开始按钮，选择 **“终端（管理员）”** 或 **“PowerShell（管理员）”**。

2. 输入以下命令并回车，检查全局 Trim 是否开启：

   PowerShell

   ```
   fsutil behavior query DisableDeleteNotify
   ```

   - 如果显示 `DisableDeleteNotify = 0`，代表 **Trim 已启用**（这是正常的）。
   - 如果显示 `1`，请输入 `fsutil behavior set DisableDeleteNotify 0` 来强制开启它。

3. **终极精准检测：** 想知道当前这块 USB 盘到底有没有成功 Trim，运行这行命令：

   PowerShell

   ```
   Optimize-Volume -DriveLetter X -Analyze -Verbose
   ```

   *(请把命令中的 **`X`** 换成你 USB 固态硬盘在系统里的实际盘符，比如 `E` 或 `F`)*

   - 观察返回的日志，如果提示中包含 `Trim` 相关字样且成功通过，说明这块盘已经彻底打通了 Trim 通道。

## 💡 给硬件爱好者的折腾小贴士

如果你手里有大量的闲置 M.2 NVMe 固态硬盘（比如从升级换下的老机器上拆下来的），想做成万能高速移动硬盘：

- 在购买外接硬盘盒时，**务必认准主控芯片**。目前市面上对 Windows 11 兼容性最好、完美稳定支持 UASP 和 Trim 的主流主控是 **RTL9210B**（Realtek 瑞昱）或 **ASM2464**（祥硕 USB4/雷电）。
- 尽量将硬盘盒插在电脑的原生 USB 3.2 Gen2（通常是红色、蓝色接口或 Type-C 接口）上，直接连接主板总线，Trim 指令的握手成功率会显著高于经过各种第三方 USB Hub（集线器）转接。