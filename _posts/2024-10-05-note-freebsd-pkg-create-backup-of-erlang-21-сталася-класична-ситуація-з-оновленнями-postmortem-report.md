---
layout: post
title: "Note. FreeBSD. pkg create. Backup of erlang-21. Сталася класична ситуація з оновленнями. Postmortem Report."
date: 2024-10-05 10:29:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/10/note-freebsd-pkg-create-backup-of.html
---

### Умови:

Є віртуальна машина з [FreeBSD](https://www.freebsd.org/) під керуванням [Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview).

Є застосунок '[pleroma](https://pleroma.social/)' котрий працює на базі
[erlang](https://www.erlang.org/).

### Причини:

Для підтримання інфраструктури операційна система оновлюється і пакунки
підтримуються в актуальному стані.  
Застосунок працює, в пам'яті, все
нормально. Користувачі заносять нові дані...

  

### Дії:

Snapshot VM with RAM
state - 'before\_upgrade'.  
Сервісне оновлення 'pkg upgrade'

```
Oct 1 08:51:24 pleroma pkg[25416]: erlang upgraded: 21.3.8.24_1,4 -> 25.3.2.12_1,4
Oct 1 08:52:37 pleroma pkg[25416]: elixir upgraded: 1.11.3 -> 1.16.3
```

Застосунок працює.

Вночі - сервісний перезапуск системи. Попередньо зроблено знімок віртуальної машини зі станом: Snapshot VM with
RAM state - 'before\_reboot'.

### Проблеми:

І ось після перезапуску системи - вітаннячка: pleroma  -  не
запускається вже під 'erlang -25'.

```
Error:

==> prometheus_ex
Compiling 19 files (.ex)

== Compilation error in file lib/prometheus/buckets.ex ==
** (UndefinedFunctionError) function Kernel.Utils.defdelegate/2 is undefined or private. Did you mean:

      * defdelegate_all/3
      * defdelegate_each/2

    (elixir 1.16.3) Kernel.Utils.defdelegate({:new, [line: 18, column: 12], [{:arg, [line: 18, column: 16], nil}]}, [])
    lib/prometheus/buckets.ex:18: (module)
could not compile dependency :prometheus_ex, "mix compile" failed. Errors may have been logged above. You can recompile this dependency with "mix deps.compile prometheus_ex --force", update it with "mix deps.update prometheus_ex" or clean it with "mix deps.clean prometheus_ex"
```

Ігри з "*prometheus\_ex*" - не допомагають. Оновити '[pleroma](https://pleroma.social/)' наразі не можливо, додаткові налаштування краще не чіпати зараз.

|  |
| --- |
|  |
| Proxmox VE. Disconnect netwok interface. |

  

### Рішення:

1. Відновити "erlang-21", але є, але pkg не присутній вже в репозиторії версії "FreeBSD 13.4". В архівах швидко не знайшов пакунків. З ports знайшов архівні знімки, але з них не компілюється успішно.
   Відкладено.
2. Відновлюю Snapshot VM with RAM state - 'before\_reboot'. Допомагає.
   Система знову працює. Але працює поки не перезавантажити. Через те, що вона
   тільки в пам'яті з запущеним erlang-21/elixir-1.11.. Відновитися до "before\_upgrade" не можу, з'явилася нова інформація.
3. Відновлю Snapshot VM with RAM state - 'before\_upgrade' у якості нової
   віртуальної машини 201. Через:

   ```
   qm disk-detach 201 scsi0
   zfs send storage/vm101-disk-0@before_upgrade | zfs receive -f storage/vm201-disk-0
   qm set 201 --scsi0  storage:vm-201-disk-0
   ```
4. Disconnect netwok interface віртуальної машини 201. Запуск віртуальної машини 201: qm start 201.
5. В консолі віртуальної машини 201 змінюю налаштування тимчасової IP адреси, щоб не було дублікатів. Connect netwok interface. Піддуюся SSH консоллю. Тут є ще пакунки: elixir-1.11.3 erlang-21.3.8.24\_1,4 erlang-man-21.3\_2.
6. Створення резервних пакунків erlang-21 та elixir-1.11 з робочої системи:

   ```
   $ mkdir backup_pkg
   $ pkg info -r erlang
   erlang-21.3.8.24_1,4:
           elixir-1.11.3
   $ pkg create -o backup_pkg erlang 
   $ pkg create -o backup_pkg elixir
   $ pkg create -o backup_pkg erlang-man
   $ ls -lh backup_pkg/
   total 40576
   -rw-r--r--  1 root  wheel   5.1M Oct  2 15:58 elixir-1.11.3.pkg
   -rw-r--r--  1 root  wheel    33M Oct  2 15:58 erlang-21.3.8.24_1,4.pkg
   -rw-r--r--  1 root  wheel   1.0M Oct  2 15:58 erlang-man-21.3_2.pkg
   ```
7. Перенесення файлів 'backup\_pkg/\*.pkg' на робочу віртуальну машину 101 через 'nfs' або через 'scp'.
8. В консолі віртуальної машини 101 видаляю версії "erlang-25.3.2.12\_1,4", "elixir-1.16.3", та встановлюю щойно завантаженні.

   ```
   $ pkg install erlang-21.3.8.24_1,4.pkg erlang-man-21.3_2.pkg elixir-1.11.3.pkg
   ```
9. Перезапуск застосунку і перевірка роботи, все працює.
10. Блокування пакунків від майбутнього оновлення до розв'язування питання з оновленням самого застосунку.  

    ```
    $ pkg lock erlang-21.3.8.24_1,4.pkg erlang-man-21.3_2.pkg elixir-1.11.3.pkg
    ```

### Висновок.

На production це робити - гріх 😀  
Рознести дані та застосунок за різним мікросервісам: Postgress database VM. NFS сервер для зображень.  
Знімки стану віртуальних машин перед будь-якими важливим діями, має бути правило. Дяка ZFS що робить це дуже швидко.
