---
layout: post
title: "Нотатка. Монтування дисків NFS, ZFS поверх iSCSI. FreeBSD rc.d."
date: 2023-05-12 23:17:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/05/nfs-zfs-iscsi-freebsd-rcd.html
---

Є підключення диска через iSCSI, надалі диск форматується у ZFS. При
завантаженні системи потрібно зачекати мережевого з'єднання iSCSI Initiatior з
iSCSI target. Потім, змотувати ZFS pool. Потім запустити hook скрипт, після
успішного монтування.

|  |
| --- |
| [source imgage : https://www.data-storage.uk](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhclJQXrRXtuUc6TSZiORuuabjtiiSJhbjMW8Rj7BKm_EcYcAQVfxOo1lIM1O2RFAmCEXwypOMPkLhrOWutI6HigmoLs4cfsI31n5cymx9sbEY3gVNRN5WxA3O3ecalXuTirVsXGZ8JlXoITFggtYhRef4q15POgV4PdXD0y_OAJFtTptN0ApycTcmizw/s900/iscsi-storage.jpg) |
| iSCSI Storage |

#### iSCSI

/etc/rc.conf:

```
#iSCSI
iscsid_enable="YES"
iscsictl_enable="YES"
iscsictl_flags="-Aa"
```

/etc/iscsi.conf:

```
t0 {
    TargetAddress   = nas1
    TargetName      = iqn.2023-05.net.server.nas1:server1-storage
    AuthMethod      = CHAP
    chapIName       = server1
    chapSecret      = "kdwkjdqhdww3qq@@6td3_3eiw9e7w43"
}
```

#### ZFS:

/etc/rc.conf:

```
#iSCSI_ZFS
iscsi_zfs_enable="YES"
iscsi_zfs_wait_iqn="server1-storage"
iscsi_zfs_wait_pool="storage"
iscsi_zfs_post_hook="/root/script/post-mount.sh"
iscsi_zfs_pre_hook="/root/script/pre-umount.sh"
```

Файл
[/usr/local/etc/rc.d/iscsi-zfs](https://gist.github.com/lexxai/b9017d2c1d6e7a9b2c2ed24464b1c297)
:

#### NFS:

/etc/rc.conf:

```
#NFS
rpcbind_enable="YES"
nfs_client_enable="YES"
rpc_lockd_enable="YES"
rpc_statd_enable="YES"
```

/etc/fstab:

```
# Device<-->Mountpoint<>FStype<>Options>Dump<-->Pass#
# ...
nfs-server:/storage-raid /mnt/raid  nfs rw,bg  0   0
```

Опція -o bg, дозволяє при монтуванні у /etc/rc.d/mountcritremote у випадку помилки мережі, не зависати серверу.
І тоді віддалено не зайдеш.

А так з bg, монтування продовжиться у фоновому процесі, а завантаження системи піде далі.
