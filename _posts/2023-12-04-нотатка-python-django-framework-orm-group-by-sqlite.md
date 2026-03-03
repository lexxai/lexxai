---
layout: post
title: "Нотатка: Python. Django framework. ORM \"GROUP BY\". SQLite."
date: 2023-12-04 03:49:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/12/python-django-framework-orm-group-by.html
---

### Django SQL GROUP BY.

А Ви знали що його не має в ORM у прямому вигляді ?

```
SELECT x.* FROM logs x WHERE x.username = 'user1' GROUP BY x.host
```

### DBeaver тестування

Є таблиця log доступу користувачів:

```
CREATE TABLE logs (
	id INTEGER PRIMARY KEY AUTOINCREMENT,
	date DATETIME,
	host VARCHAR,
	request VARCHAR,
	username VARCHAR
);

CREATE INDEX logs_host_IDX ON logs (host);
```

|  |
| --- |
|  |
| SEED LOG TABLE |

Стала проста задача для SQL запиту. Для отримання унікальних записів IP адрес
з яких отримував доступ певний користувач.

  

```
SELECT x.* FROM logs x WHERE x.username = 'user1' GROUP BY x.host
```

DBeaver:

|  |
| --- |
|  |
| DBeaver SQL 'GROUP BY' |

### DJANGO реалізація.

### 

```
> poetry init
> poetry shell
> poetry add django
> django-admin startproject groupby  
> cd groupby  
> python manage.py migrate
```

groupby\loganalyze\models.py:

```
class Log(models.Model):
    date = models.DateTimeField(default=timezone.now)
    host = models.CharField(max_length=128)
    request = models.CharField(max_length=128)
    username = models.CharField(max_length=128)

    class Meta:
        indexes = [
            models.Index(fields=["host"], name="host_idx")
        ]
```

```
> python manage.py makemigrations  
Migrations for 'loganalyze':  
  loganalyze\migrations\0001_initial.py  
    - Create model Log
```

```
> python manage.py migrate  
Operations to perform:  
  Apply all migrations: admin, auth, contenttypes, loganalyze, sessions  
Running migrations:  
  Applying loganalyze.0001_initial... OK 
```

Далі на GitHub
<https://github.com/lexxai/python_django_groupby>

|  |
| --- |
|  |
| Тільки користувач "user1" |

|  |
| --- |
|  |
| Тільки користувач "user1" |

```
    # Filter "user1" only  
    data0 = Log.objects.filter(username__exact = "user1")  
      
    # Select from previous result only column with name 'host' and get unique value for it with   
    # calculate minimum value on 'id' column, it should be first raw only.  
    grouped_records = data0.values('host').distinct().annotate(min_id=Min("id"))  
  
    # From clear table select founded rows by founded "id" lists.  
    data =  Log.objects.filter(id__in=grouped_records.values_list("min_id", flat=True))
```

|  |
| --- |
|  |
| Тільки користувач "user1" з унікальними IP адресами |

Django log:

```
December 04, 2023 - 05:15:32  
Django version 4.2.7, using settings 'groupby.settings'  
Starting development server at http://127.0.0.1:8000/  
Quit the server with CTRL-BREAK.  
  
(0.000) SELECT "loganalyze_log"."id", "loganalyze_log"."date", "loganalyze_log"."host", "loganalyze_log"."request", "loganalyze_log"."username" FROM "loganalyze_log" WHERE "loganalyze_log"."id" IN (SELECT DISTINCT MIN(U0."id") AS "min_id" FROM "loganalyze_log" U0 WHERE U0."username" = 'user1' GROUP BY U0."host"); args=('user1',); alias=default
```

<https://github.com/lexxai/python_django_groupby>
