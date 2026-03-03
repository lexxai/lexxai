---
layout: post
title: "Free DNS he.net + Free SSL Certificates Let's Encrypt"
date: 2023-03-04 20:02:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/03/free-dns-henet-free-ssl-certificates.html
---

### dns.he.net

Якщо у Вас є домен в DNS службі <https://dns.he.net>, то можна оновлювати TXT записи через динамічний ключ, а це надає можливість отримати безкоштовні сертифікати через [Let's Encrypt](https://letsencrypt.org/) у випадку коли у Вас наприклад поштовий сервер.

Рішення що є pligin до [certbot](https://certbot.eff.org/) - [he.net DNS Authenticator plugin for Certbot](https://github.com/gentoo-root/certbot-dns-henet) потребує повний доступ через login/password до Вашого he.net акаунта, як на мене це жирно :)

Тому було знайдено рішення оновлення через динамічний ключ для оновлення тільки одного TXT запису.

|  |
| --- |
|  |
| he.net TXT |

Для отримання ключа його можна згенерувати, наприклад: aJoSWOFyLD1A3iDG

|  |
| --- |
|  |
| he.net dynamic key |

#### 

### Оновлення he.net

Далі створюю скрипт оновлення або очищення запису.

he-dns-update.sh

```
#!/bin/sh

# Do we have everything we need?

if [ -z "$CERTBOT_DOMAIN" ] || [ -z "$CERTBOT_VALIDATION" ]; then
    echo '$CERTBOT_DOMAIN and $CERTBOT_VALIDATION environment variables required.'
    exit 1
fi

# Add all HE TXT record DDNS keys to the txt_key object
# Remember to protect this script file - chmod 700!
case $CERTBOT_DOMAIN in
 domain)
  txt_key='aJoSWOFyLD1A3iDG'
 ;;
esac

if [  "$1" == "clean" ];then
 CERTBOT_VALIDATION=' '
fi


# Create a FQDN based on $CERTBOT_DOMAIN
HE_DOMAIN="_acme-challenge.$CERTBOT_DOMAIN"

# Update HE DNS record
curl -s -X POST "https://dyn.dns.he.net/nic/update" -d "hostname=$HE_DOMAIN" -d "password=${txt_key}" -d "txt=$CERTBOT_VALIDATION"

# Sleep to make sure the change has time to propagate over to DNS
sleep 30
```

Для захисту ключа змінено права доступу: chmod 700 he-dns-update.sh

Ваш ключ для оновляння домену: domain, фіксований і є  txt\_key='aJoSWOFyLD1A3iDG'. Змініть на свої значення.

### Cертифікат Lets Encrypt

Для генерації сертифікату за допомогою пакету certbot:

```
#!/bin/sh

certbot certonly \
  --preferred-challenges dns \
  --manual \
  --manual-auth-hook "/root/script/certs/he.net/he-dns-update.sh"  \
  --manual-cleanup-hook "/root/script/certs/he.net/he-dns-update.sh clean"  \
  --manual-public-ip-logging-ok \
   -d 'domain' \
  --server https://acme-v02.api.letsencrypt.org/directory --agree-tos \
  --post-hook /root/script/certs/he.net/reloadservices.sh \
  -m postmaster@domain
```

Для оновлення роботи сертифікатів скрипт перезавантаження служб post-hook - reloadservices.sh. Шляхи до розміщення скриптів : /root/script/certs/he.net/. Ваш 'domen' домен замінити на власний.

```
#!/bin/sh

service postfix restart
service dovecot restart
```

### Додатки

У додатках змінюємо шляхи до сертифікатів:

Postfix:

```
smtpd_tls_CAfile = /usr/local/etc/letsencrypt/live/domain/fullchain.pem  
smtpd_tls_cert_file = /usr/local/etc/letsencrypt/live/domain/cert.pem  
smtpd_tls_key_file = /usr/local/etc/letsencrypt/live/domain/privkey.pem
```

Dovecot:

```
ssl_cert = </usr/local/etc/letsencrypt/live/domain/fullchain.pem  
ssl_key = </usr/local/etc/letsencrypt/live/domain/privkey.pem
```

### CAA записи DNS

Для застосування [CAA](https://en.wikipedia.org/wiki/DNS_Certification_Authority_Authorization) записів треба використати домен "letsencrypt.org"

|  |
| --- |
|  |
| CAA letsencrypt.org |
