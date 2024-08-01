# Windows

## Windows10

[详细](./Windows10/README.md)

## Windows11

[详细](./Windows11/11-WIN11安装.md)

# Windows 常用指令

## 1. chkdsk `/f`和`/r`的区别

> chkdsk命令是Windows系统中用于检查和修复‌磁盘错误的工具。 该命令可以检查文件系统的‌逻辑错误，并尝试修复这些问题。chkdsk命令提供了多个参数，其中‌/f和‌/r是两个常用的参数。‌

- `/f`参数的功能和用途：  
  `/f`参数用于修复磁盘上的错误。当使用/f参数时，chkdsk会尝试修复文件系统中的逻辑错误，包括损坏的文件、目录或文件系统表等。这个参数可以帮助恢复数据的完整性和一致性。‌

- `/r`参数的功能和用途：  
  `/r`参数用于查找坏扇区并恢复可读的信息。当使用`/r`参数时，chkdsk会扫描磁盘上的每个扇区，查找并标记损坏的扇区。如果找到可读的信息，chkdsk会尝试恢复这些信息。这个参数对于修复物理损坏或逻辑错误非常有用。‌

总结来说，`/f`参数主要用于修复文件系统中的逻辑错误，而`/r`参数则用于查找和恢复损坏的扇区。 这两个参数在磁盘维护和故障排除中都非常有用，可以根据具体的需求选择使用。‌

## 2. cleanmgr

Windows11开始，不再明显的显示 磁盘清理 功能，不过可以通过输入 `cleanmgr` 指令调出该功能。

## 3. 创建、启动 WinPE

> https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/boot-to-winpe?view=windows-11

## 4. 硬盘驱动器分区

> https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/configure-biosmbr-based-hard-drive-partitions?view=windows-11
