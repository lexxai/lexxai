---
layout: post
title: "Proxmox VE, firewall, nf_conntrack_max, nf_conntrack_count"
date: 2025-03-04 16:57:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2025/03/proxmox-ve-firewall-nfconntrackmax.html
---

Є Hypervisor, Proxmox VE з ввімкненим firewall на рівні Proxmox, через те, що запущені VM з Proxy сервером "Squid", і бажано обмежити доступ до потенційного доступу до локальної мережі.

Але за великої кількості підключень спрацьовують  обмеження на кількість одночасних сесій підключень.

|  |
| --- |
|  |
| Proxmos VE, Shell, dmesg |

Для стандартних рішень з Ubuntu є змінна файлу з додавань певних рядків у  /etc/sysctl.conf, але усі значення перевизначаються і бачимо постійно значення за замовчуванням.

|  |
| --- |
|  |
| Not help, /etc/sysctl.conf |

Але знайшов давній пост де є пропозиція використати налаштування GUI Proxmox. І це допомогло.

|  |
| --- |
|  |
| Proxmox VE, GUI, Node, Firewall. Options. nf\_conntrack\_max |

Моніторинг значень - nf\_conntrack\_count: "watch -n 1 "cat /proc/sys/net/netfilter/nf\_conntrack\_count"

|  |
| --- |
|  |
| watch -n 1 "cat /proc/sys/net/netfilter/nf\_conntrack\_count" |
