---
title: KMS 服务
subtitle: 一句命令激活 Windows / Office
type: page
comments: false # 关闭评论
date: 2019-09-30T18:56:00+08:00
---

## <center>基本情况</center>

  * 服务器地址：`kms.s1s8.com`
  * 服务端版本：`vlmcsd 2018-10-15(20?)(1112)`
  * 上传维护时间：`2019-09-18 10:45`
  * 优点：命令激活，省时简便；无需软件，纯净无毒；在线激活，自动续期，相当于永久激活！
  * 适用对象：VOL 版本的 Windows 和 Office。
  * 适用版本：截止到 Windows 10、Windows Server 2016 和 Office 2016 的所有版本。
  * 备用服务器地址：`kms.rebeta.cn`
  * 备用服务端版本：`vlmcsd 2017-06-17(1111)`
  * 上传维护时间：`2018-10-16`

## <center>使用方法</center>

### 一句命令激活 Windows

以管理员身份运行命令提示符（ CMD ），依次执行以下操作：

  1. `slmgr /skms kms.s1s8.com` %把系统 kms 服务器地址设置为 kms.s1s8.com%
  2. `slmgr /ato` %立即尝试激活当前系统%
  3. `slmgr /dlv` %查看激活信息%

### 一句命令激活 Office

以 64 位系统，Office2016 为例，其他版本可能目录有区别，注意区分！

以管理员身份运行命令提示符（ CMD ），依次执行以下操作：

  1. `cd "C:\Program Files (x86)\Microsoft Office\Office16"` %进入 Office 安装目录%
  2. `cscript ospp.vbs /sethst:kms.s1s8.com` %把kms服务器地址设置为 kms.s1s8.com%
  3. `cscript ospp.vbs /act` %立即尝试激活 Office%
  4. `cscript ospp.vbs /dstatus` %查看激活信息%

### 如果遇到报错

  1. 你的系统或 Office 是不是**批量授权**（ VOL ）版本;
  2. 查看是否以**管理员**身份运行命令提示符;
  3. 你的系统或 Office 是否修改过KEY或未安装 GVLK KEY。

  * 你可以到 [**这里**][1] 获取`Windows GVLK KEY`；或到 [**这里**][2] 或 [这里 ( 官方 )][3] 获取`Office GVLK KEY`。
  * 对于Windows，你需要先运行`slmgr /upk`卸载秘钥，再运行`slmgr /ipk “GVLK KEY”`重新安装GVLK KEY。
  * 对于Office，先运行`cscript ospp.vbs /dstatus`记下现有秘钥的后5位，再运行`cscript ospp.vbs /unpkey:“现有秘钥后5位”`将现有秘钥卸载，最后运行`cscript ospp.vbs /inpkey:“GVLK KEY”`重新安装GVLK KEY。

另外你还可以在微软官网找到可以用于KMS激活的Gvlk

  * office2016 https://technet.microsoft.com/zh-cn/library/dn385360(v=office.16).aspx
  * office2013 https://technet.microsoft.com/ZH-CN/library/dn385360.aspx
  * office2010 https://technet.microsoft.com/ZH-CN/library/ee624355(v=office.14).aspx
  * Server/Windows https://docs.microsoft.com/zh-cn/windows-server/get-started/kmsclientkeys

### KMS服务器状态检测

  * 打开命令提示符（ CMD ），把 vlmcs 拖进去，打上你要测试的地址:端口 ( 不打端口默认就是1688 ) 返回信息显示 successful 就说明kms服务器可用。
  * 更多高级用法请打 vlmcs -help 和 vlmcs -1。
  * -v 参数可以显示激活测试的详细信息。
  * -l 参数 ( 小写L ) + 产品编号或者名字 可以指定检测什么产品，打 vlmcs -help 可以查看所有支持检测的产品和编号。
  * 直接添加-l ( 小写L ) 参数可以列出服务器可激活的产品列表。

### Office2016零售版 (Retail)转批量授权(VOL)

```go-html-template
@ECHO OFF&PUSHD %~DP0
setlocal EnableDelayedExpansion&color 3e & cd /d "%~dp0"
title office2016 retail转换vol版
%1 %2
mshta vbscript:createobject("shell.application").shellexecute("%~s0","goto :runas","","runas",1)(window.close)&goto :eof
:runas
if exist "%ProgramFiles%\Microsoft Office\Office16\ospp.vbs" cd /d "%ProgramFiles%\Microsoft Office\Office16"
if exist "%ProgramFiles(x86)%\Microsoft Office\Office16\ospp.vbs" cd /d "%ProgramFiles(x86)%\Microsoft Office\Office16"
:WH
cls
echo.
echo                         选择需要转化的office版本序号
echo.
echo --------------------------------------------------------------------------------
echo                 1. 零售版 Office Pro Plus 2016 转化为VOL版
echo.
echo                 2. 零售版 Office Visio Pro 2016 转化为VOL版
echo.
echo                 3. 零售版 Office Project Pro 2016 转化为VOL版
echo.
echo. --------------------------------------------------------------------------------
set /p tsk="请输入需要转化的office版本序号【回车】确认（1-3）: "
if not defined tsk goto:err
if %tsk%==1 goto:1
if %tsk%==2 goto:2
if %tsk%==3 goto:3
:err
goto:WH
:1
cls
echo 正在安装 KMS 许可证...
for /f %%x in ('dir /b ..\root\Licenses16\proplusvl_kms*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul
echo 正在安装 MAK 许可证...
for /f %%x in ('dir /b ..\root\Licenses16\proplusvl_mak*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul
set /p y=请输入激活密钥，按回车确定:
cscript ospp.vbs /inpkey:%y%
goto :e
:2
cls
echo 正在安装 KMS 许可证...
for /f %%x in ('dir /b ..\root\Licenses16\visio???vl_kms*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul
echo 正在安装 MAK 许可证...
for /f %%x in ('dir /b ..\root\Licenses16\visio???vl_mak*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul
set /p y=请输入激活密钥，按回车确定:
cscript ospp.vbs /inpkey:%y%
goto :e
:3
cls
echo 正在安装 KMS 许可证...
for /f %%x in ('dir /b ..\root\Licenses16\project???vl_kms*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul
echo 正在安装 MAK 许可证...
for /f %%x in ('dir /b ..\root\Licenses16\project???vl_mak*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul
set /p y=请输入激活密钥，按回车确定:
cscript ospp.vbs /inpkey:%y%
goto :e
:e
echo.
echo 转化完成，按任意键退出！
pause >nul
exit
```
转载自：https://blog.csdn.net/permanlightfelder/article/details/55806189

<p style="text-align: center;">
  <strong>本页面部分内容转载自</strong><br /> <a title="蓝色随想" href="https://bluskai.com/">蓝色随想</a><br /> (https://bluskai.com/kms/)<br /> <a title="零散坑" href="https://03k.org/">零散坑</a><br /> (https://03k.org/kms.html)<br /> <a href="https://blog.csdn.net/permanlightfelder">permanlightfelder</a><br /> (https://blog.csdn.net/permanlightfelder/article/details/55806189)
</p>

 [1]: /page/windows-gvlk/
 [2]: /page/office-gvlk/
 [3]: https://docs.microsoft.com/zh-cn/DeployOffice/vlactivation/gvlks