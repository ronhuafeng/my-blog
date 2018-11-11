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

## youtube-dl video and extract audio file 

`youtube-dl --proxy socks5://127.0.0.1:1080 -x --audio-format mp3 youtube-url`

## virtualenvwrapper

- `WORKON_HOME`: which directory your environments are created in
- `/usr/local/bin/virtualenvwrapper.sh`: default location for its configuration file
- `mkvirtualenv test --python=python3`: make a virtual environment 'test' with python3
- `rmvirtualenv test`: remove a virtual environment 'test'
- `workon test3` or `lsvirtualenv -b test3`: activate a virtual environment 'test'
- `deactivate`: exit current environment
- more details: search engine
  - how to avoid globa packages
  - how to copy an environment
