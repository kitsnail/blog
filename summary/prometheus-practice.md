# prometheus practice

## steps

  - monitor
    - install node_exporter
    - install prometheus
    - install grafana
  - alert
    - install alertmanager
  - service discovery


## create `prometheus` user

```bash
useradd -s /sbin/nologin prometheus
```
## install prometheus

- download & install

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.8.0/prometheus-2.8.0.linux-amd64.tar.gz -O /tmp/prometheus.tar.gz
tar xvf /tmp/prometheus.tar.gz -C /usr/local
mv -v /usr/local/prometheus-2.8.0.linux-amd64 /usr/local/prometheus
chmod -R prometheus.prometheus /usr/local/prometheus
```

- setting `prometheus.yml`

eg. `/usr/local/prometheus/prometheus.yml`

```yaml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'node'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9100']
```

- setting systemd script `/etc/systemd/system/prometheus.service`

```bash
[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network-online.target

[Service]
User=prometheus
Restart=on-failure

#Change this line if you download the
#Prometheus on different path user
ExecStart=/usr/local/prometheus/prometheus \
  --config.file=/usr/local/prometheus/prometheus.yml \
  --storage.tsdb.path=/usr/local/prometheus/data

[Install]
WantedBy=multi-user.target
```

```
chmod u+x /etc/systemd/system/prometheus.service
chown prometheus.prometheus /etc/systemd/system/prometheus.service
```

- start & enable service

```bash
systemctl start prometheus
systemctl status prometheus
systemctl enable prometheus
```

## install node_exporter

- download & install

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v0.17.0/node_exporter-0.17.0.linux-amd64.tar.gz -O /tmp/node_exporter.tar.gz
tar xvf /tmp/node_exporter.tar.gz -C /usr/local
mv -v /usr/local/node_exporter-0.17.0.linux-amd64 /usr/local/node_exporter
chown -R prometheus.prometheus /usr/local/node_exporter
```

- setting systemd script `/etc/systemd/system/node_exporter.service`

```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/node_exporter/node_exporter

[Install]
WantedBy=default.target
```

```
chmod u+x /etc/systemd/system/node_exporter.service
chown prometheus.prometheus /etc/systemd/system/node_exporter.service
```

- start & enable service

```bash
systemctl start node_exporter
systemctl status node_exporter
systemctl enable node_exporter
```

## install grafana

- download & install

```bash
wget https://dl.grafana.com/oss/release/grafana-6.0.1-1.x86_64.rpm
sudo yum localinstall grafana-6.0.1-1.x86_64.rpm -y
```

- install plugin

```bash
grafana-cli plugins install grafana-piechart-panel
```

- start & enable service

```bash
systemctl start grafana-server
systemctl status grafana-server
systemctl enable grafana-server.service
```

## install alertmanager

- download & install

```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.16.1/alertmanager-0.16.1.linux-amd64.tar.gz -O /tmp/alertmanager.tar.gz
tar xf /tmp/alertmanager.tar.gz -C /usr/local
mv /usr/local/alertmanager-0.16.1.linux-amd64/ /usr/local/alertmanager
chown prometheus.prometheus /usr/local/alertmanager
```

- setting configure `alertmanager.yml`

```yaml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'send@gmail.com'
  smtp_hello: 'kitsnail-tokyo'
  smtp_auth_username: 'send@gmail.com'
  smtp_auth_password: 'sendpasswd'
  #smtp_require_tls: false

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'recv'

receivers:
- name: 'recv'
  email_configs:
  - to: 'recv@gmail.com'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

- setting systemd script `/etc/systemd/system/alertmanager.service`

```
[Unit]
Description=Alertmanager
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/usr/local/prometheus/alertmanager/alertmanager --config.file=/usr/local/prometheus/alertmanager/alertmanager.yml --storage.path=/usr/local/prometheus/alertmanager/data
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```
chmod u+x /etc/systemd/system/alertmanager.service
chown prometheus.prometheus /etc/systemd/system/alertmanager.service
```

- start & enable service

```bash
systemctl start alertmanager
systemctl status alertmanager
systemctl enable alertmanager.service
```

