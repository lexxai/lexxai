---
layout: post
title: "Скрипт автоматизації перейменування віртуальних машин у Proxomos VE для приєднання до кластеру"
date: 2023-01-03 23:08:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/01/proxomos-ve.html
---

[![](/assets/images/blog/Screenshot%202023-01-04%20002424.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjoApk02OJO5A5-N9faNK9c22l3Z7Sfwje8arSASKV0X0XKzUslu80qtQxAImqE75sAMxdE0ZsgGYkR4aElICL4QEQwkMNwE_MLjMX5un4y-SM2PxVNmx6ILiu2ZMuqn-h6irinUmw44DVYqzOfnp7JUhpHOrz67mu2zpJb_GeHYkhYyWkWwwCtH4GYUQ/s253/Screenshot%202023-01-04%20002424.png)

  

При переносі віртуальних машин з ноду в кластер необхідно мати пустий нод, і номера віртуальних машин що не конфліктують з іншими.

Для вирішення цієї задачі простіше і надійніше використати метод створення резервної копії і потім її відновлення в кластері.

Але я створив малий скрип що допомагає обійтися без цього методу.

Тому цей простий скрип перейменовує гуртом віртуальні машини з номерами 1ХХ на 6ХХ і де дані зберігаються на ZFS диску з назвою 'mdata'.

  

```
#!/usr/bin/env bash  
  
old=1  
new=6  
zfspool=mdata  
  
#rename zfpool disks of vm  
for i in $( zfs list -o name | grep "${zfspool}/vm-${old}" ); do  
  r=$(echo $i | sed "s/vm-${old}/vm-${new}/")  
  echo " zfs rename $i $r"  
  zfs rename $i $r  
done  
  
#rename disks on vm conf  
sed -i s/${zfspool}:vm-${old}/${zfspool}:vm-${new}/g /etc/pve/qemu-server/${old}*.conf  
  
  
#rename conf of vm  
for i in $( ls -1 /etc/pve/qemu-server/${old}*.conf); do  
  r=$(echo $i | sed -E "s/${old}(.*)\.conf/${new}\1\.conf/" )  
  echo "mv $i $r"  
  mv $i $r  
done  
       
```

Після операцій перейменувань переносимо файли конфігурацій з /etc/pve/qemu-server до резервної теки в домашньому каталозі /root/backup/vmconfigs.

|  |
| --- |
|  |
| Очищений нод ns26 у Proxmox VE |

Далі створюється кластер  на базі іншого чистого ноду  ns25. Де використовується два мережеві інтерфейси один для керування VM (10.5.0.25) інший для операцій в кластері (10.10.10.25).

P.S. Так як мережева карта з чотирьох портів, то окремий інтерфейс також використовується для самих віртуальних машин, що дає ізоляцію панелі керування від адресного простору віртуальних машин.

|  |
| --- |
|  |
| ns25 у ново створеному кластері |

Додається до новоствореного кластеру новий нод ns26.

|  |
| --- |
|  |
| ns26 додається до ново створеного кластеру |

  
 Кластер з двох нодів.  

|  |
| --- |
|  |
| Кластер з ns25 та ns26 |

  

Далі переписую назад файли конфігурацій віртуальних машин в ноді ns26 з /root/backup/vmconfigs до /etc/pve/qemu-server. Все - віртуальні машини відновлено у кластері без використання методу backup - restore.

|  |
| --- |
|  |
| Кластер з відновленими ВМ. |

  

|  |
| --- |
|  |
| Приклад запущеної ВМ з ноду ns26 в кластері |

  

Послідовно додаю інші ноди до кластеру, так як мінімум бажано мати 3 ноди у кластері для створення робочого кворуму.

P.S. Допомога з видаленням ноду з кластеру:

```
systemctl stop pve-cluster  
systemctl stop corosync  
  
pmxcfs -l  
  
rm /etc/pve/corosync.conf  
rm -r /etc/corosync/*  
  
killall pmxcfs  
  
systemctl start pve-cluster  
  
pvecm delnode <node to delete>
```

```
# ^this^ failed which the doc states is expected... so I ran  
  
pvecm expected 1  
  
pvecm delnode <node2>
```

https://forum.proxmox.com/threads/proxmox-ve-6-removing-cluster-configuration.56259/
