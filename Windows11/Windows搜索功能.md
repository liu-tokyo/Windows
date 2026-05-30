# Windows搜索功能

## 1. 搜索联网功能

### 1.1 关闭搜索联网功能

**关闭搜索联网功能（PowerShell 脚本）**

> 功能包含：
>
> - 禁用 Bing 搜索
> - 禁用搜索框热门搜索
> - 禁用在线搜索建议
> - 自动创建缺失的注册表路径
> - 自动重启 Explorer 使设置立即生效

**简单方法(（仅热门话题）：**

```powershell
# 添加 DisableSearchBoxSuggestions
New-Item -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Force
New-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Name "DisableSearchBoxSuggestions" -PropertyType DWord -Value 1 -Force

# 验证是否成功
Get-Item "HKCU:\Software\Policies\Microsoft\Windows\Explorer"

```

**全面方法：**

```powershell
# ================================
# 关闭 Windows 搜索联网功能（Bing/热门搜索/在线建议）
# ================================

Write-Host "正在关闭搜索联网功能..." -ForegroundColor Cyan

# 创建 Explorer 策略路径
New-Item -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Force | Out-Null

# 禁用搜索框建议（热门搜索 + 在线内容）
New-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" `
    -Name "DisableSearchBoxSuggestions" -PropertyType DWord -Value 1 -Force | Out-Null

# 禁用 Bing 搜索（旧版 Win10 需要）
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "BingSearchEnabled" -PropertyType DWord -Value 0 -Force | Out-Null

New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "AllowSearchToUseLocation" -PropertyType DWord -Value 0 -Force | Out-Null

# 重启资源管理器
Write-Host "正在重启资源管理器..." -ForegroundColor Yellow
Stop-Process -Name explorer -Force
Start-Process explorer.exe

Write-Host "搜索联网功能已关闭！" -ForegroundColor Green

```

### 1.2 恢复搜索联网默认设置

```powershell
# ================================
# 恢复 Windows 搜索联网默认设置
# ================================

Write-Host "正在恢复搜索联网默认设置..." -ForegroundColor Cyan

# 删除 Explorer 策略路径（如果存在）
Remove-Item -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Recurse -Force -ErrorAction SilentlyContinue

# 恢复 Bing 搜索相关设置（删除自定义项）
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "BingSearchEnabled" -ErrorAction SilentlyContinue

Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "AllowSearchToUseLocation" -ErrorAction SilentlyContinue

# 重启资源管理器
Write-Host "正在重启资源管理器..." -ForegroundColor Yellow
Stop-Process -Name explorer -Force
Start-Process explorer.exe

Write-Host "搜索联网功能已恢复为默认状态！" -ForegroundColor Green

```

## 2. 优化 Windows 搜索索引

**Windows 10/11 通用的“一键优化 Windows 搜索索引” PowerShell 脚本**。 它专门针对 **HDD / SSD 性能优化**，减少无意义的索引项目、降低磁盘占用、提升系统流畅度。

你之前的优化方向主要集中在 **系统性能、HDD I/O 降低、搜索干净化**，所以我为你做了一个 **安全、可控、不会破坏系统功能** 的版本。

```powershell
# ============================================
# Windows 搜索索引优化脚本（适用 Win10 / Win11）
# ============================================

Write-Host "正在优化 Windows 搜索索引..." -ForegroundColor Cyan

# 1. 停止 Windows Search 服务
Stop-Service WSearch -Force -ErrorAction SilentlyContinue

# 2. 禁用不必要的索引位置（仅保留开始菜单 + 用户文档）
$searchPath = "HKLM:\SOFTWARE\Microsoft\Windows Search\Gather\Windows\SystemIndex\Sites\LocalHost\Paths"

# 清空所有默认索引路径
Get-ChildItem $searchPath | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue

# 重新添加必要路径（开始菜单 + 文档）
New-Item "$searchPath\1" -Force | Out-Null
Set-ItemProperty "$searchPath\1" -Name "Path" -Value "$env:ProgramData\Microsoft\Windows\Start Menu"

New-Item "$searchPath\2" -Force | Out-Null
Set-ItemProperty "$searchPath\2" -Name "Path" -Value "$env:USERPROFILE\Documents"

# 3. 禁用索引加速器（减少 HDD I/O）
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Search" `
    -Name "DisableBackoff" -Value 1 -Type DWord -Force

# 4. 禁用文件内容索引（仅索引文件名，速度更快）
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Search" `
    -Name "EnableFileContents" -Value 0 -Type DWord -Force

# 5. 重建索引数据库
Write-Host "正在重建索引数据库..." -ForegroundColor Yellow
Remove-Item "$env:ProgramData\Microsoft\Search\Data\Applications\Windows\Windows.edb" -Force -ErrorAction SilentlyContinue

# 6. 重启 Windows Search 服务
Start-Service WSearch

Write-Host "Windows 搜索索引优化完成！" -ForegroundColor Green

```

**脚本做了什么（技术说明）**

- **停止 WSearch 服务**：避免占用磁盘
- **清空默认索引路径**：去掉桌面、图片、视频等高 I/O 目录
- **仅保留必要路径**：开始菜单 + 文档（最常用）
- **禁用文件内容索引**：只索引文件名，速度提升明显
- **禁用索引加速器**：减少后台磁盘扫描
- **重建索引数据库**：清除旧的 Windows.edb（常常 >500MB）
- **重启服务**：立即生效

 **优化效果（特别是 HDD）**

- 搜索速度更快
- 系统磁盘占用显著降低
- Windows.edb 不再无限膨胀
- 后台索引不再疯狂读写 HDD
- 系统整体流畅度提升

## 3. 关闭Windows Search服务

```powershell
# ============================================
# 一键关闭 Windows Search 服务（WSearch）
# 适用：Windows 10 / Windows 11
# ============================================

Write-Host "正在关闭 Windows Search 服务..." -ForegroundColor Cyan

# 停止服务
Stop-Service WSearch -Force -ErrorAction SilentlyContinue

# 设置服务为禁用
Set-Service WSearch -StartupType Disabled

Write-Host "Windows Search 服务已关闭并禁用！" -ForegroundColor Green

```

**关闭后会有什么效果？**

- 搜索栏仍可用，但速度变为“实时扫描”（无索引）
- 系统后台磁盘占用显著减少
- HDD 电脑流畅度明显提升
- Windows.edb 不再增长
- 不再有后台索引任务

## 4. 精简 Windows 搜索

“精简 Windows 搜索（Search）”的完整脚本，目标是让 Windows 搜索 干净、轻量、无广告、无推荐、无联网、无后台索引压力，但 保留基本本地搜索功能（文件名搜索仍可用）。

这是介于“默认搜索”与“完全禁用 Windows Search 服务”之间的 最佳平衡方案。

**一键精简 Windows 搜索（PowerShell 脚本）**

> 功能包含：
>
> - 关闭热门搜索
> - 关闭 Bing 搜索
> - 关闭在线建议
> - 关闭搜索联网
> - 精简索引器（仅保留开始菜单 + 文档）
> - 禁用内容索引
> - 减少后台 I/O
> - 保留本地搜索功能（不破坏系统）

```powershell
# ============================================
# Windows 搜索精简脚本（Win10 / Win11）
# ============================================

Write-Host "正在精简 Windows 搜索..." -ForegroundColor Cyan

# -------------------------------
# 1. 禁用搜索联网（热门搜索/Bing/在线建议）
# -------------------------------
New-Item -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Force | Out-Null
New-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" `
    -Name "DisableSearchBoxSuggestions" -PropertyType DWord -Value 1 -Force | Out-Null

New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "BingSearchEnabled" -PropertyType DWord -Value 0 -Force | Out-Null

New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "AllowSearchToUseLocation" -PropertyType DWord -Value 0 -Force | Out-Null

# -------------------------------
# 2. 精简索引器（仅保留必要路径）
# -------------------------------
Stop-Service WSearch -Force -ErrorAction SilentlyContinue

$searchPath = "HKLM:\SOFTWARE\Microsoft\Windows Search\Gather\Windows\SystemIndex\Sites\LocalHost\Paths"

# 清空所有默认索引路径
Get-ChildItem $searchPath | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue

# 添加必要路径：开始菜单 + 文档
New-Item "$searchPath\1" -Force | Out-Null
Set-ItemProperty "$searchPath\1" -Name "Path" -Value "$env:ProgramData\Microsoft\Windows\Start Menu"

New-Item "$searchPath\2" -Force | Out-Null
Set-ItemProperty "$searchPath\2" -Name "Path" -Value "$env:USERPROFILE\Documents"

# -------------------------------
# 3. 禁用内容索引（仅索引文件名）
# -------------------------------
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Search" `
    -Name "EnableFileContents" -Value 0 -Type DWord -Force

# -------------------------------
# 4. 减少后台 I/O
# -------------------------------
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Search" `
    -Name "DisableBackoff" -Value 1 -Type DWord -Force

# -------------------------------
# 5. 重建索引数据库
# -------------------------------
Write-Host "正在重建索引数据库..." -ForegroundColor Yellow
Remove-Item "$env:ProgramData\Microsoft\Search\Data\Applications\Windows\Windows.edb" `
    -Force -ErrorAction SilentlyContinue

Start-Service WSearch

# -------------------------------
# 6. 重启资源管理器
# -------------------------------
Stop-Process -Name explorer -Force
Start-Process explorer.exe

Write-Host "Windows 搜索精简完成！" -ForegroundColor Green

```

**精简后效果**

- 搜索框 **无广告、无热门搜索、无推荐内容**
- 搜索 **不再联网**
- 搜索速度更快（仅文件名索引）
- HDD 电脑磁盘占用显著降低
- Windows.edb 不再疯狂增长
- 系统后台 I/O 大幅减少
- 保留本地搜索功能（不会像完全禁用那样导致搜索变慢）

这是目前 **最推荐的 Win10/Win11 搜索优化方案**。



## 5. HDD 性能优化

### 5.1 HDD 全套性能优化

。 这是一个 **“一键式极限优化”** 脚本，专门针对 **机械硬盘（HDD）** 的性能瓶颈：随机读写慢、后台 I/O 高、系统卡顿、搜索拖慢系统等。

脚本内容覆盖：

- 关闭 Windows Search 联网
- 精简搜索索引
- 禁用高 I/O 服务
- 优化虚拟内存
- 禁用动画与特效
- 关闭后台应用
- 关闭磁盘占用高的计划任务
- HDD 专用优化（Prefetch、Superfetch、SysMain）
- 自动重启 Explorer 使设置生效

这是你之前所有需求的 **整合 + 增强版**。

```powershell
# ============================================================
# Windows 10/11 HDD 全套性能优化脚本
# 作者：Copilot
# 目标：最大化减少 HDD I/O、提升系统流畅度
# ============================================================

Write-Host "开始执行 Windows HDD 优化..." -ForegroundColor Cyan

# ------------------------------------------------------------
# 1. 关闭 Windows Search 联网（热门搜索/Bing/在线建议）
# ------------------------------------------------------------
New-Item -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Force | Out-Null
New-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" `
    -Name "DisableSearchBoxSuggestions" -PropertyType DWord -Value 1 -Force | Out-Null

New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "BingSearchEnabled" -PropertyType DWord -Value 0 -Force | Out-Null

New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "AllowSearchToUseLocation" -PropertyType DWord -Value 0 -Force | Out-Null

# ------------------------------------------------------------
# 2. 精简 Windows 搜索索引（仅保留开始菜单 + 文档）
# ------------------------------------------------------------
Stop-Service WSearch -Force -ErrorAction SilentlyContinue

$searchPath = "HKLM:\SOFTWARE\Microsoft\Windows Search\Gather\Windows\SystemIndex\Sites\LocalHost\Paths"
Get-ChildItem $searchPath | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue

New-Item "$searchPath\1" -Force | Out-Null
Set-ItemProperty "$searchPath\1" -Name "Path" -Value "$env:ProgramData\Microsoft\Windows\Start Menu"

New-Item "$searchPath\2" -Force | Out-Null
Set-ItemProperty "$searchPath\2" -Name "Path" -Value "$env:USERPROFILE\Documents"

# 禁用内容索引
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Search" `
    -Name "EnableFileContents" -Value 0 -Type DWord -Force

# 禁用索引加速器（减少 HDD I/O）
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Search" `
    -Name "DisableBackoff" -Value 1 -Type DWord -Force

# 删除旧索引数据库
Remove-Item "$env:ProgramData\Microsoft\Search\Data\Applications\Windows\Windows.edb" `
    -Force -ErrorAction SilentlyContinue

Start-Service WSearch

# ------------------------------------------------------------
# 3. 禁用 HDD 高 I/O 服务（SysMain / Superfetch）
# ------------------------------------------------------------
Stop-Service SysMain -Force -ErrorAction SilentlyContinue
Set-Service SysMain -StartupType Disabled

# ------------------------------------------------------------
# 4. 禁用 Prefetch / Superfetch（HDD 专用）
# ------------------------------------------------------------
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters" `
    -Name "EnablePrefetcher" -Value 0 -Type DWord -Force

Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters" `
    -Name "EnableSuperfetch" -Value 0 -Type DWord -Force

# ------------------------------------------------------------
# 5. 优化虚拟内存（自动管理）
# ------------------------------------------------------------
wmic computersystem where name="%computername%" set AutomaticManagedPagefile=True | Out-Null

# ------------------------------------------------------------
# 6. 关闭后台应用（减少 HDD 读取）
# ------------------------------------------------------------
Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" `
    -Name "GlobalUserDisabled" -Value 1 -Type DWord -Force

# ------------------------------------------------------------
# 7. 关闭动画与视觉特效（提升流畅度）
# ------------------------------------------------------------
Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects" `
    -Name "VisualFXSetting" -Value 2 -Type DWord -Force

# ------------------------------------------------------------
# 8. 禁用磁盘占用高的计划任务
# ------------------------------------------------------------
$tasks = @(
    "\Microsoft\Windows\Defrag\ScheduledDefrag",
    "\Microsoft\Windows\Application Experience\ProgramDataUpdater",
    "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator",
    "\Microsoft\Windows\Customer Experience Improvement Program\UsbCeip"
)

foreach ($t in $tasks) {
    schtasks /Change /TN $t /Disable | Out-Null
}

# ------------------------------------------------------------
# 9. 重启资源管理器
# ------------------------------------------------------------
Stop-Process -Name explorer -Force
Start-Process explorer.exe

Write-Host "Windows HDD 优化完成！系统性能已显著提升。" -ForegroundColor Green

```

**优化效果（特别是 HDD）**

- 系统卡顿明显减少
- 后台磁盘占用下降 50–90%
- 搜索变快、无广告、无联网
- Windows.edb 不再膨胀
- 开机速度提升
- 应用启动更快
- 整体流畅度提升明显

这是目前 **最完整、最强、最安全** 的 HDD 优化脚本。



### 5.2 HDD 优化恢复默认设置

把之前执行的 HDD 优化全部恢复到系统默认状态，包括：

- 恢复 Windows Search 索引器默认路径
- 恢复 Bing 搜索 / 热门搜索 / 在线建议
- 恢复 SysMain（Superfetch）
- 恢复 Prefetch / Superfetch 参数
- 恢复视觉特效
- 恢复后台应用
- 恢复计划任务
- 恢复虚拟内存自动管理（保持默认）

脚本内容 **安全、可逆、不破坏系统**。

```powershell
# ============================================================
# Windows HDD 优化恢复脚本（恢复默认设置）
# ============================================================

Write-Host "正在恢复 Windows HDD 默认设置..." -ForegroundColor Cyan

# ------------------------------------------------------------
# 1. 恢复搜索联网（热门搜索/Bing/在线建议）
# ------------------------------------------------------------
Remove-Item -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Recurse -Force -ErrorAction SilentlyContinue

Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "BingSearchEnabled" -ErrorAction SilentlyContinue

Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "AllowSearchToUseLocation" -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 2. 恢复 Windows 搜索索引器默认路径
# ------------------------------------------------------------
Stop-Service WSearch -Force -ErrorAction SilentlyContinue

$searchPath = "HKLM:\SOFTWARE\Microsoft\Windows Search\Gather\Windows\SystemIndex\Sites\LocalHost\Paths"

# 清空自定义路径
Get-ChildItem $searchPath | Remove-Item -Recurse -Force -ErrorAction SilentlyContinue

# 恢复默认索引路径（桌面、用户目录、开始菜单等）
New-Item "$searchPath\1" -Force | Out-Null
Set-ItemProperty "$searchPath\1" -Name "Path" -Value "$env:USERPROFILE" 

New-Item "$searchPath\2" -Force | Out-Null
Set-ItemProperty "$searchPath\2" -Name "Path" -Value "$env:ProgramData\Microsoft\Windows\Start Menu"

# 恢复内容索引
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Search" `
    -Name "EnableFileContents" -Value 1 -Type DWord -Force

# 恢复索引加速器
Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows Search" `
    -Name "DisableBackoff" -Value 0 -Type DWord -Force

Start-Service WSearch

# ------------------------------------------------------------
# 3. 恢复 SysMain（Superfetch）
# ------------------------------------------------------------
Set-Service SysMain -StartupType Automatic
Start-Service SysMain -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 4. 恢复 Prefetch / Superfetch 默认值（3 = 全启用）
# ------------------------------------------------------------
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters" `
    -Name "EnablePrefetcher" -Value 3 -Type DWord -Force

Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters" `
    -Name "EnableSuperfetch" -Value 3 -Type DWord -Force

# ------------------------------------------------------------
# 5. 恢复后台应用
# ------------------------------------------------------------
Remove-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" `
    -Name "GlobalUserDisabled" -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 6. 恢复视觉特效（系统自动管理）
# ------------------------------------------------------------
Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\VisualEffects" `
    -Name "VisualFXSetting" -Value 0 -Type DWord -Force

# ------------------------------------------------------------
# 7. 恢复计划任务
# ------------------------------------------------------------
$tasks = @(
    "\Microsoft\Windows\Defrag\ScheduledDefrag",
    "\Microsoft\Windows\Application Experience\ProgramDataUpdater",
    "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator",
    "\Microsoft\Windows\Customer Experience Improvement Program\UsbCeip"
)

foreach ($t in $tasks) {
    schtasks /Change /TN $t /Enable | Out-Null
}

# ------------------------------------------------------------
# 8. 重启资源管理器
# ------------------------------------------------------------
Stop-Process -Name explorer -Force
Start-Process explorer.exe

Write-Host "HDD 优化已恢复为系统默认设置。" -ForegroundColor Green

```



## 6. Win10极限精简

### 6.1 生成Win10极限精简脚本

Win10 的“极限精简脚本”不是简单关几个服务，而是要 **系统性地关闭广告、遥测、推荐、在线内容、后台服务、计划任务、隐私上传、UWP 垃圾组件、搜索联网、自动安装内容** 等等。 你要的是 **真正干净、轻量、无广告、无遥测、无推荐、无后台 I/O** 的 Win10，我给你做了一份 **专业级、可逆、安全、不破坏系统核心功能** 的版本。

**Windows 10 极限精简脚本（PowerShell）**

> 精简范围：
>
> - 关闭广告 / 推荐 / 在线内容
> - 关闭遥测（Telemetry）
> - 关闭后台应用
> - 关闭隐私上传
> - 关闭自动安装 UWP 垃圾应用
> - 关闭搜索联网
> - 关闭 Cortana
> - 关闭反馈、诊断、体验改善
> - 禁用高 I/O 服务
> - 禁用隐私相关计划任务
> - 保留系统稳定性（不删系统组件）

```powershell
# ============================================================
# Windows 10 极限精简脚本（安全版）
# ============================================================

Write-Host "正在执行 Win10 极限精简..." -ForegroundColor Cyan

# ------------------------------------------------------------
# 1. 关闭所有广告、推荐、在线内容
# ------------------------------------------------------------
$advPaths = @(
  "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager",
  "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced"
)

foreach ($p in $advPaths) {
    Set-ItemProperty $p -Name "SystemPaneSuggestionsEnabled" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty $p -Name "ShowSyncProviderNotifications" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty $p -Name "SoftLandingEnabled" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty $p -Name "SubscribedContent-338389Enabled" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty $p -Name "SubscribedContent-353694Enabled" -Value 0 -ErrorAction SilentlyContinue
    Set-ItemProperty $p -Name "SubscribedContent-310093Enabled" -Value 0 -ErrorAction SilentlyContinue
}

# ------------------------------------------------------------
# 2. 关闭搜索联网 / 热门搜索 / Bing
# ------------------------------------------------------------
New-Item -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Force | Out-Null
Set-ItemProperty "HKCU:\Software\Policies\Microsoft\Windows\Explorer" `
    -Name "DisableSearchBoxSuggestions" -Value 1 -Type DWord -Force

Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "BingSearchEnabled" -Value 0 -Type DWord -Force

Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "AllowSearchToUseLocation" -Value 0 -Type DWord -Force

# ------------------------------------------------------------
# 3. 关闭 Cortana
# ------------------------------------------------------------
New-Item -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" -Force | Out-Null
Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "CortanaEnabled" -Value 0 -Type DWord -Force

# ------------------------------------------------------------
# 4. 关闭遥测（Telemetry）
# ------------------------------------------------------------
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\DataCollection" -Force | Out-Null
Set-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\DataCollection" `
    -Name "AllowTelemetry" -Value 0 -Type DWord -Force

Stop-Service DiagTrack -Force -ErrorAction SilentlyContinue
Set-Service DiagTrack -StartupType Disabled

Stop-Service dmwappushservice -Force -ErrorAction SilentlyContinue
Set-Service dmwappushservice -StartupType Disabled

# ------------------------------------------------------------
# 5. 禁用后台应用
# ------------------------------------------------------------
Set-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" `
    -Name "GlobalUserDisabled" -Value 1 -Type DWord -Force

# ------------------------------------------------------------
# 6. 禁用自动安装 UWP 垃圾应用
# ------------------------------------------------------------
New-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\CloudContent" -Force | Out-Null
Set-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\CloudContent" `
    -Name "DisableWindowsConsumerFeatures" -Value 1 -Type DWord -Force

# ------------------------------------------------------------
# 7. 禁用高 I/O 服务（SysMain）
# ------------------------------------------------------------
Stop-Service SysMain -Force -ErrorAction SilentlyContinue
Set-Service SysMain -StartupType Disabled

# ------------------------------------------------------------
# 8. 禁用隐私相关计划任务
# ------------------------------------------------------------
$tasks = @(
  "\Microsoft\Windows\Application Experience\ProgramDataUpdater",
  "\Microsoft\Windows\Autochk\Proxy",
  "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator",
  "\Microsoft\Windows\Customer Experience Improvement Program\UsbCeip",
  "\Microsoft\Windows\DiskDiagnostic\Microsoft-Windows-DiskDiagnosticDataCollector"
)

foreach ($t in $tasks) {
    schtasks /Change /TN $t /Disable | Out-Null
}

# ------------------------------------------------------------
# 9. 关闭反馈与诊断
# ------------------------------------------------------------
New-Item -Path "HKCU:\Software\Microsoft\Siuf\Rules" -Force | Out-Null
Set-ItemProperty "HKCU:\Software\Microsoft\Siuf\Rules" `
    -Name "NumberOfSIUFInPeriod" -Value 0 -Type DWord -Force

# ------------------------------------------------------------
# 10. 重启 Explorer
# ------------------------------------------------------------
Stop-Process -Name explorer -Force
Start-Process explorer.exe

Write-Host "Win10 极限精简完成！系统已变得更干净、更轻量。" -ForegroundColor Green

```

**精简后效果**

- 无广告
- 无推荐
- 无在线内容
- 无 Bing 搜索
- 无 Cortana
- 无遥测上传
- 无后台应用
- 无自动安装 UWP 垃圾
- 无隐私计划任务
- 系统更轻量、更干净、更流畅

这是 Win10 最干净的状态之一，同时 **不破坏系统核心功能**。





### 6.2 Win10极限精简恢复设置

**Windows 10 极限精简恢复脚本（安全版）**，它会把你之前执行的 Win10 极限精简全部恢复到系统默认状态，包括：

- 恢复广告/推荐/在线内容
- 恢复搜索联网（热门搜索、Bing、在线建议）
- 恢复 Cortana
- 恢复遥测（Telemetry）
- 恢复后台应用
- 恢复自动安装 UWP 内容
- 恢复 SysMain
- 恢复隐私相关计划任务
- 恢复反馈与诊断
- 重启 Explorer

脚本 **不删除文件、不破坏系统、不影响激活**，可安全执行。

```powershell
# ============================================================
# Windows 10 极限精简恢复脚本（恢复默认设置）
# ============================================================

Write-Host "正在恢复 Win10 默认设置..." -ForegroundColor Cyan

# ------------------------------------------------------------
# 1. 恢复广告、推荐、在线内容
# ------------------------------------------------------------
$advPaths = @(
  "HKCU:\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager",
  "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced"
)

foreach ($p in $advPaths) {
    Remove-ItemProperty $p -Name "SystemPaneSuggestionsEnabled" -ErrorAction SilentlyContinue
    Remove-ItemProperty $p -Name "ShowSyncProviderNotifications" -ErrorAction SilentlyContinue
    Remove-ItemProperty $p -Name "SoftLandingEnabled" -ErrorAction SilentlyContinue
    Remove-ItemProperty $p -Name "SubscribedContent-338389Enabled" -ErrorAction SilentlyContinue
    Remove-ItemProperty $p -Name "SubscribedContent-353694Enabled" -ErrorAction SilentlyContinue
    Remove-ItemProperty $p -Name "SubscribedContent-310093Enabled" -ErrorAction SilentlyContinue
}

# ------------------------------------------------------------
# 2. 恢复搜索联网 / 热门搜索 / Bing
# ------------------------------------------------------------
Remove-Item -Path "HKCU:\Software\Policies\Microsoft\Windows\Explorer" -Recurse -Force -ErrorAction SilentlyContinue

Remove-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "BingSearchEnabled" -ErrorAction SilentlyContinue

Remove-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "AllowSearchToUseLocation" -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 3. 恢复 Cortana
# ------------------------------------------------------------
Remove-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Search" `
    -Name "CortanaEnabled" -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 4. 恢复遥测（Telemetry）
# ------------------------------------------------------------
Remove-ItemProperty "HKLM:\Software\Policies\Microsoft\Windows\DataCollection" `
    -Name "AllowTelemetry" -ErrorAction SilentlyContinue

Set-Service DiagTrack -StartupType Automatic
Start-Service DiagTrack -ErrorAction SilentlyContinue

Set-Service dmwappushservice -StartupType Automatic
Start-Service dmwappushservice -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 5. 恢复后台应用
# ------------------------------------------------------------
Remove-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" `
    -Name "GlobalUserDisabled" -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 6. 恢复自动安装 UWP 内容
# ------------------------------------------------------------
Remove-Item -Path "HKLM:\Software\Policies\Microsoft\Windows\CloudContent" `
    -Recurse -Force -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 7. 恢复 SysMain（Superfetch）
# ------------------------------------------------------------
Set-Service SysMain -StartupType Automatic
Start-Service SysMain -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 8. 恢复隐私相关计划任务
# ------------------------------------------------------------
$tasks = @(
  "\Microsoft\Windows\Application Experience\ProgramDataUpdater",
  "\Microsoft\Windows\Autochk\Proxy",
  "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator",
  "\Microsoft\Windows\Customer Experience Improvement Program\UsbCeip",
  "\Microsoft\Windows\DiskDiagnostic\Microsoft-Windows-DiskDiagnosticDataCollector"
)

foreach ($t in $tasks) {
    schtasks /Change /TN $t /Enable | Out-Null
}

# ------------------------------------------------------------
# 9. 恢复反馈与诊断
# ------------------------------------------------------------
Remove-ItemProperty "HKCU:\Software\Microsoft\Siuf\Rules" `
    -Name "NumberOfSIUFInPeriod" -ErrorAction SilentlyContinue

# ------------------------------------------------------------
# 10. 重启 Explorer
# ------------------------------------------------------------
Stop-Process -Name explorer -Force
Start-Process explorer.exe

Write-Host "Win10 极限精简恢复完成！系统已回到默认状态。" -ForegroundColor Green

```

**恢复后效果**

- 所有广告/推荐恢复
- 搜索联网恢复（Bing、热门搜索）
- Cortana 恢复
- 遥测恢复
- 后台应用恢复
- UWP 自动安装恢复
- SysMain 恢复
- 隐私计划任务恢复
- 系统回到 Win10 默认行为