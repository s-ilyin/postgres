
## Ильин Сергей Евгеньевич
### Урок 1. Архитектура PostgreSQL. Поток 2

#### ДЗ

1. Развернуть ВМ (Linux) с PostgreSQL (у вас есть ВМ в ВБ, любой другой способ, в т.ч.
докер)

2. Залить Тайские перевозки в минимальном варианте

3. Посчитать количество поездок - select count(*) from book.tickets;

4. Не забываем ВМ остановить/удалить


Решение:

1. ```docker run -dit --name pg-master -p 6000:5432 --network pg-network -e POSTGRES_PASSWORD=dev_dev -e POSTGRES_USER=postgres -e PGDATA=/var/lib/postgresql/data/pg_data postgres:16```

2. ```wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql -h localhost -p 6000 -U postgres < thai.sql```

3. ```5185505 - select count(*) from book.tickets;```

4. ```docker stop pg-master```