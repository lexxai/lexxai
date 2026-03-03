---
layout: post
title: "Нотатка для себе. Використання TCP IP round robin для ffmpeg за допомогою SQUID Proxy"
date: 2023-06-17 15:38:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/06/tcp-ip-round-robin-ffmpeg-squid-proxy.html
---

#### Умови

* Задача балансувати зовнішнє підключення програми ffmpeg за допомогою пулу різних IP
  адрес.
* Firewall server з встановленим Squid. У мене це сервер "[pfSense+](https://www.netgate.com/pfsense-plus-software)".
* Мережа реальних IP адрес  /24.

#### 

#### pfSense+ Virtual IP

Налаштовую на інтерфейсі додаткові зовнішні адреси як Аліаси або Virtual IPs у
pfSense. Це десять  реальних  IP.

|  |
| --- |
|  |
| Virtual IPs у pfSense |

#### Squid

Далі стандартно налаштовую Squid на порту 3159 і інтерфейсі де будуть
підключатися клієнти.

|  |
| --- |
|  |
| Squid у pfSense. |

Тепер додаю додаткові порти прослуховування для Squid для одного інтерфейсу
через додаткові налаштування (Advanced Features/ Custom Options (Before Auth)
).

|  |
| --- |
|  |
| Squid listening |

  

Тепер додаю правила acl для Squid за портом прослуховування також через
додаткові налаштування (Advanced Features/ Custom Options (Before Auth) ).

```
acl outgoing160 localport 3160
acl outgoing161 localport 3161
acl outgoing162 localport 3162
acl outgoing163 localport 3163
acl outgoing164 localport 3164
acl outgoing165 localport 3165
acl outgoing166 localport 3166
acl outgoing167 localport 3167
acl outgoing168 localport 3168
acl outgoing169 localport 3169
```

Додаю вибір вихідної IP адреси для зовнішніх підключень.

```
tcp_outgoing_address XX.XX.XX.160 outgoing160
tcp_outgoing_address XX.XX.XX.161 outgoing161
tcp_outgoing_address XX.XX.XX.162 outgoing162
tcp_outgoing_address XX.XX.XX.163 outgoing163
tcp_outgoing_address XX.XX.XX.164 outgoing164
tcp_outgoing_address XX.XX.XX.165 outgoing165
tcp_outgoing_address XX.XX.XX.166 outgoing166
tcp_outgoing_address XX.XX.XX.167 outgoing167
tcp_outgoing_address XX.XX.XX.168 outgoing168
tcp_outgoing_address XX.XX.XX.169 outgoing169
```

Налаштовую зовнішні проксі (Remote Proxy Settings/Remote Cache) у ручних
налаштуваннях (Custom Options (Before Auth)) тому що GUI не розуміє додавання
опції "name" до "cache\_peer". Використання "name" дає використати одну адресу з
різними портами.

```
#Remote proxies
cache_peer XX.XX.XX.1 parent 3160 0 no-query proxy-only round-robin name=rrobin0
cache_peer XX.XX.XX.1 parent 3161 0 no-query proxy-only round-robin name=rrobin1
cache_peer XX.XX.XX.1 parent 3162 0 no-query proxy-only round-robin name=rrobin2
cache_peer XX.XX.XX.1 parent 3163 0 no-query proxy-only round-robin name=rrobin3
cache_peer XX.XX.XX.1 parent 3164 0 no-query proxy-only round-robin name=rrobin4
cache_peer XX.XX.XX.1 parent 3165 0 no-query proxy-only round-robin name=rrobin5
cache_peer XX.XX.XX.1 parent 3166 0 no-query proxy-only round-robin name=rrobin6
cache_peer XX.XX.XX.1 parent 3167 0 no-query proxy-only round-robin name=rrobin7
cache_peer XX.XX.XX.1 parent 3168 0 no-query proxy-only round-robin name=rrobin8
cache_peer XX.XX.XX.1 parent 3169 0 no-query proxy-only round-robin name=rrobin9
```

Забороняю пряме підключення до мережі для клієнтів що підключилися на порт 3159,
форсуючи таким чином підключення через "Remote Cache"

```
acl roudrobinport localport 3159
prefer_direct on
never_direct allow roudrobinport
never_direct deny all
```

Таким чином клієнти що підключаються на порт проксі сервера 3159, на кожен новий
запит будуть використовувати інший проксі порт, а той в свою чергу буде вже
виходити через свою налаштовану зовнішню адресу.

#### 

#### Перевірка

### curl

```
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.169
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.168
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.167
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.166
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.165
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.164
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.163
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.162
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.161
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.160
 provider       : 
 location       : 
% curl -x "http://XX.XX.XX.1:3159" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.169
 provider       : 
 location       :
```

Якщо треба просто іншу зовнішню IP  без round-robin, то вибираємо інший
порт:

```
% curl -x "http://XX.XX.XX.1:3164" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.164
 provider       : 
 location       :  
% curl -x "http://XX.XX.XX.1:3164" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.164
 provider       : 
 location       :  
% curl -x "http://XX.XX.XX.1:3164" "2ip.me"
 ip             : unknown, unknown, XX.XX.XX.164
 provider       : 
 location       :
```

### ffmpeg

```
 % ffmpeg -http_proxy http://XX.XX.XX.1:3159 -hide_banner -y -i https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/360/Big_Buck_Bunny_360_10s_5MB.mp4 demo.mp4
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/360/Big_Buck_Bunny_360_10s_5MB.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    title           : Big Buck Bunny, Sunflower version
    artist          : Blender Foundation 2008, Janus Bager Kristensen 2013
    composer        : Sacha Goedegebure
    encoder         : Lavf57.63.100
    comment         : Creative Commons Attribution 3.0 - http://bbb3d.renderfarming.net
    genre           : Animation
  Duration: 00:00:10.00, start: 0.000000, bitrate: 4183 kb/s
  Stream #0:0(und): Video: h264 (High) (avc1 / 0x31637661), yuv420p, 640x360 [SAR 1:1 DAR 16:9], 4180 kb/s, 30 fps, 30 tbr, 15360 tbn, 60 tbc (default)
    Metadata:
      handler_name    : VideoHandler
      vendor_id       : [0][0][0][0]
Stream mapping:
  Stream #0:0 -> #0:0 (h264 (native) -> h264 (libx264))
Press [q] to stop, [?] for help
[libx264 @ 0x86074c700] using SAR=1/1
[libx264 @ 0x86074c700] using cpu capabilities: MMX2 SSE2Fast SSSE3 SSE4.2 AVX FMA3 BMI2 AVX2
[libx264 @ 0x86074c700] profile High, level 3.0, 4:2:0, 8-bit
[libx264 @ 0x86074c700] 264 - core 164 - H.264/MPEG-4 AVC codec - Copyleft 2003-2022 - http://www.videolan.org/x264.html - options: cabac=1 ref=3 deblock=1:0:0 analyse=0x3:0x113 me=hex subme=7 psy=1 psy_rd=1.00:0.00 mixed_ref=1 me_range=16 chroma_me=1 trellis=1 8x8dct=1 cqm=0 deadzone=21,11 fast_pskip=1 chroma_qp_offset=-2 threads=11 lookahead_threads=1 sliced_threads=0 nr=0 decimate=1 interlaced=0 bluray_compat=0 constrained_intra=0 bframes=3 b_pyramid=2 b_adapt=1 b_bias=0 direct=1 weightb=1 open_gop=0 weightp=2 keyint=250 keyint_min=25 scenecut=40 intra_refresh=0 rc_lookahead=40 rc=crf mbtree=1 crf=23.0 qcomp=0.60 qpmin=0 qpmax=69 qpstep=4 ip_ratio=1.40 aq=1:1.00
Output #0, mp4, to 'demo.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    title           : Big Buck Bunny, Sunflower version
    artist          : Blender Foundation 2008, Janus Bager Kristensen 2013
    composer        : Sacha Goedegebure
    genre           : Animation
    comment         : Creative Commons Attribution 3.0 - http://bbb3d.renderfarming.net
    encoder         : Lavf58.76.100
  Stream #0:0(und): Video: h264 (avc1 / 0x31637661), yuv420p(progressive), 640x360 [SAR 1:1 DAR 16:9], q=2-31, 30 fps, 15360 tbn (default)
    Metadata:
      handler_name    : VideoHandler
      vendor_id       : [0][0][0][0]
      encoder         : Lavc58.134.100 libx264
    Side data:
      cpb: bitrate max/min/avg: 0/0/0 buffer size: 0 vbv_delay: N/A
frame=  300 fps=283 q=-1.0 Lsize=    1077kB time=00:00:09.90 bitrate= 891.2kbits/s speed=9.34x
video:1072kB audio:0kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.435770%
[libx264 @ 0x86074c700] frame I:2     Avg QP:21.25  size: 76941
[libx264 @ 0x86074c700] frame P:76    Avg QP:23.09  size: 11200
[libx264 @ 0x86074c700] frame B:222   Avg QP:28.67  size:   416
[libx264 @ 0x86074c700] consecutive B-frames:  1.3%  0.0%  0.0% 98.7%
[libx264 @ 0x86074c700] mb I  I16..4:  0.1% 55.4% 44.5%
[libx264 @ 0x86074c700] mb P  I16..4:  0.0%  0.2%  0.1%  P16..4: 38.5% 29.4% 25.3%  0.0%  0.0%    skip: 6.6%
[libx264 @ 0x86074c700] mb B  I16..4:  0.0%  0.0%  0.0%  B16..8: 38.6%  1.4%  0.2%  direct: 0.2%  skip:59.7%  L0:41.5% L1:52.3% BI: 6.3%
[libx264 @ 0x86074c700] 8x8 transform intra:57.0% inter:54.4%
[libx264 @ 0x86074c700] coded y,uvDC,uvAC intra: 98.9% 99.9% 95.9% inter: 13.2% 14.5% 7.0%
[libx264 @ 0x86074c700] i16 v,h,dc,p:  0%  0%  0% 100%
[libx264 @ 0x86074c700] i8 v,h,dc,ddl,ddr,vr,hd,vl,hu: 15% 18% 15%  6%  8%  8% 11%  7% 12%
[libx264 @ 0x86074c700] i4 v,h,dc,ddl,ddr,vr,hd,vl,hu: 14% 10% 13%  9% 10% 10% 11% 11% 12%
[libx264 @ 0x86074c700] i8c dc,h,v,p: 55% 21%  9% 15%
[libx264 @ 0x86074c700] Weighted P-Frames: Y:0.0% UV:0.0%
[libx264 @ 0x86074c700] ref P L0: 52.8% 21.3% 11.3% 14.6%
[libx264 @ 0x86074c700] ref B L0: 83.5%  9.7%  6.8%
[libx264 @ 0x86074c700] ref B L1: 89.5% 10.5%
[libx264 @ 0x86074c700] kb/s:877.90
```

#### Squid log

|  |
| --- |
|  |
| Squid log |

  

### 

#### За матеріалами:

* [squid : tcp\_outgoing\_address configuration directive](http://www.squid-cache.org/Doc/config/tcp_outgoing_address)
* [squid : cache\_peer configuration directive](http://www.squid-cache.org/Doc/config/cache_peer)
* [Cache peer configuration for squid - Notes\_Wiki](https://www.sbarjatiya.com/notes_wiki/index.php/Cache_peer_configuration_for_squid)
* [FFmpeg Protocols Documentation](https://www.ffmpeg.org/ffmpeg-protocols.html#toc-http)
