---
title: 申威  ( sw64 ) 架构高版本 Golang ( go-1.24.10 ) 适配小记
author: ShadoWalker
type: post
date: 2025-11-18T19:13:00+08:00
tags:
  - 笔记
---

#### 1. 逐步编译高版本 golang

系统中自带的 golang 版本为：
```bash
go version go1.16.4 linux/sw64 (Git-commit:7c7ef9447 Date:20220324)
```

这一版本直接编译 go-1.24.10 源码会报错：

```bash
Building Go cmd/dist using /usr/lib/golang. (go1.16.4 linux/sw64 (Git-commit:7c7ef9447 Date:20220324))
//go:build comment without // +build comment
cmd/dist/buildtool.go:16:2: cannot find package "go/version" in any of:
	/usr/lib/golang/src/go/version (from $GOROOT)
	/root/go/src/go/version (from $GOPATH)
```

这是由于当前版本过低，不支持 1.24.10 源码中使用的新语法导致的。

所以我们需要前往申威官网下载更高版本的 golang 源码，逐步编译获取到能得到的最高版本的 golang。官网地址：
https://developer.wxiat.cn/#/ecology/basicsoftware/detail?id=10007

升级路径为：
go1.16.4 -> go-sw64-1.17.7 -> go-sw64-1.20.10 -> go-sw64-1.22.6 。

go-sw64-1.17.7 编译步骤为：
```bash
tar zxvf go-sw64-1.17.7-master.tar.gz
cd go-sw64-1.17.7/src
./make.bash
```

go-sw64-1.20.10 编译步骤为：
```bash
tar zxvf go-sw64-1.20.10-release-branch.go1.20.sw64.tar.gz
cd go-sw64-1.20.10/src
export GOROOT_BOOTSTRAP=/path-to-1.17/go-sw64-1.17.7
./make.bash
```

go-sw64-1.22.6 编译步骤为：
```bash
tar zxvf go-sw64-1.22.6-release-branch.go1.22.6.sw64.tar.gz
cd go-sw64-1.22.6/src
export GOROOT_BOOTSTRAP=/path-to-1.20/go-sw64-1.20.10
./make.bash
```

至此我们完成了编译环境的准备。


#### 2. 提取 golang 1.22.6 版本的 patch

前往 golang 官网 ( https://go.dev/dl/ ) ，下载原始版本的 golang 1.22.6 源码。

```bash
wget https://go.dev/dl/go1.22.6.src.tar.gz
```

将源码解压后，基于当前源码创建 git 仓库。
```bash
tar zxvf go1.22.6.src.tar.gz
cd go
git init
git add .
git commit -m init
```

将修改后的源码覆盖到原始版本中，生成 patch 文件。
```bash
cd ..
cp -r go-sw64-1.22.6/* go/
cd go
git diff > sw.patch
```


#### 3. 将 go-sw64-1.22.6 的 patch 应用到原始版本的 1.24.10 源码中

前往 golang 官网 ( https://go.dev/dl/ ) ，下载原始版本的 golang 1.22.6 源码。

```bash
wget https://go.dev/dl/go1.24.10.src.tar.gz
```

将源码解压后，基于当前源码创建 git 仓库。
```bash
tar zxvf go1.24.10.src.tar.gz
cd go
git init
git add .
git commit -m init
```

应用 patch。
```bash
git apply --reject sw.patch
```



#### 4. 手工解决由于冲突未能合入的代码

执行步骤 3 后，大部分修改可以直接合入，未能合入的代码会在对应位置生成文件同名的 rej 文件。

未能合入的原因基本都是因为 golang 迭代导致代码行数对不上导致的，逐个查看 rej 文件手工合入代码。


#### 5. 尝试编译

完成所有代码合入后，可以开始尝试进行编译：

```bash
cd src
export GOROOT_BOOTSTRAP=/path-to-1.22.6/go-sw64-1.22.6
./make.bash
```

#### 6. 解决编译报错

合入 1.22.6 的所有 patch 后，代码编译依旧会报错，主要的原因和解决方式为：

1. golang 版本迭代导致某些模块路径发生变化。最显著存在这一问题的包为 go-sw64-1.22.6/src/runtime/internal/syscall & go-1.24.10/src/internal/runtime/syscall 。1.22 版本中这个包内的 asm_linux_sw64.s、defs_linux_sw64.go 两个文件未被正确合入 1.24 版本的源码，会导致出现编译时报错大量变量和函数未定义。


2. golang 版本迭代新增的常量在 1.22.6 版本的申威架构源码中没有定义。最显著存在这一问题的源码为 go-1.24.10/src/internal/runtime/syscall/defs_linux_sw64.go ，1.24 引入了 SYS_EVENTFD2、EFD_NONBLOCK、SYS_MPROTECT 这些变量在 1.22 中未定义。需要查找系统中的头文件定义找到正确的数值并补充到源码中。

例如 EFD_NONBLOCK 的查找方式为：
```bash
grep -r "EFD_NONBLOCK" /usr/include/
```
检索到的数值为八进制，golang 中使用的是 16 进制，注意进行数值转换。