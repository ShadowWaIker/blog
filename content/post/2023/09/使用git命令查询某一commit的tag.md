---
title: 中科方德服务器系统编译 lmbench 缺少 rpc.h 及 netconfig.h 等依赖的解决方法
author: ShadoWalker
type: post
date: 2023-09-13T16:13:00+08:00
tags:
  - 笔记
---

#### 1. 安装 libtirpc-devel

```bash
sudo yum -y install libtirpc-devel
```

#### 2. 添加编译指令

```bash
export LDFLAGS='-ltirpc'
```

#### 3. 开始测试

```bash
make results
```
