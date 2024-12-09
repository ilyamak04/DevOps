[Cheat sheet](https://gist.github.com/Kartones/dd3ff5ec5ea238d4c546)

Общие команды

- `sudo -u postgres psql postgres`
- `CREATE DATABASE имя_базы_данных;`
- `DROP DATABASE mydatabase;`
- `psql -U postgres`


- `\l` - просмотр списка бд
- `\q` - выход

--- 
Работа внутри конкретной бд 
- `\c имя_базы_данных [имя_пользователя]` - подключение к бд или из bash `psql -U username -d database_name`

- `\dt` - просмотр списка таблиц
- `\d [имя таблицы]` - показывает столбцы, типы данных и индексы
- `\dn` - список схем
- `\dt schema_name.*` - список таблиц в схеме
---

- `CREATE TABLE имя_таблицы (id SERIAL PRIMARY KEY, колонка1 тип, колонка2 тип, ...);` - создание таблицы

- `CREATE USER имя_пользователя WITH PASSWORD 'пароль';`
- `CREATE ROLE имя_роли;`
- `GRANT ALL PRIVILEGES ON DATABASE имя_базы TO имя_пользователя;`
- `DROP USER имя_пользователя;`
- `DROP ROLE имя_роли;`

--- 

- `SELECT * FROM pg_stat_activity;` - просмотр текущий подключений
- `SELECT * FROM pg_locks;` - просмотр блокировок
--- 
- `psql -U postgres -d my_database -f ./script.sql` - запуск скрипта

- `/usr/pgsql-12/bin/pg_dump -U u_ensi -t address_several_juridical_entity -h 127.0.0.1 db_ensi | gzip > <dir>/address_several_juridical_entity.sql.gz` - снятие дампов

- `/zcat /tmp/SvJul_4_07.sql.gz | /usr/pgsql-12/bin/psql -h 127.0.0.1 -U u_ensi -d db_ensi -t SvJul_4_07 >> ensi.log` - восстановление таблицы
---
`ALTER TABLE` — это команда в SQL, которая используется для изменения структуры уже существующей таблицы в базе данных.
```bash
ALTER TABLE имя_таблицы
ADD имя_столбца тип_данных;
```
```bash
ALTER TABLE имя_таблицы
DROP COLUMN имя_столбца;
```
```bash
ALTER TABLE имя_таблицы
ALTER COLUMN имя_столбца SET DATA TYPE новый_тип;
```
```bash 
ALTER TABLE имя_таблицы
ADD CONSTRAINT имя_ограничения тип_ограничения (столбец);
```

```bash
ALTER TABLE employees
ADD CONSTRAINT age_check CHECK (age >= 18);
```


SELECT pid, usename, datname, client_addr, client_port, application_name, state
FROM pg_stat_activity;


SELECT usename, client_addr, client_port, state
FROM pg_stat_activity;


psql -U postgres -t -c "SELECT datname FROM pg_database WHERE datistemplate = false;"

psql -U postgres -d postgres -c "SELECT datname FROM pg_database WHERE datistemplate = false;" -t -A


--- 

`SELECT pg_terminate_backend( pid ) FROM pg_stat_activity WHERE pid <> pg_backend_pid( ) AND datname = 'name of database';` - отключить все подключения к бд

---

`ALTER DATABASE "downloader8097" RENAME TO "downloader_vo_8097";` - переименовать БД

--- 
Анализ БД 

```
SELECT pid, usename, datname, client_addr, client_port, application_name, state
FROM pg_stat_activity
WHERE datname = current_database();
```
```
SELECT pid,
	   now() - query_start AS runtime_query,
       state,
       query,
       usename,
       datname
FROM pg_stat_activity
WHERE state = 'active' and datname = current_database()
ORDER BY runtime_query DESC;
``` 
```
SELECT pid,
	   now() - xact_start  AS transaction_time,
       state,
       query,
       usename,
       datname
FROM pg_stat_activity
WHERE state IN ('active', 'idle in transaction') and datname = current_database()
ORDER BY transaction_time DESC;
```
```
SELECT 
    pid,                                       -- Идентификатор процесса
    state,                                     -- Состояние процесса
    now() - query_start AS query_runtime,      -- Время выполнения текущего запроса
    now() - xact_start AS transaction_runtime, -- Время выполнения транзакции
    query,                                     -- Текущий запрос
    usename,                                   -- Пользователь
    datname,                                   -- База данных
    xact_start                                 -- Время начала транзакции
FROM pg_stat_activity
WHERE state IN ('active', 'idle in transaction') 
  AND datname = current_database()              
ORDER BY transaction_runtime DESC;              
```
```
# использование памяти и ресурсов
SELECT *
FROM pg_stat_database;
```
- `datid` — Идентификатор базы данных (OID).
- `datname` — Имя базы данных.
- `numbackends` — Количество активных соединений (клиентов) с этой базой.
- `xact_commit` — Общее количество успешно завершённых транзакций.
- `xact_rollback` — Общее количество откатов транзакций.
- `blks_read` — Количество блоков, считанных с диска.
- `blks_hit` — Количество блоков, найденных в кеше (попадания в shared_buffers).
- `tup_returned` — Количество строк, возвращённых клиенту.
- `tup_fetched` — Количество строк, извлечённых (например, SELECT).
- `tup_inserted` — Количество вставленных строк (INSERT).
- `tup_updated` — Количество обновлённых строк (UPDATE).
- `tup_deleted` — Количество удалённых строк (DELETE).
- `conflicts` — Количество конфликтов (например, из-за репликации).
- `temp_files` — Количество временных файлов, созданных сервером.
- `temp_bytes` — Количество данных, записанных во временные файлы (в байтах).
- `deadlocks` — Количество взаимоблокировок.
- `blk_read_time` — Общее время чтения блоков с диска (в миллисекундах).
- `blk_write_time` — Общее время записи блоков на диск (в миллисекундах).

```
# системная инфа
SELECT *
FROM pg_stat_bgwriter;
```
```
# Блокировки
SELECT 
 blocked_locks.pid AS blocked_pid,
 blocked_activity.usename AS blocked_user,
 blocking_locks.pid AS blocking_pid,
 blocking_activity.usename AS blocking_user,
 blocking_activity.state AS blocking_state,
 blocked_activity.query AS blocked_statement,
 blocking_activity.query AS current_statement_in_blocking_process,
 blocked_locks.locktype AS blocked_locktype, 
 blocking_locks.locktype AS blocking_locktype
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity 
  ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
  ON blocking_locks.locktype = blocked_locks.locktype
 AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
 AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
 AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
 AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
 AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
 AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
 AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
 AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
 AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
 AND blocking_locks.pid != blocked_locks.pid 
JOIN pg_catalog.pg_stat_activity blocking_activity 
  ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.GRANTED;
```
```
EXPLAIN ANALYZE
SELECT <ваш_запрос>;
```
```
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE pid = <process_id>;
```