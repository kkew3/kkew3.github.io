---
layout: post
title:  "用TCL（工具命令语言）管理Tornado (for VxWorks) 可启动工程"
date:   2016-03-25 23:33:31 +0800
categories: tcl
---

尽管多数情况下要写VxWorks嵌入式应用程序代码常用Tornado编程环境，但有时可能会需要在命令行下完成简单的Tornado工程管理。本教程教授了如何将简单的工程管理迁移到Tornado外部并在命令行下实现（虽然这样做无法体验Tornado下的一些方便的功能）。

0. 准备Tornado软件。首先得有Tornado的全套软件。我的是Tornado2.2。Tornado是否经过破解或是否安装都问题不大，只要有它的安装目录就可以。

1. 配置环境。Tornado环境中已经配好了各种环境变量，所以我们要向在普通cmd下实现Tornado的基本功能，也需要手动配置相应的环境。a) 新建环境变量`WIND_BASE`，其值为Tornado的安装目录（例如我的Tornado安装在D盘Tornado2.2目录下，那么`WIND_BASE`值为`D:\Tornado2.2`；b) 新建环境变量`WIND_HOST_TYPE`，如果是Windows用户，那么需要将其值设为`x86-win32`，如果不是Windows用户，那么凭本人的知识就不太清楚了；c) 将`%WIND_BASE%\host\%WIND_HOST_TYPE%\bin`加入`PATH`环境变量；d) 新建环境变量`DIABLIB`，其值为`%WIND_BASE%/host/diab`（注意斜杠的方向）。注意这些变量必须真得加到系统环境变量中而不是仅在命令行上输`SET WIND_BASE=D:\Tornado2.2`等等。

2. 配置`diab`和`gnu`工具链。在cmd中执行以下两条批处理命令：

```cmd
wtxtcl.exe %WIND_BASE%/host/resource/tcl/app-config/Project/gnuInfoGen.tcl diab
wtxtcl.exe %WIND_BASE%/host/resource/tcl/app-config/Project/gnuInfoGen.tcl gnu
```

3. 基本的工程管理方法（建议将下面的每条内容都写到TCL脚本文件中以方便调用）

a) 建立新工程（本例中BSP（板级支持包）以三星的嵌入式开发板`S3c2410BP`为例）

```tcl
# 加载过程库文件cmpScriptLib.tcl，其中定义了工程管理所需的各种方法
source [wtxPath host resource tcl app-config Project]cmpScriptLib.tcl

# 尝试创建名为"Project0"的可启动工程，注意Project0一定不能是已经存在的工程
# 新工程位于%WIND_BASE%\target\proj目录下，该目录由可接受任意个参数的命令wtxPath指定
# S3c2410BP是BSP名，BSP应放在%WIND_BASE%\target\config目录下
cmpProjCreate S3c2410BP [wtxPath target proj Project0]Project0.wpj
cmpProjClose
```

b) 删除工程（以删除工程"Project0"为例）

```tcl
source [wtxPath host resource tcl app-config Project]cmpScriptLib.tcl

cmpProjOpen [wtxPath target proj Project0]Project0.wpj
cmpProjDelete
```

c) 向工程（以`Project0`为例）中添加文件（以`D:\my_directory\my_source_file.c`为例）

```tcl
source [wtxPath host resource tcl app-config Project]cmpScriptLib.tcl
cmpProjOpen [wtxPath target proj Project0]Project0.wpj
cmpFileAdd d:/my_directory/my_source_file.c
cmpProjClose
```

d) 从工程（以`Project0`为例）中移除文件（以`D:\my_directory\my_source_file.c`为例）

```tcl
source [wtxPath host resource tcl app-config Project]cmpScriptLib.tcl
cmpProjOpen [wtxPath target proj Project0]Project0.wpj
cmpFileRemove d:/my_directory/my_source_file.c
cmpProjClose
```

e) 获取工程中包含的文件列表（一行一个文件名，以`Project0`为例）

```tcl
source [wtxPath host resource tcl app-config Project]cmpScriptLib.tcl
set projId [cmpProjOpen [wtxPath target proj Project0]Project0.wpj]
set file_list [prjFileListGet $projId]
cmpProjClose
foreach item $file_list {
    puts $item
}
```

f) 重新编译工程（以`Project0`为例）

```tcl
source [wtxPath host resource tcl app-config Project]cmpScriptLib.tcl
cmpProjOpen [wtxPath target proj Project0]Project0.wpj
cmpBuild clean
cmpBuild
cmpProjClose
```

本教程至此结束，若对TCL语言不很熟悉，请参阅工具命令语言（TCL）的相关教程。
