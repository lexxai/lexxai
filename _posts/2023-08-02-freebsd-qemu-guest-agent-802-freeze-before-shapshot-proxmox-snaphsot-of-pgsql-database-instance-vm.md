---
layout: post
title: "FreeBSD qemu-guest-agent 8.0.2 freeze before shapshot. Proxmox snaphsot of pgsql database instance VM."
date: 2023-08-02 00:57:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/08/freebsd-qemu-guest-agent-802-freeze.html
---

Задача - коректно зберігати базу даних при створені знімку віртуальної машини в Proxmox.

Що є:

Proxmox VE, VM з сервером FreeBSD 13, база даних postgresql.

### Виконання:

Встановлюю qemu-guest-agent.

```
pkg version -x qemu-guest-agent  
qemu-guest-agent-8.0.2 
```

Налаштовую qemu-guest-agent:

/etc/rc.d

```
qemu_guest_agent_enable="YES"  
qemu_guest_agent_flags="-d -v --fsfreeze-hook" 
```

### Скрипт для оброки freeze-hook

[scripts/qemu-guest-agent/fsfreeze-hook](https://raw.githubusercontent.com/qemu/qemu/master/scripts/qemu-guest-agent/fsfreeze-hook)

```
#!/bin/sh

# This script is executed when a guest agent receives fsfreeze-freeze and
# fsfreeze-thaw command, if it is specified in --fsfreeze-hook (-F)
# option of qemu-ga or placed in default path (/etc/qemu/fsfreeze-hook).
# When the agent receives fsfreeze-freeze request, this script is issued with
# "freeze" argument before the filesystem is frozen. And for fsfreeze-thaw
# request, it is issued with "thaw" argument after filesystem is thawed.

LOGFILE=/var/log/qga-fsfreeze-hook.log
FSFREEZE_D=$(dirname -- "$0")/fsfreeze-hook.d

# Check whether file $1 is a backup or rpm-generated file and should be ignored
is_ignored_file() {
    case "$1" in
        *~ | *.bak | *.orig | *.rpmnew | *.rpmorig | *.rpmsave | *.sample | *.dpkg-old | *.dpkg-new | *.dpkg-tmp | *.dpkg-dist | *.dpkg-bak | *.dpkg-backup | *.dpkg-remove)
            return 0 ;;
    esac
    return 1
}

# Iterate executables in directory "fsfreeze-hook.d" with the specified args
[ ! -d "$FSFREEZE_D" ] && exit 0
for file in "$FSFREEZE_D"/* ; do
    is_ignored_file "$file" && continue
    [ -x "$file" ] || continue
    printf "$(date): execute $file $@\n" >>$LOGFILE
    "$file" "$@" >>$LOGFILE 2>&1
    STATUS=$?
    printf "$(date): $file finished with status=$STATUS\n" >>$LOGFILE
done

exit 0
```

/usr/local/etc/qemu/fsfreeze-hook.d/pg-flush.sh

```
#!/bin/sh

# Flush PGSQL tables to the disk before the filesystem is frozen.
# At the same time, this keeps a read lock in order to avoid write accesses
# from the other clients until the filesystem is thawed.

case "$1" in
    freeze)
        logger -t pg-flush "freeze pg before stop service"
        service postgresql stop
        sync
        logger -t pg-flush "freeze pg after sync"
        ;;

    thaw)
        logger -t pg-flush "thaw pg starting service"
        service postgresql start
        logger -t pg-flush "thaw pg service `service postgresql status`"  
        ;;

    *)
        exit 1
        ;;
esac 
```

|  |
| --- |
|  |
| Proxmox snapshot VM |

tail -f /var/log/messages

```
Aug  1 17:39:24 pleroma pg-flush[40320]: freeze pg before stop service
Aug  1 17:39:24 pleroma postgres[29734]: [10-1] 2023-08-01 17:39:24.116 [29734] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[29739]: [10-1] 2023-08-01 17:39:24.116 [29739] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[29736]: [10-1] 2023-08-01 17:39:24.116 [29736] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[29733]: [10-1] 2023-08-01 17:39:24.116 [29733] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[29738]: [10-1] 2023-08-01 17:39:24.116 [29738] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[29730]: [10-1] 2023-08-01 17:39:24.116 [29730] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[29735]: [10-1] 2023-08-01 17:39:24.116 [29735] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[29737]: [10-1] 2023-08-01 17:39:24.116 [29737] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[29731]: [10-1] 2023-08-01 17:39:24.116 [29731] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[16588]: [10-1] 2023-08-01 17:39:24.116 [16588] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[37548]: [10-1] 2023-08-01 17:39:24.116 [37548] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[47096]: [10-1] 2023-08-01 17:39:24.116 [47096] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[29742]: [10-1] 2023-08-01 17:39:24.116 [29742] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[47073]: [10-1] 2023-08-01 17:39:24.116 [47073] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma postgres[47076]: [10-1] 2023-08-01 17:39:24.116 [47076] FATAL:  terminating connection due to administrator command
Aug  1 17:39:24 pleroma pg-flush[40334]: freeze pg after sync
Aug  1 17:39:24 pleroma pg-flush[40343]: thaw pg starting service
Aug  1 17:39:24 pleroma pg-flush[40365]: thaw pg service pg_ctl: server is running (PID: 41234) /usr/local/bin/postgres "-D" "/var/db/postgres/data12"
```
