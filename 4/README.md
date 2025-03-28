## Ильин Сергей Евгеньевич
### Урок 4. Блокировки». Поток 2

#### ДЗ

1. Создать таблицу accounts(id integer, amount numeric);
2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).
3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.

#### Решение

1. ```create table wb.accounts(id integer, amount numeric);```
2. ```insert into wb.accounts values (1,100.00), (2,150.00), (3,200.00);```
3. консоль 1: ```begin;update wb.accounts set amount = amount + 1 where id = 1;```
4. консоль 2: ```begin;update wb.accounts set amount = amount + 1 where id = 2;```
5. консоль 1: ```begin;update wb.accounts set amount = amount + 1 where id = 2;```
6. консоль 2: ```begin;update wb.accounts set amount = amount + 1 where id = 1;```
7. ```ERROR:  deadlock detected DETAIL:  Process 24986 waits for ShareLock on transaction 947;blocked by process 24975. Process 24975 waits for ShareLock on transaction 948; blocked by process 24986. HINT:  See server log for query details. CONTEXT:  while updating tuple (0,1) in relation "accounts"```