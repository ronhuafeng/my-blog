---
title: Useful Commands
date: 2018-07-22
categories:
  - 技术记录
  - tech-notes
tags: 
---

## Convert images to a video

```bash
ffmpeg -r 30 -start_number 3455 -i _IMG%d.jpg -s 960X600 -pix_fmt yuv420p 30fps-960.mov
```

- `-r 30`: 30 frames per second
- `-s 960X600`: resolution
- `-pix_fmt yuv420p`: for OsX
