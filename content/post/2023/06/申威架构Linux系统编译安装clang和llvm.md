---
title: 申威架构Linux系统编译安装clang和llvm
author: ShadoWalker
type: post
date: 2023-07-21T11:02:00+08:00
tags:
  - 笔记
---

## sw 架构 Linux 系统 ( NFS Server RTM4.0 ) 编译安装 clang 和 llvm

#### 1. 下载源码

```bash
git clone git@github.com:llvm/llvm-project.git -b release/12.0.1
```

#### 2. 准备构建目录

```bash
cd llvm-project && mkdir build && cd build
```

#### 3. 准备编译

```bash
cmake -G "Unix Makefiles" ../llvm -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;"
```

#### 3. 编译
使用 -j64 指定多核编译，加快编译速度
```bash
make -j64
```

#### 3. 安装
```bash
make install
```