## Странное соединение 

| Событие | Название | Категория | Сложность |
|:--------|:---------|:----------|----------:|
| VKA-CTF`2020 | Логи | Форенсика | АБИТУРА  |


### Описание
> Автор: WaffeSoul
> Старенький кафедральный компьютер доживает свои последние деньки. Его по-хорошему надо бы уже давно списать, но начальство приказало ему долго жить. Но все что вы можете, так это посмотреть логи, вдруг что-нибудь найдете.

### Решение 

Дан файл с логами, 100к сторок. Написано, что все события good. Значит надо искать обратное. Пишим не большой скрипт по поиску строки где нет good .
```file = open('system.log','r')
for line in file:
    index = line.find('good')
    if index == -1:
        print(line)
file.close()```

Получаем строку 

>[13057.082628]	System event {766B617B6C3036355F77336C6C5F31665F345F6C3137376C337D} - evil

Переводи hex в text получаем флаг.

**Флаг:**

>vka{l065_w3ll_1f_4_l177l3}
