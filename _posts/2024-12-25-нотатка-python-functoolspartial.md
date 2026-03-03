---
layout: post
title: "Нотатка. Python. functools.partial."
date: 2024-12-25 22:16:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/12/python-functoolspartial.html
---

🔨 [functools.partial(func, /, \*args, \*\*keywords)](https://docs.python.org/uk/3.13/library/functools.html#functools.partial)

|  |
| --- |
|  |
| functools.partial |

📍
Повертає новий частковий об’єкт, який під час виклику поводитиметься
як func, що викликається з позиційними аргументами *args*і ключовими
аргументами *keywords*. Якщо до виклику надається більше аргументів, вони
додаються до *args*. Якщо надаються додаткові ключові аргументи, вони
розширюють і замінюють ключові слова.

📌
Примітка. [Більшість функцій планування asyncio не дозволяють передавати ключові аргументи](https://docs.python.org/uk/3.13/library/asyncio-eventloop.html). Для цього скористайтеся *[functools.partial()](https://docs.python.org/uk/3.13/library/functools.html#functools.partial)*:

```
# will schedule "print("Hello", flush=True)"
loop.call_soon(
   functools.partial(print, "Hello", flush=True)
   )
```

🎧 Використання часткових об’єктів зазвичай зручніше, ніж використання
[лямбда-виразів](https://acode.com.ua/lambda-python/), оскільки asyncio може краще відтворювати часткові
об’єкти в повідомленнях про налагодження та помилки.

```
# лямбда-вираз
loop.call_soon(
   lambda : print("Hello")
   )
```
