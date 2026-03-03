---
layout: post
title: "Нотатка: concurrent.futures.ProcessPoolExecutor  pyinstaller та python.  freeze_support()"
date: 2023-09-11 20:41:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2023/09/concurrentfuturesprocesspoolexecutor.html
---

Знайшов проблему використання concurrent.futures.ProcessPoolExecutor з pyinstaller.

Програма випадала у такі помилки.

|  |
| --- |
|  |
| concurrent.futures.ProcessPoolExecutor з pyinstaller |

  

За рішенням: <https://stackoverflow.com/questions/28631288/concurrent-futures-works-well-in-command-line-not-when-compiled-with-pyinstal>

Допомогло використання:

```
from multiprocessing import freeze_support


def main()
    freeze_support()
    ...
```
