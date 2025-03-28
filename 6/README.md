## Ильин Сергей Евгеньевич
### Урок 6. Физическая и логическая репликация. Поток 2

#### ДЗ

#### Решение
Немного отличается, но каскадную репликацию настроил

## Создаем пг-шки
master 
```bash 
docker run -dit --name pg-master -p 6000:5432 --network pg-network -e POSTGRES_PASSWORD=dev_dev -e POSTGRES_USER=postgres -e PGDATA=/var/lib/postgresql/data/pg_data postgres:17
```

sync rpl
```bash
docker run -dit --name pg-slave -p 6001:5432 --network pg-network -e POSTGRES_PASSWORD=dev_dev -e POSTGRES_USER=postgres -e PGDATA=/var/lib/postgresql/data/pg_data postgres:17
```

async rpl 
```bash
docker run -dit --name pg-async-slave -p 6002:5432 --network pg-network -e POSTGRES_PASSWORD=dev_dev -e POSTGRES_USER=postgres -e PGDATA=/var/lib/postgresql/data/pg_data postgres:17
```

## Настроем master

настройка postgres.conf
```conf
// настройка postgres.conf
wal_level = replica
max_wal_senders = 10 
max_replication_slots = 10
```
подключимся
```bash 
docker exec -it pg-master su - postgres -c psql
``` 

создадим роля для репликации лоя репликации
```sql 
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'dev_dev';
```
создаем слот для надежности
```sql
SELECT pg_create_physical_replication_slot('test');
```

получаем маску подсети 
```bash
docker network inspect pg-network | grep Subnet
```
настройка pg_hba.conf 
```conf
host replication replicator 172.18.0.0/16 scram-sha-256
```

создадим таблицу с данными
```sql 
create table public.users(id varchar(8));
insert into public.users values ('1'), ('2');
``` 

обновим мастер
```bash
docker restart pg-master
```


## Настроем синхронной реплики

коннектимся
```
docker exec -it pg-slave bash
```

теперь магия - делаем раз, два, три
```bash
rm -rf /var/lib/postgresql/data/pg_data
pg_basebackup -h pg-master -p 5432 -D /var/lib/postgresql/data/pg_data -U replicator -R -S test
```

обновим конфиги в pg-sync-slave в postgres.conf
```
primary_conninfo = host=pg-master port=5432 user=replicator password=dev_dev application_name=pg-slave
```

## Настроем асинхронной реплики

коннектимся
```
docker exec -it pg-async-slave bash
```

теперь магия - делаем раз, два, три
```bash
rm -rf /var/lib/postgresql/data/pg_data
pg_basebackup -h pg-master -p 5432 -D /var/lib/postgresql/data/pg_data -U replicator -R -S test
```

обновим конфиги в pg-async-slave в postgres.conf
```
primary_conninfo = host=pg-master port=5432 user=replicator password=dev_dev application_name=pg-slave
```

## Проверка master

```bash
docker exec -it pg-master su - postgres -c psql
```

в мастере postgresql.conf
```
```bash
synchronous_commit = on
synchronous_standby_names = 'FIRST 1 (pg-slave)'
```

```sql
select application_name, sync_state from pg_stat_replication;
select pg_reload_conf();
```

Вроде все, ничего не забыл! С докером промучался очень сильно :)