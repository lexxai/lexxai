---
layout: post
title: "Performance Comparison of Python Function Implementations for Equivalent Output"
date: 2024-11-13 21:47:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/11/performance-comparison-of-five-python.html
---

😄 Цікаво інколи оптимізувати код.  
Отримав такі рішення для випадку коли не завжди є рядок з всіма параметрами і його треба розбити на частини.  
  
📅 Рядок category може бути "W", "P-V-00", "S", "P-V-01", "L-X", "L-X-A-B-C-D"  
Проведено 10\_000\_000 замірів 8 разів і отримано середні значення часу виконання.

|  |
| --- |
|  |
| Perfomance Comparison |

🔖Наведено топ 3 результати вимірювань.

1. Як не дивно, на першому місці
   Version 5, коли є всі елементи в категорії, але на останньому коли не
   всі елементи є - вітання до try-except.   
   Можна провести алегорію з
   приказками "Як тривога, то до Бога", "Без біди Бога не кличуть".
2. На другому місці Version 6 та Version 9 в інших випадках.

|  |
| --- |
|  |
| versions code |
