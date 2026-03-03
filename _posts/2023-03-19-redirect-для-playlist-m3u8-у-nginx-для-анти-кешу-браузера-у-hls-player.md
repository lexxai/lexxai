---
layout: post
title: "redirect для playlist .m3u8 у nginx для анти кешу браузера у HLS player"
date: 2023-03-19 16:46:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/03/redirect-playlist-m3u8-nginx-hls-player.html
---

Стала задача як форсувати отримання свіжої версії файлу playlist [.m3u8](https://docs.fileformat.com/uk/audio/m3u8/) для застуванні у [HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) ([HTTP Live Streaming](https://developer.apple.com/streaming/)) медіа плеєру. Свіжу версію мається на увазі те що вона не буде зчитана з кешу браузера у будь-якому випадку незалежно від заголовків expire.

Вирішення задачі це додавати до імені файлу поточну дату у вигляді додаткового аргументу на кшталт: index.m3u8?m=202303191711.

Використовується Web сервер - [nginx](https://uk.wikipedia.org/wiki/Nginx).

#### nginx

### 

### time\_iso8601

Сформуємо змінну *$formatted\_date* з датою у необхідному форматі. Є вбудована змінна у nginx для дати у форматі [ISO8601](https://uk.wikipedia.org/wiki/ISO_8601) [$time\_iso8601](http://nginx.org/en/docs/http/ngx_http_core_module.html#var_time_iso8601).

Для формування дати у потрібному форматі застосовується [послідовність map у контексті http.](https://stackoverflow.com/questions/46704516/get-current-time-in-format-iso-8601-nginx)

|  |
| --- |
|  |
| map - nginx |

```
map $time_iso8601 $year {  
  default             'year';  
  '~^(?<yyyy>\d{4})-' $yyyy;  
}  
map $time_iso8601 $month {  
  default             'month';  
  '~^\d{4}-(?<mm>\d{2})-' $mm;  
}  
map $time_iso8601 $day {  
  default             'day';  
  '~^\d{4}-\d{2}-(?<dd>\d{2})' $dd;  
}  
map $time_iso8601 $hour {  
  default             'hour';  
  '~^\d{4}-\d{2}-\d{2}T(?<hh>\d{2})' $hh;  
}  
map $time_iso8601 $min {  
  default             'minute';  
  '~^\d{4}-\d{2}-\d{2}T\d{2}:(?<mn>\d{2})' $mn;  
}  
map $time_iso8601 $sec {  
  default             'seconds';  
  '~^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:(?<se>\d{2})' $se;  
}  
map $time_iso8601 $formatted_date {  
  default                         'date-not-found';  
  '~^(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})' $year$month$day$hour$min$sec;  
}
```

Тепер ми маємо змінну *$formatted\_date* у вигляді *$year$month$day$hour$min$sec*.

Але є одне але. Спільне використання змінної у регулярному виразі *location*.

### location

 Наприклад "location ~ ^(\*.\.m3u8)$" і спробі використати результат пошуку змінної "$1" у location для rewrite.

```
location ~ ^/t/(.*\.m3u8)$ {
  rewrite ^ $1?m=$formatted_date&$args? permanent;
}
```

Надасть ось такий результат: "http://lexxai.pp.ua/t/09?m=20230319083509&".

Тому що використана буде остання змінна "$1" з використання команди map, вже після *location*.

Для вирішення цієї проблеми використаємо таку конструкцію регулярного виразу:

```
location ~ ^/t/(?<filename>.+\.m3u8) {
 rewrite ^ $filename?m=$formatted_date&$args? permanent;
}
```

Зараз все буде добре з заміною, але буде інша проблема - зациклювання:

"http://lexxai.pp.ua/t/index.m3u8?m=20230319084535&m=20230319084535&m=20230319084535&m=20230319084535&m=20230319084535&m=20230319084534&m=20230319084534&m=20230319084534&m=20230319084534&m=20230319084534&m=20230319084533&m=20230319084533&m=20230319084533&m=20230319084533&m=20230319084532&m=20230319084532&m=20230319084532&m=20230319084532&m=20230319084532&m=20230319084531&...."

Тому додаємо умову на наявність аргументу "m=":

```
location ~ ^/t/(?<filename>.+\.m3u8) {
 if ($args !~ m=){
  rewrite ^ $filename?m=$formatted_date&$args? permanent;
 }
}
```

Надасть ось такий результат: "http://lexxai.pp.ua/t/index.m3u8?m=20230319085241&".

Але є знову одне але. Використання "[redirect 301](https://elit-web.ua/ua/blog/301-redirekt)" для HLS плеєра.

### HLS плеєр.

Для програвання на сторінці браузера файлів у [форматі HLS](https://en.wikipedia.org/wiki/HTTP_Live_Streaming) має бути його підтримка. Так як корні формату належать Apple ([application/vnd.apple.mpegurl](https://en.wikipedia.org/wiki/HTTP_Live_Streaming)), тому повноцінну підтримку на сьогодні має тільки браузер ([Safari](https://en.wikipedia.org/wiki/Safari_(web_browser) "Safari (web browser)")). Для інших браузерів має бути стороннє рішення. Для себе я знайшов проєкт JavaScript - [hls.js](https://github.com/video-dev/hls.js). Він використовує стандартний HTML5 video tag, і у випадку вбудованої підтримки HLS, більше нічого не застосовується. Для інших браузерів вже застосовується додатковий javascript код котрий завантажує маленькі блоки відео і з'єднує їх у пам'яті для програвання відео через об'єкт [javascript blob](https://uk.javascript.info/blob).

Так от, використання redirect 301, не дозволено у HLS, можливо з метою безпеки. Тому було винайдено інший шлях - згенерувати новий [master m3u8 playlist](https://developer.apple.com/documentation/http_live_streaming/example_playlists_for_http_live_streaming/creating_a_multivariant_playlist) засобами nginx.

### 

### master m3u8 playlist

Все доволі просто, у *location*:

```
return 200 "#EXTM3U\n#EXT-X-STREAM-INF:BANDWIDTH\n$filename?m=$formatted_date&$args";
```

І в результаті буде таке повернення від сервера на запит: http://lexxai.pp.ua/t/index.m3u8

```
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH
/t/index.m3u8?m=20230319093225&
```

### CORS

Не забуваємо про [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) заголовки для програвання HLS.

```
location ~ ^(?.+\.m3u8)$ {
 if ($args !~ m=){
  add_header Pragma "public";
  add_header Cache-Control "public";
  add_header Storm-Control "public";
  add_header X-Cache $upstream_cache_status;
  # CORS setup
  add_header 'Access-Control-Allow-Origin' "$cors_origin_header" always;
  add_header 'Access-Control-Expose-Headers' 'Content-Length';
  add_header 'Access-Control-Allow-Methods' 'GET, HEAD';
  expires 1s;
  return 200 "#EXTM3U\n#EXT-X-STREAM-INF:BANDWIDTH\n$filename?m=$formatted_date";
 }
 add_header Pragma "public";
 add_header Cache-Control "public";
 add_header Storm-Control "public";
 add_header X-Cache $upstream_cache_status;
 # CORS setup
 add_header 'Access-Control-Allow-Origin' "$cors_origin_header" always;
 add_header 'Access-Control-Expose-Headers' 'Content-Length';
 add_header 'Access-Control-Allow-Methods' 'GET, HEAD';
 expires 5m;
}
```

Де $cors\_origin\_header визначається через map для Ваших дозволенних доменів:

```
map $http_origin $cors_origin_header {
 default '';
 # all your domains
 https://cdn1.domain1.com  "$http_origin";
 https://cdn2.domain1.com  "$http_origin";
 https://cdn.domain2.com   "$http_origin";
}
```

|  |
| --- |
|  |
| location - nginx |

Приклад готового проєкту:

|  |
| --- |
| [STORMTRACKER.US - Previous 48 Hours Weather Data, updated every 5 min.](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgATyTWNAwg0V0IjMKdcYD0qWQ0lsvCAeLOqSh3migjI58XqXIfWtwT7wk2dHNG94E43g_pgwiGH1acmj0rVFn9qT-EVtPI49RtF-EYQh7sEmxEYLTJkh49w_w3sA4DJvcxcSIkP0ZuKVaTm47vUrhfiDbi0XuwvjP4jEV66vKvGy-zgKAajlMYd6AEQQ/s1896/Screenshot%202023-03-19%20202328.png) |
| <https://stormtracker.us> |
