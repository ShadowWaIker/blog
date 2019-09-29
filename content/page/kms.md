---
title: KMS 服务
subtitle: 一句命令激活 Windows / Office
author: ShadoWalker
type: page
date: 2018-10-16T07:17:52+00:00

---
**基本情况**

  * 服务器地址：`kms.s1s8.com`
  * 服务端版本：`vlmcsd 2018-10-15(20?)(1112)`
  * 备用服务器地址：`kms.rebeta.cn`
  * 备用服务端版本：`vlmcsd 2017-06-17(1111)`
  * 优点：命令激活，省时简便；无需软件，纯净无毒；在线激活，自动续期，相当于永久激活！
  * 上传维护时间：`2018-10-16`
  * 适用对象：VOL版本的Windows和Office
  * 使用版本：截止到Windows 10、Windows Server 2016和Office 2016的所有版本

**使用方法**

_一句命令激活Windows_

以管理员身份运行命令提示符（CMD），依次执行以下操作：

  1. `slmgr /skms kms.rebeta.cn` %把系统kms服务器地址设置为kms.rebeta.cn%
  2. `slmgr /ato` %立即尝试激活当前系统%
  3. `slmgr /dlv` %查看激活信息%

_一句命令激活Office_

以64位系统，Office2016为例，其他版本可能目录有区别，注意区分！

以管理员身份运行命令提示符（CMD），依次执行以下操作：

  1. `cd "C:\Program Files (x86)\Microsoft Office\Office16"` %进入Office安装目录%
  2. `cscript ospp.vbs /sethst:kms.rebeta.cn` %把kms服务器地址设置为kms.rebeta.cn%
  3. `cscript ospp.vbs /act` %立即尝试激活Office%
  4. `cscript ospp.vbs /dstatus` %查看激活信息%

**如果遇到报错**

  1. 你的系统或Office是不是**批量授权**（VOL）版本;
  2. 查看是否以**管理员**身份运行命令提示符;
  3. 你的系统或Office是否修改过KEY或未安装GVLK KEY。

  * 你可以到[**这里**][1]获取`Windows GVLK KEY`；或到[**这里**][2]或[这里(官方)][3]获取取`Office GVLK KEY`。
  * 对于Windows，你需要先运行`slmgr /upk`卸载秘钥，再运行`slmgr /ipk “GVLK KEY”`重新安装GVLK KEY。
  * 对于Office，先运行`cscript ospp.vbs /dstatus`记下现有秘钥的后5位，再运行`cscript ospp.vbs /unpkey:“现有秘钥后5位”`将现有秘钥卸载，最后运行`cscript ospp.vbs /inpkey:“GVLK KEY”`重新安装GVLK KEY。

另外你还可以在微软官网找到可以用于KMS激活的Gvlk

  * office2016 https://technet.microsoft.com/zh-cn/library/dn385360(v=office.16).aspx
  * office2013 https://technet.microsoft.com/ZH-CN/library/dn385360.aspx
  * office2010 https://technet.microsoft.com/ZH-CN/library/ee624355(v=office.14).aspx
  * Server/Windows https://docs.microsoft.com/zh-cn/windows-server/get-started/kmsclientkeys

**KMS服务器状态检测**

  * 打开命令提示符（CMD），vlmcs拖进去，打上你要测试的地址:端口（不打端口默认就是1688）返回信息显示 successful 就说明kms服务器可用。
  * 更多高级用法请打vlmcs -help和vlmcs -1。
  * -v参数可以显示激活测试的详细信息。
  * -l 参数（小写L）+产品编号或者名字 可以指定检测什么产品，打vlmcs -help可以查看所有支持检测的产品和编号。
  * 直接添加-l（小写L）参数可以列出服务器可激活的产品列表。

**Office2016零售版(Retail)转批量授权(VOL)**
  
`@ECHO OFF&PUSHD %~DP0<br />
setlocal EnableDelayedExpansion&color 3e & cd /d "%~dp0"<br />
title office2016 retail转换vol版<br />
%1 %2<br />
mshta vbscript:createobject("shell.application").shellexecute("%~s0","goto :runas","","runas",1)(window.close)&goto :eof<br />
:runas<br />
if exist "%ProgramFiles%\Microsoft Office\Office16\ospp.vbs" cd /d "%ProgramFiles%\Microsoft Office\Office16"<br />
if exist "%ProgramFiles(x86)%\Microsoft Office\Office16\ospp.vbs" cd /d "%ProgramFiles(x86)%\Microsoft Office\Office16"<br />
:WH<br />
cls<br />
echo.<br />
echo                         选择需要转化的office版本序号<br />
echo.<br />
echo --------------------------------------------------------------------------------<br />
echo                 1. 零售版 Office Pro Plus 2016 转化为VOL版<br />
echo.<br />
echo                 2. 零售版 Office Visio Pro 2016 转化为VOL版<br />
echo.<br />
echo                 3. 零售版 Office Project Pro 2016 转化为VOL版<br />
echo.<br />
echo. --------------------------------------------------------------------------------<br />
set /p tsk="请输入需要转化的office版本序号【回车】确认（1-3）: "<br />
if not defined tsk goto:err<br />
if %tsk%==1 goto:1<br />
if %tsk%==2 goto:2<br />
if %tsk%==3 goto:3<br />
:err<br />
goto:WH<br />
:1<br />
cls<br />
echo 正在安装 KMS 许可证...<br />
for /f %%x in ('dir /b ..\root\Licenses16\proplusvl_kms*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul<br />
echo 正在安装 MAK 许可证...<br />
for /f %%x in ('dir /b ..\root\Licenses16\proplusvl_mak*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul<br />
set /p y=请输入激活密钥，按回车确定:<br />
cscript ospp.vbs /inpkey:%y%<br />
goto :e<br />
:2<br />
cls<br />
echo 正在安装 KMS 许可证...<br />
for /f %%x in ('dir /b ..\root\Licenses16\visio???vl_kms*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul<br />
echo 正在安装 MAK 许可证...<br />
for /f %%x in ('dir /b ..\root\Licenses16\visio???vl_mak*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul<br />
set /p y=请输入激活密钥，按回车确定:<br />
cscript ospp.vbs /inpkey:%y%<br />
goto :e<br />
:3<br />
cls<br />
echo 正在安装 KMS 许可证...<br />
for /f %%x in ('dir /b ..\root\Licenses16\project???vl_kms*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul<br />
echo 正在安装 MAK 许可证...<br />
for /f %%x in ('dir /b ..\root\Licenses16\project???vl_mak*.xrm-ms') do cscript ospp.vbs /inslic:"..\root\Licenses16\%%x" >nul<br />
set /p y=请输入激活密钥，按回车确定:<br />
cscript ospp.vbs /inpkey:%y%<br />
goto :e<br />
:e<br />
echo.<br />
echo 转化完成，按任意键退出！<br />
pause >nul<br />
exit`
  
转载自：https://blog.csdn.net/permanlightfelder/article/details/55806189

<p style="text-align: center;">
  <strong>本页面部分内容转载自</strong><br /> <a title="蓝色随想" href="https://bluskai.com/">蓝色随想</a><br /> (https://bluskai.com/kms/)<br /> <a title="零散坑" href="https://03k.org/">零散坑</a><br /> (https://03k.org/kms.html)<br /> <a href="https://blog.csdn.net/permanlightfelder">permanlightfelder</a><br /> (https://blog.csdn.net/permanlightfelder/article/details/55806189)
</p>

 [1]: https://blog.tracewalker.com/?page_id=212
 [2]: https://blog.tracewalker.com/?page_id=215
 [3]: https://docs.microsoft.com/zh-cn/DeployOffice/vlactivation/gvlks