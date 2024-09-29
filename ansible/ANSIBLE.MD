- Устанавливаем ansible на машину 
```bash 
# Для Ubuntu
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```
- Создаём инветарный файл в котором описываются хосты (сервера), которыми будет управлять ansible 
```ini
[web_servers]
archers ansible_host=176.119.89.45  ansible_user=archer  ansible_private_key=/home/ilyamak04/.ssh/archers
```
#### Про инвентарный файл (что можно делать?)
- Разбивать сервера на группы
```ini
[dev_servers]
webserver1 ansible_host=192.168.1.10
webserver2 ansible_host=192.168.1.11
[prod_servers]
webserver3 ansible_host=192.168.1.12
webserver4 ansible_host=192.168.1.13
```
[dev_servers] — это группа серверов, которая представляет собой логическую коллекцию хостов

- Создавать групповые переменные 
```ini
[web_servers:vars]
ansible_user=ubuntu
ansible_port=22
```
Это задаст пользователя и порт по умолчанию для всех серверов в группе web_servers
- Параметры подключения: Можно управлять параметрами подключения для отдельных серверов или групп:
    - `ansible_user`: имя пользователя для SSH
    - `ansible_port`: порт для SSH-подключения
    - `ansible_host`: IP-адрес или имя хоста
    - `ansible_ssh_private_key_file`: путь к файлу с приватным ключом для SSH
    - `ansible_become`: определяет, нужно ли использовать привилегированные права (sudo) для выполнения задач
- Группы групп
```ini
[production:children]
web_servers
db_servers
```
- Команды:

    `ansible-inventory -i hosts.ini --list` - отображает список хостов, распределение хостов по группам в json-овидном формате

    `ansible-inventory -i hosts.ini --graph` - отображает список хостов, распределение хостов по группам в древовидном формате
> Ansible case sensetive!

> По умолчанию любой сервер входит в 2 группы: 1) all 2) ungrouped или пользовательская группа

#### Про конфигурационный файл `ansible.cfg`
- Ansible ищет конфигурационный файл в нескольких местах, в следующем порядке при запуске:

    1. В текущем каталоге — если существует файл ansible.cfg в директории, из которой запускается Ansible.
    2. В домашней директории пользователя — файл ~/.ansible.cfg.
    3. В системной директории — обычно /etc/ansible/ansible.cfg.

Ansible будет использовать первый найденный конфигурационный файл. Это позволяет иметь разные конфигурации для разных проектов.

Файл ansible.cfg лучше хранить в проекте, инфраструктуру когорого менеджерит ansible, а вот инвентарный файл hosts.ini можно хранить где угодно на Ansible-Master хосте, главное в .cfg задать путь к hosts.ini

#### Основные секции `ansible.cfg`
- [defaults] - Это основная секция для установки параметров по умолчанию
```ini
[defaults]
inventory = ./hosts          # Указывает путь к инвентарному файлу
remote_user = ubuntu         # Пользователь для подключения по умолчанию
host_key_checking = False    # Отключение проверки ключей SSH хоста
retry_files_enabled = False  # Отключение создания файлов retry
timeout = 10                 # Время ожидания для SSH-соединения
forks = 10                   # Количество параллельных задач
retry_files_enabled = True
retry_files_save_path = ~/ansible-retries             
```
Создание retry-файла: Если во время выполнения плейбука Ansible сталкивается с ошибкой на одном или нескольких серверах, он автоматически создаёт retry-файл. По умолчанию этот файл создаётся в директории, где был запущен плейбук, и имеет формат имя_плейбука.retry.

Повторный запуск плейбука: Можно использовать retry-файл, чтобы запустить плейбук только на тех серверах, которые указаны в файле. Для этого нужно указать имя retry-файла с опцией --limit.
`ansible-playbook site.yml --limit @site.retry
`
- [privilege_escalation] - Эта секция управляет параметрами для выполнения команд от имени суперпользователя (с помощью sudo)
```ini 
[privilege_escalation]
become = True                # Включение использования sudo (become)
become_method = sudo         # Метод получения привилегий (по умолчанию sudo)
become_user = root           # Пользователь, от имени которого выполняются команды
become_ask_pass = False      # Отключение запроса пароля при использовании sudo
```
- [ssh_connection] - Эта секция отвечает за параметры подключения через SSH.
```ini
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
control_path = %(directory)s/%%h-%%r
```
`ssh_args`: Дополнительные параметры для команды ssh. В данном примере включена поддержка многократного использования одного SSH-соединения (ControlMaster).

`pipelining`: Опция для ускорения выполнения команд за счёт уменьшения количества вызовов SSH.

`control_path`: Путь для хранения файлов управления соединением SSH.
-  [log] - Настройка логирования
```ini
[log]
log_path = /var/log/ansible.log
```
### Ad-Hoc команды
`ansible [опции] <группа-хостов> -m <модуль> -a <аргументы>` - структура Ad-Hoc команд

- `ansible -i hosts.ini all -m setup` - выводит подробную информацию про конфигурацию хостов.
- `ansible -i hosts.ini all -m ping` - проверка доступности хостов.
- `ansible -i hosts.ini all -m shell -a "rm -rf /"` - запускает команду в кавычках на хостах, выводит вывод команды со всех хостов в консоль. 
- `ansible -i hosts.ini all -m command -a "rm -rf /"` - Команда(ы) не будут обрабатываться через shell, поэтому переменные типа $HOSTNAME и операции типа "*", "<", ">", "|", ";" и "&" не будут работать. Используйте модуль ansible.builtin.shell, если вам нужны эти функции. То есть command более секьюрная.
- `ansible -i hosts.ini all -m copy -a "src=filename.txt dest=/home mode=777" -b` - ну тут понятно что команда делает, флаг `-b` - это выполнить от суперпользователя 

