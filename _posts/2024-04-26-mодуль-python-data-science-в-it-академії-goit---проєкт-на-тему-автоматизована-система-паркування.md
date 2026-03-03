---
layout: post
title: "Mодуль \"Python Data Science\"  в IT академії GoIT - проєкт на тему \"Автоматизована система паркування\"."
date: 2024-04-26 16:35:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/04/m-python-data-science-it-goit.html
---

💡 Закінчили навчання за останнім третім модулем "[Python](https://uk.wikipedia.org/wiki/Python)
[Data Science](https://uk.wikipedia.org/wiki/%D0%9D%D0%B0%D1%83%D0%BA%D0%B0_%D0%BF%D1%80%D0%BE_%D0%B4%D0%B0%D0%BD%D1%96)" в[IT академії GoIT - start your career in IT](https://www.linkedin.com/school/goit-start-your-career-in-it/?originalSubdomain=ua).  
🚩 Було створено команду 🇺🇦 розробників - "Neuron Curve Bobr".  
🚗 В
результаті командної роботи був виконанний груповий проєкт на тему
"Автоматизована система паркування".  
  
Дякую 🔥 команді за нашу спільну
роботу впродовж напружених 🔨 двох тижнів.  

#### Курсовий проєкт на тему "Автоматизована система паркування".

Згідно з технічним завданням на проєкт: Система автоматично може визначати
номери автомобільних знаків на зображеннях з використанням методів машинного
навчання.  
  

### Функції застосунку:

* Управління обліковими записами користувачів
* Візуалізація про кількість вільних та зайнятих паркомісць.
* Приймання зображень від користувача (симуляція роботи відеокамери)
* Виявлення та виділення області з номерним знаком
* Детекція номерного знаку
* Розпізнавання символів для ідентифікації тексту номерного знаку
* Перевірка номера авто у базі зареєстрованих транспортних засобів
* Запис часу в'їзду/виїзду кожного разу, коли визначається номерний знак
* Відстеження тривалості паркування
* Розрахунок загальної тривалості паркування
* Зберігання даних про тривалість паркування в базі даних;
* Розрахунок вартості паркування. Керування тарифами. Чек за
  [QR-кодом](https://uk.wikipedia.org/wiki/QR-%D0%BA%D0%BE%D0%B4);
* Генерація звітів про розрахунки, які можна експортувати у форматі
  [CSV](https://uk.wikipedia.org/wiki/CSV)
* Оплата послуги паркування

### | | | --- | | | | Проєкт на тему "[Автоматизована система паркування]( https://drive.google.com/file/d/1LBgz3OKKVZoJlBMAKc7QhZdnN__WX0LO/view?usp=drive_link)". |

### Data sciense & Computer vision

Пошук зони знаку використовує алгоритми
[computer vision](https://uk.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BC%D0%BF%27%D1%8E%D1%82%D0%B5%D1%80%D0%BD%D0%B8%D0%B9_%D0%B7%D1%96%D1%80)
модуля [OpenCV](https://opencv.org/) -
[Haar Cascades](https://docs.opencv.org/4.9.0/db/d28/tutorial_cascade_classifier.html).   
А розпізнавання кожного номеру реалізовано класифікацією зображення
на 36 класів з використанням простої моделі з використанням згорткової
нейронної мережі
[CNN](https://uk.wikipedia.org/wiki/%D0%97%D0%B3%D0%BE%D1%80%D1%82%D0%BA%D0%BE%D0%B2%D0%B0_%D0%BD%D0%B5%D0%B9%D1%80%D0%BE%D0%BD%D0%BD%D0%B0_%D0%BC%D0%B5%D1%80%D0%B5%D0%B6%D0%B0)
в
[Keras](https://www.tensorflow.org/guide/keras)
([Tensorflow](https://www.tensorflow.org/)).

Модель тренована була на власних додаткових зображеннях для поліпшення
якості розпізнавання і виявлення проблем з подібними знаками такими як
літера О і цифра 0.

В процесі вибору алгоритму були протестовані різні рішення, вибрано цей
метод за умовною простотою і гарними результатами.

|  |
| --- |
|  |
| CNN модель |

За причини конфліктів версій модулів для Tensoflow було обмежено максимальну
версію Python до версії 3.11.

```
tensorflow-io-gcs-filesystem==0.31.0 ; python_version >= "3.10" and python_version < "3.12"
```

### Front-end

З метою швидкості реалізації візуальної частини у межах посталеного терміну
на проєкт два тижні було вибрано фреймворк
[Django](https://www.djangoproject.com/). В цьому
проєкті використано версію Django 5.0.4.  
Але були заділи  та ідеї
на використання фреймворку
[FastAPI](https://fastapi.tiangolo.com/) для
реалізації  чистого backend, є початкова гілка у гіт проєктів для
цього, але час є час.  
Django прекрасно справлявся з задачею побудови та
виявлення змін у моделі  бази даних створюючи версійність міграцій
змін. У ролі бази даних використано
[Postgress](https://www.postgresql.org/).

|  |
| --- |
|  |
| Проєкт у [Visual Studio Code](https://code.visualstudio.com/) |

  

|  |
| --- |
|  |
| Структура бази даних |

  

### Sheduler

Для роботи додаткового функціонала для системи  сповіщень та роботу
Telegram Bot використано власний модуль sheduler.py  якій виконується
як окремий процес.  
З навчальною метою та без наявності постійного
хостингу модуль Telegram Bot працює використовуючи метод pooling без
використання web-hooks.  
Для зберігання даних необхідних для роботи
бота, використано системний файловий кеш Django, але в ідеалі мав би бути
використаний зовнішній сервер Redis або сервер Memcache.

  

### CI/CD

В проєкті реалізовано найпростіше рішення для CI/CD що дозволило повністю
реалізувати автоматичне розгортання на орендованому сервері.   
Для
неперервної інтеграції (англ. Continuous Integration) -
[CI](https://uk.wikipedia.org/wiki/%D0%9D%D0%B5%D0%BF%D0%B5%D1%80%D0%B5%D1%80%D0%B2%D0%BD%D0%B0_%D1%96%D0%BD%D1%82%D0%B5%D0%B3%D1%80%D0%B0%D1%86%D1%96%D1%8F)
використано функціонал GitHub Action для перевірки коду на можливі помилки
перед кожним merge у кожному pull-request від розробників команди.   
Для
безперервного доставлення (англ. Continuous delivery) -
[CD](https://uk.wikipedia.org/wiki/%D0%91%D0%B5%D0%B7%D0%BF%D0%B5%D1%80%D0%B5%D1%80%D0%B2%D0%BD%D0%B0_%D0%B4%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BA%D0%B0)
використано власний
[bash](https://uk.wikipedia.org/wiki/Bash)
скрипт  котрий перевіряє гілку
[git](https://uk.wikipedia.org/wiki/Git) за розкладом у
[cron](https://uk.wikipedia.org/wiki/Cron), та
перебудовує
[docker](https://uk.wikipedia.org/wiki/Docker)
контейнери у разі виявлення змін.  

### Посилання

* GitHub:
  [Проєкт](https://github.com/lexxai/GoIT-DS-TeamProject-PlateN)
* YouTube:
  [запис презентації при захисті проєкту](https://youtu.be/iH9t5hpYwqI)
* YouTube:
  [встановлення та інтерфейс](https://youtu.be/BkJmMh1Nvzk)
* TikTok: [встановлення та інтерфейс](https://vm.tiktok.com/ZMMpcjdCV/)
* Live Demo: <https://fastparking.x-server.net>

### Склад команди:

1. Team Lead:
   [Artem Dorofeev](https://www.linkedin.com/in/artem-dorofeev-ua/). [GitHub](https://github.com/artem-dorofeev)
2. Scrum Master:
   [Rostyslav Bysko](https://www.linkedin.com/in/rostyslav-bysko-8328a251/). [GitHub](https://github.com/RostislavB74)
3. Django expert:
   [Tetiana Chyrkova](https://www.linkedin.com/in/tetiana-chyrkova-ab9481273/). [GitHub](https://github.com/lumi-ua),
   [GitHub](https://github.com/diixo)
4. Full stack devops :
   [Lex Xai](https://www.linkedin.com/in/lexxai/).  [GitHub](https://github.com/lexxai)
5. Art director:
   [Евгений Плахотин](https://www.linkedin.com/in/plahotin033/). [GitHub](https://github.com/plaha303)
6. Tester:
   [Oleksander Borovyk](https://www.linkedin.com/in/oleksander-borovyk/).  [GitHub](https://github.com/AlexanderBgit)

### Youtube

  
  

### Слайд шоу

[![](/assets/images/blog/01.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhhxyNpmPIq4MIQpzNaXx_cFgeNksNbU3LKh5FNlzbeAlxKwlfi-MfULfpcFVErGHWmYj7eDUjiIB3798jBoMiW_rxoICuU10NsjA64j33VNz4KPuhgITIrGVmyluQSxKyhkPIEmWAGwT_WafYeqPQLQuJAr0ZUeKzP8CTxMtrLr9UJz5sOhEk_J9yaI93P/s960/01.png)

  

[![](/assets/images/blog/02.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhsQ_L1bGcw4iKiQyLlSgjGzSSk0twrLED0TuKUSCKeoBILg77iQGpyY235AxMpHKDJgktLCgZioajyACZ3utDq77L0SYHzYMvyfZEpX9NZ5rESOuuuOuaq3eUICpNKeejfKSzwGXBqWMSjcdhMBIonav2ccZPJf7kssaOH_jTSad_AVY-eYlvL8c3fkMk9/s960/02.png)

  

[![](/assets/images/blog/03.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg1l_yphl4flyA6iL1umshFiTIWBsoJkuiooZvaTnL9OfQ1wWaL9PAlGBbW_zBdxzcD985T-1Nn97HV3AC5_RlqVIS_JlKbS2_6vQIgEjtFUHHQ4kMZcsTz_rzBbkbwfEhaVJGiLf1ox9xl9vKL-S-VWlkuZsp7hm8TdpPbaYwo6qvjPDo6oKvF0bn-4K0F/s960/03.png)

  

[![](/assets/images/blog/04.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgCvxX08esCg1zJ2rEeHNxzVVhdDXQgaI6hgNkXgzmbl7xZ8TNeLRVXiy4zgbUFs64CqrmP-hrIMgCHdUA1rorh7daEn76xOaBxSao4Zbegiz8DPjJ4hZVhiNXrkqWq_FThztJgi58Mf_qN3Erwz2bZ-47yevcn5rdJF1den9zppbl_lUqv4vbBQ5tPI9jD/s960/04.png)

  

[![](/assets/images/blog/05.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgbp7z3Be2P2UcCVcQW_qKncoEUlcDyK3qkFnBYOhbQ8RibjB7XOWBw2mFZ0DiaSy5o2fI8D0bQADz9Z41ThsKSnH5FrOm8ebeDKwmsr56D9c0LyrNMX6VfLrKgpfmibngOoGzKEB7AmJag7Y8cdSyTol8FpCq5M9212yGf7M6_1MpXWlcmLoe3ts5AUYiN/s960/05.png)

  

[![](/assets/images/blog/06.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEio54Ch8Fks6TOY499rfYzaUejo9T5JIbInSVV1o3ZS7aUPF2h9pJ2zZV7tbtvpHyUVW7jyUM9_dFhKtks5gTyJDB_CBjZWTffGqOc2RHOY0CqA5WZB_sFmaHPttriACwtzBMCLx0Iss8zF-KMrz6FJOTrwP0-DvxAMyOVlAPRGRAyrsBK1kaecXtDRrPeQ/s960/06.png)

  

[![](/assets/images/blog/07.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjpsSlSUdrOuoZe8XgPOHopH-VJ4wd-9oBv1Gs9dggyPWlSvnWin5bQYow7FCloV5hgBi7dkpNkjsR9CQly1948-0oVJ_nu1IYSX6e0q6UUAiTWPosmiuNm5F62Bj7LQU62xDPGbies1pbgS8frNzYgRJ6wpssN6q84mmzLI7bN1WrLh7ObYUg4fypgqDgw/s960/07.png)

  

[![](/assets/images/blog/08.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjOFnoZqc48VFhB66IRESL1PeG_ZKz2WFxN_WA-VtRz9_mUjm_B1ZJSdSwaD-efuIxuZ7q9eD2xeFXQTVB8QqcwNpWxS33OVoZC5gnuL8BHoVGhyphenhyphenkfvnivnfspGwYhhYDhdGcGgHOAEyP1WvblX0fN7Eq5jKiYEnUyXrCPj4EuvijLgsuunGdRJgFdhRkAf/s960/08.png)

  

[![](/assets/images/blog/09.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjX1k3yJ2dbd7osLuc7FcVNnH_yhlNfZxbASoKEVFnqOWM6jm5FH91T1MmxwrPfKZ0lh0XkPoBGfoMxlNM2AY4fW4C2zaYYXRvJjT5iJJ1JnJuE2NyEdEV8O1baq2NAj648Ea4teT40dQUDVJnwALXfHtDI2mH3oiVXEBL8cHJH69-GvBQ2n3ModZpszfZn/s960/09.png)

  

[![](/assets/images/blog/10.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgcE1Birc28Z4KBjm8OasRNVEoFVDb4d6BDrlDVRSYoNTLNoyte6qcH4eOB6A3qBhX731qsESqSImgNloT3k9ZhIntuk0TIEpAw2LhPwk3vTURa-jc3QpR93wKZU7kHmovopF58eB6C3WZjgpysHwc760q5NFQewdRdvv4ALI_8j6M5PJQmKy_f5wPVnJHR/s960/10.png)

  

[![](/assets/images/blog/11.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjKJM0OoXSaXiySllcHGCgxnRbjDcrFgRrwTqH0tfsA2CeHn23pjwf6TolcOtfB_RGrdPEB90d7KAJJehUucFHAZqS13cdGuTOBY1f1cIV1_OxBviHJ-C3K8Pll1QlXPT555pj-Zw-OvkobAJsDuyTVfgtW-rzWEBnx2QaOS0UuVTaOFJ6Y7L_QnMyjiP8s/s960/11.png)

  

[![](/assets/images/blog/12.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgQOACxlJqUkjvViEBkyyLLzUB067r9xkO6RKj0yVO19jTb4QwzOxUbSPHmiRU3zhph77HoBFlCavoE-nrbF3Zte0vZoOinxSGkTCKVobN6Qv7mVHsfE6zWX-gJsWwvR4gWkJWG7wflUjKaF2RfMTapqQSd4O0JVtEKkemVyKid2HKVe9X4mAddg5WkRuAE/s960/12.png)

  

[![](/assets/images/blog/13.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEglp24NC9fZcQ4l-7AJFrOrBKqPhg7BrdA_puEU5xEJM_543R6MF0KtSp3xNzlCbKKoOmVJ0atuJWXPZPyDdGUaVwKuwMQknDykh2H8rakkfkLNfK8UVT-HJ7tYV1hRmfQVs1tN5HKX-tvDkwaYITVwEpaK_pI0Yzjwnz1OkCSOmb-euZFZiaonR0ur_KD1/s960/13.png)

  

[![](/assets/images/blog/14.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiS259sKlw1615Doy66T0LJx9t2bMrxOVa5H9fCOOW-3kiO_Cm8XJsiHJUCniSLRLrBDfIu_1nQdxeGRHuqK5vPRzySPaG9d7hbPiM9wyyi6ggwis4lY9RTT28TOpo9Tx0U3xRdiFPBkUX2Hs8m9vUANrdfWVdJajOCyIkP8-VhU0s3fm51NHYMnps0gWdj/s960/14.png)

  

[![](/assets/images/blog/15.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj7glebtCFMfwn9gOGqi_vJNxuLUdogdMaBCSvffDos_09Pc-BYXOGZ-p35uwFVocpKXKXT0YQydssJE5yHKnzp2rpS4gJzNEpjOJXA7LDvY2ixNesdLs5zYMw2K2ll1B0YZsF0m8xsW0zyMoWXuqchllRhkGJkXQhcAAVDjC4sHHq7r_lskRZjcDr7Wzcy/s960/15.png)

  

[![](/assets/images/blog/16.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjrraTD92Nm8Dl2QzlpSme96DbJJEiLr09Eb4PiziVZei2VXdmgBgtIO9ExAQYN9T8W_fwoSH4IpSAZB1F7rF78kuLkPqR_gsZVr_trIlAHi2nb7wh5cUQBcV8X5PZUEisCOLAl4aPCS-D8cqJdv8fNG7TaM95e_DUnSgFTCmAnMfWxQktIlTtEnX9qe9Vd/s960/16.png)

  

[![](/assets/images/blog/17.png)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgILOSvTRLfLRUX_qiYde4acNz2tBSFsbeehMAP7-AJ4BzPzW45XuoOIb0wzke-2xjQ5Tqu0bXcp2Nco3QUfzNHHz9BKXcGAS6-M1IoFHx3kMTtnV7XalcRMqEgunDp_6a2zt9iUfwXfNCMlqqLmViMwxVwXJO8PsP51yWFgjGvA9BpDnYnBHUxk-3tANi4/s960/17.png)

  
[PDF]( https://drive.google.com/file/d/1LBgz3OKKVZoJlBMAKc7QhZdnN__WX0LO/view?usp=drive_link)  
  
[#python](https://www.linkedin.com/feed/hashtag/?keywords=python&highlightedUpdateUrns=urn%3Ali%3Aactivity%3A7190691132626870272) [#datascience](https://www.linkedin.com/feed/hashtag/?keywords=datascience&highlightedUpdateUrns=urn%3Ali%3Aactivity%3A7190691132626870272) [#machinelearning](https://www.linkedin.com/feed/hashtag/?keywords=machinelearning&highlightedUpdateUrns=urn%3Ali%3Aactivity%3A7190691132626870272) [#django](https://www.linkedin.com/feed/hashtag/?keywords=django&highlightedUpdateUrns=urn%3Ali%3Aactivity%3A7190691132626870272) [#tensorflow](https://www.linkedin.com/feed/hashtag/?keywords=tensorflow&highlightedUpdateUrns=urn%3Ali%3Aactivity%3A7190691132626870272) [#opencv](https://www.linkedin.com/feed/hashtag/?keywords=opencv&highlightedUpdateUrns=urn%3Ali%3Aactivity%3A7190691132626870272)
