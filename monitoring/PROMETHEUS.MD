### Install and Configure
- `wget https://github.com/prometheus/prometheus/releases/download/v2.55.0-rc.0/prometheus-2.55.0-rc.0.linux-amd64.tar.gz` - скачиваем прометеус сервер

- `tar xvfz *.tar.gz`
- `sudo mv prometheus /usr/bin/`
- `sudo mkdir /etc/prometheus/`
- `sudo mkdir /etc/prometheus/data`
- `useradd -rs /bin/false prometheus` - создаём системного пользователя для работы с prometheus
- `chown prometheus:prometheus /usr/bin/prometheus` 
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
ExecStart=/usr/bin/prometheus \
  --config.file       /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /etc/prometheus/data

[Install]
WantedBy=multi-user.target
```
- `sudo systemctl daemon-reload` - обновить systemd

- `sudo systemctl start prometheus`
- `sudo systemctl enable prometheus`



 