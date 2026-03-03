---
layout: post
title: "Нотатка: Python, logger,  модулі та спільні logging Handlers"
date: 2023-09-26 00:53:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/09/python-logger-logging-handlers.html
---

[![](/assets/images/blog/Screenshot%202023-09-26%20035327.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgTG-hOlPpzQ8wNGjYGZA8BsM2HxXhxDyU4RESCmSBqCgEOgOA3xkSzD--Gl-6_kmA4mjRdXyDsj8uR1aF9UwynUNyrdFff9DVENcQ3xnK7KpAX1XD3BE_oI6Jiy5jEAsGFVVAu4fnNunvWqLnBsmYmgtUsMv_IUcovo5uURR-edKzLqHo9wN5n3tQjA_v7/s633/Screenshot%202023-09-26%20035327.png)

### Задача

У Python налаштувати спеціальний logger, з єдиними налаштунками для всіх модулів пакету.

Файли проєкту : <https://github.com/lexxai/DemoLogger>

  

package/logger.py:

```
import logging
from pathlib import Path

logged_format = "%(asctime)s [%(levelname)s] #%(process)s (%(name)s) %(message)s"
logged_format_date = "%Y-%m-%d %H:%M:%S"
logged_formatter = logging.Formatter(logged_format, datefmt=logged_format_date)


def get_logger(name: str, log_path: Path | None = None) -> logging:
    log_path: Path = Path() if log_path is None else log_path
    log_path.mkdir(exist_ok=True, parents=True)

    # create file handler for DEBUG, INFO
    file_name = "debug.log"
    file_path = log_path.joinpath(file_name)
    logged_handler_file = logging.FileHandler(str(file_path))
    logged_handler_file.setLevel(logging.DEBUG)
    logged_handler_file.setFormatter(logged_formatter)

    # create file handler for ERRORS
    file_name = "error.log"
    file_path = log_path.joinpath(file_name)
    logged_handler_error_file = logging.FileHandler(str(file_path))
    logged_handler_error_file.setLevel(logging.ERROR)
    logged_handler_error_file.setFormatter(logged_formatter)

    # create console handler INFO, ERROR
    logged_handler_stream = logging.StreamHandler()
    logged_handler_stream.setLevel(logging.INFO)
    logged_handler_stream.setFormatter(logged_formatter)

    # create main logger for all child modules of thi package
    my_logger: logging = logging.getLogger(name)
    my_logger.addHandler(logged_handler_file)
    my_logger.addHandler(logged_handler_error_file)
    my_logger.addHandler(logged_handler_stream)
    my_logger.setLevel(logging.DEBUG)
    return my_logger
```

package/module1.py:

```
import logging

logger: logging = logging.getLogger(__name__)

def some_code1():
    logger.info("INFO")
    logger.debug("DEBUG")
    logger.error("ERROR")
```

package/module2.py:

```
import logging

logger: logging = logging.getLogger(__name__)

def some_code2():
    logger.info("INFO")
    logger.debug("DEBUG")
    logger.error("ERROR")
```

package/main.py:

```
import logging
from pathlib import Path

from package.logger import get_logger
from package.module1 import some_code1
from package.module2 import some_code2


def init_logger(arg_log_path: Path):
    return get_logger("package", arg_log_path)


logger: logging

if __name__ == "__main__":
    logger = init_logger(Path("logs"))
    logger.info("INFO")
    logger.debug("DEBUG")
    logger.error("ERROR")
    some_code1()
    some_code2()
```

RESULT CONSOLE:

```
2023-09-26 03:28:29 [INFO] #15964 (package) INFO
2023-09-26 03:28:29 [ERROR] #15964 (package) ERROR
2023-09-26 03:28:29 [INFO] #15964 (package.module1) INFO M1
2023-09-26 03:28:29 [ERROR] #15964 (package.module1) ERROR M1
2023-09-26 03:28:29 [INFO] #15964 (package.module2) INFO M2
2023-09-26 03:28:29 [ERROR] #15964 (package.module2) ERROR M2
```

RESULT FILE : logs/debug.log

```
2023-09-26 03:28:29 [INFO] #15964 (package) INFO
2023-09-26 03:28:29 [DEBUG] #15964 (package) DEBUG
2023-09-26 03:28:29 [ERROR] #15964 (package) ERROR
2023-09-26 03:28:29 [INFO] #15964 (package.module1) INFO M1
2023-09-26 03:28:29 [DEBUG] #15964 (package.module1) DEBUG M1
2023-09-26 03:28:29 [ERROR] #15964 (package.module1) ERROR M1
2023-09-26 03:28:29 [INFO] #15964 (package.module2) INFO M2
2023-09-26 03:28:29 [DEBUG] #15964 (package.module2) DEBUG M2
2023-09-26 03:28:29 [ERROR] #15964 (package.module2) ERROR M2
```

RESULT FILE : logs/error.log

```
2023-09-26 03:28:29 [ERROR] #15964 (package) ERROR
2023-09-26 03:28:29 [ERROR] #15964 (package.module1) ERROR M1
2023-09-26 03:28:29 [ERROR] #15964 (package.module2) ERROR M2 
```

Вся сила у тому що ім'я підлеглих loggers будуються в залежності від імені кореневого файлу з конфігурацію.

Так  це для :

* main.py = ''packge"
* package/module1.py  = "package.module1"
* package/module2.py  = "package.module2"

Тому зробивши налаштування у  logging.getLogger( ''packge"), ці налаштування отримають  всі підлеглі "package.module1", "package.module1",  "package.\*"

За матеріалами:

* [Logging cookbook](https://docs.python.org/3/howto/logging-cookbook.html#logging-cookbook)
