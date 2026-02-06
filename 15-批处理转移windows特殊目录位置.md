# 批处理转移windows特殊目录位置

在 Windows 11 中，可以通过修改一些系统设置或使用批处理脚本来将下载、文档、图片等特殊文件夹（例如 "Downloads"）转移到另一个磁盘或分区。下面是如何使用批处理脚本来实现这一点的步骤：

### 1. **转移 "下载" 文件夹**

可以将下载文件夹转移到其他磁盘（例如 `D:` 盘）。以下是批处理脚本的示例：

```
@echo off
:: 创建目标文件夹，如果不存在的话
mkdir "D:\Downloads"

:: 将原 "下载" 文件夹的内容转移到新位置
robocopy "%USERPROFILE%\Downloads" "D:\Downloads" /E /MOVE

:: 使用符号链接将原位置指向新位置
mklink /D "%USERPROFILE%\Downloads" "D:\Downloads"

echo Downloads folder has been moved to D:\Downloads
pause
```

### 2. **转移 "文档" 文件夹**

类似地，你可以转移其他特殊文件夹，例如 "Documents"（文档）文件夹：

```
@echo off
:: 创建目标文件夹，如果不存在的话
mkdir "D:\Documents"

:: 将原 "文档" 文件夹的内容转移到新位置
robocopy "%USERPROFILE%\Documents" "D:\Documents" /E /MOVE

:: 使用符号链接将原位置指向新位置
mklink /D "%USERPROFILE%\Documents" "D:\Documents"

echo Documents folder has been moved to D:\Documents
pause
```

### 3. **通用方法（转移多个文件夹）**

如果你想要转移多个文件夹（如 "下载"、"文档"、"图片" 等），可以将它们一起写入批处理脚本中：

```
@echo off
:: 转移下载文件夹
mkdir "D:\Downloads"
robocopy "%USERPROFILE%\Downloads" "D:\Downloads" /E /MOVE
mklink /D "%USERPROFILE%\Downloads" "D:\Downloads"

:: 转移文档文件夹
mkdir "D:\Documents"
robocopy "%USERPROFILE%\Documents" "D:\Documents" /E /MOVE
mklink /D "%USERPROFILE%\Documents" "D:\Documents"

:: 转移图片文件夹
mkdir "D:\Pictures"
robocopy "%USERPROFILE%\Pictures" "D:\Pictures" /E /MOVE
mklink /D "%USERPROFILE%\Pictures" "D:\Pictures"

echo All specified folders have been moved to D:\
pause
```

### 4. **注意事项**

- **符号链接 (`mklink`)**: 使用符号链接可以让系统认为这些文件夹依然位于默认位置，但实际上它们被转移到新位置。
- **`robocopy` 命令**: `robocopy` 是一个强大的命令，用来复制文件及其目录结构，`/E` 参数表示包括子目录，`/MOVE` 参数表示移动文件而不是复制。
- **管理员权限**: 运行这些脚本时，确保你使用管理员权限来执行批处理文件，因为更改系统文件夹位置需要足够的权限。

### 5. **批处理文件的保存**

1. 将这些代码保存为 `.bat` 文件（例如 `move_folders.bat`）。
2. 右键点击批处理文件，选择 **以管理员身份运行**。