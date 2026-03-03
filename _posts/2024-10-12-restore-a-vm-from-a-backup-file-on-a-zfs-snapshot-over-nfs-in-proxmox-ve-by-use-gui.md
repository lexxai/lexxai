---
layout: post
title: "Restore a VM from a backup file on a ZFS snapshot over NFS in Proxmox VE by use GUI."
date: 2024-10-12 14:19:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/10/restore-vm-from-backup-file-on-zfs.html
---

### Умови:

Proxmox VE server  періодично робить резервні копії до NFS спільної теки котра розміщена на NAS сервері.

NAS це TrueNAS сервер що зберігає дані у ZFS файловій системі. TrueNAS автоматично налаштований робити періодичні знімки (snapshots) ZFS dataset де розміщенні дані для NFS теки.

У VM сервера знайдено підозрілі файли за назвою "." розміром 1024 bytes з бінарним вмістом. Необхідно провести аналіз в "offline copy" сервера.

### Задача:

Відновити віртуальну машину з попередньої резервної копії.

### Рішення:

Під'єднати попередні періодичні знімки (snapshots) ZFS dataset, що створені на стороні NAS сервера, для відновлення віртуальної машини з її резервної копії.

## Підключення до консолі Proxmox Node.

* Proxmox node: ns21
* NFS share: nfs-ns58-10g
* Target data: 2024-10-11
* Target VM ID: 150

|  |
| --- |
|  |
| Proxmox console. |

ZFS snapshots зберігаються за замовчуванням у прихованій теці ".zfs".   

Знайдено цільовий шлях де зберігаються потрібні резервні копії: /mnt/pve/nfs-ns58-10g/.zfs/snapshot/auto-2024-10-11\_00-00

## Створення Datacenter STORAGE (Directory).

|  |
| --- |
|  |
| Directory 'nfssnap' from premounted NFS folder on ns21. |

  

|  |
| --- |
|  |
| Directory 'nfssnap' on Datacenter of cluster |

## Пошук резервної копії у 'nfssnap'.

|  |
| --- |
|  |
| Backup file for VM ID 150. |

## Відновлення VM 150.

|  |
| --- |
|  |
| Start of restore |

  

|  |
| --- |
|  |
| Finish of restore |

  

## Аналіз VM.

[![](/assets/images/blog/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-12%20171542.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh4R_-6F5fyzJbkXkZn8ukI_BHBVA9pFd6MMKKSiDPorfe-G0E3NlZJyKg_JbWmULXr7pSp_6Dz8253V0Y9LUaS65YpRlZwTT2cHKjk8FNwf018JsnSmNePWZ9ydpKgesZF-uh2-KjbkJ9CIYkN-7oVsg7K89mlVo2DThp-Q__11W-RtgmCbPmpfguYtTt5/s372/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-10-12%20171542.png)

Не забуваємо видалити тимчасове storage 'nfssnap'
