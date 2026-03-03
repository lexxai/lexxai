---
layout: post
title: "Як завершити async-завдання, створене з функції що блокує через asyncio.to_thread(): експеримент та рішення."
date: 2024-12-25 02:03:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/12/async-asynciotothread.html
---

[![](/assets/images/blog/DALL%C2%B7E%202024-12-25%2003.59.33%20-%20An%20illustrative%20experiment%20showing%20how%20to%20terminate%20an%20async%20task%20created%20from%20a%20blocking%20function%20using%20asyncio.to_thread().%20The%20scene%20depicts%20two%20di.webp)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEizPzQyXDhCW3-WNZxk02uT_1-Q7iCtK_X1UEpdOoT_qUjMagCDPHQsnOdoApx9yYsdj2h5aSRJvkpuuKAc3vbDU4V1XnnXNYsWt7uLFgEyVWGg88jd8hvp7EWPjvBBe0LD2GYsPzRGoh3OCRU-eaRlV6WKR3T7Jy0u4Usb6S48nf6Dyndw_ZqOFm9NfEra/s1024/DALL%C2%B7E%202024-12-25%2003.59.33%20-%20An%20illustrative%20experiment%20showing%20how%20to%20terminate%20an%20async%20task%20created%20from%20a%20blocking%20function%20using%20asyncio.to_thread().%20The%20scene%20depicts%20two%20di.webp)

Маленький експеримент як завершити [*async task*](https://docs.python.org/uk/3.13/library/asyncio-task.html#creating-tasks) якщо вона була створена з функції що блокує.

Тобто створена з не асинхронної функції (proc2) за допомогою *[asyncio.to\_thread()](https://docs.python.org/uk/3.13/library/asyncio-task.html#asyncio.to_thread)*.

Як видно з результатів без використання *[threading.Event()](https://docs.python.org/uk/3.13/library/threading.html#threading.Event)*, програма буде постійно працювати навіть після закінчення всіх асинхронних функцій в *[async loop](https://docs.python.org/uk/3.13/library/asyncio-eventloop.html)*, тому що окремий потік породжений у цій програмі ще не завершився й [*async loop*](https://docs.python.org/uk/3.13/library/asyncio-eventloop.html) буде чекати на нього.
