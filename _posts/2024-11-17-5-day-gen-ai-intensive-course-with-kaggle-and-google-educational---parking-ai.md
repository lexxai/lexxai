---
layout: post
title: "5-Day Gen AI Intensive Course with Kaggle and Google. Educational - Parking AI."
date: 2024-11-17 17:57:00 +0000
tags: []
blogger_orig_link: https://lexxai.blogspot.com/2024/11/5-day-gen-ai-intensive-course-with.html
---

Ось і закінчився
[5 денний інтенсив на тему Генеративного ШІ](https://www.kaggle.com/discussions/general/545082).   
Захопило це після прослуховування LiveStreams кожного дня, але у запису
коли зʼявлялися субтитри котрі вже можна перекласти.  
По закінченню отримав
такий
[бейдж від Kaggle](https://www.kaggle.com/lexxai)
😉

|  |
| --- |
|  |
| Badge Kaggle |

З позитивних моментів те що захотілося спробувати отриману інформацію
особисто, і спробував з моделювати найпростішого помічника до
[нашого навчального проєкту IT академії GoIT - на тему "Автоматизована
система паркування"](https://lexxai.github.io/2024/04/m-python-data-science-it-goit.html).

Оформив простий Colab projet
[ParkingAI.ipynb](https://colab.research.google.com/drive/1yj30lBghix7LeVH45E0pBZzxjJ6uPrsg?usp=sharing). Для його виконання необхідно отримати GOOGLE\_API\_KEY від
[aistudio.google.com](https://aistudio.google.com/app/apikey?authuser=0&utm_source=colab.research.google.com&utm_medium=referral&utm_campaign=colab-integration&utm_content=owned-promo)
і додати до секретних ключів.  Зверніть увагу на список країн де AI
Studio може працювати.

### Ось ядро логіки роботи з models/gemini-1.5-flash

|  |
| --- |
|  |
| In general the state diagram is. |

### 

```
# https://github.com/google-gemini/generative-ai-python
# https://colab.research.google.com/github/google-gemini/cookbook/blob/main/quickstarts/Function_calling.ipynb
# List of tools/functions for the parking AI system with explicit parameter types

parking_tools = [get_free_space, get_statistics, get_user_preferences, check_parked_car]

# Instruction to the model on how to use the tools
instruction = """
You are a helpful AI assistant for a smart parking app. Users can ask you questions about the app's features,
their parking status, and how to use the app. Use the available tools (get_free_space, get_statistics,
get_user_preferences, check_parked_car) to access relevant information and provide clear and concise answers.
"""

model = genai.GenerativeModel(
    "models/gemini-1.5-flash-latest",
    tools=parking_tools,
    system_instruction=instruction,
    )
user_query = input("Ask your question (commands: [e]xit, [h]istory): ")
response = chat.send_message(user_query)
print(response.text)
```

### Приклади запитів:

```
Ask your question (commands: [e]xit, [h]istory): user 23 what preferred place ?

    User 23 prefers Zone A and Zone B.

Ask your question (commands: [e]xit, [h]istory): user 24 and 33 ?

    User 24 prefers Zone B and Zone C. There is no preference on record for user 33.

Ask your question (commands: [e]xit, [h]istory): h
user -> [{'text': 'user 23 what preferred place ?'}]
--------------------------------------------------------------------------------
model -> [{'function_call': {'name': 'get_user_preferences', 'args': {'user_id': 23.0}}}]
--------------------------------------------------------------------------------
user -> [{'function_response': {'name': 'get_user_preferences', 'response': {'preferred_locations': ['Zone A', 'Zone B']}}}]
--------------------------------------------------------------------------------
model -> [{'text': 'User 23 prefers Zone A and Zone B.\n'}]
--------------------------------------------------------------------------------
user -> [{'text': 'user 24 and 33 ?'}]
--------------------------------------------------------------------------------
model -> [{'function_call': {'name': 'get_user_preferences', 'args': {'user_id': 24.0}}}, {'function_call': {'name': 'get_user_preferences', 'args': {'user_id': 33.0}}}]
--------------------------------------------------------------------------------
user -> [{'function_response': {'name': 'get_user_preferences', 'response': {'preferred_locations': ['Zone B', 'Zone C']}}}, {'function_response': {'name': 'get_user_preferences', 'response': {'preferred_locations': []}}}]
--------------------------------------------------------------------------------
model -> [{'text': 'User 24 prefers Zone B and Zone C.  There is no preference on record for user 33.\n'}]
--------------------------------------------------------------------------------
Ask your question (commands: [e]xit, [h]istory): q
Exiting the chat. Goodbye!
```

```
Ask your question (type 'exit' to quit): What you can?
I can:

* Tell you how many free parking spaces are available.
* Provide overall usage statistics for the app.
* Tell you your preferred parking locations (if you provide your user ID).
* Check if your car is currently parked (if you provide your license plate number).

Ask your question (type 'exit' to quit): how many parking space ?
There are currently 10 free parking spaces.

Ask your question (type 'exit' to quit): My car ABC123
OK.  And what about your car with license plate ABC123?  Do you want to know if it's currently parked?

Ask your question (type 'exit' to quit): yes 
Yes, the car with license plate ABC123 is currently parked.

Ask your question (type 'exit' to quit): What about car AJJS11, is parked ?
No, the car with license plate AJJS11 is not currently parked.

Ask your question (type 'exit' to quit): Скільки вільних місць зараз є?
There are currently 10 free parking spaces.

Ask your question (type 'exit' to quit): Відповідай мені укаріїнською мовою.
Гаразд, я буду відповідати українською.  Задавайте питання!

Ask your question (type 'exit' to quit):  Скільки вільних місць зараз є? І який номер моєї машини?
Зараз є 10 вільних паркувальних місць.  Повідомте мені номер вашої машини, щоб я міг перевірити, чи вона припаркована.

Ask your question (type 'exit' to quit): Я вже відповідав, чи ти не мамʼятаєш ?
Вибачте, я не маю пам'яті попередніх розмов.  Для того, щоб перевірити, чи ваша машина припаркована, будь ласка, вкажіть її номерний знак.

Ask your question (type 'exit' to quit): ABC123
Так, автомобіль з номером ABC123 припаркований.

Ask your question (type 'exit' to quit): Яка сататистика використання ?
Загальна кількість зареєстрованих користувачів становить 100. Середнє використання додатку за тиждень складає 5.
```

```
Аsk your question (type 'exit' to quit, 'history' for history show): history
Ask your question (type 'exit' to quit, 'history' for history show): Мова спілкування українська
Добре, я буду спілкуватися з вами українською.  Задавайте питання.

Ask your question (type 'exit' to quit, 'history' for history show): Номер моєї машини ABC123, вона припаркована ?
Так, ваш автомобіль з номером ABC123 припаркований.

Ask your question (type 'exit' to quit, 'history' for history show): history
user -> [{'text': 'Мова спілкування українська'}]
--------------------------------------------------------------------------------
model -> [{'text': 'Добре, я буду спілкуватися з вами українською.  Задавайте питання.\n'}]
--------------------------------------------------------------------------------
user -> [{'text': 'Номер моєї машини ABC123, вона припаркована ?'}]
--------------------------------------------------------------------------------
model -> [{'text': 'Перевіряю...\n\n'}, {'function_call': {'name': 'check_parked_car', 'args': {'license_plate': 'ABC123'}}}]
--------------------------------------------------------------------------------
user -> [{'function_response': {'name': 'check_parked_car', 'response': {'is_parked': True}}}]
--------------------------------------------------------------------------------
model -> [{'text': 'Так, ваш автомобіль з номером ABC123 припаркований.\n'}]
--------------------------------------------------------------------------------
Ask your question (type 'exit' to quit, 'history' for history show): яка моя улюблена зона ?
Для того, щоб відповісти на це питання, мені потрібен ваш ідентифікатор користувача.  Будь ласка, вкажіть його.

Ask your question (type 'exit' to quit, 'history' for history show): 23
Ваші улюблені зони паркування - це Зона A та Зона B.

Ask your question (type 'exit' to quit, 'history' for history show): history
user -> [{'text': 'Мова спілкування українська'}]
--------------------------------------------------------------------------------
model -> [{'text': 'Добре, я буду спілкуватися з вами українською.  Задавайте питання.\n'}]
--------------------------------------------------------------------------------
user -> [{'text': 'Номер моєї машини ABC123, вона припаркована ?'}]
--------------------------------------------------------------------------------
model -> [{'text': 'Перевіряю...\n\n'}, {'function_call': {'name': 'check_parked_car', 'args': {'license_plate': 'ABC123'}}}]
--------------------------------------------------------------------------------
user -> [{'function_response': {'name': 'check_parked_car', 'response': {'is_parked': True}}}]
--------------------------------------------------------------------------------
model -> [{'text': 'Так, ваш автомобіль з номером ABC123 припаркований.\n'}]
--------------------------------------------------------------------------------
user -> [{'text': 'яка моя улюблена зона ?'}]
--------------------------------------------------------------------------------
model -> [{'text': 'Для того, щоб відповісти на це питання, мені потрібен ваш ідентифікатор користувача.  Будь ласка, вкажіть його.\n'}]
--------------------------------------------------------------------------------
user -> [{'text': '23'}]
--------------------------------------------------------------------------------
model -> [{'text': 'Добре, перевіряю ваші налаштування...\n\n'}, {'function_call': {'name': 'get_user_preferences', 'args': {'user_id': 23.0}}}, {'text': '\n'}]
--------------------------------------------------------------------------------
user -> [{'function_response': {'name': 'get_user_preferences', 'response': {'preferred_locations': ['Zone A', 'Zone B']}}}]
--------------------------------------------------------------------------------
model -> [{'text': 'Ваші улюблені зони паркування - це Зона A та Зона B.\n'}]
--------------------------------------------------------------------------------
```

#### Що було на курсі?

#### 

### [Day 1 Assignments] 5-Day Gen AI Intensive:

💡What You’ll Learn  
  
Today you’ll explore the evolution of LLMs,
from transformers to techniques like fine-tuning and inference acceleration.
You’ll also get trained in the art of prompt engineering for optimal LLM
interaction.  
  
The code lab will walk you through getting started
with the Gemini API and cover several prompt techniques and how different
parameters impact the prompts.

📼
[Day 1 Livestream with Paige Bailey](https://www.youtube.com/watch?v=kpRyiJUUFxY)

### [Day 2 Assignments] 5-Day Gen AI Intensive:

💡 What You’ll Learn  
  
Today you will learn about the conceptual
underpinning of embeddings and vector databases and how they can be used to
bring live or specialist data into your LLM application. You’ll also explore
their geometrical powers for classifying and comparing textual data.

 📼
[Day 2 Livestream with Paige Bailey](https://www.youtube.com/watch?v=86GZC56rQCc)

### [Day 3 Assignments] 5-Day Gen AI Intensive:

💡 What You’ll Learn  
  
Learn to build sophisticated AI agents by
understanding their core components and the iterative development process.  
  
The
code labs cover how to connect LLMs to existing systems and to the real world.
Learn about function calling by giving SQL tools to a chatbot, and learn how
to build a LangGraph agent that takes orders in a café.

 📼
[Day 3 Livestream with Paige Bailey](https://www.youtube.com/watch?v=HQUtMWoTAD4)

### [Day 4 Assignments] 5-Day Gen AI Intensive:

💡 What You’ll Learn  
  
In today’s reading, you’ll delve into the
creation and application of specialized LLMs like SecLM and MedLM/Med-PaLM,
with insights from the researchers who built them.  
  
In the code labs
you will learn how to add real world data to a model beyond its knowledge
cut-off by grounding with Google Search.  You will also learn how to
fine-tune a custom Gemini model using your own labeled data to solve custom
tasks.

 📼
[Day 4 Livestream with Paige Bailey](https://www.youtube.com/watch?v=odvuLMJWUSU)

### [Day 5 Assignments] 5-Day Gen AI Intensive:

💡 What You’ll Learn  
  
Discover how to adapt MLOps practices for
Generative AI and leverage Vertex AI's tools for foundation models and
generative AI applications.

 📼
[Day 5 Livestream with Paige Bailey](https://www.youtube.com/watch?v=uCFW0i9xrBc)
