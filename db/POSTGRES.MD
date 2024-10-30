[Cheat sheet](https://gist.github.com/Kartones/dd3ff5ec5ea238d4c546)

- `sudo -u postgres psql postgres`
- `CREATE DATABASE имя_базы_данных;`
- `\l` - просмотр списка бд
- `\q` - выход

--- 
- `\c имя_базы_данных [имя_пользователя]` - подключение к бд или из bash `psql -U username -d database_name`

- `\dt` - просмотр списка таблиц
- `\d [имя таблицы]` - показывает столбцы, типы данных и индексы
- `\dn` - список схем
- `\dt schema_name.*` - список таблиц в схеме
---

- `CREATE TABLE имя_таблицы (id SERIAL PRIMARY KEY, колонка1 тип, колонка2 тип, ...);` - создание таблицы

- `DROP TABLE имя_таблицы;`
- `CREATE USER имя_пользователя WITH PASSWORD 'пароль';`
- `CREATE ROLE имя_роли;`
- `GRANT ALL PRIVILEGES ON DATABASE имя_базы TO имя_пользователя;`
- `DROP USER имя_пользователя;`
- `DROP ROLE имя_роли;`

--- 

- `SELECT * FROM pg_stat_activity;` - просмотр текущий подключений
- `SELECT * FROM pg_locks;` - просмотр блокировок


