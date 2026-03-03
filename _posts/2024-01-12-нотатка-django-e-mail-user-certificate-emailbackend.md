---
layout: post
title: "Нотатка. Django. E-mail. User Certificate. EmailBackend."
date: 2024-01-12 16:04:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/01/django-e-mail-user-certificate.html
---

Для відправлення листів електронною поштою на певних поштових системах
потрібно застосовувати свій персональний сертифікат користувача, у парі з
закритим ключем.

|  |
| --- |
|  |
| Властивості сертифіката для підключення клієнта |

Але, пробуючи відправити листа у Django з'ясував що листи не відправляються з
помилкою:

```
SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1000)')
```

Аналізуючи приклад з прямим надсиланням email через *smtplib*, з'ясував -
що працює відправлення коли контекст створений з використанням
***purpose  = ssl.Purpose.SERVER\_AUTH***.

А у базовому *EmailBackend* у Dajngo контекст без цього.

```
class EmailBackend(BaseEmailBackend):
...
    @cached_property
    def ssl_context(self):
        if self.ssl_certfile or self.ssl_keyfile:
            ssl_context = ssl.SSLContext(protocol=ssl.PROTOCOL_TLS_CLIENT)
            ssl_context.load_cert_chain(self.ssl_certfile, self.ssl_keyfile)
            return ssl_context
        else:
            return ssl.create_default_context()
```

Тому створив новий власний Backend котрий розширює EmailBackend.  
Для розв'язання проблем з сертифікатами у macOS  помилки (ssl.SSLCertVerificationError: [SSL: CERTIFICATE\_VERIFY\_FAILED] certificate verify failed: unable to get local issuer certificate (\_ssl.c:1000)) додано використання "certifi"
та cafile=certifi.where().

cert\_email\_backend.py:

```
import ssl  
import certifi
  
from django.core.mail.backends.smtp import EmailBackend
from django.utils.functional import cached_property


class CertEmailBackend(EmailBackend):
      
    @cached_property
    def ssl_context(self):
        if self.ssl_certfile or self.ssl_keyfile:
            ssl_context = ssl.create_default_context(ssl.Purpose.SERVER_AUTH, cafile=certifi.where())
            # ssl_context = ssl.SSLContext(protocol=ssl.PROTOCOL_TLS_CLIENT)
            ssl_context.load_cert_chain(self.ssl_certfile, self.ssl_keyfile)
            return ssl_context
        else:
            return ssl.create_default_context(cafile=certifi.where())
```

А в налаштуваннях *settings.py* змінив EMAIL\_BACKEND.

```
# EMAIL_BACKEND = "django.core.mail.backends.smtp.EmailBackend"
EMAIL_BACKEND = "your_project.cert_email_backend.CertEmailBackend"
```

Змінні оточення використовую в *settings.py* для визначення шляху до файлу
сертифіката та ключа що знаходяться у теці *.certs* так:

```
BASE_DIR = Path(__file__).resolve().parent.parent  
  
EMAIL_SSL_CERTFILE = os.getenv("EMAIL_SSL_CERTFILE")
EMAIL_SSL_KEYFILE = os.getenv("EMAIL_SSL_KEYFILE")

if EMAIL_SSL_CERTFILE:
    EMAIL_SSL_CERTFILE = BASE_DIR.parent.joinpath(".certs").joinpath(EMAIL_SSL_CERTFILE)
if EMAIL_SSL_KEYFILE:
    EMAIL_SSL_KEYFILE = BASE_DIR.parent.joinpath(".certs").joinpath(EMAIL_SSL_KEYFILE)
```

Для тестування налаштувань зручно використати django-admin tools 
manage.py

```
python manage.py sendtestemail some@example.net
```

Для конвертування ключів з файлу .p12 до файлу .pem використати можна openssl:

```
openssl pkcs12 -info -in path.p12  -nodes -nocerts -out newfile.pem
```
