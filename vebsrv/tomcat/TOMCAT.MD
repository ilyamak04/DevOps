
[Документация](https://tomcat.apache.org/tomcat-8.5-doc/introduction.html)
- `wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.96/bin/apache-tomcat-9.0.96.tar.gz`
- `CATALINA_HOME` : представляет собой корень вашей установки Tomcat, например /home/tomcat/apache-tomcat-9.0.10 или C:\Program Files\apache-tomcat-9.0.10.
- `CATALINA_BASE` : представляет корень конфигурации времени выполнения определенного экземпляра Tomcat. Если вы хотите иметь несколько экземпляров Tomcat на одной машине, используйте свойство CATALINA_BASE.
- установка и настройка `tomcat`
    - `cd /opt/`
    - `wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.73/bin/apache-tomcat-9.0.73.tar.gz`
    - `sudo useradd -r -m -U -d /opt/tomcat tomcat`
    - `sudo chown -R tomcat:tomcat /opt/tomcat`
    - `which java`
    - `readlink -f /usr/bin/java`

    - `cd /opt/tomcat/bin`
    - `./startup.sh`

- **Создаём systemd unit**
- `sudo vi /etc/systemd/system/tomcat.service` 

```bash
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=CATALINA_HOME=/opt/tomcat/
Environment=CATALINA_BASE=/opt/tomcat/
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

User=tomcat
Group=tomcat
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```
- `sudo systemctl daemon-reload`
- `sudo systemctl start tomcat`
- `sudo systemctl enable tomcat`
- `sudo systemctl status tomcat`

```
/opt/tomcat
├── conf
│   ├── server.xml          # Основной файл конфигурации серверов и виртуальных хостов
│   ├── logging.properties  # Общие настройки логирования
│   ├── [host1]             # Каталог для настроек виртуального хоста host1 (опционально)
│   │   └── app1.xml        # Конфигурация приложения для host1/app1
│   └── [host2]             # Каталог для настроек виртуального хоста host2 (опционально)
│       └── app2.xml        # Конфигурация приложения для host2/app2
├── logs                    # Общая директория для логов (или отдельные каталоги для каждого хоста)
│   ├── host1               # Логи виртуального хоста host1 (если настроено)
│   ├── host2               # Логи виртуального хоста host2 (если настроено)
│   └── catalina.out        # Основной лог Tomcat
├── webapps                 # Общая папка для приложений (используется по умолчанию для всех виртуальных хостов)
│   ├── ROOT                # Основное приложение по умолчанию
│   ├── examples            # Пример приложения
│   └── [host-specific-apps]
│       ├── host1/app1      # Приложение app1 для host1
│       └── host2/app2      # Приложение app2 для host2
└── temp
└── work
└── bin
└── lib
```

Содержимое CATALINA_HOME в `/opt/tomcat`:

- `/bin` — скрипты для запуска и остановки Tomcat (startup.sh, catalina.sh)
- `/lib` — библиотеки, необходимые для работы сервера (catalina.jar, jasper.jar)


`CATALINA_BASE` — директория с конфигурациями, временными файлами и приложениями, которые специфичны для каждого экземпляра Tomcat. Для нескольких инстансов рекомендуется создать отдельный `CATALINA_BASE` для каждого инстанса, например:

`/etc/tomcat-instance1`, `/etc/tomcat-instance2` и т.д.

- `/conf` — конфигурационные файлы для инстанса (server.xml, web.xml, context.xml).
- `/webapps` — развернутые приложения и .war файлы для данного инстанса.
- `/logs` — папка с логами инстанса.
- `/temp` — временные файлы инстанса.
- `/work` — кешированные объекты и скомпилированные JSP для данного инстанса.

---

Про библиотеку **Logback** для логирования Java приложений 

В logback.xml основные блоки конфигурации включают:

`<configuration>` — корневой блок файла, внутри которого настраиваются все остальные элементы.

`<appender>` — определяет, куда будут направляться логи (например, в файл, консоль, удалённый сервер):

- `ConsoleAppender` — вывод в консоль (обычно используется для отладки).
- `FileAppender` — запись логов в файл.
- `RollingFileAppender` — запись в файл с ротацией по размеру или дате.
- `SocketAppender` — отправка логов на удалённый сервер.

`<encoder>` — внутри аппендера определяет форматирование логов (например, шаблоны с датой, уровнем лога и сообщением).

`<root>` — задаёт общий уровень логгирования (например, INFO, DEBUG) и связывает его с аппендерами.

`<logger>` — настраивает уровень логгирования для отдельных классов или пакетов приложения (переопределяет `<root>`).


- `TRACE`: Самый детализированный уровень, для отладки.
- `DEBUG`: Для отладки, но с меньшей детализацией, чем TRACE.
- `INFO`: Обычные информационные сообщения, отображающие состояние приложения.
- `WARN`: Предупреждения о возможных проблемах.
- `ERROR`: Сообщения об ошибках, когда выполнение операции не удалось

WEB-INF/classes или src/main/resources          
`<configuration scan="true" scanPeriod="30 seconds">` 