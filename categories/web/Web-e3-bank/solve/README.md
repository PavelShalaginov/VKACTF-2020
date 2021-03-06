## Банк

| Событие | Название | Категория | Сложность |
|:--------|:---------|:----------|----------:|
| VKA-CTF`2020 | Банк | Web | КМБ |

### Описание
> Автор: [ 𝕂𝕣𝕒𝕦𝕤𝕖 ]
>
> Недавний [проект](http://army-invest.vkactf.tk) от Департамента Обороны, позволявший военнослужащим получать проценты от вкладов, ВНЕЗАПНО обанкротился. Нам удалось воссоздать его копию, чтобы вы выяснили причины произошедшего инцидента.

> Копия: http://army-invest-two.vkactf.tk

### Решение

![](imgs/1.PNG)

На странице с вкладами `/invest/` в коде страницы видим js метод `window.setInterval`, который ежесекундно выполняет функцию `update`
```js
intervalId = window.setInterval(update, 1000)
```
Если попытаться найти где объявлена функция `update`, то можно найти обфусцированный `main.js` файл

Используя [JavaScript Beautifier](https://beautifier.io/) и консоль разработчика вашего браузера, можно понять, что функция `update` делает запрос на `/update_balance/` и получает в ответ json
```json
{'balance': 150.0, 'invest': 100, 'proc': 1.0}
```
где `balance` это доход, `invest` это вклад клиента, а `proc` это 1 процент от вклада, который ежесекундно увеличивает доход

Увеличение дохода происходит после отправки POST запроса. Следовательно, можно написать [скрипт](solve_update_balance.py), который будет выполнять запросы быстрее, что позволит накопить необходимую сумму достаточно быстро

**Флаг:**

> vka{1_4l50_w4n7_70_r4153_m0n3y_7h15_w4y}