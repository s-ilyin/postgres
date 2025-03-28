### Внутренний курс по Postgresql. ПОТОК 2

Полезные команды:
```sql
// получить живые строки, мертвые, последний автовакуум
SELECT relname, n_live_tup, n_dead_tup,
trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'test';
```

```psql
// Посмотреть размер файла с таблицей
\dt+ test
```