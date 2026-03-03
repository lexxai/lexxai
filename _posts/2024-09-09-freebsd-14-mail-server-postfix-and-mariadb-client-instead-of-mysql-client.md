---
layout: post
title: "FreeBSD 14. Mail server POSTFIX and mariadb-client instead of mysql-client"
date: 2024-09-09 14:22:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/09/freebsd-14-mail-server-postfix-and.html
---

Маю операційну систему FreeBSD 14.1-RELEASE у віртуальному середовищі Proxmox VE.

Щойно оновив поштовий сервер з FreeBSD 13.1-RELEASE, і з'ясувалося що я не можу тепер встановити POSTFIX та mariadb-client одночасно, як це було раніше. Тому це нотатка мені як я  розв'язав цю проблему, щоб не наступати на ті самі граблі двічі.

Коли встановлено у Вас *mariadb-server* та *mariadb-client* на одному сервері, то при встановленні поштового сервера *postfix* як пакунок через *pkg install postfix-mysql*, або з портів з опцією *MySQL*.

|  |
| --- |
|  |
| postfix freebsd port, mysql option |

  
 Вам буде пропоновано видалити *mariadb-server* та mariadb-client і встановити *mysql-client*.

|  |
| --- |
|  |
| видалити mariadb-server та mariadb-client і встановити mysql-client |

У такому випадку необхідно зробити dump до sql з mariadb server встановити *mysql-server*, *mysql-client*, та імпортувати базу з dump до нововстановленого *mysql-server* і потім встановити *postfix-mysql*.

Але якщо мені потрібно зберегти *mariadb-server*, то треба використовувати збирання *postfix* з портів, але сталося не все як хотілося.

|  |
| --- |
|  |
| Збирання postfix з портів з MySQL підтримкою. |

Як видно на зображені, не знайдено бібліотеки *===>   postfix-3.9.0\_1,1 depends on shared library: libmysqlclient.so.21 - not found*, і за допомоги файлу *Makefile* починають збирати відсутній пакунок  */usr/ports/databases/mysql80-client*.  

Для розв'язання проблеми я створив символічне посилання для цього файлу що не існує  *libmysqlclient.so.21*.

|  |
| --- |
|  |
| ln -s /usr/local/lib/mysql/libmysqlclient.so /usr/local/lib/mysql/libmysqlclient.so.21 |

|  |
| --- |
|  |
| Successful building of postfix 3.9 with mariadb1011-client-10.11.8\_1 |
