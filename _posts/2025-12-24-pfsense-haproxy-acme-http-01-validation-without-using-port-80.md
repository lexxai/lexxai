---
layout: post
title: "pfSense + HAProxy + ACME: HTTP-01 validation without using port 80"
date: 2025-12-24 11:09:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2025/12/pfsense-haproxy-acme-http-01-validation.html
---

#### Вступ

У багатьох інсталяціях [pfSense](https://www.pfsense.org/) використовується не лише як firewall, а і як
точка термінації HTTPS для внутрішніх вебсервісів. Типовий стек виглядає
так:

* pfSense - firewall та reverse-proxy
* [HAProxy](https://www.haproxy.org/) - маршрутизація HTTP/HTTPS трафіку
* [ACME (Let’s Encrypt)](https://letsencrypt.org/docs/client-options/) - автоматична генерація HTTPS сертифікатів
* Внутрішні вебсервери - працюють по HTTP у локальній мережі

У цій публікації як розгляну як коректно пройти ACME HTTP-01 валідацію, коли
порт 80 вже зайнятий HAProxy, без зупинки сервісів і без DNS-01.

[![](/assets/images/blog/intro.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh4RgHhDrXg0zyKKmCdk_qT6lfl5G4BO1_cJ9_OlpEsrG-MBq6b91pq5Bopfwu5SzyonxUsgNL9LwyED6I5m7iFEvHDVL9TWu3A4vYbiWvbSzvc3klnQ5Kao0KH4Js-d2Tlh7UF6JDc07yNO4ep8RHuzjNzwi8MYmFdXgDAlfP4f7bDZhcuufa4m9pu9r_5/s737/intro.png)

  

#### Вихідні умови

## Інфраструктура

* pfSense firewall
* У локальній мережі є декілька вебсерверів, яким потрібні HTTPS сертифікати
* Доступ до них забезпечує HAProxy, що працює безпосередньо у pfSense

### HAProxy

Прослуховує:

+ 80/tcp (HTTP)
+ 443/tcp (HTTPS)

Функції:

+ HTTP → HTTPS redirect
+ TLS termination
+ Маршрутизація за доменним іменем (ACL по Host)
+ Робота з HTTP backend (без TLS усередині LAN)

  

### ACME

* Сертифікати генеруються через: *Services → ACME → Certificates*
* Використовується **HTTP-01** validation
* Стандартний порт 80 недоступний, бо його вже використовує HAProxy

*Проблема***ACME HTTP-01 завжди звертається на порт 80:**

```
http://example.com/.well-known/acme-challenge/<token>
```

Але:

* порт 80 вже зайнятий HAProxy
* зупиняти HAProxy під час renew — погана ідея
* перенаправлення на інший порт (81, 8080) не підтримується
  **Let’s Encrypt**

#### Рішення: HAProxy як проксі для ACME standalone

|  |
| --- |
|  |
| Services Acme Certificate options |

#### 

### Ключова ідея

* HAProxy залишається на порту 80
* ACME під час валідації:

+ піднімає тимчасовий standalone webserver
+ на іншому локальному порту, наприклад 81

* HAProxy:

+ перехоплює лише запити до */.well-known/acme-challenge/*
+ проксіює їх на **127.0.0.1:81**

Let’s Encrypt нічого не знає про порт 81 — для нього все відбувається через
порт 80  
  

### ACME standalone webserver

Приклад команди, яку використовує pfSense ACME package під час валідації:
  

```
/usr/local/pkg/acme/acme.sh --issue \    
  --domain example.com \  
  --standalone --httpport 81 --listen-v4
```

Особливості:

* вебсервер тимчасовий
* порт 81 відкритий лише на час валідації
* доступний тільки локально

### Налаштування HAProxy

|  |
| --- |
|  |
| Services HAProxy Backend Edit |

#### 

### Backend для ACME

|  |
| --- |
|  |
| Services HAProxy Frontend Edit |

Створюємо окремий backend, наприклад **local\_acme**:   

* Address: 127.0.0.1
* Port: 81
* Mode: http
* Health checks - вимкнені

*Важливо: Standalone webserver не працює постійно. Якщо увімкнути health
check - backend завжди буде в статусі DOWN і HAProxy повертатиме
503.*

### ACL правило

У frontend для HTTP (port 80) додаємо ACL:   

```
path_beg /.well-known/acme-challenge/
```

### Routing rule

Перше правило у frontend:

```
if path_beg /.well-known/acme-challenge/ use_backend local_acme
```

Порядок правил критичний — ACME правило має бути перед redirect на
HTTPS.   
  

### Повний потік запиту

1. Let’s Encrypt → *http://example.com/.well-known/acme-challenge/...*
2. Запит приходить на pfSense, порт 80
3. HAProxy:
   * бачить *path\_beg /.well-known/acme-challenge/*
   * НЕ робить redirect
   * проксіює на 127.0.0.1:81
4. ACME standalone webserver віддає challenge
5. Сертифікат успішно згенеровано
6. Standalone webserver завершує роботу

### Переваги рішення

* без зупинки HAProxy
* без відкриття порту 81 у WAN
* без DNS-01
* повністю автоматичне renew
* production-ready схема

### Типові помилки

* redirect
  HTTP → HTTPS для ACME path
* увімкнені health checks у backend
* path rewrite або security filtering
* неправильний порядок ACL правил

#### Висновок

Ця схема
дозволяє коректно використовувати ACME HTTP-01 validation, навіть якщо порт 80
повністю зайнятий HAProxy.  
HAProxy виступає не як перешкода, а як тонкий маршрутизатор для ACME
challenge, що робить рішення стабільним і безпечним.

|  |
| --- |
|  |
|  |
