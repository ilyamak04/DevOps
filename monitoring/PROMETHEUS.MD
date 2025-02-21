### Установка Prometheus server и Node exporter
#### Prometheus server install

- `wget https://github.com/prometheus/prometheus/releases/download/v2.55.0-rc.0/prometheus-2.55.0-rc.0.linux-amd64.tar.gz` - скачиваем прометеус сервер
- `tar xvfz *.tar.gz`
- `cd prometheus-2.55.0-rc.0.linux-amd64.tar.gz`
- `sudo mv prometheus /usr/local/bin/`
- `sudo mv promtool /usr/local/bin`
- `sudo mkdir /etc/prometheus/`
- `sudo mkdir /etc/prometheus/data`
- `sudo mv prometheus.yml /etc/prometheus/`

```yml
# prometheus.yml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
        
  - job_name: "archers-paradox-servers"
    static_configs:
      - targets:
          - 89.22.241.241:9100
```
- `useradd -rs /bin/false prometheus` - создаём системного пользователя для работы с prometheus
- `chown prometheus:prometheus /usr/local/bin/prometheus` 
- `chown -R prometheus:prometheus /etc/prometheus` 
- `vi /etc/systemd/system/prometheus.service` - создаём systemd юнит

```bash
[Unit]
Description=Prometheus Server
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=4s
ExecStart=/usr/local/bin/prometheus \
  --config.file       /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /etc/prometheus/data

[Install]
WantedBy=multi-user.target
```
- `sudo systemctl daemon-reload` - обновить systemd
- `sudo systemctl start prometheus`
- `sudo systemctl enable prometheus`

#### Node exporter install
- `wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-arm64.tar.gz` - скачиваем node exporter на удалённый хост

- `tar xvfz *.tar.gz`
- `sudo mv node_exporter /usr/local/bin/`

-  `sudo useradd -rs /bin/false node_exporter`
-  `sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter`
- `sudo vi /etc/systemd/system/node_exporter.service`

```bash
# node_exporter.service
[Unit]
Description=Prometheus Node Exporter
After=network.target 

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=4s
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
- `systemctl daemon-reload`
- `sudo systemctl start node_exporter`
- `sudo systemctl enable node_exporter`


### Про PromQL
- `increase()` возвращает общий прирост за указанный интервал времени.
```
increase(<метрика>[<интервал>])
```
- `rate()` вычисляет скорость изменения за этот же интервал времени, и в результате возвращает количество изменений в секунду.
- `increase(http_requests_total[5m])` — покажет, сколько запросов пришло за последние 5 минут.
- `rate(http_requests_total[5m])` — покажет среднее количество запросов в секунду за последние 5 минут.

### Полезности

- `promtool check config prometheus.yml`
- `promtool check rules rules.yml`
- `promtool test rules rules_test.yml`