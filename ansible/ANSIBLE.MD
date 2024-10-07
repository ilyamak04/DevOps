### Quick start
- Устанавливаем ansible на машину (Ansible Master)
```bash 
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
### Про инвентарный файл (что можно делать?)
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

### Про конфигурационный файл `ansible.cfg`
- Ansible ищет конфигурационный файл в нескольких местах, в следующем порядке при запуске:

    1. В текущем каталоге — если существует файл ansible.cfg в директории, из которой запускается Ansible.
    2. В домашней директории пользователя — файл ~/.ansible.cfg.
    3. В системной директории — обычно /etc/ansible/ansible.cfg.

Ansible будет использовать первый найденный конфигурационный файл. Это позволяет иметь разные конфигурации для разных проектов.

Файл ansible.cfg лучше хранить в проекте, инфраструктуру когорого менеджерит ansible, а вот инвентарный файл hosts.ini можно хранить где угодно на Ansible-Master хосте, главное в .cfg задать путь к hosts.ini

### Основные секции `ansible.cfg`
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

- `ansible -i hosts.ini all -m setup` - модуль, который собирает детальную информацию о системе каждого хоста (например, информацию о процессоре, операционной системе, памяти, сетевых интерфейсах и т.д.). Эта информация затем может быть использована в плейбуках или для отладки.
- `ansible -i hosts.ini all -m ping` - проверка доступности хостов.
- `ansible -i hosts.ini all -m shell -a "rm -rf /"` - запускает команду в кавычках на хостах, выводит вывод команды со всех хостов в консоль. 
- `ansible -i hosts.ini all -m command -a "rm -rf /"` - Команда(ы) не будут обрабатываться через shell, поэтому переменные типа $HOSTNAME и операции типа "*", "<", ">", "|", ";" и "&" не будут работать. Используйте модуль ansible.builtin.shell, если вам нужны эти функции. То есть command более секьюрная.
- `ansible -i hosts.ini all -m copy -a "src=filename.txt dest=/home mode=777" -b` - ну тут понятно что команда делает, флаг `-b` - это выполнить от суперпользователя 

Про модуль `file` -  С помощью этого модуля можно изменять права доступа, владельцев, группы, создавать или удалять файлы и директории, а также устанавливать символические ссылки. 

Вот основные параметры, которые можно использовать с модулем file:

- `path`: путь к файлу или директории, с которыми нужно работать.
- `state`: определяет, что должно быть сделано с файлом или директорией. Возможные значения:
    - `touch`: создать пустой файл, если он не существует, или обновить время доступа и модификации, если файл уже существует.
    - `absent`: удалить файл или директорию.
    - `directory`: создать директорию.
    - `file`: создать файл.
    - `link`: создать символическую ссылку.
    - `hard`: создать жесткую ссылку.
- `owner`: владелец файла или директории.
- `group`: группа файла или директории.
- `mode`: права доступа к файлу или директории, указанные в виде числового значения (например, 0644).
- `recurse`: рекурсивно изменяет права, владельцев или группы для директорий и их содержимого (поддиректорий и файлов).

- `ansible -i hosts.ini all -m file -a "path=/home/privet.txt state=absent" -b` - удаляет файл
- `ansible -i hosts.ini all -m get_url -a "url=https://link/on/download/file dest=/home" -b` - скачивает файл из интернета
- `ansible -i hosts.ini all -m apt -a "name=nginx state=present" --become` - устанаваливает пакет на Ubuntu/Debian (-b и --become эквивалентны)
- `ansible -i hosts.ini all -m uri -a "url=https://www.content.ru return_content=yes"` - Модуль uri в Ansible используется для взаимодействия с веб-сервисами через HTTP или HTTPS. Этот модуль позволяет выполнять запросы к REST API, загружать данные, отправлять данные на удалённый сервер и проверять доступность веб-сервисов.

Модуль `service` в Ansible используется для управления системными сервисами на удалённых хостах. Он позволяет запускать, останавливать, перезапускать и изменять состояние сервисов, а также управлять их включением при загрузке системы.

- `name`: Название сервиса, который нужно управлять. Например, apache2, nginx, ssh, и т. д.
- `state`: Указывает желаемое состояние сервиса. Возможные значения:
    - `started`: Запустить сервис.
    - `stopped`: Остановить сервис.
    - `restarted`: Перезапустить сервис.
    - `reloaded`: Перезагрузить конфигурацию сервиса без остановки.
    - `enabled`: Включить сервис при загрузке системы.
    - `disabled`: Отключить сервис от автозагрузки.
- `enabled`: Указывает, должен ли сервис автоматически запускаться при загрузке системы. Значения могут быть true или false.
- `ansible -i hosts.ini all -m service -a "name=nginx state=started enabled=yes" -b` 
> Все демоны могут считаться сервисами, но не все сервисы являются демонами

### group_vars
Директория `group_vars` должна находиться в директории с инвентарным файлом.
```css
├── ansible.cfg
├── inventory/
│   ├── hosts
├── group_vars/
│   ├── all.yml
│   ├── web_servers.yml
│   └── db_servers.yml
└── playbook.yml
```
Внутри директории `group_vars` могут находиться файлы с именами групп из инвентаря. Эти файлы содержат переменные, которые будут применяться ко всем хостам, принадлежащим к соответствующей группе.

- Файл `web_servers.yml` содержит переменные, которые будут применяться ко всем серверам группы web_servers.

- Файл `all.yml` содержит переменные для всех хостов, независимо от группы.

#### Основные моменты
- **Приоритеты**: Если переменные определены в нескольких местах, например, в файлах group_vars, host_vars или playbook, Ansible применяет переменные в следующем порядке:

    1. Переменные, определённые внутри задач в плейбуке (самый высокий приоритет).

    2. Переменные, указанные для конкретных хостов (host_vars).

    3. Переменные из группы хостов (group_vars).

    4. Переменные для всех хостов (например, файл group_vars/all.yml).
    
    5. Значения по умолчанию.

- **Наследование переменных**: Переменные, определённые в файле для всех хостов (например, `group_vars/all.yml`), могут быть переопределены переменными, определёнными в группе конкретных хостов (например, `group_vars/web_servers.yml`).

- **Формат файлов**: Файлы в директории `group_vars` могут быть в формате YAML или INI, хотя YAML используется чаще.

### host_vars

- Ansible автоматически подгружает переменные для конкретного хоста из файла, если этот файл находится в специальной директории `host_vars`
- На уровне директории, где находится ваш инвентарь (файл `inventory`), вы создаёте папку с именем `host_vars` 
```
# Пример структуры файлов
├── inventory
├── host_vars/
│   ├── server1.yaml
│   ├── server2.yaml
```
- Переменные из файлов `host_vars` будут доступны при выполнении плейбука на соответствующих хостах
```yml
# host_vars/server1.yaml
ansible_user: admin
http_port: 8080
db_name: production_db
```
```yml
# Пример плейбука, использующего переменные из host_vars
- hosts: server1
  tasks:
    - name: Показать значение переменной
      ansible.builtin.debug:
        var: http_port
```

### Флаги
- `-b` - от суперюзера
- `-k` - `--ask-pass`: ask for connection password
- `-K` - `--ask-become-pass`: ask for privilege escalation password
- `-v`, `-vv`, `-vvv`, `-vvvv` - подробный вывод для дебага 

### Плейбуки

Ansible playbook — это файл в формате YAML, который описывает последовательность действий для настройки хостов (серверов). Playbooks позволяют автоматизировать задачи, такие как установка программ, управление конфигурациями, деплой приложений и многое другое. Они состоят из набора инструкций, называемых tasks (задачи), которые выполняются на удалённых хостах, указанных в инвентаре. (определение chat gpt, на мой взгляд неплохое) 

- **Hosts (хосты)** - это часть плейбука, которая указывает, на каких серверах выполнять задачи 
    - `all` означает, что команды будут выполняться на всех хостах из инвентаря.

    - Также можно указывать конкретные группы или отдельные хосты.
```yml
- hosts: web_servers  # Выполнить задачи на всех хостах группы 'web_servers'
```
- **Tasks (задачи)** - Каждая задача в плейбуке представляет собой отдельное действие (например, установка пакета, редактирование файла, запуск сервиса). Задачи выполняются по порядку, сверху вниз.
```yml
tasks:
  - name: Установить Apache
    ansible.builtin.apt:
      name: apache2
      state: present
```
- **Vars (переменные)** - Можно использовать переменные для хранения данных, таких как имена пользователей, порты, пути к файлам и так далее.
```yml
- hosts: web_servers
  become: true

  vars:
    http_port: 8080
    document_root: /var/www/html

  tasks:
    - name: Установить Apache
      ansible.builtin.apt:
        name: apache2
        state: present

    - name: Настроить Apache на нужный порт
      ansible.builtin.lineinfile:
        path: /etc/apache2/ports.conf
        regexp: '^Listen'
        line: "Listen {{ http_port }}"  # Использование переменной 

    - name: Создать директорию DocumentRoot
      ansible.builtin.file:
        path: "{{ document_root }}"  # Использование переменной
        state: directory
```
- **Handlers (обработчики)** - это специальные задачи, которые выполняются **только в случае, если их вызвали**. Их часто используют для таких задач, как перезагрузка сервиса после изменения конфигурационного файла.
```yml
tasks:
  - name: Изменить конфигурацию Apache
    ansible.builtin.template:
      src: templates/apache.conf.j2
      dest: /etc/apache2/apache2.conf
    notify:
      - Перезапустить Apache

handlers:
  - name: Перезапустить Apache
    ansible.builtin.service:
      name: apache2
      state: restarted
```
- **Become (повышение привилегий)** - Некоторые команды требуют прав администратора (sudo). Для этого используется `become`.
```yml
tasks:
  - name: Установить Apache
    ansible.builtin.apt:
      name: apache2
      state: present
    become: true  # Повышение привилегий до sudo
```
- **Loops (циклы)** - Ansible поддерживает выполнение задач в цикле
```yml
tasks:
  - name: Установить список пакетов
    ansible.builtin.apt:
      name: "{{ item }}"
      state: present
    loop:
      - apache2
      - mysql-server
      - php
```
- **Conditions (условия)** - Ansible позволяет выполнять задачи только при выполнении определённых условий с помощью `when`.
```yml 
tasks:
  - name: Установить Apache только на Ubuntu
    ansible.builtin.apt:
      name: apache2
      state: present
    when: ansible_facts['os_family'] == "Debian"
``` 

### Блоки 

Для чего блоки:

1. **Группировка задач**: Несколько связанных задач можно объединить для удобства.

2. **Управление ошибками**: Можно задать специальную логику для обработки ошибок с помощью блоков `rescue` (обработчики ошибок) и `always` (выполняются всегда, независимо от успеха или неудачи).

3. **Условия и циклы**: Можно использовать блоки для использования `when`, `with_items` или `loop` и тп 

```yml
# Пример с rescue и always
- name: Основной блок
  block:
    - name: Первая задача
      ansible.builtin.shell: echo "Running task 1"
    
    - name: Вторая задача
      ansible.builtin.shell: echo "Running task 2"
    
  rescue:
    - name: Обработка ошибок
      ansible.builtin.debug:
        msg: "Произошла ошибка!"
  
  always:
    - name: Выполняется всегда
      ansible.builtin.debug:
        msg: "Этот шаг выполнится всегда, даже если были ошибки."
```
1. `block`: — основной блок, в котором определяются задачи. Если в нём произойдёт ошибка, выполнение перейдёт в секцию rescue.
2. `rescue`: — блок для обработки ошибок. Выполняется, если одна из задач в блоке завершилась с ошибкой.
3. `always`: — блок для задач, которые должны выполняться всегда, вне зависимости от того, произошла ошибка или нет (например, очистка или уведомления).
4. `rescue` и `always` используются только с блоками и относятся к конкретному блоку   

```yml
# Пример блока с условием
- hosts: localhost
  tasks:
    - name: Пример блока с условием
      block:
        - name: Выполнить команду echo
          ansible.builtin.command: echo "Hello, World!"
      when: ansible_facts['os_family'] == "Debian"
```
```yml
# Блок с циклом
- hosts: localhost
  tasks:
    - name: Установить несколько пакетов
      block:
        - name: Установить пакеты
          ansible.builtin.apt:
            name: "{{ item }}"
            state: present
          with_items:
            - git
            - vim
            - htop

      when: ansible_facts['os_family'] == "Debian"
```
```yml
# Пример использования блоков с условиями
- hosts: all
  tasks:
    - name: Проверка системы и установка пакетов
      block:
        - name: Установить curl
          ansible.builtin.apt:
            name: curl
            state: present

        - name: Установить git
          ansible.builtin.apt:
            name: git
            state: present

      when: ansible_facts['os_family'] == "Debian"

    - name: Установить пакеты для RedHat
      block:
        - name: Установить curl
          ansible.builtin.yum:
            name: curl
            state: present

        - name: Установить git
          ansible.builtin.yum:
            name: git
            state: present

      when: ansible_facts['os_family'] == "RedHat"
```
```yml
- name: Add several users
  ansible.builtin.user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  loop:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }
```
### Хендлеры (Handlers)

Это специальные задачи, которые выполняются только при вызове через `notify`. Обычно они используются для выполнения действий, которые должны происходить после того, как одна или несколько задач изменили состояние системы. Например, изменение конфигурации сервиса. 

- **Запускаются один раз за плейбук**: Если хендлер был вызван несколько раз в одном плейбуке, он выполнится только один раз, после выполнения всех остальных задач, изменивших состояние.

- **Выполняются в конце выполнения задач**: Хендлеры выполняются после того, как все основные задачи завершились.

- **Хендлеры выполняются по требованию**: Если задача не изменила состояние, хендлер не вызовется, даже если есть директива `notify`.

- **Внутри хендлера можно использовать `when`**

```yml
- hosts: all
  tasks:
    - name: Обновить конфигурацию приложения
      ansible.builtin.copy:
        src: /path/to/config.yml
        dest: /etc/myapp/config.yml
      notify:
        - Restart App
        - Send Notification

  handlers:
    - name: Restart App
      ansible.builtin.systemd:
        name: myapp
        state: restarted

    - name: Send message
      ansible.builtin.debug:
        msg: "Конфигурация приложения обновлена, уведомление."
```
### Модули

// TODO Информацию о модулях выше, перенести в эту главу

- **Модуль Dedug** - Модуль debug в Ansible используется для вывода отладочной информации во время выполнения плейбуков. Он помогает отображать значения переменных, выводить сообщения.
```yml
- name: Вывести сообщение
  ansible.builtin.debug:
    msg: "Задача завершена!"
```
```yml 
- name: Вывести значение переменной
  ansible.builtin.debug:
    var: http_port
```
- **Модуль set_fact** - для всяких разных операций с переменными в плейбуке

- **Модуль  template** - Используется для создания конфигурационных файлов и других текстовых файлов на целевых хостах на основе шаблонов Jinja2. Это позволяет динамически генерировать файлы с переменными, которые могут меняться в зависимости от окружения или настроек. Модуль `template` читает шаблон, подставляет значения переменных и создает итоговый файл на целевом хосте.
```yml
# Пример синтаксиса модуля
- name: Копировать шаблон конфигурационного файла
  ansible.builtin.template:
    src: /path/to/template.j2   # Путь к шаблону на Ansible Master
    dest: /path/to/destination.conf  # Путь, куда будет записан итоговый файл на целевом хосте
```
Боевой ример исользования `tenplate`
```j2
# Файл-шаблон nginx.conf.j2
server {
    listen {{ http_port }};
    server_name {{ server_name }};
    
    location / {
        proxy_pass http://{{ backend_service }};
    }
}
```
```yml
- hosts: web_servers
  vars:
    http_port: 80
    server_name: example.com
    backend_service: backend:5000
  tasks:
    - name: Копировать шаблон конфигурации Nginx
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/conf.d/example.conf
      notify:
        - перезапустить nginx

  handlers:
    - name: перезапустить nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```
Таска `Копировать шаблон конфигурации Nginx` берет шаблон `nginx.conf.j2`, рендерит его с подстановкой переменных и записывает в файл `/etc/nginx/conf.d/example.conf` на целевом хосте.

> `ansible-galaxy collection install ansible.<collection_name>` - установить модуль

### Директивы

- **`register`** - используется для сохранения результатов выполнения задачи в переменной. Это позволяет вам использовать результаты задачи в последующих задачах внутри playbook.
```yml
- name: Register loop output as a variable
  ansible.builtin.shell: "echo {{ item }}"
  loop:
    - "one"
    - "two"
  register: echo
```
После выполнения задачи, `echo` будет содержать информацию о каждой итерации, включая стандартный вывод, статус выполнения и другие данные.
```yml
- name: Print the registered output
  ansible.builtin.debug:
    var: echo
```
После выполнения задачи, вы можно использовать зарегистрированную переменную в следующих задачах
```json
{
    "changed": true,
    "msg": "All items completed",
    "results": [
        {
            "changed": true,
            "cmd": "echo \"one\" ",
            "delta": "0:00:00.003110",
            "end": "2013-12-19 12:00:05.187153",
            "invocation": {
                "module_args": "echo \"one\"",
                "module_name": "shell"
            },
            "item": "one",
            "rc": 0,
            "start": "2013-12-19 12:00:05.184043",
            "stderr": "",
            "stdout": "one"
        },
        {
            "changed": true,
            "cmd": "echo \"two\" ",
            "delta": "0:00:00.002920",
            "end": "2013-12-19 12:00:05.245502",
            "invocation": {
                "module_args": "echo \"two\"",
                "module_name": "shell"
            },
            "item": "two",
            "rc": 0,
            "start": "2013-12-19 12:00:05.242582",
            "stderr": "",
            "stdout": "two"
        }
    ]
}
```
Примерная структура переменной в `register`
```yml
# Просто пример использования
- name: Run shell command
  hosts: all
  tasks:
    - name: List files and count them
      ansible.builtin.shell: "ls -l | wc -l"
      register: file_count

    - name: Print the number of files
      ansible.builtin.debug:
        msg: "There are {{ file_count.stdout }} files in the directory."
```

- **`until`** - конструкция, которая используется для повторного выполнения задачи до тех пор, пока не будет выполнено определенное условие
```yml
# Основной синтаксис
- name: Task description            # Условие, при выполнении которого задача завершится
  ansible.builtin.module_name:      
    parameters                      
  register: result_variable         # Сохраняет результат выполнения задачи в указанной переменной, которая затем используется в условии until.
  until: result_variable.condition  # Условие, при выполнении которого задача завершится.
  retries: number_of_retries        # Общее количество повторных попыток, которые будут сделаны, если условие until не выполнено
  delay: delay_in_seconds           # Время (в секундах) для ожидания между попытками выполнения задачи
```
```yml
# Пример
- name: Start a service
  ansible.builtin.systemd:
    name: my_service
    state: started

- name: Wait for the service to be active
  ansible.builtin.shell: "systemctl is-active my_service"
  register: result
  until: result.stdout == "active"
  retries: 5
  delay: 2

- name: Notify that the service is active
  ansible.builtin.debug:
    msg: "The service is now active."
```
- **`with_fileglob`** - используется для итерации по файлам в директории, соответствующим определённому шаблону
    - Работает только с файлами на локальном хосте (где запущен Ansible).
    - Если в шаблоне не найдено ни одного файла, Ansible просто пропустит задачу.
    - Путь в `with_fileglob` должен быть **абсолютным**. 
```yml
- hosts: all
  become: true
  tasks:
    - name: Копировать конфигурационные файлы и установить правильные разрешения
      ansible.builtin.copy:
        src: "{{ item }}"            # Локальный путь к файлам
        dest: /etc/myapp/configs/
        mode: '0644'                 
      with_fileglob:
        - "/etc/myapp/configs/*.conf"
```
```yml
# Можно использовать несколько шаблонов
- hosts: webservers
  become: true
  tasks:
    - name: Копировать конфигурационные и скриптовые файлы
      ansible.builtin.copy:
        src: "{{ item }}"
        dest: /etc/myapp/
      with_fileglob:                # Можно использовать несколько шаблонов
        - "/etc/myapp/configs/*.conf"
        - "/etc/myapp/scripts/*.sh"
```
- **`any_errors_fatal : (true || false)`** - прекращает выполнение плейбука, если хоть одна таска на одном из хостов падает с ошибкой
```yml
- name: Ansible 
  hosts: all
  any_errors_fatal: true
  become: yes

  tasks:
    - name: ...
```
- **`ignore_errors : (yes || no)`** - если таска падает с ошибкой, плейбук продолжает выполняться
```yml
tasks: 
  - name: Task Number
    apt:
      name: nginx
      state: present
    ignore_errors: yes 
```

- **`failed_when`** - таска падает или не падает в зависимости от условия. `failed_when` принимает логическое выражение. Если это выражение истинно (true), задача считается неудачной, даже если команда выполнена успешно (с кодом возврата 0)
```yml
- name: Example task
  command: /path/to/some/command
  register: result
  failed_when: result.rc != 0 and result.stdout != "expected output"
```
```yml
- name: Check if service is running
  command: systemctl status my_service
  register: service_status
  failed_when: "'inactive' in service_status.stdout"
```
```yml
- name: Run a command
  command: /path/to/some/command
  register: command_result
  failed_when: command_result.rc != 0 and 'error' in command_result.stderr
```

- **`changed_when`** - позволяет кастомно определить условие `changed`, не останавливает выполнение задачи, влияет только на вывод работы плейбука. благодаря `changed_when` можно контролировать, как Ansible оценивает, изменила ли она что-либо в системе.

```yml
- name: Example task
  command: /path/to/some/command
  register: result
  changed_when: result.stdout != "no changes"
```
- **`group_by`** - в Ansible используется для динамического создания групп хостов на основе определённых условий. Модуль `group_by` может создавать группы на основе любых фактов, доступных через `ansible_facts`, а также на основе данных из инвентаря или других переменных. `group_by` не изменяет инвентарь
```yml
# Синтаксис
- name: Group hosts by operating system
  group_by:
    key: "os_{{ ansible_facts['distribution'] }}"
```
```yml
# Пример
- hosts: os_Ubuntu
  tasks:
    - name: Install package on Ubuntu hosts
      apt:
        name: vim
        state: present

- hosts: os_CentOS
  tasks:
    - name: Install package on CentOS hosts
      yum:
        name: vim
        state: present
```
```yml
# Группировка хостов по оперативной памяти
- name: Group hosts by available RAM
  group_by:
    key: "ram_{{ (ansible_facts['memtotal_mb'] // 1024) }}GB"
```
- **`tags`** - теги (tags) позволяют управлять выполнением отдельных задач или целых плейбуков, предоставляя возможность запускать только те части плейбука, которые вам нужны, вместо выполнения всех задач. Теги особенно полезны для ускорения процесса, когда нужно протестировать только определенные задачи или фрагменты плейбука

    - Управление выполнением задач: Вы можете назначать тег задачам или блокам задач, а затем запускать playbook с флагом `--tags`, чтобы выполнить только эти задачи.

    - Исключение задач: С помощью `--skip-tags` можно исключать выполнение задач с определенными тегами.
```yml
# Теги тасок 
- name: Install Nginx
  apt:
    name: nginx
    state: present
  tags:
    - install
    - webserver

- name: Copy configuration file
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  tags:
    - config
```
```yml
# Теги плейбука 
- hosts: webservers
  tags: 
    - setup
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
    - name: Start Apache service
      service:
        name: apache2
        state: started
```
```yml
# Теги для ролей
- hosts: all
  roles:
    - role: common
      tags: common
    - role: webserver
      tags: web
```
- `ansible-playbook site.yml --tags "install,config"` - поддерживается работа с несколькими тегами
- `ansible-playbook site.yml --tags "install" --skip-tags "config"` - выполнить только установку серверов пропуская конфигурацию (пример использования) 


### Секреты (Ansible Vault)
Позволяет шифровать конфиденциальные данные, такие как файлы с переменными, плейбуки, конфигурации или любые другие файлы, требующие защиты
- `ansible-vault create secret.yml` - создание зашифрованного файла
- `ansible-vault edit secret.yml` - редактирование зашифрованного файла
- `ansible-vault encrypt secret.yml` - шифрование уже сущ. файла
- `ansible-vault decrypt secret.yml` - расшифровка файла
- `ansible-vault view secret.yml` - cat для зашифрованного файла 
- `ansible-vault rekey secret.yml` - поменять пароль для зашифрованного файла 
- `ansible-playbook playbook_vault.yml --vault-password-file mypassword.txt` - пароль для запуска плейбука берётся из файла
- **`ansible-playbook playbook.yml --ask-vault-pass`** - запуск плейбука с расшифровкой
- `ansible-vault encrypt_string --stdin-name` "MyPassword" - зашифровать строку
- `echo -n "MyPassword" | ansible-vault encrypt_string` - зашифровать строку
---
- `ansible all -m ping -b --extra-vars '@vault.yml' --extra-vars "ansible_become_pass={{ vault_sudo_passwords[inventory_hostname] }}" --ask-vault-pass` - пинг хостов с разными паролями юзеров    

### Роли 
```bash
# init role 
mkdir roles
cd roles
ansible-galaxy init first_role
```
```bash
# Структура роли
└── first_role
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml
```
Можно указать путь к роли в инвентаре
```ini
[defaults]
roles_path = ./roles:/etc/ansible/roles:/another/path/to/roles # : - это разделение нескольких путей
```

Можно закинуть роль в глобальный каталог
```bash
ansible-galaxy install <роль> -p /etc/ansible/roles
```

`tasks/main.yml`: Здесь находятся задачи, которые выполняет роль. Это основной файл задач, аналогичный тем, что вы пишете в плейбуках. Можно разделить задачи по отдельным файлам и включать их в main.yml.

`handlers/main.yml`: Хендлеры, которые вызываются при изменении состояния задач. Они перезапускают сервисы, если это необходимо.

`files/`: В эту папку вы кладете файлы, которые должны быть скопированы на удаленные хосты с помощью модуля copy.

`templates/`: Здесь хранятся шаблоны Jinja2, которые можно использовать для создания динамических файлов конфигурации, применяя переменные.

`vars/main.yml`: Здесь определяются переменные для роли. Эти переменные имеют более высокий приоритет по сравнению с переменными из defaults.

`defaults/main.yml`: Здесь определяются переменные с наименьшим приоритетом. Они могут быть переопределены переменными из других файлов.

`meta/main.yml`: В этом файле содержатся метаданные роли, такие как ее зависимости от других ролей.

`tests/`: Папка для тестирования роли. Здесь можно хранить тестовые плейбуки и инвентарь.

**Приоритет переменных**
1. Переменные командной строки
2. Переменные, объявленные в `vars` внутри плейбука.
3. Переменные, объявленные в `vars` внутри роли.
5. Переменные, определенные в `host_vars`.
4. Переменные, определенные в `group_vars`.
6. Переменные, объявленные в `defaults` внутри роли.

Зависимости можно указать в файле `meta/main.yml`. Например, если роль зависит от другой роли, то она будет автоматически вызвана:
```yml
dependencies:
  - role: common
  - role: security
```

Роль можно протестировать с помощью специального тестового плейбука. Например, в папке `tests/` может быть файл `test.yml`, который проверяет работу роли:
```yml
- hosts: localhost
  roles:
    - myrole
```
Для тестирования можно просто запустить этот плейбук:
```bash
ansible-playbook tests/test.yml
```

**Разница между `defaults/` и `vars/`**

- `defaults/` может содержать такие переменные, как версии ПО, которые можно легко изменить для разных окружений.

- `vars/` будет содержать важные и фиксированные настройки, такие как адреса серверов или пути к файлам, которые редко меняются.

```yml
# Пример плейбука
- name: Playbook
  hosts: all
  become: yes

  roles:
    - { role: my_role, when: ansible_system == "Linux" }
```
В условии можно использовать логически операторы (`and`, `or` и `not (!=)`)
```yml
- { role: my_role, when: ansible_distribution == "Ubuntu" and ansible_distribution_version == "20.04" }
```
Если переменная может быть не определена, можем проверить её наличие 
```yml
- { role: my_role, when: my_var is defined }
```
Можно проверять значения словаря в списках
```yml
- { role: my_role, when: my_dict.key == "value" }
```
### extra-vars

```yml
- name: Playbook
  hosts: "{{ MYHOST }}"
  become: yes

  roles:
    - my_role
```
**Как передать переменную в плейбук?**
```bash
ansible-playbook playbook.yml --extra-var "MYHOSTS=STAGING"
```
Можно писать:
- --extra-var
- --extra-vars
- -e

`extra-var` имеет наивысший приоритет и переопределяет все остальные переменные, т.к. это оп сути переменная внутри плейбука

### import и include

- `import_*` - загружает все задачи из указанного файла на этапе **разбора плейбука (parse time)**, то есть перед выполнением задач
- `include_*` - Загружает задачи динамически на этапе выполнения **(runtime)**
- `include_tasks`
    ```yml
    - name: Include tasks dynamically
      include_tasks: dynamic_tasks.yml
    ```
- `import_playbook` - загружает целый плейбук на этапе разбора **(parse time)**
- `include_playbook` - устарела
- `import_role`
    ```yml
    - name: Import a role
      import_role:
      name: webserver
    ```

Когда использовать `import`, а когда `include`?

 `import`:
- Когда нужно загрузить фиксированный набор задач, плейбуков или ролей.
- Когда условия и переменные известны на этапе разбора (parse time).

 `include`:
- Когда требуется динамическое выполнение в зависимости от условий.
- Когда необходимо передать переменные или использовать цикл для включения.

### Best practice
- Писать `hosts : all`, а потом юзать в тасках `when` - **bad practice**
- Инвентарь в `yml` формате
- `host_vars` - это временное решение (на хостах завязываться не стоит), лучше объединить в группу с одним хостом в инвентаре и прописывать переменные в `group_vars` 
- Можно писать переменные внутри инвентаря, если это общая переменная для многих хостов, чтобы создавать лишние `group_vars` и не плодить сущности
- Не управляем пользователями и контейнерами с помощью `Ansible` для этого есть специализированные инструменты.
- Писать роль отвечающую за компонент целиком (например, роль для NGINX)

- Разные инвентари для разные сред (инвентарь для прода, дева и тд)
- Группируй однотипные задачи с помощью `block`
- Роли долнжны быть слабосвязаны, избегать зависимости между ролями
- Проверьте, какие задачи будут запущены перед выполнением: Вы можете использовать флаг `--list-tasks` для подтверждения того, какие задачи будут запущены без их фактического выполнения. Вы можете использовать флаг `--list-hosts`: так вы убедитесь в том, на какие хосты повлияет плейбук, но при этом не запустить его
- Убедитесь в том, что вы собрались менять, но при этом не запускайте обновления: Используйте флаг `--check` для прогнозирования изменений, которые могут произойти. Объедините его с флагом `--diff`, чтобы показать различия в измененных файлах.
- Пример архитектуры
```yml 
production                # inventory file for production servers
staging                   # inventory file for staging environment

group_vars/
   group1.yml             # here we assign variables to particular groups
   group2.yml
host_vars/
   hostname1.yml          # here we assign variables to particular systems
   hostname2.yml

library/                  # if any custom modules, put them here (optional)
module_utils/             # if any custom module_utils to support modules, put them here (optional)
filter_plugins/           # if any custom filter plugins, put them here (optional)

site.yml                  # master playbook
webservers.yml            # playbook for webserver tier
dbservers.yml             # playbook for dbserver tier

roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
```
- Пример архитектуры
```yml 
inventories/
   production/
      hosts               # inventory file for production servers
      group_vars/
         group1.yml       # here we assign variables to particular groups
         group2.yml
      host_vars/
         hostname1.yml    # here we assign variables to particular systems
         hostname2.yml

   staging/
      hosts               # inventory file for staging environment
      group_vars/
         group1.yml       # here we assign variables to particular groups
         group2.yml
      host_vars/
         stagehost1.yml   # here we assign variables to particular systems
         stagehost2.yml

library/
module_utils/
filter_plugins/

site.yml
webservers.yml
dbservers.yml

roles/
    common/
    webtier/
    monitoring/
    fooapp/
```
- Пример статического инвентаря 
```ini
# file: production
[atlanta_webservers]
www-atl-1.example.com
www-atl-2.example.com

[boston_webservers]
www-bos-1.example.com
www-bos-2.example.com

[atlanta_dbservers]
db-atl-1.example.com
db-atl-2.example.com

[boston_dbservers]
db-bos-1.example.com

# webservers in all geos
[webservers:children]
atlanta_webservers
boston_webservers

# dbservers in all geos
[dbservers:children]
atlanta_dbservers
boston_dbservers

# everything in the atlanta geo
[atlanta:children]
atlanta_webservers
atlanta_dbservers

# everything in the boston geo
[boston:children]
boston_webservers
boston_dbservers
```
- В `site.yml` мы импортируем плейбуки, которые определяют всю нашу инфраструктуру
```yml
---
# file: site.yml
- import_playbook: webservers.yml
- import_playbook: dbservers.yml
```
- Идея здесь в том, что мы можем выбрать настройку всей нашей инфраструктуры, «запустив» `site.yml`, или мы могли бы просто выбрать запуск подмножества, запустив `webservers.yml`. Это аналогично параметру `«–limit» в ansible`, но немного более явно
```bash 
ansible-playbook site.yml --limit webservers
ansible-playbook webservers.yml
```
- Только для серверов в Бостоне
```bash
ansible-playbook -i production webservers.yml --limit boston
```
- Для первых 10, потом следущие 10 
```bash
ansible-playbook -i production webservers.yml --limit boston[0:9]
ansible-playbook -i production webservers.yml --limit boston[10:19]
```
```bash
# confirm what task names would be run if I ran this command and said "just ntp tasks"
ansible-playbook -i production webservers.yml --tags ntp --list-tasks
```
```bash
# confirm what hostnames might be communicated with if I said "limit to boston"
ansible-playbook -i production webservers.yml --limit boston --list-hosts
```
- Как также упоминалось выше, хороший способ разделить среды подготовки (или тестирования) и производства - использовать отдельный файл инвентаризации для dev и prod. Таким образом, вы выбираете с помощью `-i` на инвентарь. Хранение их всех в одном файле может привести к непрогнозируемому поведению! Тестирование промежуточной среде перед тем, как выкатывать на prod, всегда является отличной идеей. Ваши среды не обязательно должны быть одинакового размера, и вы можете использовать групповые переменные для управления различиями между этими средами.
- Параметр `'state'` является необязательным для многих модулей. Будь то `'state=present'` или `'state=absent'`, всегда лучше оставить этот параметр в своих плейбуках, чтобы сделать его понятным, особенно потому, что некоторые модули поддерживают дополнительные состояния.
- Если вам приходится иметь дело с параметром, который отличается в двух разных операционных системах, отличным способом решения этой проблемы является использование модуля `group_by`. Это создает динамическую группу хостов, соответствующих определенным критериям, даже если эта группа не определена в файле инвентаризации. Это позволит объединить все системы в динамическую группу на основе имени операционной системы.
```yml
 - name: talk to all hosts just so we can learn about them
   hosts: all
   tasks:
     - name: Classify hosts depending on their OS distribution
       group_by:
         key: os_{{ ansible_facts['distribution'] }}

 # now just on the CentOS hosts...

 - hosts: os_CentOS
   gather_facts: False
   tasks:
     - # tasks that only happen on CentOS go here
```
Если необходимы настройки, специфичные для группы, это также можно сделать. Например:
```yml
---
# file: group_vars/all
asdf: 10

---
# file: group_vars/os_CentOS
asdf: 42
```
- Используй `name` в тасках
- Используйте контроль версий. Храните ваши плейбуки и файл `inventory` в `git` (или другой системе контроля версий) и фиксируйте изменения, когда вносите в них изменения. Таким образом, у вас будет аудиторский след, описывающий, когда и почему вы изменили правила, автоматизирующие вашу инфраструктуру.

- **Переменные и секреты:** начнём с `group_vars/` подкаталога, названного в честь группы. Внутри этого подкаталога создайте два файла с именами `vars` и `vault`. Внутри файла `vars` определите все необходимые переменные, включая любые конфиденциальные. Затем скопируйте все конфиденциальные переменные в файл `vault` и добавьте к этим переменным префикс `vault_`. Вам следует настроить переменные в `vars` файле так, чтобы они указывали на соответствующие `vault_переменные`, используя синтаксис `jinja2`, и убедиться, что `vault` файл зашифрован с помощью хранилища.
