---
layout: post
title: "ladder on loong"
subtitle: ""
date: 2024-10-03
author: "Real_Roller"
header-img: "img/post-bg-2015.jpg"
tags: []
---

## 以防我忘记这个
由于clash-meta,clash-verge都是aur的私货,所以只能自己打
`bash
git clone https://aur.archlinux.org/clash-geoip.git
cd clash-geoip.git
makepkg -si
git clone https://aur.archlinux.org/clash-meta.git
cd clash-meta.git
makepkg -si
`
然后下载clash-verge的yaml和db文件,放到`~/.config/clash/`下,然后启动clash-meta即可
```bash
sudo clash-meta -d ~/.config/clash/ -f ~/.config/clash/config.yaml
```

- 补充

后来改用mihomo,和tailscale在重启之后不能兼容(两者均使用tun模式)

- 解决方法

```bash
systemctl stop mihomo tailscaled
systemctl start tailscaled
systemctl start mihomo
```
只需保证正确启动顺序即可(~~其实试了很多办法没有用~~)
