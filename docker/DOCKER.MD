- Параметры для `restart`
    - `no` (или не указывать restart вовсе) — контейнер не будет перезапускаться автоматически. Это значение по умолчанию.
    - `always` — контейнер будет автоматически перезапущен при любом завершении работы (даже при ручной остановке через docker stop). Он также перезапустится после перезагрузки Docker или хоста
    - `on-failure` — контейнер будет перезапущен только в случае выхода с ненулевым статусом (ошибкой). Вы можете также указать максимальное количество перезапусков, например,`on-failure:3`.
    - `unless-stopped` — контейнер будет автоматически перезапущен при сбоях и после перезагрузки Docker или системы, но не перезапустится, если его остановили вручную.

- `docker stats`
    - CONTAINER ID - Уникальный идентификатор контейнера.
    - NAME - Имя контейнера.
    - CPU % - Использование CPU в процентах.
    - MEM USAGE / LIMIT	- Использование памяти / установленный лимит.
    - MEM %	-   Доля использования памяти от лимита.
    - NET I/O - Сетевой ввод/вывод данных.
    - BLOCK I/O	- Объем операций ввода/вывода на диске.
    - PIDS	- Количество процессов внутри контейнера.

- `docker info`

- `docker volume ls`
- `docker volume inspect <volume_name>`
- `docker inspect <container_name> | grep -i mount`
- `docker inspect <container_name> | grep -i volumes`
- `docker port <container_name>`
- `docker inspect <container_name> | grep -i port`