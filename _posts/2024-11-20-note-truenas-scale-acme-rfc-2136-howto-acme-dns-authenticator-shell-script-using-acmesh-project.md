---
layout: post
title: "Note. TrueNAS Scale. ACME RFC 2136. Howto: ACME DNS-Authenticator shell script using acmesh project"
date: 2024-11-20 00:10:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/11/note-truenas-scale-acme-rfc-2136-howto.html
---

### Умови:

Є контрольований DNS server (BIND) з можливістю оновлювати записи з
використанням nsupdate
[RFC 2136](https://datatracker.ietf.org/doc/html/rfc2136).

Є
[TrueNAS SCALE](https://www.truenas.com/truenas-scale/)
Server Dragonfish-24.04.2.5.

### Задача:

Додати автоматично оновлюваний сертифікат безпеки  від
[Let’s Encrypt](https://letsencrypt.org/uk/docs/challenge-types/),  відповідно до стандартів ACME ([RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555)).

[![](/assets/images/blog/Gemini_Generated_Image_wvjsyewvjsyewvjs.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi6sAN3Xc8RBslNbcZdsepNeCsM7bo4KBkqpZQ1u4uxYo0MX2x1uEK55We3UtVrxfYBNnTKLFfCgKiEOkE0JfRud3mWEqs5cpn_U_jrufbeRcoycd70jiZuLN9wvEDfdO0Badtol229LyElU30gNWhyphenhypheniFUQmLA76CqmizkwcSB40wBVD50Wy6K4PtqU9yKr/s2048/Gemini_Generated_Image_wvjsyewvjsyewvjs.jpg)

### Основі матеріали:

* [Howto: ACME DNS-Authenticator shell script using acmesh project](https://www.truenas.com/community/threads/howto-acme-dns-authenticator-shell-script-using-acmesh-project.107252/)
* [TrueNAS SCALE - Adding LetsEncrypt Certificates](https://youtu.be/TJ5fDiDRcbU?si=28mGnptWufo1siWK)

### Дії:

1. Створити ZFS dataset: *system\_tools*.  
   Повний шлях монтування з
   pool mdata - */mnt/mdata/system\_tools.*
2. Отримати доступ до консолі, через GUI або SSH.
3. Створити робочу теку */mnt/mdata/system\_tools.*
4. Завантажити та встановити
   [acme.sh](https://github.com/acmesh-official/acme.sh/wiki/How-to-install)  до теки /mnt/mdata/system\_tools/acme.  
   mkdir
   /mnt/mdata/system\_tools/acme  
   cd /mnt/mdata/system\_tools/acme  
   git
   clone --depth 1 https://github.com/acmesh-official/acme.sh.git
5. Створити скрипт *dns\_acme.sh* на основі
   [публікації](https://www.truenas.com/community/threads/howto-acme-dns-authenticator-shell-script-using-acmesh-project.107252/), де змінити данні для змінних NSUPDATE\_SERVER, NSUPDATE\_KEY на власні:
6. Створити файл */mnt/mdata/system\_tools/acme/.nsupdate.key*, де secret
   можна отримати командую "*openssl rand -base64 64*"  

   ```
   key "some-key" {
       algorithm hmac-sha512;
       secret " … ==";
   };
   ```
7. Змінити права доступу до файлу: *chmod 600 /mnt/mdata/system\_tools/acme/.nsupdate.key*
8. Додати ACME DNS-Authenticators dns\_acme.  
     

   |  |
   | --- |
   |  |
   | dns\_acme |

     

   [![](/assets/images/blog/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-11-20%20%D0%BE%2002.03.13.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEilKSDzzPTpXQsFxV3PQzF0ws09PQ9f8J1Yvt-6LoeGnWL9pwam9Z8Y4AhE97jPn7_EFD_SgS6kS-e86DKLix9ouTMuZW5IMGsAW5oOeWJlzf2_u_GL-XvyBSIvczOXoGbSqj_AkL4HgjEVCx20V1qStS8AIu0Oa1uR3z7_WFzzJCbwBLPip9mClDt6NRps/s1158/%D0%97%D0%BD%D1%96%D0%BC%D0%BE%D0%BA%20%D0%B5%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-11-20%20%D0%BE%2002.03.13.png)
9. Зробити налаштування відповідно до
   [відео інструкції.](https://youtu.be/TJ5fDiDRcbU?si=Dd3Mvg6Kn12xQYEf&t=99) з використанням dns\_acme при створені сертифікату.
