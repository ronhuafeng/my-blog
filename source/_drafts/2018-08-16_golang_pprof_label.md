---
title: golang pprof profiler label
date: 2018-08-16
categories:
  - 技术记录
  - tech-notes
tags: 
---

```bash
git clone -b release-branch.go1.10 https://github.com/golang/go.git release-branch.go1.10
cd release-branch.go1.10/src
./make.bash
```

```
Building Go cmd/dist using /usr/lib/go-1.10.
Building Go toolchain1 using /usr/lib/go-1.10.
Building Go bootstrap cmd/go (go_bootstrap) using Go toolchain1.
Building Go toolchain2 using go_bootstrap and Go toolchain1.
Building Go toolchain3 using go_bootstrap and Go toolchain2.
Building packages and commands for linux/amd64.
---
Installed Go for linux/amd64 in ~/Projects/release-branch.go1.10
Installed commands in ~/Projects/release-branch.go1.10/bin
```