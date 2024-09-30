### Базовая настройка "чистого" сервера (ubuntu)
- Подключитесь к серверу с вашего компьютера через SSH
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
username ALL=(ALL:ALL) NOPASSWD: /usr/bin/systemctl  # Для юзера
%group ALL=(ALL:ALL) NOPASSWD: /usr/bin/example_command # Для группы
```
> нужно указать полные путь к команде, чтобы узнать его: `which example_command`