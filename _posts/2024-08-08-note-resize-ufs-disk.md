---
layout: post
title: "Note. Resize UFS Disk"
date: 2024-08-08 19:29:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/08/note-resize-ufs-disk.html
---

OS: FreeBSD.   
Disk: GPT, UFS.

Proxmox increase live disk of VM.

```
gpart commit vtbd1
gpart show vtbd1
gpart resize -i 1 vtbd1
gpart show vtbd1
df -h
growfs /dev/gpt/cloud-data
df -h
```
