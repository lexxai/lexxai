---
layout: post
title: "Windows Docker. Virtual image file just only grow size of .vhds file."
date: 2026-01-30 15:17:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2026/01/windows-docker-virtual-image-file-just.html
---

### Куди зникає місце на диску?

Docker у Windows працює через WSL (Windows Subsystem for Linux), яка в свою чергу використовує віртуалізацію Hyper‑V. Це означає, що всі дані зберігаються у файлах образів віртуальних дисків .vhdx.

З часом диск починає стрімко розростатися, і рано чи пізно місце на системному SSD закінчується. Тоді виникає питання: куди ж воно поділося?

На допомогу приходить утиліта WinTree (diskanalyzer.com), яка дозволяє швидко побачити, що саме займає простір.

Саме так я й з’ясував, що проблема була у Docker. Але коли перевірив сам Docker - там усе вже очищено, а файл .vhdx продовжував залишатися гігантським.

```
docker system df

TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          2         2         329.2MB   0B (0%)
Containers      3         0         2B        2B (100%)
Local Volumes   62        1         2.309GB   2.125GB (92%)
Build Cache     0         0         0B
```

### Чому VHDX росте безконтрольно.

Реальний розмір віртуального тому завжди більший за фактичні дані. Це відбувається тому, що .vhdx‑файл у WSL та Docker може лише збільшуватися у міру потреби.

Коли ти видаляєш образи чи очищаєш дані, простір всередині Linux‑файлової системи звільняється, але сам файл на диску не стискається автоматично.

Тому він продовжує залишатися гігантським, навіть якщо даних там майже немає.

|  |
| --- |
|  |
| Docker .vhdx before compact |

### Стискаємо образ диску

Виходимо з застосунку "Docker Desktop" та робимо завершення "WSL":

```
wsl --shutdown
```

Стискаємо VHD файл відкриваючи консоль PowerShell як Administrator.

```
Optimize-VHD -Path "$env:LOCALAPPDATA\Docker\wsl\disk\docker_data.vhdx" -Mode Full
```

Тепер набагато краще.

|  |
| --- |
|  |
| Docker .vhdx after compact |
