---
layout: post
title: "Proxmos VE v.8.X and MB with VGA Matrox MGA G200eW WPCM450"
date: 2024-04-27 23:02:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/04/proxmos-8-and-mb-with-vga-matrox-mga.html
---

Зараз є проблема оновлення з 7 до 8 версій Proxmos у власників не нових
серверів з  VGA Matrox MGA G200eW WPCM450 -[є чорний екран](https://forum.proxmox.com/threads/black-screen-vga-on-proxmox-ve-8-1-installer-boot.137408/).

Собі щоб не забути зробив нотатки.

### Upgrade 7 to 8

Спочатку [етапи оновлення](https://pve.proxmox.com/wiki/Upgrade_from_7_to_8) але без перезавантаження.

```
lspci | grep VGA
08:01.0 VGA compatible controller: Matrox Electronics Systems Ltd. MGA G200eW WPCM450 (rev 0a)
```

Основні зміни в /etc/default/grub:

```
GRUB CMDLINE LINUX="nomodeset"
GRUB TERMINAL=console
GRUB GFXMODE=nomodeset
```

|  |
| --- |
|  |
| /etc/default/grub |

```
grub-update
reboot
```

|  |
| --- |
|  |
| PROXMOX VE 8.2.2 |

### Install ISO

### 

[![](/assets/images/blog/Screenshot%202024-04-28%20185520.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhUL8KiC7rJn9HfXKijWmHV-pPebGpXUwefv59BkT0NDA6ImSCLx_YDZ9f8eGnGUUhDAn8c_qEdJ2jf9zQwTk59f2DoTKtnW_ajT9DeZ50bslRjRpQaRgg6KcshDs_1nVINx5I5p7_gq5WDgAM5a6yttkKou3p8_bCmLOQYkVG6iBXDDFER2CVAbMwbg9MD/s1440/Screenshot%202024-04-28%20185520.png)
