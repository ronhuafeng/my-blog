---
title: Upgrade DigitalOcean's Ubuntu 17.04 release which reaches/passes its End of Life (EOL)
date: 2018-06-23
categories:
  - 技术记录
  - tech-notes
tags: 
---

[//]: # (Image References)
[Dolores]: https://cdn.vox-cdn.com/thumbor/NxFcQHzIb1eqgk5EXafSzFwy2UU=/0x0:5100x3400/920x613/filters:focal(1816x555:2632x1371):format(webp)/cdn.vox-cdn.com/uploads/chorus_image/image/52220535/westworld_dolores.0.jpeg
[release-end-of-life-date]: https://assets.ubuntu.com/v1/f02f0a4b-r-eol-ubuntu-full-2018-02-28.png


## What happened?

I found my proxy for accessing some websites stopped working today, so I had to change my VPS's IP address.
After some trials, everything seemed OK and I started watching a skiing video made by NorthFace on Youtube.

Emmmm, I noticed 12 packages needed to be updated. 
Well, I typed `sudo apt-get update` and got messages like this (I didn't save the error messages then):

```
  404  Not Found [IP: 91.189.91.23 80]
Ign:10 http://us.archive.ubuntu.com/ubuntu zesty-backports InRelease
Err:11 http://us.archive.ubuntu.com/ubuntu zesty Release
  404  Not Found [IP: 91.189.91.26 80]
Err:12 http://us.archive.ubuntu.com/ubuntu zesty-updates Release
  404  Not Found [IP: 91.189.91.26 80]
Err:13 http://us.archive.ubuntu.com/ubuntu zesty-backports Release
  404  Not Found [IP: 91.189.91.26 80]
Reading package lists... Done
E: The repository 'http://security.ubuntu.com/ubuntu zesty-security Release' does no longer have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
E: The repository 'http://us.archive.ubuntu.com/ubuntu zesty Release' does no longer have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
E: The repository 'http://us.archive.ubuntu.com/ubuntu zesty-updates Release' does no longer have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
E: The repository 'http://us.archive.ubuntu.com/ubuntu zesty-backports Release' does no longer have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```

Wow, It was the first time I encountered such things. 
After some digging, I realized it was because Ubuntu 17.04 (zesty) was not supported any longer.

According to https://wiki.ubuntu.com/ZestyZapus/ReleaseNotes

> Ubuntu 17.04 will be supported for 9 months until January 2018. If you need Long Term Support, it is recommended you use Ubuntu 16.04 LTS instead.

It means my current release version (Ubuntu 17.04) won't receive any updates in the future. 

![release-end-of-life-date][release-end-of-life-date]

## How to upgrade

I was struggled to reveal some steps to upgrade my Ubuntu 17.04 (End-of-Life) to 18.04 (LTS version).

The official guide to upgrade from an Ubuntu release which reaches its “end of life” is: https://help.ubuntu.com/community/EOLUpgrades

### 1. Update sources.list

The path of *sources.list* is */etc/apt/sources.list*.

```
## EOL upgrade sources.list
# Required
deb http://old-releases.ubuntu.com/ubuntu/ CODENAME main restricted universe multiverse
deb http://old-releases.ubuntu.com/ubuntu/ CODENAME-updates main restricted universe multiverse
deb http://old-releases.ubuntu.com/ubuntu/ CODENAME-security main restricted universe multiverse

# Optional
#deb http://old-releases.ubuntu.com/ubuntu/ CODENAME-backports main restricted universe multiverse
```

In my case, `CODENAME` should be replaced by `zesty`.

### 2. Upgrade

Try `apt-get upgrade`, and I got:

```
dpkg: dependency problems prevent configuration of ubuntu-standard:
 ubuntu-standard depends on libpam-systemd; however:
  Package libpam-systemd:amd64 is not configured yet.
```

By some method suggested by stackoverflowers, I finally got over it by these commands:

```bash
sudo dpkg --force-all -P libpam-systemd
sudo apt-get -f install libpam-systemd
```

### 3. Dist Upgrade

These are some normal steps for an upgrade.

```
sudo apt-get dist-upgrade
sudo do-release-upgrade
```

If you're unlucky like me, you will not upgrade to a newer release completely.
You may see:

```
Package grub-efi-amd64 is not configured yet.
```

According to [a question](https://askubuntu.com/questions/330531/unable-to-fix-broken-package-system) asked by someone, you can try this:

```bash
sudo dpkg --force-all -P grub-efi-amd64
sudo dpkg --force-all -P grub-efi-amd64-signed
```

Execute `sudo apt-get update` and `sudo apt-get upgrade`.

### 4. Thoughts

I first upgraded to Ubuntu 17.10 then Ubuntu 18.04.

To see if I have done the right things, I check the release version by `sudo lsb_release -a`:

```
Distributor ID:    Ubuntu
Description:       Ubuntu 18.04 LTS
Release:           18.04
Codename:          bionic
```

Is it the DigitalOcean's customized release makes the upgrade so complex for a non-expert programmer or the Ubuntu release itself?

But apparently, I'm not the only one who is confused by the error messages and asks questions on the web. Luckily I've found helping answers I need. 

The commands with words like `force` or `-f` make me feel anxious.

> These violent delights have violent ends. 
> --- by Shakespeare

![Dolores][Dolores]

Anyway, it works.
















