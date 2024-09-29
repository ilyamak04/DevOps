### Конфигурация ssh сервера 
- `ssh-keygen -t ed25519 -f ~/.ssh/<path>` - создаёт пару ssh-ключей (лучше защитить ключ паролем)
- `ssh-copy-id -i ~/.ssh/<path> <username>@<servername>`- добавляет публичный ключ на сервер  
- В файле `/etc/ssh/sshd_config` должны быть настройки (после изменения файла перезагрузить демон ssh `systemctl restart sshd`):
    - PermitRootLogin no - исключает возможность подключения на сервер под рутом
    - PubkeyAuthentication yes
    - PasswordAuthentication no
### Конфигурация ssh клиента 
- `vi ~/.ssh/config` - создаёт файл конфигурации 
- Поместить в файл (заполнить своими данными):
```
Host *.com
        User user
        ForwardAgent yes
        IdentityFile /Users/username/.ssh/id_rsa
        IdentitiesOnly yes
        AddKeysToAgent yes
        ServerAliveInterval 60
        ServerAliveCountMax 1200
```
### SSH Agent
SSH Agent выбирает ssh-ключи для подключения к соответствующим серверам

Также использование ssh-agent позволяет вводить пароль от ключа только один раз, при первом подключении к серверу

`eval $(ssh-agent)` - запускает ssh-agent

`ssh-add ~/.ssh/id_rsa` - добавляет ключ в ssh-agent
