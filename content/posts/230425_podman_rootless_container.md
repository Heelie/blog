---
title: "关于AlmaLinux WSL运行使用systemctl --user的问题"
subtitle: ""
date: 2023-04-25T20:23:17+08:00
draft: false
author: "Jay"
authorLink: "https://github.com/Heelie"
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: true
weight: 0

tags:
  - WSL
  - Podman
categories:
  - Podman

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: [ ]

repost:
  enable: false
  url: ""
---

## 起因
设置podman无根容器开机自启，已经创建好了service文件，但是使用systemctl --user daemon-reload的时候报错
```shell
$ ll ~/.config/systemd/user
total 4.0K
-rw-r--r-- 1 jay jay 874 Apr 24 20:46 nginx.service
$ systemctl --user daemon-reload
Failed to connect to bus: No such file or directory
```

然后大概想了一下，以为是因为不是ssh登陆导致的，然后 `su -` 切到root，再 `ssh jay@localhost` 结果还是报错

```shell
$ systemctl --user daemon-reload
Failed to connect to bus: No medium found
```

## 原因
最后Google了好久，找到原因是因为没有启systemd-logind.service，直接启动
```shell
$ sudo systemctl start systemd-logind.service
Failed to start systemd-logind.service: Unit systemd-logind.service is masked.
```
还被禁用了！不知道为什么。直接解除禁用
```shell
$ sudo systemctl unmask systemd-logind.service
Removed "/etc/systemd/system/systemd-logind.service".
```
切回Windows的powershell，关机wsl
```powershell
wsl --shutdown
```
再次打开wsl，执行systemctl --user daemon-reload就好了

如果报下面的错就是需要用ssh来登录用户即可
```shell
$ systemctl --user daemon-reload
Failed to connect to bus: Permission denied
```
