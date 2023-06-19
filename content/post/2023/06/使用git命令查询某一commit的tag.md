---
title: 使用 git 命令查询某一 commit 的 tag
author: ShadoWalker
type: post
date: 2023-06-19T10:13:00+08:00
tags:
  - 笔记
---

#### 1. 查询某一 commit 的 tag

```bash
git describe --tags --abbrev=0 --exact-match <commit>
```

#### 2. 查询当前版本的 tag

```bash
git describe --tags --abbrev=0 --exact-match $(git rev-parse --short=8 HEAD)
```
