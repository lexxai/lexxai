---
layout: post
title: "Нотатка: IPMI Watchdog timer, скидання та встановлення через ipmitool"
date: 2023-04-04 20:05:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/04/ipmi-watchdog-timer-ipmitool.html
---

Сучасні серверні материнські плати мають можливість контролювати свій "живий" стан через спеціальний сторожовий таймер (Watchdog timer).

|  |
| --- |
|  |
| Налаштування Watchdog Timer у BIOS. |

|  |
| --- |
|  |
| WatchDog Timer HP server |

Принцип його простий - запускається таймер з певним початковим значенням, на рівні BIOS спеціальної материнської плати (BMC), і  якщо значення таймеру досягне значення 0 сек., тоді виконається певна дія.   
Ці  дії можуть бути: перезавантажити материнську плату - reset, вимкнути живлення - power off,  або вимкнути та ввімкнути живлення послідовно - power cycle.

#### Налаштування

Є старенький сервер HP ProLiant DL380e Gen8 з iLO 4 (HPE Integrated Lights-Out 4).

|  |
| --- |
|  |
| iLO 4 ProLiant HP server |

На сервері встановлено систему 'Proxmox'. Тому я можу виконувати в консолі unix команди, та встановлювати пакунки.

[Встановлюю: ipmitool за допомогою apt](https://advantech-ncg.zendesk.com/hc/en-us/articles/360033313052-How-to-Install-IPMItool).

Тепер можу прочитати поточні значення сторожового таймера:

```
# ipmitool mc watchdog get
Watchdog Timer Use:     SMS/OS (0x04)
Watchdog Timer Is:      Stopped
Watchdog Timer Actions: No action (0x00)
Pre-timeout interval:   0 seconds
Timer Expiration Flags: 0x00
Initial Countdown:      300 sec
Present Countdown:      0 sec
```

Щоб встановити дію та нові значення для таймера з консолі, можна скористатися публікацією: [How to use ipmitool command to set BMC watchdog timer ?](https://advantech-ncg.zendesk.com/hc/en-us/articles/360028285872-How-to-use-ipmitool-command-to-set-BMC-watchdog-timer-)

```
# ipmitool raw 0x06 0x24 0x04 0x01 0x00 0x00 0x70 0x17
Watchdog Timer Use:     SMS/OS (0x04)
Watchdog Timer Is:      Stopped
Watchdog Timer Actions: Hard Reset (0x01)
Pre-timeout interval:   0 seconds
Timer Expiration Flags: 0x00
Initial Countdown:      600 sec
Present Countdown:      0 sec
```

Де # ipmitool raw 0x06 0x24 0xWW 0xXX 0x00 0x00 0xYY 0xZZ  
0xXX: WDT action mode

|  |
| --- |
|  |
| WDT action |

0xYY 0xZZ: Встановити початкові значення для сторожового таймера BMC

|  |
| --- |
|  |
| WDT Timer |

Timer проміжок від 0.1(0X0001) до 6553.5(0XFFFF) секунд, де одиниця виміру - 0.1 секунди.   
600 sec: 6000 DEC (0.1 sec) = 1770 HEX = 0x70 0x17

#### Приклад роботи

Таким чином команда, налаштує таймер на 10 хвилин, і з дією перезапустити сервер.

```
ipmitool raw 0x06 0x24 0x04 0x01 0x00 0x00 0x70 0x17
```

Для того щоб показати що сервер працює за допомогою cron ми раз на хвилину запускаємо команду по скиданню таймера до початкових значень та перезапуску.

```
# ipmitool mc watchdog reset  
# ipmitool mc watchdog get  
Watchdog Timer Use:     SMS/OS (0x44)  
Watchdog Timer Is:      Started/Running  
Watchdog Timer Actions: Hard Reset (0x01)  
Pre-timeout interval:   0 seconds  
Timer Expiration Flags: 0x00  
Initial Countdown:      600 sec  
Present Countdown:      592 sec
```

Таким чином бачимо що таймер запущений і залишилось 592 секунди до перезавантаження сервера, якщо таймер не скинути.
