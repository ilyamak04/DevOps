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
