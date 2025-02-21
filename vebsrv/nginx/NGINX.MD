- файл `/etc/nginx/nginx.conf`: 
```
http {
    # Включение основной конфигурации из conf.d
    include /etc/nginx/conf.d/*.conf;

    # Включение дополнительных конфигураций из extra-conf.d
    include /etc/nginx/extra-conf.d/*.conf;
}
```
- Блок **http**

Блок `http` используется для настроек на уровне всего HTTP-сервера и обычно располагается в основном файле конфигурации `/etc/nginx/nginx.`conf. Он содержит общие настройки, которые могут применяться ко всем server блокам (виртуальным хостам) внутри него.

Примеры того, что может быть настроено в http блоке:

- Глобальные параметры для всех сайтов:

    - Настройки кеширования, сжатия, таймаутов, буферов.
    - Настройки логирования (например, access_log, error_log).
    - Параметры загрузки статических файлов.
    - Общие правила для обработки MIME-типов и дефолтных заголовков.

- Определение подключаемых конфигураций:

    - Через директивы include, например, для файлов в conf.d/ и extra-conf.d/.

- Настройки для upstream

    - Создание блоков upstream для балансировки нагрузки между несколькими бэкэндами (например, для проксирования запросов к пулу приложений).

```
http {
    # Логирование запросов
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Сжатие
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;

    # Пулы серверов для балансировки нагрузки
    upstream backend {
        server backend1.example.com;
        server backend2.example.com;
    }

    # Подключение всех файлов в conf.d/
    include /etc/nginx/conf.d/*.conf;
}
```

- Блок **server**

Блок server используется для настройки конкретного хоста (сайта) или группы настроек для определенного домена или IP-адреса. В одном http блоке может быть несколько server блоков, каждый из которых настраивает определенный виртуальный хост, обслуживающий свои уникальные домены, поддомены, или порты.

Домен и порт:
Директивы server_name и listen, чтобы указать, на каких доменах и портах сервер принимает запросы.

Обработка запросов:
Можно настроить обработку корневых запросов (/), редиректы и маршрутизацию.

Проксирование запросов на бэкенд:
Настройка обратного прокси-сервера через директиву proxy_pass.

Доступ к статическим файлам:
Использование директивы root или alias для указания расположения статических файлов.
```
server {
    listen 80;                   
    server_name example.com; 

    root /var/www/example.com;

    # Настройка проксирования для API-запросов
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # Страница ошибки 404
    error_page 404 /404.html;
    location = /404.html {
        internal;
    }
}
```

Блок http: Настройки, определенные здесь, применяются ко всем server блокам, если они не переопределены внутри конкретного server блока. Например, включение сжатия в http блоке (gzip on;) будет автоматически применяться ко всем виртуальным хостам.

Блок server: Определяет параметры для конкретных сайтов или доменов. 

--- 

- `/etc/nginx/sites-available:` В этой директории хранятся файлы конфигураций для всех доступных сайтов или приложений. Каждый файл здесь обычно соответствует одному сайту или домену, и содержит настройки для конкретного виртуального хоста.

- `/etc/nginx/sites-enabled`: Эта директория содержит символические ссылки на файлы из sites-available. Файлы в sites-enabled представляют собой активированные сайты. Nginx загружает только те сайты, для которых конфигурационные файлы находятся в sites-enabled.

для активации сайта:

`sudo ln -s /etc/nginx/sites-available/example.conf /etc/nginx/sites-enabled/`

для деактивации:

`sudo rm /etc/nginx/sites-enabled/example.conf`

`sudo systemctl reload nginx` - необходимо перезагрузить nginx

```
/etc/nginx/
├── nginx.conf
├── sites-available/
│   ├── example.com
│   └── example.org
└── sites-enabled/
    ├── example.com -> /etc/nginx/sites-available/example.com
```

потом `sites-enabled` подключается к `/etc/nginx/nginx.conf`
```
http {
    # Общие настройки HTTP
    include /etc/nginx/sites-enabled/*;
}
```

--- 

`error_page 500 502 503 504 /50x.html;`

---

```
upstream server-group {
    server srv-server1.bft.local:8080;
    server srv-server2.bft.local:8080;
    keepalive 60;
}
```
---
```
location / {
    return 301 https://srvname.test.srv/app;
}
```
`return <status_code> [<url>];`

```
error_page 500 502 503 504 = @custom_error;
location @custom_error {
    return 302 http://example.com/error-page;
}
```
---

● Last-modified (If-Modified-Since)
● Expires
● Etag (If-None-Match)
● Cache-control
add_header Cache-Control "max-age=31536000, public, no-transform, immutable";


 Gzip
gzip on;
gzip_static on;
gzip_types text/plain text/css text/xml application/javascript application/json image/svg+xml 
application/font-ttf;
gzip_comp_level 6;
gzip_proxied any;
gzip_min_length 1000;
gzip_vary on;
● Brotli*
brotli_static on;
brotli on;
brotli_comp_level 6;
brotli_types text/plain text/xml text/css application/javascript application/json image/x-icon 
image/svg+xml;


статическое и динамическое сжатие


eepalive_timeout 300;
keepalive_requests 10000;
ssl_prefer_server_ciphers on;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
ssl_ciphers 
ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-S
HA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-S
HA384:DHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE
-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:
DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA256:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AE
S128-SHA:AES256-SHA:DES-CBC3-SHA;
ssl_early_data on;
ssl_session_tickets on;
ssl_session_timeout 28h;
ssl_session_cache shared:SSL:10m;