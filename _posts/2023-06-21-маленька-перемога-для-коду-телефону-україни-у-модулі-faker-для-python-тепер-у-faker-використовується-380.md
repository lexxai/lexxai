---
layout: post
title: "Маленька перемога для коду телефону України у модулі Faker для Python. Тепер у Faker використовується: +380"
date: 2023-06-21 01:42:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/06/faker-python-faker-380.html
---

Знайдено було невідповідність у модулі [Faker](https://github.com/joke2k/faker) для Python запису [телефонного коду України +380](/2017/08/380-38.html).

Виправив, подав заявку, прийняли.

|  |
| --- |
|  |
| commit до faker - прийнято та об'єднано |

  
  

|  |
| --- |
|  |
| Нова версія faker v.18.11.1 з підтримкою +380 |

### Тестування з faker==18.11.0

```
(.venv-win-3913) ~ pip install faker==18.11.0               
Collecting faker==18.11.0  
  Downloading Faker-18.11.0-py3-none-any.whl (1.7 MB)  
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.7/1.7 MB 730.1 kB/s eta 0:00:00  
Requirement already satisfied: python-dateutil>=2.4 in .venv-win-3913\lib\site-packages (from faker==18.11.0) (2.8.2)  
Requirement already satisfied: six>=1.5 in .venv-win-3913\lib\site-packages (from python-dateutil>=2.4->faker==18.11.0) (1.16.0)  
Installing collected packages: faker  
Successfully installed faker-18.11.0  
 
```

```
(.venv-win-3913) ~ python test_faker.py       
+38 (068) 218-86-12  
041 561-44-60  
+38 042 641-22-27  
004 383-81-94  
+38 (075) 916-40-90  
+38 021 451-89-78  
+38 041 198 21 07  
610-40-16  
365-06-53  
+38 074 499-71-69
```

```
(.venv-win-3913) ~ pip uninstall Faker   
Found existing installation: Faker 18.11.0  
Uninstalling Faker-18.11.0:  
  Would remove:  
    .venv-win-3913\lib\site-packages\faker-18.11.0.dist-info\*  
    .venv-win-3913\lib\site-packages\faker\*  
    .venv-win-3913\scripts\faker.exe  
Proceed (Y/n)? y  
  Successfully uninstalled Faker-18.11.0
```

### Тестування з faker останньої версії

```
(.venv-win-3913) ~ pip install faker
Collecting faker
  Downloading Faker-18.11.1-py3-none-any.whl (1.7 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.7/1.7 MB 1.0 MB/s eta 0:00:00
Collecting python-dateutil>=2.4 (from faker)
  Using cached python_dateutil-2.8.2-py2.py3-none-any.whl (247 kB)
Collecting six>=1.5 (from python-dateutil>=2.4->faker)
  Using cached six-1.16.0-py2.py3-none-any.whl (11 kB)
Installing collected packages: six, python-dateutil, faker
Successfully installed faker-18.11.1 python-dateutil-2.8.2 six-1.16.0
```

```
(.venv-win-3913) ~ python test_faker.py  
+380 02 027-32-25  
047 273-10-40  
311-38-59  
+380 (47) 634-44-69  
764 22 36  
+380 10 467-80-44  
059 869 77 78  
729-90-62  
+380 66 226-18-56  
+380 (10) 460-16-76 
```

Місію виконано.
