# 🚨 Integrating Grafana with Alertmanager

![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?style=for-the-badge\&logo=prometheus\&logoColor=white)
![Alertmanager](https://img.shields.io/badge/Alertmanager-Alert%20Management-E6522C?style=for-the-badge\&logo=prometheus\&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-Visualization-F46800?style=for-the-badge\&logo=grafana\&logoColor=white)
![Node Exporter](https://img.shields.io/badge/Node%20Exporter-Metrics-76D04B?style=for-the-badge\&logo=prometheus\&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-E95420?style=for-the-badge\&logo=ubuntu\&logoColor=white)
![Python](https://img.shields.io/badge/Python-Flask-3776AB?style=for-the-badge\&logo=python\&logoColor=white)
![YAML](https://img.shields.io/badge/Config-YAML-000000?style=for-the-badge\&logo=yaml\&logoColor=white)

> 📊 **Build a complete monitoring and alerting stack with Prometheus, Alertmanager, Grafana, and Node Exporter.**

---

## 📌 Lab Overview

This lab demonstrates how to integrate **Grafana with Prometheus Alertmanager** to create a complete monitoring and alert-management solution on a Linux machine.

The environment collects infrastructure metrics using **Node Exporter**, stores and evaluates metrics with **Prometheus**, manages and routes alerts using **Alertmanager**, and visualizes system health through **Grafana**.

The complete alert pipeline is:

```text
┌─────────────────┐
│  Node Exporter  │
│  System Metrics │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    Prometheus   │
│ Metrics + Rules │
└────────┬────────┘
         │ Alerts
         ▼
┌─────────────────┐
│   Alertmanager  │
│ Routing/Grouping│
└───────┬─────────┘
        │
        ├──────────────► 📧 Email
        │
        ├──────────────► 🔗 Webhook
        │
        ▼
┌─────────────────┐
│     Grafana     │
│ Visualization   │
│ Alert Management│
└─────────────────┘
```

---

## 🎯 Learning Objectives

By completing this lab, you will learn how to:

* 🟢 Install and configure Prometheus
* 🟢 Install and configure Node Exporter
* 🟢 Install and configure Alertmanager
* 🟢 Configure Prometheus alerting rules
* 🟢 Connect Prometheus to Alertmanager
* 🟢 Install and configure Grafana
* 🟢 Configure Prometheus as a Grafana data source
* 🟢 Configure Alertmanager as a Grafana data source
* 🟢 Build a monitoring dashboard
* 🟢 Configure alert notifications
* 🟢 Create a webhook receiver
* 🟢 Generate test alerts
* 🟢 Verify alert delivery
* 🟢 Test alert resolution
* 🟢 Troubleshoot the complete monitoring pipeline

---

## 🧰 Technology Stack

| Technology            | Purpose                                                |
| --------------------- | ------------------------------------------------------ |
| 🟠 **Prometheus**     | Metrics collection and alert evaluation                |
| 🔴 **Alertmanager**   | Alert grouping, routing, inhibition, and notifications |
| 🟠 **Grafana**        | Metrics visualization and alert management             |
| 🟢 **Node Exporter**  | Linux system metrics                                   |
| 🐍 **Python + Flask** | Webhook testing                                        |
| 🐧 **Linux**          | Lab operating system                                   |
| 📄 **YAML**           | Prometheus and Alertmanager configuration              |
| ⚙️ **systemd**        | Service management                                     |
| 🔥 **stress**         | Alert testing                                          |

---

# 🏁 Prerequisites

Before starting, you should have:

* Basic Linux command-line knowledge
* Familiarity with YAML
* Basic Prometheus knowledge
* Basic Grafana knowledge
* Understanding of monitoring concepts
* Experience with `nano` or `vim`
* Basic understanding of HTTP/webhooks

---

# ☁️ Lab Environment

The lab can be performed on an **Al Nafi Linux cloud machine**.

The machine is assumed to be a clean Linux environment, so the required monitoring components are installed during the lab.

### Main Services

| Service       | Default Port |
| ------------- | -----------: |
| Prometheus    |       `9090` |
| Alertmanager  |       `9093` |
| Grafana       |       `3000` |
| Node Exporter |       `9100` |
| Flask Webhook |       `5001` |

---

# 🚀 Task 1 — Set Up Prometheus and Node Exporter

## 🔹 Step 1 — Update the System

```bash
sudo apt update
sudo apt install -y wget curl tar
```

## 🔹 Step 2 — Create Service Users

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo useradd --no-create-home --shell /bin/false node_exporter
```

## 🔹 Step 3 — Create Prometheus Directories

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

## 🔹 Step 4 — Install Prometheus

```bash
cd /tmp

wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz

tar xvf prometheus-2.45.0.linux-amd64.tar.gz

cd prometheus-2.45.0.linux-amd64

sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/

sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool

sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus

sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

## 🔹 Step 5 — Install Node Exporter

```bash
cd /tmp

wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz

tar xvf node_exporter-1.6.1.linux-amd64.tar.gz

cd node_exporter-1.6.1.linux-amd64

sudo cp node_exporter /usr/local/bin

sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

---

# ⚙️ Task 2 — Configure Prometheus Alerting

## 🔹 Step 1 — Create Prometheus Configuration

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - localhost:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

## 🔹 Step 2 — Create Alert Rules

```bash
sudo nano /etc/prometheus/alert_rules.yml
```

Add:

```yaml
groups:
- name: system_alerts
  rules:

  - alert: HighCPUUsage
    expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage detected"
      description: "CPU usage is above 80% for more than 2 minutes on {{ $labels.instance }}"

  - alert: HighMemoryUsage
    expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High memory usage detected"
      description: "Memory usage is above 85% for more than 2 minutes on {{ $labels.instance }}"

  - alert: DiskSpaceLow
    expr: (1 - (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"})) * 100 > 90
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Disk space is running low"
      description: "Disk usage is above 90% on {{ $labels.instance }} filesystem {{ $labels.mountpoint }}"

  - alert: ServiceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Service is down"
      description: "{{ $labels.job }} service is down on {{ $labels.instance }}"
```

## 🔹 Step 3 — Set Ownership

```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
sudo chown prometheus:prometheus /etc/prometheus/alert_rules.yml
```

## 🔍 Validate the Configuration

```bash
/usr/local/bin/promtool check config /etc/prometheus/prometheus.yml
/usr/local/bin/promtool check rules /etc/prometheus/alert_rules.yml
```

---

# 🚨 Task 3 — Install and Configure Alertmanager

## 🔹 Step 1 — Create Alertmanager User

```bash
sudo useradd --no-create-home --shell /bin/false alertmanager

sudo mkdir /etc/alertmanager
sudo mkdir /var/lib/alertmanager

sudo chown alertmanager:alertmanager /etc/alertmanager
sudo chown alertmanager:alertmanager /var/lib/alertmanager
```

## 🔹 Step 2 — Download Alertmanager

```bash
cd /tmp

wget https://github.com/prometheus/alertmanager/releases/download/v0.26.0/alertmanager-0.26.0.linux-amd64.tar.gz

tar xvf alertmanager-0.26.0.linux-amd64.tar.gz

cd alertmanager-0.26.0.linux-amd64

sudo cp alertmanager /usr/local/bin
sudo cp amtool /usr/local/bin

sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
sudo chown alertmanager:alertmanager /usr/local/bin/amtool
```

## 🔹 Step 3 — Configure Alertmanager

```bash
sudo nano /etc/alertmanager/alertmanager.yml
```

Example configuration:

```yaml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alertmanager@example.com'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'

receivers:

- name: 'web.hook'
  webhook_configs:
  - url: 'http://localhost:5001/webhook'
    send_resolved: true

- name: 'email-notifications'
  email_configs:
  - to: 'admin@example.com'
    subject: 'Alert: {{ .GroupLabels.alertname }}'
    body: |
      {{ range .Alerts }}
      Alert: {{ .Annotations.summary }}
      Description: {{ .Annotations.description }}
      {{ end }}

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal:
      - 'alertname'
      - 'dev'
      - 'instance'
```

> ⚠️ Replace the example SMTP server and email addresses with valid values before using email notifications in a real environment.

## 🔹 Step 4 — Set Ownership

```bash
sudo chown alertmanager:alertmanager /etc/alertmanager/alertmanager.yml
```

---

# ⚙️ Task 4 — Create systemd Services

## 🔹 Node Exporter

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

## 🔹 Prometheus

```bash
sudo nano /etc/systemd/system/prometheus.service
```

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

## 🔹 Alertmanager

```bash
sudo nano /etc/systemd/system/alertmanager.service
```

```ini
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
    --config.file=/etc/alertmanager/alertmanager.yml \
    --storage.path=/var/lib/alertmanager/

[Install]
WantedBy=multi-user.target
```

## 🔹 Start All Services

```bash
sudo systemctl daemon-reload

sudo systemctl start node_exporter
sudo systemctl enable node_exporter

sudo systemctl start prometheus
sudo systemctl enable prometheus

sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

## 🔍 Verify

```bash
sudo systemctl status node_exporter
sudo systemctl status prometheus
sudo systemctl status alertmanager
```

---

# 📊 Task 5 — Install Grafana

## 🔹 Step 1 — Install Dependencies

```bash
sudo apt-get install -y software-properties-common
```

## 🔹 Step 2 — Add Grafana Repository

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" | \
sudo tee -a /etc/apt/sources.list.d/grafana.list
```

## 🔹 Step 3 — Install Grafana

```bash
sudo apt-get update
sudo apt-get install -y grafana
```

## 🔹 Step 4 — Start Grafana

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
sudo systemctl status grafana-server
```

---

# 🌐 Task 6 — Access Grafana

Open:

```text
http://localhost:3000
```

Default lab credentials:

```text
Username: admin
Password: admin
```

> 🔐 Change the default password immediately in any non-lab environment.

---

# 🔗 Task 7 — Configure Grafana Data Sources

## 🔹 Prometheus Data Source

Create:

```bash
cat > /tmp/prometheus-datasource.json << EOF
{
  "name": "Prometheus",
  "type": "prometheus",
  "url": "http://localhost:9090",
  "access": "proxy",
  "isDefault": true
}
EOF
```

Add it:

```bash
curl -X POST \
  http://admin:admin@localhost:3000/api/datasources \
  -H 'Content-Type: application/json' \
  -d @/tmp/prometheus-datasource.json
```

## 🔹 Alertmanager Data Source

Create:

```bash
cat > /tmp/alertmanager-datasource.json << EOF
{
  "name": "Alertmanager",
  "type": "alertmanager",
  "url": "http://localhost:9093",
  "access": "proxy"
}
EOF
```

Add it:

```bash
curl -X POST \
  http://admin:admin@localhost:3000/api/datasources \
  -H 'Content-Type: application/json' \
  -d @/tmp/alertmanager-datasource.json
```

---

# 📈 Task 8 — Create Monitoring Dashboard

The dashboard contains:

* 🖥️ CPU Usage
* 🧠 Memory Usage
* 🚨 Active Alerts
* ⏱️ Automatic refresh
* 📊 Prometheus-based metrics

Create the dashboard JSON:

```bash
cat > /tmp/monitoring-dashboard.json << 'EOF'
{
  "dashboard": {
    "id": null,
    "title": "System Monitoring with Alerts",
    "tags": ["monitoring", "alerts"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "CPU Usage",
        "type": "stat",
        "targets": [
          {
            "expr": "100 - (avg by(instance) (irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "steps": [
                {"color": "green", "value": null},
                {"color": "yellow", "value": 70},
                {"color": "red", "value": 80}
              ]
            },
            "unit": "percent"
          }
        },
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0}
      },
      {
        "id": 2,
        "title": "Memory Usage",
        "type": "stat",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "steps": [
                {"color": "green", "value": null},
                {"color": "yellow", "value": 75},
                {"color": "red", "value": 85}
              ]
            },
            "unit": "percent"
          }
        },
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0}
      },
      {
        "id": 3,
        "title": "Active Alerts",
        "type": "alertlist",
        "targets": [],
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 8}
      }
    ],
    "time": {"from": "now-1h", "to": "now"},
    "refresh": "5s"
  }
}
EOF
```

Import it:

```bash
curl -X POST \
  http://admin:admin@localhost:3000/api/dashboards/db \
  -H 'Content-Type: application/json' \
  -d @/tmp/monitoring-dashboard.json
```

---

# 🔔 Task 9 — Configure Grafana Alert Notifications

Create the notification configuration:

```bash
cat > /tmp/notification-channel.json << EOF
{
  "name": "alertmanager-webhook",
  "type": "webhook",
  "settings": {
    "url": "http://localhost:9093/api/v1/alerts",
    "httpMethod": "POST"
  }
}
EOF
```

Then:

```bash
curl -X POST \
  http://admin:admin@localhost:3000/api/alert-notifications \
  -H 'Content-Type: application/json' \
  -d @/tmp/notification-channel.json
```

> ℹ️ Grafana alerting APIs and notification configuration have changed across Grafana releases. For newer Grafana versions, use the current Grafana Alerting UI/API model if the legacy endpoint above is unavailable.

---

# 🧪 Task 10 — Build a Test Webhook Server

## 🔹 Step 1 — Install Flask

```bash
sudo apt install -y python3 python3-pip
pip3 install flask
```

## 🔹 Step 2 — Create Webhook Server

```bash
cat > /tmp/webhook_server.py << 'EOF'
from flask import Flask, request, jsonify
import json
from datetime import datetime

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.get_json()
    timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')

    print(f"\n[{timestamp}] Alert received:")
    print(json.dumps(data, indent=2))

    with open('/tmp/alerts.log', 'a') as f:
        f.write(f"[{timestamp}] {json.dumps(data)}\n")

    return jsonify({"status": "received"}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001, debug=True)
EOF
```

## 🔹 Step 3 — Start Webhook

```bash
cd /tmp

python3 webhook_server.py &

WEBHOOK_PID=$!

echo $WEBHOOK_PID > /tmp/webhook.pid
```

---

# 🔥 Task 11 — Generate Test Alerts

## 🔹 CPU Stress

Install `stress`:

```bash
sudo apt install -y stress
```

Start CPU stress:

```bash
stress --cpu 4 --timeout 300s &

STRESS_PID=$!

echo "Stress test started with PID: $STRESS_PID"
```

## 🔹 Check Prometheus Alerts

```bash
curl -s http://localhost:9090/api/v1/alerts | \
python3 -m json.tool
```

## 🔹 Check Alertmanager

```bash
curl -s http://localhost:9093/api/v1/alerts | \
python3 -m json.tool
```

## 🔹 Monitor Webhook

```bash
tail -f /tmp/alerts.log
```

---

# 🖥️ Task 12 — Verify Alerts in Grafana

Open:

```text
http://localhost:3000
```

Check the monitoring dashboard.

Verify:

* 📈 CPU usage increases
* 🚨 Active alerts appear
* 🔔 Alert rules change state
* 🔗 Alertmanager receives alerts
* 📡 Webhook receives notifications

Then open:

```text
http://localhost:9093
```

Verify:

* Active alerts
* Alert grouping
* Alert labels
* Alert status
* Notification activity

---

# ✅ Task 13 — Test Alert Resolution

Stop the stress process:

```bash
kill $STRESS_PID
echo "Stress test stopped"
```

Wait for the alert to resolve:

```bash
sleep 120
```

Check Alertmanager:

```bash
curl -s http://localhost:9093/api/v1/alerts | \
python3 -m json.tool
```

Check webhook resolution messages:

```bash
tail -10 /tmp/alerts.log
```

Because `send_resolved: true` is configured, the webhook can receive resolution notifications.

---

# 🧠 Task 14 — Test Memory Alerts

Generate memory pressure:

```bash
stress --vm 2 --vm-bytes 1G --timeout 180s &

MEMORY_STRESS_PID=$!

echo "Memory stress test started with PID: $MEMORY_STRESS_PID"
```

Monitor:

```bash
curl -s http://localhost:9090/api/v1/alerts | \
grep -E "(HighCPUUsage|HighMemoryUsage|DiskSpaceLow)"
```

---

# 💾 Task 15 — Test Disk Alerts

> ⚠️ Use caution when consuming disk space.

```bash
dd if=/dev/zero of=/tmp/large_file bs=1M count=100
```

Check alerts:

```bash
curl -s http://localhost:9090/api/v1/alerts
```

Remove the test file afterward:

```bash
rm -f /tmp/large_file
```

---

# 🧹 Task 16 — Clean Up

Stop stress processes:

```bash
pkill stress
```

Remove test files:

```bash
rm -f /tmp/large_file
```

Stop the webhook:

```bash
if [ -f /tmp/webhook.pid ]; then
    kill $(cat /tmp/webhook.pid)
    rm /tmp/webhook.pid
fi
```

---

# 🔧 Troubleshooting

## ❌ Problem 1 — Services Are Not Starting

Check logs:

```bash
sudo journalctl -u prometheus -f
sudo journalctl -u alertmanager -f
sudo journalctl -u grafana-server -f
```

Check status:

```bash
sudo systemctl status prometheus
sudo systemctl status alertmanager
sudo systemctl status grafana-server
```

---

## ❌ Problem 2 — Alerts Are Not Firing

Validate configuration:

```bash
/usr/local/bin/promtool check config \
/etc/prometheus/prometheus.yml
```

Validate rules:

```bash
/usr/local/bin/promtool check rules \
/etc/prometheus/alert_rules.yml
```

Check rules:

```bash
curl -s http://localhost:9090/api/v1/rules
```

---

## ❌ Problem 3 — Alertmanager Is Not Receiving Alerts

Check Prometheus targets:

```bash
curl http://localhost:9090/api/v1/targets
```

Check Alertmanager:

```bash
curl http://localhost:9093/api/v1/alerts
```

---

## ❌ Problem 4 — Grafana Cannot Connect to Prometheus

Test Prometheus:

```bash
curl http://localhost:9090/-/healthy
```

Check Grafana:

```bash
curl http://localhost:3000/api/health
```

---

## ❌ Problem 5 — Webhook Is Not Receiving Alerts

Check Alertmanager configuration:

```bash
/usr/local/bin/amtool config show \
--config.file=/etc/alertmanager/alertmanager.yml
```

Check webhook:

```bash
curl http://localhost:5001/webhook
```

Check logs:

```bash
tail -f /tmp/alerts.log
```

---

# 🔍 Verification Commands

## 🟢 Check All Services

```bash
sudo systemctl status \
node_exporter \
prometheus \
alertmanager \
grafana-server
```

## 🟢 Check Prometheus Targets

```bash
curl -s http://localhost:9090/api/v1/targets
```

## 🟢 Check Alert Rules

```bash
curl -s http://localhost:9090/api/v1/rules
```

## 🟢 Check Prometheus Alerts

```bash
curl -s http://localhost:9090/api/v1/alerts
```

## 🟢 Check Alertmanager

```bash
curl -s http://localhost:9093/api/v1/status
```

## 🟢 Check Grafana

```bash
curl -s http://localhost:3000/api/health
```

---

# 🧩 Alert Severity Model

This lab uses two primary severity levels:

| Severity      | Example      | Purpose                          |
| ------------- | ------------ | -------------------------------- |
| 🟡 `warning`  | CPU > 80%    | Requires attention               |
| 🔴 `critical` | Memory > 85% | Requires immediate investigation |

Example:

```yaml
labels:
  severity: critical
```

Severity labels can later be used by Alertmanager to create sophisticated routing policies.

---

# 🔀 Alert Routing Concept

Alertmanager provides several important alert-management capabilities:

```text
                  Prometheus
                      │
                      │ Alert
                      ▼
               ┌─────────────┐
               │ Alertmanager│
               └──────┬──────┘
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Grouping    Routing    Inhibition
          │           │           │
          ▼           ▼           ▼
       Reduce      Select       Suppress
        Noise     Receiver     Warnings
```

### Grouping

Combines related alerts into a single notification.

### Routing

Determines where alerts should be delivered.

### Inhibition

Suppresses lower-priority alerts when a more severe alert is already active.

### Deduplication

Prevents repeated notifications for the same alert event.

---

# 📚 What You Learned

After completing this lab, you should understand the role of each component:

### 🟠 Node Exporter

Collects Linux system metrics such as:

* CPU
* Memory
* Disk
* Filesystem
* Network
* System availability

### 🔴 Prometheus

Responsible for:

* Scraping metrics
* Storing time-series data
* Evaluating PromQL alert expressions
* Sending firing alerts to Alertmanager

### 🚨 Alertmanager

Responsible for:

* Alert grouping
* Alert routing
* Deduplication
* Inhibition
* Notification delivery
* Resolution notifications

### 📊 Grafana

Responsible for:

* Visualization
* Dashboards
* Alert visualization
* Alert management
* Monitoring system health

---

# 🌟 Key Benefits

This architecture provides:

* 🚨 Centralized alert management
* 📊 Real-time monitoring
* 🔔 Flexible notifications
* 🔀 Advanced alert routing
* 🧩 Alert grouping
* 🔇 Noise reduction
* 🔄 Alert resolution tracking
* 📈 Historical metric visualization
* 🛠️ Easier incident troubleshooting
* ☁️ A foundation for scalable observability

---

# 🏆 Final Architecture

```text
                         ┌───────────────────┐
                         │    Linux Host     │
                         │                   │
                         │   Node Exporter   │
                         └─────────┬─────────┘
                                   │
                                   │ Metrics
                                   ▼
                         ┌───────────────────┐
                         │    Prometheus     │
                         │                   │
                         │  TSDB + PromQL    │
                         │   Alert Rules     │
                         └─────────┬─────────┘
                                   │
                                   │ Firing Alerts
                                   ▼
                         ┌───────────────────┐
                         │   Alertmanager    │
                         │                   │
                         │ Grouping          │
                         │ Routing           │
                         │ Inhibition        │
                         │ Deduplication     │
                         └──────┬──────┬─────┘
                                │      │
                    ┌───────────┘      └───────────┐
                    ▼                              ▼
             ┌─────────────┐                ┌─────────────┐
             │   Webhook   │                │    Email    │
             └─────────────┘                └─────────────┘

                                   ▲
                                   │
                                   │ Alert Data
                                   │
                         ┌─────────┴─────────┐
                         │      Grafana      │
                         │                   │
                         │ Dashboards        │
                         │ Alert Visualization│
                         └───────────────────┘
```

---

# 🎓 Conclusion

This lab demonstrates how to build a complete **Prometheus + Alertmanager + Grafana monitoring and alerting platform**.

You installed and configured:

* ✅ Prometheus
* ✅ Node Exporter
* ✅ Alertmanager
* ✅ Grafana
* ✅ Python/Flask webhook

You also created system-level alert rules for:

* 🖥️ High CPU usage
* 🧠 High memory usage
* 💾 Low disk space
* 🔴 Service failures

Finally, you tested the complete lifecycle:

```text
Metric Collection
       ↓
Prometheus Scraping
       ↓
Rule Evaluation
       ↓
Alert Firing
       ↓
Alertmanager
       ↓
Grouping / Routing
       ↓
Notification
       ↓
Grafana Visualization
       ↓
Alert Resolution
```

This architecture provides a strong foundation for **enterprise observability, proactive incident detection, infrastructure monitoring, and DevOps/SRE operations**.

---

## 🚀 Future Enhancements

The lab can be extended with:

* 📧 Production SMTP integration
* 💬 Slack notifications
* 📱 Microsoft Teams notifications
* 📲 PagerDuty integration
* ☁️ Cloud monitoring
* 🐳 Docker monitoring
* ☸️ Kubernetes monitoring
* 🔐 TLS authentication
* 🔑 API authentication
* 🧭 Advanced Alertmanager routing trees
* 📊 Custom Prometheus exporters
* 📈 Advanced Grafana dashboards
* 🏢 Multi-node Prometheus architecture
* 🌍 High-availability Alertmanager
* 🗄️ Long-term metrics storage with Thanos

---

<div align="center">

### ⭐ Monitoring • Alerting • Observability ⭐

**Prometheus + Alertmanager + Grafana + Node Exporter**

**Built for proactive infrastructure monitoring and incident response.**

</div>
