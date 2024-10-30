- `ps -ef | grep -v grep | grep postgres` - определяем, есть ли процесс постгрес 
- `which psql` - есть ли в локальном окружении 
- `ss -tulpn` - ищем порт на котором слушает постгрес
- `sudo docker ps` - смотрим, какие контейнеры запущены
- `docker ps --format "{{.Names}}"` - только имена контейнеров 
- `sudo docker inspect <имя_контейнера> | grep postgres` - есть ли постгрес в контейнере

- `psql -V` или `/usr/bin/psql -V` - версия постгрес
- `psql -U postgres -c "SELECT pg_database.datname AS database_name, pg_size_pretty(pg_database_size(pg_database.datname)) AS size FROM pg_database;"` - смотрим базы и их размер
- `psql -U postgres -c "SELECT pg_size_pretty(SUM(pg_database_size(pg_database.datname))) AS total_size FROM pg_database;"` - общий размер всех баз

- `psql -U postgres -c "SHOW data_directory;"`

- `df -sh <path_to_data_dir>` - размер директории с данными баз, логами, конфигами, транзакциями и т.д.
- `psql -U postgres -c "SHOW port;"`

---
`sudo find / -type f -name "*.backup" -o -name "*.sql" -o -name "*.dump"` - этой командой можно поискать бэкапы

--- 
- `docker volume ls`
- `docker volume inspect <volume_name>`
- `docker inspect <container_name> | grep -i mount`
- `docker inspect <container_name> | grep -i volumes`
- `docker port <container_name>`
- `docker inspect <container_name> | grep -i port`
 