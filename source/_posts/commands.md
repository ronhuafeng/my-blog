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

## node && npm

npm complains: `Error: Cannot find module 'process-nextick-args'`

Uninstall node, `brew uninstall node`, then by [this stackoverflow post](https://stackoverflow.com/questions/11177954/how-do-i-completely-uninstall-node-js-and-reinstall-from-beginning-mac-os-x):

```
sudo rm -rf /usr/local/bin/npm /usr/local/share/man/man1/node* /usr/local/lib/dtrace/node.d ~/.npm ~/.node-gyp 
sudo rm -rf /opt/local/bin/node /opt/local/include/node /opt/local/lib/node_modules
sudo rm -rf /usr/local/bin/npm /usr/local/share/man/man1/node.1 /usr/local/lib/dtrace/node.d
```

Just delete something, then `brew install npm`.

## delve (dlv) tips

- funcs [regexp] : get function list
- call : call a function (in a newer a go version, dlv should be installed in the newer go version too)

## shadowsocks

`ssserver -c /etc/shadowsocks/config.json`

`sslocal` and `ssserver` are all from `apt-get install shadowsocks`.

## WSL 2 && VMware switch

After enabling WSL 2 on Windows 10 insider preview, VMware virtual machine is disabled.
This is because collision between Hyper-V and VMware.

Turn on VMware

```
 bcdedit /set hypervisorlaunchtype off
```

Turn on Hyper-V && WSL 2

```
 bcdedit /set hypervisorlaunchtype auto
```

reference: https://blog.minirplus.com/10268/
