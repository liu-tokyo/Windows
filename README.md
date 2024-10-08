# Windows

## Windows10

[详细](./Windows10/README.md)

## Windows11

[详细](./Windows11/11-WIN11安装.md)

# Windows 常用指令

## 1. 磁盘检查 CHKDSK

### 1.1 chkdsk `/f`和`/r`的区别

> chkdsk命令是Windows系统中用于检查和修复‌磁盘错误的工具。 该命令可以检查文件系统的‌逻辑错误，并尝试修复这些问题。chkdsk命令提供了多个参数，其中‌/f和‌/r是两个常用的参数。‌

- `/f`参数的功能和用途：  
  `/f`参数用于修复磁盘上的错误。当使用/f参数时，chkdsk会尝试修复文件系统中的逻辑错误，包括损坏的文件、目录或文件系统表等。这个参数可以帮助恢复数据的完整性和一致性。‌

- `/r`参数的功能和用途：  
  `/r`参数用于查找坏扇区并恢复可读的信息。当使用`/r`参数时，chkdsk会扫描磁盘上的每个扇区，查找并标记损坏的扇区。如果找到可读的信息，chkdsk会尝试恢复这些信息。这个参数对于修复物理损坏或逻辑错误非常有用。‌

总结来说，`/f`参数主要用于修复文件系统中的逻辑错误，而`/r`参数则用于查找和恢复损坏的扇区。 这两个参数在磁盘维护和故障排除中都非常有用，可以根据具体的需求选择使用。‌

### 1.2 查找坏扇区速度极慢

CHKDSK查找坏扇区速度极慢的原因可能包括磁盘中存在大量需要恢复的数据或大量的坏扇区需要修复，‌这可能导致CHKDSK的工作时间很长，‌甚至可能卡住。‌此外，‌如果磁盘碎片文件过多，‌也可能会导致CHKDSK的检查、‌修复速度变慢，‌甚至卡住。‌这些情况都是正常的，‌尤其是在处理大量数据或修复坏扇区时。‌

为了解决CHKDSK查找坏扇区速度慢的问题，‌可以采取以下措施：‌  
- 确保磁盘中没有正在进行的读写操作，‌因为任何运行中的软件或系统文件损坏都可能影响CHKDSK的运行速度。‌  
- 避免在CHKDSK运行时对磁盘进行其他操作，‌以减少磁盘碎片文件过多的情况，‌从而提高CHKDSK的运行效率。‌  
- 在WinPE环境下进行检测，‌这有助于减少其他软件对CHKDSK运行的影响，‌提高检测的准确性。‌  
- 备份重要数据，‌特别是在检测出坏道后，‌应尽快备份数据，‌以防数据丢失。‌

如果CHKDSK卡住无法继续，‌可以尝试联系磁盘制造商或专业数据恢复服务以获取帮助。‌在处理坏扇区或硬盘问题时，‌务必谨慎操作，‌以避免数据丢失或硬盘进一步损坏。

确保硬盘没几个月进行一次整体的 `chkdsk /B` 检查。否则不仅是影响速度，更可怕的是造成数据损失。  
※ 如果检查速度过慢，最好是再次检查，看看是否变快，否则尽快放弃该磁盘，避免造成不必要的数据损失。  
※ 坏簇检查前后，比较硬盘的空间大小，看看多少 簇 被标记为 坏簇。

## 2. cleanmgr

Windows11开始，不再明显的显示 磁盘清理 功能，不过可以通过输入 `cleanmgr` 指令调出该功能。

## 3. 创建、启动 WinPE

> https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/boot-to-winpe?view=windows-11

## 4. 硬盘驱动器分区

> https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/configure-biosmbr-based-hard-drive-partitions?view=windows-11
