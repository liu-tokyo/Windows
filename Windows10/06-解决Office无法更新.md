# 解决Office无法更新

## 1. Office产品更新无法被勾选

- 现象：  
  `更新 Windows 时接收其他 Microsoft 产品的更新` 项目灰色，无法被勾选：

- 解决：  
  按`Windows+R`输入`gpedit.msc`打开组策略，在左侧选`用户配置`—`管理模板`—`Windows组件`—`Windows Update`—在右侧选“删除使用所有Windows Update功能的访问”双击它，在打开的对话框中选择“已启用”然后按应用确定，重启电脑即可。
- 注册表方式：  
  1. 如果已是未配置状态，那么进行下一步。
  2. 打开注册表编辑器 regedit，转到 计算机  
     `\HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU`，删掉  `NoAutoUpdate` 项。先备份下注册表。  
     - 设置成 0 为禁用(灰色) 
     - 设置成 1 就可以修改了
  3. 然后刷新下界面，就可以选择啦。



## 2. 注册表方式

解决方法如下:
- win+r，打开运行，输入“regedit”命令，执行；
- B.、展开左侧的目录一直到：`HKEY_LOCAL_MACHINE/Software/Policies/Microsoft/Windows`，展开`Windows`之后，您可能会找到有一个叫做`WindowsUpdate`的资料夹（机码），在`WindowsUpdate`上按鼠标右键选删除，即可。
（注：正常的 注册表 里是没有叫做`WindowsUpdate`的资料夹的，这个是被人添加上去的，它的作用就是让您的电脑无法正常的自动更新！）



## 3. 起他解决办法

亦或是这样解决：

1. 右击我的电脑－管理－服务和应用程序，双击服务，找到`automatic updates`双击，设置为自动；

2. 开始——运行——gpedit.msc——本地计算机策略——计算机配置——管理 模板 ——windows组件——windows update——配置自动更新——双击——已启用——在“配置自动更新”处选择“ 5-允许本地管理员设置”；

3. 开始运行regedit 展开 `HKEY_LOCAL_MACHINE/SOFTWARE/Policies/Microsoft/Windows/WindowsUpdate/AU` 二进制值 `NoAutoUpdate`  
   设置成 `0` 为禁用(灰色)
   设置成 `1` 就可以修改了
