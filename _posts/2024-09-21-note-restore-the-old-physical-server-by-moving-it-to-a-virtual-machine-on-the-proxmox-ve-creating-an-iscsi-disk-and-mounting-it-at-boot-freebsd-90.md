---
layout: post
title: "Note. Restore the old physical server by moving it to a virtual machine on the Proxmox VE. Creating an iSCSI disk and mounting it at boot. FreeBSD 9.0."
date: 2024-09-21 16:41:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/09/note-restore-old-physical-server-by.html
---

Примітка. Відновлення старого фізичного сервера, перемістивши його на
віртуальну машину на [Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview). Створення [iSCSI](https://uk.wikipedia.org/wiki/ISCSI) диску та його монтування при
завантажені операційної системи. [FreeBSD 9.0](https://www.freebsd.org/releases/9.0R/announce/).

#### Історія

Основне правило адміністратора, працює не чіпай. Призводить то того що рано чи
пізно це вилазить боком. На кшталт не можеш нічого оновити та встановити.

Тому мені історично діставався цей сервер котрий треба перемістити до
віртуального середовища поки "/raid1" масив ще працює після нового його збою,
але варто зауважити міцний горішок.

Є декілька способів як перемістити систему з фізичного до віртуального. Але я
не хотів переносити усі накопиченні проблеми, і вирішив перенести все окрім
ядра та базової системи використовуючи    [rsync](https://uk.wikipedia.org/wiki/Rsync).

#### До віртуальної машини

Сервер ще живий, операційна система  FreeBSD 9.0. Тому я створив з
 [архівного ISO](http://ftp-archive.freebsd.org/pub/FreeBSD-Archive/old-releases/amd64/9.0-RELEASE/) чисту віртуальну машину.

FreeBSD 9.0 (amd64): Default (i440fx),
CPU (x86-64-v2-AES),  SCSI Controler LSI 53C895A, SCSI disk writeback
cache, Ethernet Intel E1000E.

Ось тільки так операційна система
завантажилася.

[![](/assets/images/blog/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-21%20%D0%BE%2017.40.48.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg0LxOtfM0a5eoV3m4jCTCmpq_OtfnCIyD-v69I9GjgjKmmdRdTAVV9ByD_PDQd6etfYuHhohZIEHseooqfpPPw_jizV6Q9ULqyV1qWieor8QqSFwPPrkWY8VeqqsDgycsN_ExfKs2X6NFkcXgSCGQwgeR9TbrAq1sD6SFxZaiN8y_H11a9Y8fXpj3gRLC4/s1578/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-09-21%20%D0%BE%2017.40.48.png)

Два мережевих інтерфейси для того, щоб основний інтерфейс вести в офлайн (link\_down=1),
а інший використати для підключення через SSH (ecdsa-sha2-nistp256) та підключення [iSCSI](https://uk.wikipedia.org/wiki/ISCSI)
диска.

#### Адаптуємо віртуальну машину

Bridge - vmbr0 має в налаштуваннях [MTU](https://uk.wikipedia.org/wiki/MTU) 9000.  Тому й у віртуальній машині
використано для мережевого адаптера MTU 9000 для ефективності [iSCSI](https://uk.wikipedia.org/wiki/ISCSI) через те,
що фізична мережа у гіпервізора [Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview)  - 10 Гб/c і це дало приріст
швидкості у два рази.

Додатково вимкнув перевірку контрольних сум мережевим адаптером у віртуальному середовищі:

```
ifconfig_em1_ipv6="inet6 XXXX:YYYY::101/64 mtu 9000 -txcsum -rxcsum -lro -tso"
```

Операційна система при завантажені перевіряла пам'ять і це робило певну
затримку відключив це

```
boot/loader.conf : hw.memtest.tests=0
```

#### iSCSI

Для можливості приміщення віртуальної машини по [кластеру Proxmox](https://pve.proxmox.com/wiki/Cluster_Manager), прийнято рішення не створювати локальний диск розміром "1 TB" і робити його періодичну ZFS реплікацію на інші ноди кластеру, а створити мережевий [iSCSI](https://uk.wikipedia.org/wiki/ISCSI) диск на [NAS](https://uk.wikipedia.org/wiki/Network-attached_storage) сервері ([TrueNAS Scale](https://www.truenas.com/)) і змонтувати його у віртуальному середовищі.

|  |
| --- |
|  |
| TrueNAS Scale - iSCSI |

#### Конфігурація iSCSI в VM

Для підключення  iSCSI диска, завантажую модуль ядра iscsi\_initiator\_load. Додаткових програм я вже поставити не можу,  тому маю те що маю.

Перевірка що завантажується модуль:

```
kldload iscsi_initiator_load
```

Автоматичне завантаження:   

```
boot/loader.conf : iscsi_initiator_load=YES
```

Налаштування iSCSI ініціатора:

/etc/iscsi.conf:

```
raid1old50 {
    TargetAddress = nas-server.local
    TargetName = iqn.2005-10.org.freenas.ctl:raid1old50
}
```

Монтування при завантаженні:

Результат монтування:

```
$ cat /etc/fstab
# Device        Mountpoint      FStype  Options Dump    Pass#
/dev/da0p2      /               ufs     rw      1       1
/dev/da0p3      none            swap    sw      0       0
/dev/gpt/raid1 /raid1    ufs rw,noauto,noatime   0 0

$ mount
/dev/da0p2 on / (ufs, local, journaled soft-updates)
devfs on /dev (devfs, local, multilabel)
/dev/gpt/raid1 on /raid1 (ufs, local, noatime, journaled soft-updates)
```
