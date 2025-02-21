#### Создание юнита

- в каталоге для хранения unit файлов `cd /etc/systemd/system/` создаём файл с расширением `.service` для службы
- ```bash
  sudo vi /etc/systemd/system/myservice.service
  ```
- пишем `unit` файл
```bash
[Unit]
Description=Описание сервиса
After=network.target  # Определяет, когда сервис должен запускаться (например, после сети)

[Service]
ExecStart=/path/to/your/executable_or_script  # Команда для запуска сервиса
ExecStop=/path/to/stop/script (необязательно)  # Команда для остановки сервиса, если нужно
ExecReload=/path/to/reload/script (необязательно)  # Команда для перезагрузки
User=your_user  # Пользователь, от которого запустится сервис (если нужно)
Group=your_group  # Группа (если нужно)
Restart=always  # Политика перезапуска (может быть "on-failure", "always", "no")
RestartSec=5  # Задержка перед перезапуском
Environment="ENV_VAR=value"  # Можно задавать переменные окружения

[Install]
WantedBy=multi-user.target  # Указывает, что сервис должен запускаться в multi-user режиме (стандартный для большинства серверов)
```
>  `multi-user` - многопользовательский текстовый режим
- `sudo systemctl daemon-reload` - обновить конфигурацию systemd
- `sudo systemctl enable myservice.service` - включает автозапуск юнита при перезагрузке системы
- `sudo systemctl start myservice.service` - запустить юнит
- `sudo systemctl status myservice.service` - статус юнита

#### Где же живут эти ваши юниты 

- `/etc/systemd/system/` - юниты созданные пользователем
- `/usr/lib/systemd/system/ или /lib/systemd/system/` - юниты, которые создаются менеджерами пакетов при установке ПО
- `/run/systemd/system/` - временные юниты, созданные во время работы чего-то (например, контейнера) 


