---
layout: post
title: "nginx як проксі для серверів pop3, imap. Додаткова авторизація через ssl client, та блокування у fail2ban."
date: 2023-03-30 04:44:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/03/nginx-proxy-pop3-imap-ssl-client-fail2ban.html
---

#### Задача

Є задача підключити nginx як ssl proxy для служб електронної пошти, з
перевіркою авторизації у nginx. Використання nginx дозволить використати
сучасні ssl сертифікати, у випадку коли поштові сервіси не можуть це зробити
за певних умов. Наприклад коли необхідна додаткова
[перевірка авторизації через сертифікати ssl клієнта](https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html).

|  |
| --- |
|  |
| Додаткова авторизація через SSL client certificate. |

Для підтримки авторизації через SSL client certificate сторона сервера посилає
додатковий запит клієнту на те що очікує від нього особистого сертифікату.
Запит описує чи має бути відновіть від клієнта з особистим сертифікатом
клієнту обов'язковим чи за бажанням. Надалі клієнт надсилає, а сервер
перевіряє отриманий особистий сертифікат клієнту чи був він виданий певним
центром сертифікації (CA) та чи він не був відкликаний, або не слив термін
придатності. На цьому перевірка закінчена, будь який дійсний сертифікат
виданий центром сертифікації (CA) буде прийнятним для сервера.

### Дійсність сертифікатів

Тому для визначення дійсності попередньо виданих сертифікатів є важливим
перевіряти сертифікати зі
[списком відкликаних сертифікатів](https://uk.wikipedia.org/wiki/%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA_%D0%B2%D1%96%D0%B4%D0%BA%D0%BB%D0%B8%D0%BA%D0%B0%D0%BD%D0%B8%D1%85_%D1%81%D0%B5%D1%80%D1%82%D0%B8%D1%84%D1%96%D0%BA%D0%B0%D1%82%D1%96%D0%B2). Це можна робити через файл відкликаних сертифікатів (CRL) котрий
генерується центром сертифікації (CA), або через онлайн запит OCSP.

У моєму випадку є доступ до локального файлу відкликаних сертифікатів (CRL).

Наступним етапом є перевірка цифрового відбитку особистого сертифікату клієнта
у форматі
[X.509](https://uk.wikipedia.org/wiki/X.509 "X.509").

### Цифровий відбиток (fingerprints) особистого сертифікату

Наскільки унікальні відбитки пальців людей, настільки ж унікальні і відбитки
кожного сертифіката. Відбиток це хеш-значення сертифіката користувача, що
показано як одне з його властивостей.

|  |
| --- |
|  |
| Приклад інформації про відбитку особистого сертифікату клієнта. |

Раніше цифровий відбиток ([fingerprints](https://en.wikipedia.org/wiki/Public_key_fingerprint)) використовував
[хеш-значення](https://uk.wikipedia.org/wiki/%D0%A5%D0%B5%D1%88-%D1%84%D1%83%D0%BD%D0%BA%D1%86%D1%96%D1%8F)
за алгоритмом [SHA-1](https://uk.wikipedia.org/wiki/SHA-1).
Важаться, що
[SHA-1](https://uk.wikipedia.org/wiki/SHA-1) не
гарантує достатнього захисту проти атак. Вже в 2005 дослідниками були відкриті
методи атаки на SHA-1, які поставили під сумнів тривалість використання цього
алгоритму[1]. Тому вже з 2010 року низка організацій та компаній стали
рекомендувати використання SHA-2 або SHA-3 замість нього. Microsoft, Google,
Apple та Mozilla оголосили, що їхні веббраузери припинять приймати SSL
сертифікати з SHA-1 починаючи з 2017 року.

Тому зараз цифровий відбиток має додаткову інформацію про
[хеш-значення](https://uk.wikipedia.org/wiki/%D0%A5%D0%B5%D1%88-%D1%84%D1%83%D0%BD%D0%BA%D1%86%D1%96%D1%8F)
за алгоритмом версії 2 у реалізації
[SHA-256](https://uk.wikipedia.org/wiki/SHA-2).

### Перевірка цифрового відбитку сертифікату

Маючи унікальне значення цифрового відбитку для кожного сертифікату
користувача за алгоритмом
[SHA-256](https://uk.wikipedia.org/wiki/SHA-2) його
зберігають у базі даних користувача під час генерації і видачі особистого
сертифікату.

Під час авторизації перевіряється чи співпадає отримане хеш значення
[SHA-256](https://uk.wikipedia.org/wiki/SHA-2) від
сертифіката клієнта з тим значенням що зберігається у базі даних клієнта. Це
унеможливлює повторне використання одного сертифікату для авторизації багатьма
іншими користувачами, так і  унеможливлює використання анульованих
сертифікатів для користувача кому його видано. Також перевіряється ім'я
користувача з
[полями subject сертифікату](https://www.rfc-editor.org/rfc/rfc5280#section-4.1.2.6)
де міститься інформація про сертифікат, наприклад:  CN, emailAddress або
[x500UniqueIdentifier](https://www.ietf.org/rfc/rfc2798.txt). Це можна передягнути через команду:

```
openssl x509 -in cert.pem -noout -subject
```

#### NGINX

### Модуль mail

[Nginx](https://nginx.org/en/docs/) може працювати
в
[якості проксі-сервера для протоколів IMAP, POP3 та SMTP](https://docs.nginx.com/nginx/admin-guide/mail-proxy/mail-proxy/).

Так я налаштував проксі сервер для
[POP3](https://uk.wikipedia.org/wiki/POP3) та
[IMAP](https://uk.wikipedia.org/wiki/IMAP)
протоколів.

Для протоколу
[POP3](https://uk.wikipedia.org/wiki/POP3)
використовується класичний відкритий 110 порт TCP з можливістю використання
реалізації
[StartTLS](https://uk.wikipedia.org/wiki/StartTLS),
та шифрований Secure POP3 ([SSL](https://uk.wikipedia.org/wiki/SSL "SSL")-POP) 995 порт TCP.

Для протоколу
[IMAP](https://uk.wikipedia.org/wiki/IMAP)
використовується  шифрований Secure IMAP (IMAPS) він працює через 993
порт [TCP](https://uk.wikipedia.org/wiki/TCP "TCP").

```
mail {
 server_name pop3.lexxai.pp.ua;
 auth_http   127.0.0.1:99/auth.php;
 proxy_pass_error_message on;
 error_log /var/log/nginx/mail_proxy_error.log;
 pop3_auth     plain cram-md5;
 imap_auth     plain login;
 ssl_certificate      /usr/local/etc/ssl/letscript/pop3.lexxai.pp.ua/fullchain.pem;
 ssl_certificate_key  /usr/local/etc/ssl/letscript/pop3.lexxai.pp.ua/privkey.pem;
 ssl_protocols        TLSv1.2 TLSv1.3;
 server {
        listen    110;
        listen    [::]:110;
        starttls  on;
        protocol  pop3;
 }
 server {
        listen    995 ssl;
        listen    [::]:995 ssl;
        protocol  pop3;
 }
 server {
        listen    993 ssl;
        listen    [::]:993 ssl;
        protocol  imap;
        server_name imap.lexxai.pp.ua;
        auth_http_pass_client_cert on;
        ssl_verify_client       optional;
        ssl_certificate         /usr/local/etc/ssl/letscript/imap.lexxai.pp.ua/fullchain.pem;
        ssl_certificate_key     /usr/local/etc/ssl/letscript/imap.lexxai.pp.ua/privkey.pem;
        ssl_client_certificate  /usr/local/etc/ssl/mail-user/ca/mailCA.crt;
        ssl_trusted_certificate /usr/local/etc/ssl/mail-user/ca/mailCA.crt;
        ssl_crl                 /usr/local/etc/ssl/mail-user/crl/mailCA.crl;
        ssl_verify_depth        2;
 }
}
```

### Сертифікати підключення

За шляхом */usr/local/etc/ssl/letscript* зберігаються сертифікати для
підключення до сервера корті отриманні від сервісу
[letsencrypt.org](https://letsencrypt.org/uk/).

### Сертифікати особисті

За шляхом */usr/local/etc/ssl/mail-user* зберігаються згенеровані
особисті сертифікати для авторизації. Вони створенні з використанням центру
сертифікації (CA) за допомогою *openssl*. Ним підписанні усі сертифікати
поштових клієнтів, а також генерується файл список для відкликаних
сертифікатів (CRL).

### Авторизація

Авторизація віддалених клієнтів проходить через модуль:
[auth\_http](https://nginx.org/en/docs/mail/ngx_mail_auth_http_module.html)  через онлайн запит: 127.0.0.1:99/auth.php;   
Де,
127.0.0.1:99/auth.php це  web сервер з PHP скриптом котрий обробляє GET
запити від модуля mail.

Сервер 127.0.0.1:99 описаний як http частина того ж самого nginx сервера, що
прослуховує запити за адресою 127.0.0.1 на порту 99 TCP.

```
server  {
 listen       127.0.0.1:99 default_server;
 access_log   none;
 error_log    /var/log/nginx/mail_proxy_error.log;
 location ~ \.php$ {
            set $root_path /var/www/mail-auth;
            fastcgi_pass   unix:/var/run/php-fpm-www.sock;
            include        fastcgi_params;
            fastcgi_param  SCRIPT_FILENAME $root_path$fastcgi_script_name;
            fastcgi_param  SERVER_NAME $host;
            fastcgi_param  DOCUMENT_ROOT $root_path;
 }
}
```

### Скрипт авторизації auth.php

Інформація про формат обміну даних описано в
[nginx\_mail\_auth\_http\_module](https://nginx.org/en/docs/mail/ngx_mail_auth_http_module.html), приклад скрипту auth.php є у статті:
[Using a PHP Script on an Apache Server as the IMAP Auth Backend | NGINX](https://www.nginx.com/resources/wiki/start/topics/examples/imapauthenticatewithapachephpscript/).

Необхідно доповнити функціонал з прикладу у функції
*- authuser($user,$pass,$protocol)*.

Але перед цим хочу зауважити що за
[документацією](https://nginx.org/en/docs/mail/ngx_mail_auth_http_module.html#auth_http_pass_client_cert)
при авторизації з використанням
[ssl\_client\_certificate](https://nginx.org/en/docs/mail/ngx_mail_ssl_module.html#ssl_client_certificate), передається змінна про цифровий відбиток "Auth-SSL-Fingerprint" з змінної
*$ssl\_client\_fingerprint* модуля
[ngx\_http\_ssl](http://nginx.org/en/docs/http/ngx_http_ssl_module.html)
за хеш алгоритмом
[SHA-1](https://uk.wikipedia.org/wiki/SHA-1).

```
GET /auth HTTP/1.0
Host: localhost
Auth-Method: plain
Auth-User: user
Auth-Pass: password
Auth-Protocol: imap
Auth-Login-Attempt: 1
Client-IP: 192.0.2.42
Auth-SSL: on
Auth-SSL-Protocol: TLSv1.3
Auth-SSL-Cipher: TLS_AES_256_GCM_SHA384
Auth-SSL-Verify: SUCCESS
Auth-SSL-Subject: /CN=example.com
Auth-SSL-Issuer: /CN=example.com
Auth-SSL-Serial: C07AD56B846B5BFF
Auth-SSL-Fingerprint: 29d6a80a123d13355ed16b4b04605e29cb55a5ad
```

Тому було додано до php коду функцію *CertificatGetFingerPrintSHA256()*
для отримання цифрового відбитку сертифіката за алгоритмом
[SHA-256](https://uk.wikipedia.org/wiki/SHA-2), зі
змінної *HTTP\_AUTH\_SSL\_CERT* котра містить сертифікат клієнта у PEM
форматі (urlencoded).

Для авторизації клієнта безпосередньо за login/password використовується запит
до LDAP сервера через додаткову функцію
*ldapAuth($username,$password,$finger)*.  Також тут може
перевірятися ssl-fingerprint, за наявністю.

```
<?php  
       
    /*  
    NGINX sends headers as  
    Auth-User: somuser  
    Auth-Pass: somepass  
    On my php app server these are seen as  
    HTTP_AUTH_USER and HTTP_AUTH_PASS  
    */  
    if (!isset($_SERVER["HTTP_AUTH_USER"] ) || !isset($_SERVER["HTTP_AUTH_PASS"] )){  
      fail();  
    }  
       
    $username=$_SERVER["HTTP_AUTH_USER"] ;  
    $userpass=$_SERVER["HTTP_AUTH_PASS"] ;  
    $protocol=$_SERVER["HTTP_AUTH_PROTOCOL"] ;  
  
      
    // default backend port  
    $backend_port=110;  
   
    if ($protocol=="imap") {  
      $backend_port=143;  
    }  
       
    if ($protocol=="smtp") {  
      $backend_port=25;  
    }  
       
    // NGINX likes ip address so if your  
    // application gives back hostname, convert it to ip address here  
    $backend_ip["mailhost1"] ="10.1.10.100";  
    $backend_ip["mailhost2"] ="10.1.10.101";  
    $backend_ip["localhost"] ="127.0.0.1";  
  
       
    // Authenticate the user or fail  
    $auth=authuser($username,$userpass,$protocol);  
    if ($auth !== true){  
      fail($auth);  
      exit;  
    }  
  
    $userserver=getmailserver($username);  
       
    // Get the server for this user if we have reached so far  
  
    $server_ip=(isset($backend_ip[$userserver]))?$backend_ip[$userserver] :$userserver;  
    
    // Pass!  
    pass($server_ip, $backend_port);  
       
    //END  
       
    function authuser($user,$pass,$protocol){  
  
      // password characters encoded by nginx:  
      // " " 0x20h (SPACE)  
      // "%" 0x25h  
      // see nginx source: src/core/ngx_string.c:ngx_escape_uri(...)  
      $pass = str_replace('%20',' ', $pass);  
      $pass = str_replace('%25','%', $pass);  
  
      $LOGIN_ATT=$_SERVER["HTTP_AUTH_LOGIN_ATTEMPT"];  
      $CLIENT_IP=$_SERVER["HTTP_CLIENT_IP"] ;  
       
      if ($LOGIN_ATT>3){  
        $error='ERROR_ATTEMPT';  
        error_log("MAIL($protocol): $error, user: $user, rclient: $CLIENT_IP, att: $LOGIN_ATT", 0);  
        return $error;  
      }  
  
      $A_SSL_V=(isset($_SERVER["HTTP_AUTH_SSL_VERIFY"]))?$_SERVER["HTTP_AUTH_SSL_VERIFY"]:'';  
       
      if (!empty($A_SSL_V) && $A_SSL_V != "NONE") {  
       $A_SSL=$_SERVER["HTTP_AUTH_SSL"];  
       $A_SSL_P=$_SERVER["HTTP_AUTH_SSL_PROTOCOL"];  
       $A_SSL_C=$_SERVER["HTTP_AUTH_SSL_CIPHER"];  
       $A_SSL_V=$_SERVER["HTTP_AUTH_SSL_VERIFY"];  
       $A_SSL_S=$_SERVER["HTTP_AUTH_SSL_SUBJECT"];  
       $A_SSL_I=$_SERVER["HTTP_AUTH_SSL_ISSUER"];  
       $A_SSL_SER=$_SERVER["HTTP_AUTH_SSL_SERIAL"];  
       $A_SSL_F=$_SERVER["HTTP_AUTH_SSL_FINGERPRINT"];  
       $A_SSL_F256=CertificatGetFingerPrintSHA256();  
      }else{  
       $A_SSL='';  
       $A_SSL_P='';  
       $A_SSL_C='';  
       $A_SSL_S='';  
       $A_SSL_I='';  
       $A_SSL_SER='';  
       $A_SSL_F='';  
       $A_SSL_F256='';  
      };  
  
       if ($protocol == "imap" ) {  
        if ( $A_SSL_V != "SUCCESS") {  
         $error='ERROR_SSLVERIFY';  
         error_log("MAIL($protocol): $error, user: $user, rclient: $CLIENT_IP, att: $LOGIN_ATT, sub: $A_SSL_S, verify: $A_SSL_V, fing: $A_SSL_F", 0);  
         return $error;  
        }elseif(strpos($A_SSL_S, "x500UniqueIdentifier=".$user.',') === false){  
         $error='ERROR_SSLOWNER';  
         error_log("MAIL($protocol): $error, user: {$user}, rclient: {$CLIENT_IP}, att: {$LOGIN_ATT}, sub: {$A_SSL_S}, fing: {$A_SSL_F}", 0);  
         return $error;  
        }  
      }  
  
      if (!ldapAuth($user,$pass,$A_SSL_F256)){  
         $error='ERROR_AUTH.'.$A_SSL;  
         error_log("MAIL($protocol): $error, user: $user, rclient: $CLIENT_IP, a_ssl: $A_SSL", 0);  
         return $error;  
      }  
  
      return true;  
    }  
       
    function getmailserver($user){  
        return "mailhost1";  
    }  
       
    function fail($err){  
      header("Auth-Status: Invalid login or password. $err");  
      exit;  
    }  
       
    function pass($server,$port){  
      header("Auth-Status: OK");  
      header("Auth-Server: $server");  
      header("Auth-Port: $port");  
      exit;  
    }  
      
    function ldapAuth($username,$password,$finger){  
	$LDAPSERVER='10.1.10.101';  
	$BN='OU=MAIL';  
	if (!empty($finger)){  
	 $hash=$finger;  
	 $formattedHash = chunk_split(strtoupper($hash), 2, ':');  
	 $formattedHash = rtrim($formattedHash, ':');  
	 $currentDate = gmdate("YmdHis") . "Z";  
	 $query_filter="(&(fingerprint={$formattedHash})(fingerprintexpire>={$currentDate})(uid={$username})(mail=*))";  
	}else{  
	 $query_filter="(&(uid={$username})(mail=*))";  
	}  
	$result = false;  
        $ldapconn = ldap_connect($LDAPSERVER);  
        if ( $ldapconn ) {  
         $ldhb =@ldap_bind($ldapconn, "uid=". $username .', '. $BN, $password);  
         if ( $ldhb ) {  
    	     if ( $res = ldap_list($ldapconn, $BN, $query_filter, array('mail')) ) {  
        	 if ( $entry = ldap_first_entry($ldapconn, $res) ) {  
                   $result = true;  
                 }  
               }  
             }  
             ldap_close($ldapconn);  
      }  
      return $result;  
    }  
      
    function CertificatGetFingerPrintSHA256(){  
       if (!empty($_SERVER["HTTP_AUTH_SSL_CERT"])){  
    	    $cert=$_SERVER["HTTP_AUTH_SSL_CERT"];  
	    $cert = str_replace('%20', ' ', $cert);  
	    $cert = str_replace('%0A', "\n", $cert);  
	    $hash = openssl_x509_fingerprint($cert, 'sha256');  
	    return $hash;  
       }  
       return '';  
    }  
?>
```

У випадку помилок авторизації все записується до log файлу nginx:*/var/log/nginx/mail\_proxy\_error.log*.  У форматі:   
*error\_log("MAIL($protocol): $error, user: $user, rclient: $CLIENT\_IP,
a\_ssl: $A\_SSL", 0)*

Приклад:

```
2023/03/29 13:33:44 [error] 78983#101330: *711901 FastCGI sent in stderr: "PHP message: MAIL(pop3): ERROR_AUTH., user: dizzylittle@lexxai.pp.ua, rclient: 219.XX.XXX.161, att: 1" while reading response header from upstream, client: 127.0.0.1, server: , request: "GET /auth.php HTTP/1.0", upstream: "fastcgi://unix:/var/run/php-fpm-www.sock:", host: "127.0.0.1"  
2023/03/29 15:13:22 [error] 78983#101330: *722540 FastCGI sent in stderr: "PHP message: MAIL(pop3): ERROR_AUTH., user: feqernando@lexxai.pp.ua, rclient: 2804:XXXX:XXXX:XXXX:XXXX:XXXX:XXXX:be30, att: 1" while reading response header from upstream, client: 127.0.0.1, server: , request: "GET /auth.php HTTP/1.0", upstream: "fastcgi://unix:/var/run/php-fpm-www.sock:", host: "127.0.0.1"
```

#### Поштовий клієнт.

Таким чином поштовий клієнт при підключенні підключається тільки до nginx
proxy сервера, а у випадку успішної автизації,  nginx підключається до
внутрішнього поштового сервера за адресами що повертає *auth.php* скрипт
з $backend\_port та $backend\_ip.

```
header("Auth-Status: OK");
header("Auth-Server: $server");
header("Auth-Port: $port");
```

Підключення до локального поштового сервера проходить без ssl з авторизацією
за login/password. І результат повертається до nginx, а далі до віддаленого
клієнта. Якщо авторизація не пройшла у nginx то поштовий сервер і не знає про
це так як не отримує підключення.

Тому локальний поштовий сервер нічого не знає про віддалену адресу клієнта, і
не може блокувати IP адресу "поганих" користувачів, тому ця задача
переноситься на сторону де встановлений nginx з
[fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page).

#### fail2ban

[fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page)
надає можливість блокувати IP адресу за допомогою аналізу лог. файлу
використовуючи правила та команди Вашого брандмауера.

### Фільтри

Я створив новий файл з фільтрами для аналізу лог файлу:
*mail\_proxy\_error.log* котрий генерується сервером nginx.

*filter.d/nginx-mail.conf*:

```
[Definition]  
mode = aggressive  
  
mdre-auth = ^\s*\[error\] \d+#\d+: \*\d+ FastCGI sent in stderr: "PHP message: MAIL\((?P<prot>.+)\): ERROR.+, user: <F-ALT_USER>.+</F-ALT_USER>, rclient: <ADDR>,  
mdre-fallback = ^\s*\[crit\] \d+#\d+: \*\d+ SSL_shutdown\(\) failed \(SSL: error:.+, client: <ADDR>, .*login: "<F-ALT_USER>.+</F-ALT_USER>"  
  
mdre-normal = %(mdre-auth)s  
mdre-aggressive = %(mdre-auth)s  
                  %(mdre-fallback)s  
  
failregex = <mdre-<mode>>  
  
ignoreregex =  
  
datepattern = {^LN-BEG}
```

### Налаштування

jail.local:

```
[nginx-mail]
enabled = true
port   = imaps,pop3,pop3s
logpath = /var/log/nginx/mail_proxy_error.log
backend = %(syslog_backend)s
maxretry = 3
bantime = 20m
findtime = 1h
banaction = pf
```

### Action

Так як для banaction використовується action pf, і це використання pf firewall
в FreeBSD.

Якщо бути зовсім безжальним до користувачів котрі використовують адреси IP
версії v6, і використовують свою підмережу /64 для отримання різних тимчасовий
і довільних IP адрес, можна блокувати всю підмережу з першої спроби
блокування. Для цього створюється свій action файл на базі pf.conf:
*action.d/pf-ipv6-64.conf* з наступними змінами:

```
actionban = echo "<ip>" | grep -q ":" && <pfctl> -t <tablename>-<name> -T add <ip>/64 || <pfctl> -t <tablename>-<name> -T add <ip>  
actionunban = echo "<ip>" | grep -q ":" && <pfctl> -t <tablename>-<name> -T delete <ip>/64 || <pfctl> -t <tablename>-<name> -T delete <ip>
```

### Правила firewall

Для правил брандмауера *pf* додаємо до */etc/pf.conf* файлу запис
для quik блокування IP адрес.

```
# anchor for fail2ban rules
anchor "f2b/*"
```

За anhor  "*f2b/nginx-mail*" можемо переглянути правила і таблицю
заблокованих файлів без посередньо з таблиць firewall. А якщо засобами
*fail2ban* то так: *fail2ban-client get nginx-mail banned*.

```
pfctl -sr -a f2b/nginx-mail  
pfctl -t f2b-nginx-mail -T show -a f2b/nginx-mail
```

```
block drop quick proto tcp from <f2b-nginx-mail> to any port = imaps  
block drop quick proto tcp from <f2b-nginx-mail> to any port = pop3  
block drop quick proto tcp from <f2b-nginx-mail> to any port = pop3s  
   46.XXX.XXX.239  
   50.XXX.XXX.29  
   50.XXX.XXX.94  
   51.XXX.XXX.17  
   58.XXX.XXX.50  
   58.XXX.XXX.28  
   59.XXX.XXX.131  
   63.XXX.XXX.28  
   64.XXX.XXX.155  
   2804:XXXX:XXXX:1f40::/64
```

#### Блокування підмережі

Є проект [fail2ban-block-ip-range](https://github.com/lexxai/fail2ban-block-ip-range) автора:[WKnak](https://github.com/WKnak). Що дозволяє аналізувати за розкладом cron log файл fail2ban, шукає адреси jails і групує до підмережі і блокує командами fail2ban:

```
fail2ban-client set courier-auth banip 78.128.113.66/32  
fail2ban-client set postfix-sasl banip 45.142.120.0/24  
fail2ban-client set sshd banip 193.56.28.160/32  
fail2ban-client set apache-auth banip 45.150.206.112/29
```
