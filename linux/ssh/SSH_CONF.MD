- `ssh-keygen -t ed25519 -f ~/.ssh/<path>` - создаёт пару ssh-ключей (лучше защитить ключ паролем)
- `ssh-copy-id -i ~/.ssh/<path> <username>@<servername>`- добавляет публичный ключ на сервер  
### Конфигурация ssh сервера 

- В файле `/etc/ssh/sshd_config` должны быть настройки (после изменения файла перезагрузить демон ssh `systemctl restart sshd`):
    - PermitRootLogin no - исключает возможность подключения на сервер под рутом
    - PubkeyAuthentication yes
    - PasswordAuthentication no
-  Публичные ключи хранятся в директории `~/.ssh/authorized_keys`
- Директория `~/.ssh` должна иметь права 700.
- Файл `authorized_keys` должен иметь права 600.

### Конфигурация ssh клиента 
- `vi ~/.ssh/config` - создаёт файл конфигурации 
- Поместить в файл (заполнить своими данными):
```bash 
Host myserver
        HostName 213.171.31.5 # или *.com
        User user
        ForwardAgent yes    # Перенаправляем SSH-агент на удалённый сервер
        IdentityFile /Users/username/.ssh/id_rsa
        IdentitiesOnly yes  # Используем только указанный ключ для аутентификации
        AddKeysToAgent yes  # Добавляем ключ в SSH-агент после первого использования
        ServerAliveInterval 60
        ServerAliveCountMax 1200
```
### SSH Agent
SSH Agent выбирает ssh-ключи для подключения к соответствующим серверам

Также использование ssh-agent позволяет вводить пароль от ключа только один раз, при первом подключении к серверу

`eval $(ssh-agent)` - запускает ssh-agent

`ssh-add ~/.ssh/id_rsa` - добавляет ключ в ssh-agent
