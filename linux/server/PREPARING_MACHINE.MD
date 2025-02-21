### Базовая настройка "чистого" сервера (ubuntu\debian)
- Подключиться с локальной машины к серверу по shh
    ```
    ssh root@server_ip
    ```
- Обновить пакеты на сервере
    ```
    apt update && apt upgrade -y
    ```
- Создать нового пользователя
    ```
    adduser user
    # passwd для смены пароля пользователя
    ```
- Добавить в группу `sudo`
    ```
    usermod -aG sudo user
    ```
- Настроить [**ssh**](../ssh/SSH_CONF.MD)

- Можно установить **ufw**, он упрощает настройку правил файрвола, через **iptables** сложнее

---
Немного про **sudo**
- `sudo visudo` - используется для безопасного редактирования файла конфигурации `sudoers`

- Проверить дополнительные файлы конфигурации 
```bash
sudo ls /etc/sudoers.d
sudo visudo -f /etc/sudoers.d/filename
```
- Можно разрешить выполнять команды, которые требуют привелегий суперюзера без запроса пароля 
```bash
# Для юзера
username ALL=(ALL:ALL) NOPASSWD: /usr/bin/systemctl  
# Для группы
%group ALL=(ALL:ALL) NOPASSWD: /usr/bin/example_command 
```
> нужно указать абсолютный путь к команде, чтобы узнать его: `which example_command`