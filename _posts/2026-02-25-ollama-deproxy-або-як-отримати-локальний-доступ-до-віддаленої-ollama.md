---
layout: post
title: "Ollama DeProxy або як отримати локальний доступ до віддаленої Ollama"
date: 2026-02-25 15:43:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2026/02/ollama-deproxy-ollama.html
---

|  |
| --- |
|  |
| Ollama DeProxy |

Я маю віддалений сервер із GPU, на якому запущена Ollama.

Водночас середовище розробки зазвичай очікує, що Ollama доступна локально -
наприклад, за адресою *http://localhost:11434* або в межах локальної мережі
(*http://192.168.0.111:11434*).

### Класичні рішення

Найпростіший варіант — SSH-тунель:

```
ssh remote@server -L 11434:localhost:11434
```

Після цього локальний *localhost:11434* проксуватиметься на віддалений сервер.

Якщо ж розробників декілька і вони працюють з різних офісів або через інтернет
— можна використати VPN. Це теж робоче рішення, але воно потребує додаткової
інфраструктури та адміністрування.

### Проблема з авторизацією

У моєму випадку Ollama використовується разом із OpenWebUI, який проксіює
доступ до Ollama через власний API з токен-авторизацією.

Однак більшість застосунків, що інтегруються з Ollama, очікують простий доступ
до http://localhost:11434 без жодної авторизації. Через це вони не можуть
напряму працювати через OpenWebUI.

### Рішення — Ollama DeProxy

Щоб спростити інтеграцію, я написав невеликий застосунок -
**[Ollama DeProxy](ollama-deproxy)**.

Його ідея проста:

* локально він виглядає як звичайна Ollama (localhost:11434);
* всередині - проксіює запити на віддалений сервер;
* за потреби додає авторизацію;
* дозволяє централізовано керувати доступом.

Таким чином:

* розробникам не потрібно налаштовувати SSH-тунелі;
* не обов’язково піднімати VPN;
* клієнтські застосунки продовжують працювати так, ніби Ollama запущена
  локально.

У результаті віддалений GPU-сервер використовується прозоро, без змін у
клієнтському коді.

### Репозиторій:

 **Ollama DeProxy** -
<https://github.com/lexxai/ollama-deproxy>  
  

### Приклади використання: | | | --- | | | | Docker container Ollama DeProxy |

### | | | --- | | | | VS Code + Continue + Ollama DeProxy | | | | --- | | | | VS Code + Copilot GitHub (Old version) + Ollama DeProxy |

|  |
| --- |
|  |
| VS Code + Copilot GitHub (0.37.9) + Ollama DeProxy (CORRECT\_NUMBERED\_MODEL\_NAMES=True) |

  

### За матеріалами:

* [Оllama](https://github.com/ollama/ollama)
* [openwebui.com - api proxy](https://docs.openwebui.com/reference/api-endpoints#-ollama-api-proxy-support)
* [ollama-deproxy](https://github.com/lexxai/ollama-deproxy)
