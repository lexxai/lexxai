---
layout: post
title: "Python Enum, keys(), values(), items() with cached values using lru_cache."
date: 2024-11-07 16:13:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/11/python-enum-keys-values-items-with.html
---

Читаючи
[How to get all values from python enum class?](https://stackoverflow.com/questions/29503339/how-to-get-all-values-from-python-enum-class)
знайшов цікаве для себе рішення для застосування
[class enum.Enum Python](https://docs.python.org/uk/3/library/enum.html#enum.Enum)
у парі з
[@functools.lru\_cache](https://docs.python.org/uk/3/library/functools.html#functools.lru_cache) .

[![](/assets/images/blog/code.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhCX_xnAkl3TVUs75Y5z4lGKkD93SR6sd4qcwcA03tJbKpu-Zh5_na-oCEkAkg_EfL-gqibv_ApAgSlyzyBFMGPQ38MYcrFbGLdsN1ABTjLujQkUjn0ErCL9vvE2xKYtPMUbvFlIxvTwNsrX2X-DKRaBLe4M35W3zyrs2w3dIOvbXEGwXfnl0yySfrr-Gnj/s1706/code.png)

  

#### Продовжуючи гратися, інші методи:
