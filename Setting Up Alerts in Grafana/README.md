# 🚨 Setting Up Alerts in Grafana

![Grafana](https://img.shields.io/badge/Grafana-Alerting-orange?style=for-the-badge\&logo=grafana)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?style=for-the-badge\&logo=prometheus)
![Alertmanager](https://img.shields.io/badge/Alertmanager-Notifications-FF6F00?style=for-the-badge)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-E95420?style=for-the-badge\&logo=ubuntu)
![Node Exporter](https://img.shields.io/badge/Node%20Exporter-Metrics-5C5C5C?style=for-the-badge)

> 🛡️ **A practical monitoring and alerting lab using Grafana, Prometheus, Node Exporter, and Alertmanager.**

---

## 📖 Lab Overview

This lab demonstrates how to build a complete **monitoring and alerting stack** on a single Linux machine.

You will install and configure:

* 📊 **Grafana** — visualization and alert management
* 🔥 **Prometheus** — metrics collection and alert-rule evaluation
* 🖥️ **Node Exporter** — Linux system metrics
* 🚨 **Alertmanager** — alert routing, grouping, silencing, and notifications

The lab also demonstrates how to create alerts for **CPU, memory, disk, service availability, and network traffic**, then deliver notifications through **email and Slack**.

---

## 🎯 Learning Objectives

By completing this lab, you will be able to:

* 🚀 Install and configure Grafana on Linux
* 📡 Configure Prometheus as a Grafana data source
* 📈 Collect system metrics using Node Exporter
* 🚨 Create Prometheus and Grafana alert rules
* 📧 Configure email notifications
* 💬 Configure Slack notifications
* 🔀 Route alerts according to severity
* 🔇 Silence unwanted alerts
* 📦 Group related alerts
* 🧪 Generate system load to test alerts
* 🔍 Troubleshoot alerting failures
* ✅ Verify the complete monitoring pipeline

---

## 🧠 Monitoring Architecture

```text
                   ┌─────────────────────┐
                   │     Linux Server    │
                   │                     │
                   │   Node Exporter     │
                   │   CPU / RAM / Disk  │
                   │   Network / System  │
                   └──────────┬──────────┘
                              │
                              │ Metrics
                              ▼
                   ┌─────────────────────┐
                   │     Prometheus      │
                   │                     │
                   │ • Scrapes metrics   │
                   │ • Evaluates rules   │
                   │ • Stores metrics    │
                   └──────────┬──────────┘
                              │
                   Alerts    │
                              ▼
                   ┌─────────────────────┐
                   │    Alertmanager     │
                   │                     │
                   │ • Routing            │
                   │ • Grouping           │
                   │ • Silencing          │
                   │ • Inhibition         │
                   └──────┬────────┬─────┘
                          │        │
                    Email │        │ Slack
                          ▼        ▼
                     📧 Alerts   💬 Alerts

                   ┌─────────────────────┐
                   │       Grafana       │
                   │                     │
                   │ Dashboards + Alerts │
                   └──────────┬──────────┘
                              │
                              ▼
                       👨‍💻 Administrator
```

---

# 🧰 Prerequisites

Before beginning, you should have:

* 🐧 Basic Linux command-line knowledge
* 📊 Understanding of monitoring concepts
* 🌐 Basic networking knowledge
* 📝 Familiarity with YAML
* 🔗 Basic HTTP/web-service knowledge
* 🔐 `sudo` privileges

---

# ☁️ Lab Environment

The lab is designed for an **Al Nafi Linux cloud machine**.

The environment provides a bare Linux machine, so required software is installed manually.

All components run on **one Linux machine**:

| Component     | Purpose               | Default Port |
| ------------- | --------------------- | -----------: |
| Grafana       | Dashboards & alerting |       `3000` |
| Prometheus    | Metrics & rules       |       `9090` |
| Alertmanager  | Alert routing         |       `9093` |
| Node Exporter | System metrics        |       `9100` |

---

# 🛠️ Task 1 — Environment Setup

## 🔄 Step 1: Update the System

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y wget curl software-properties-common apt-transport-https
```

### ✅ Expected Result

The operating system is updated and required installation utilities are available.

---

# 📡 Step 2: Install Prometheus

Create the Prometheus user:

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

Create directories:

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

Download Prometheus:

```bash
cd /tmp

wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz

tar xvf prometheus-2.45.0.linux-amd64.tar.gz
```

Install the binaries:

```bash
sudo cp prometheus-2.45.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.45.0.linux-amd64/promtool /usr/local/bin/

sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

Copy Prometheus console files:

```bash
sudo cp -r prometheus-2.45.0.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.45.0.linux-amd64/console_libraries /etc/prometheus

sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

---

# ⚙️ Step 3: Configure Prometheus

Create the configuration:

```bash
sudo tee /etc/prometheus/prometheus.yml > /dev/null <<EOF
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
EOF
```

Set ownership:

```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

### 🔎 Configuration Highlights

* `scrape_interval: 15s` → collects metrics every 15 seconds
* `evaluation_interval: 15s` → evaluates alert rules every 15 seconds
* `rule_files` → loads alerting rules
* `alerting` → connects Prometheus to Alertmanager
* `scrape_configs` → defines monitored targets

---

# ⚙️ Step 4: Create Prometheus Service

```bash
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
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
EOF
```

---

# 🖥️ Step 5: Install Node Exporter

Download Node Exporter:

```bash
cd /tmp

wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz

tar xvf node_exporter-1.6.1.linux-amd64.tar.gz
```

Install it:

```bash
sudo cp node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/node_exporter
```

Create its systemd service:

```bash
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF
```

---

# 📊 Step 6: Install Grafana

Add the Grafana repository:

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" | \
sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Install Grafana:

```bash
sudo apt update
sudo apt install -y grafana
```

> 💡 **Note:** The repository/key method shown above follows the lab material. For a new production deployment, use Grafana's current official installation instructions because repository configuration can change over time.

---

# 🚀 Step 7: Start Monitoring Services

```bash
sudo systemctl daemon-reload

sudo systemctl enable prometheus node_exporter grafana-server

sudo systemctl start prometheus node_exporter grafana-server
```

Check the services:

```bash
sudo systemctl status prometheus
sudo systemctl status node_exporter
sudo systemctl status grafana-server
```

### ✅ Expected Result

All three services should show:

```text
Active: active (running)
```

---

# 🖥️ Task 2 — Configure Grafana

## 🌐 Step 1: Access Grafana

Open:

```text
http://localhost:3000
```

Default credentials:

```text
Username: admin
Password: admin
```

You will normally be prompted to change the password after the first login.

---

# 🔌 Step 2: Add Prometheus Data Source

In Grafana:

```text
⚙️ Configuration
      ↓
Data Sources
      ↓
Add data source
      ↓
Prometheus
```

Configure:

```text
Name: Prometheus
URL: http://localhost:9090
Access: Server
```

Click:

```text
Save & Test
```

### ✅ Expected Result

Grafana should confirm that the Prometheus data source is working.

---

# 📈 Step 3: Import Node Exporter Dashboard

Navigate to:

```text
+ → Import
```

Enter:

```text
1860
```

This imports the **Node Exporter Full** dashboard.

Select:

```text
Prometheus
```

as the data source and click:

```text
Import
```

You should now see CPU, memory, disk, network, and system information.

---

# 🚨 Task 3 — Create Prometheus Alert Rules

Create the alert rules file:

```bash
sudo tee /etc/prometheus/alert_rules.yml > /dev/null <<EOF
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
      description: "CPU usage is above 80% for more than 2 minutes on {{ \$labels.instance }}"

  - alert: HighMemoryUsage
    expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High memory usage detected"
      description: "Memory usage is above 85% for more than 2 minutes on {{ \$labels.instance }}"

  - alert: DiskSpaceLow
    expr: (1 - (node_filesystem_avail_bytes{fstype!="tmpfs"} / node_filesystem_size_bytes{fstype!="tmpfs"})) * 100 > 90
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Disk space is running low"
      description: "Disk usage is above 90% on {{ \$labels.instance }} filesystem {{ \$labels.mountpoint }}"

  - alert: ServiceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Service is down"
      description: "{{ \$labels.job }} service is down on {{ \$labels.instance }}"
EOF
```

Set ownership:

```bash
sudo chown prometheus:prometheus /etc/prometheus/alert_rules.yml
```

---

# 🚨 Alert Rules Explained

| Alert                | Condition                  | Severity |
| -------------------- | -------------------------- | -------- |
| 🔥 `HighCPUUsage`    | CPU > 80% for 2 minutes    | Warning  |
| 🧠 `HighMemoryUsage` | Memory > 85% for 2 minutes | Warning  |
| 💾 `DiskSpaceLow`    | Disk usage > 90%           | Critical |
| 🔴 `ServiceDown`     | Target unavailable         | Critical |

The `for` field prevents temporary spikes from immediately generating alerts.

---

# 🚨 Task 4 — Install Alertmanager

Create the Alertmanager user:

```bash
sudo useradd --no-create-home --shell /bin/false alertmanager
```

Create directories:

```bash
sudo mkdir /etc/alertmanager
sudo mkdir /var/lib/alertmanager

sudo chown alertmanager:alertmanager /etc/alertmanager
sudo chown alertmanager:alertmanager /var/lib/alertmanager
```

Download Alertmanager:

```bash
cd /tmp

wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz

tar xvf alertmanager-0.25.0.linux-amd64.tar.gz
```

Install binaries:

```bash
sudo cp alertmanager-0.25.0.linux-amd64/alertmanager /usr/local/bin/
sudo cp alertmanager-0.25.0.linux-amd64/amtool /usr/local/bin/

sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
sudo chown alertmanager:alertmanager /usr/local/bin/amtool
```

---

# 📧 Configure Alertmanager

Create:

```bash
sudo tee /etc/alertmanager/alertmanager.yml > /dev/null <<EOF
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alerts@yourdomain.com'
  smtp_auth_username: 'your-email@gmail.com'
  smtp_auth_password: 'your-app-password'

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'

receivers:
- name: 'web.hook'
  email_configs:
  - to: 'admin@yourdomain.com'
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
    equal: ['alertname', 'dev', 'instance']
EOF
```

Set ownership:

```bash
sudo chown alertmanager:alertmanager /etc/alertmanager/alertmanager.yml
```

> 🔐 **Security:** Never commit real SMTP passwords, app passwords, or Slack webhook URLs to GitHub. Use protected configuration, environment-specific secrets, or a secret manager.

---

# ⚙️ Create Alertmanager Service

```bash
sudo tee /etc/systemd/system/alertmanager.service > /dev/null <<EOF
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
    --storage.path=/var/lib/alertmanager/ \
    --web.listen-address=0.0.0.0:9093

[Install]
WantedBy=multi-user.target
EOF
```

Start Alertmanager:

```bash
sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
```

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

Verify:

```bash
sudo systemctl status alertmanager
sudo systemctl status prometheus
```

---

# 📬 Task 5 — Configure Notification Channels

## 📧 Email Testing

Install mail utilities:

```bash
sudo apt install -y mailutils
```

Test:

```bash
echo "Test email from Alertmanager" | mail -s "Test Alert" $USER
```

Check local mail:

```bash
mail
```

---

# 💬 Slack Integration

Create a Slack webhook and place it in the Alertmanager configuration.

Example:

```yaml
receivers:

- name: 'critical-alerts'
  slack_configs:
  - api_url: 'YOUR_SLACK_WEBHOOK_URL_HERE'
    channel: '#alerts'
    title: 'Critical Alert'
    text: |
      {{ range .Alerts }}
      Alert: {{ .Annotations.summary }}
      Description: {{ .Annotations.description }}
      {{ end }}
```

Restart Alertmanager:

```bash
sudo systemctl restart alertmanager
```

### 🔀 Severity-Based Routing

```text
                 Alert
                   │
                   ▼
            ┌──────────────┐
            │ Alertmanager │
            └──────┬───────┘
                   │
          ┌────────┴────────┐
          │                 │
      Critical            Warning
          │                 │
      📧 Email          📧 Email
      💬 Slack
```

---

# 📊 Task 6 — Configure Grafana Alerts

Depending on your Grafana version, menu names may differ from older tutorials.

Typical workflow:

```text
Alerting
   ↓
Alert rules
   ↓
New alert rule
```

Configure:

```text
Name: High CPU Usage
Evaluate every: 10s
For: 1m
Condition: CPU > 80
```

Add an appropriate notification/contact point and save the rule.

### 🎯 Example Alert

```text
Name: High CPU Usage

Condition:
CPU > 80%

Evaluation:
Every 10 seconds

Duration:
1 minute

Severity:
Warning
```

---

# 🧪 Task 7 — Test CPU Alerts

Create a CPU stress script:

```bash
cat > /tmp/cpu_stress.sh << 'EOF'
#!/bin/bash

echo "Starting CPU stress test..."

for i in {1..4}; do
    yes > /dev/null &
done

echo "CPU stress test started."
echo "PIDs: $(jobs -p)"
echo "To stop: kill $(jobs -p)"
EOF

chmod +x /tmp/cpu_stress.sh
```

Run:

```bash
/tmp/cpu_stress.sh
```

Monitor:

```bash
top -n 1 | grep "Cpu(s)"
```

Wait for the alert evaluation period:

```bash
sleep 180
```

Stop the test:

```bash
killall yes
```

---

# 🧠 Task 8 — Test Memory Alerts

Install `stress-ng`:

```bash
sudo apt install -y stress-ng
```

Create the test script:

```bash
cat > /tmp/memory_stress.sh << 'EOF'
#!/bin/bash

echo "Starting memory stress test..."

stress-ng --vm 2 --vm-bytes 75% --timeout 300s &

echo "Memory stress test started."
echo "PID: $!"
EOF

chmod +x /tmp/memory_stress.sh
```

Run:

```bash
/tmp/memory_stress.sh
```

Monitor the alert through Prometheus, Alertmanager, and Grafana.

---

# 🔍 Task 9 — Verify Alert Functionality

## Prometheus

Open:

```text
http://localhost:9090/alerts
```

Verify:

```text
FIRING
PENDING
INACTIVE
```

---

## Alertmanager

Open:

```text
http://localhost:9093
```

Verify that Prometheus alerts are arriving.

---

## Grafana

Navigate to:

```text
Alerting → Alert Rules
```

Verify the state of your configured alert rules.

---

## 📧 Notification Verification

Check:

* Email inbox
* Local mail using `mail`
* Slack `#alerts` channel
* Grafana alert history
* Alertmanager UI

---

# ✅ Task 10 — Test Alert Resolution

Stop all stress processes:

```bash
sudo killall stress-ng
sudo killall yes
```

Wait:

```bash
sleep 180
```

Verify that alerts transition:

```text
🔥 FIRING
     ↓
✅ RESOLVED
```

Check the state in:

* Prometheus
* Alertmanager
* Grafana

---

# 🌐 Task 11 — Create Network Traffic Alert

Append a network alert:

```bash
sudo tee -a /etc/prometheus/alert_rules.yml > /dev/null <<EOF

  - alert: HighNetworkTraffic
    expr: rate(node_network_receive_bytes_total[5m]) > 10000000
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "High network traffic detected"
      description: "Network receive traffic is above 10MB/s on {{ \$labels.instance }} interface {{ \$labels.device }}"
EOF
```

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

---

# 🔇 Task 12 — Configure Alert Silencing

Alertmanager supports temporary silencing.

Open:

```text
http://localhost:9093
```

Then:

```text
Active Alert
    ↓
Silence
    ↓
Set Duration
    ↓
Add Reason
    ↓
Create
```

### 💡 Why Silence Alerts?

Silencing is useful during:

* 🔧 Planned maintenance
* 🚀 Deployments
* 🧪 Testing
* 🔄 Infrastructure migration
* 🛠️ Known temporary incidents

---

# 📦 Task 13 — Configure Alert Grouping

Example:

```yaml
route:
  group_by: ['cluster', 'alertname']
  group_wait: 10s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'default'
```

### 🔎 Important Parameters

| Parameter         | Purpose                                     |
| ----------------- | ------------------------------------------- |
| `group_by`        | Combines related alerts                     |
| `group_wait`      | Waits before sending the first notification |
| `group_interval`  | Controls updates to an alert group          |
| `repeat_interval` | Controls repeated notifications             |
| `receiver`        | Defines where notifications are delivered   |

Grouping prevents an alert storm from generating hundreds of individual notifications.

---

# 🧠 Advanced Alert Concepts

## 🔀 Alert Routing

Routes alerts based on labels such as:

```yaml
severity: critical
```

or:

```yaml
severity: warning
```

This allows different alerts to reach different teams or channels.

---

## 🚫 Alert Inhibition

Inhibition prevents lower-priority alerts from creating unnecessary noise when a more serious alert is already active.

Example:

```text
CRITICAL alert
      │
      ▼
Suppress related
WARNING alerts
```

---

## 🔇 Alert Silencing

Silencing temporarily suppresses notifications without deleting the alert rule.

---

## 📦 Alert Grouping

Grouping combines related alerts into a single notification.

---

# 🛠️ Troubleshooting

## ❌ Issue 1 — Prometheus Not Starting

Validate configuration:

```bash
sudo -u prometheus /usr/local/bin/promtool \
check config /etc/prometheus/prometheus.yml
```

Check logs:

```bash
sudo journalctl -u prometheus -f
```

---

## ❌ Issue 2 — Alert Rules Not Loading

Run:

```bash
sudo -u prometheus /usr/local/bin/promtool \
check rules /etc/prometheus/alert_rules.yml
```

Query Prometheus:

```bash
curl http://localhost:9090/api/v1/query?query=up
```

---

## ❌ Issue 3 — Email Notifications Not Working

Check the mail service:

```bash
sudo systemctl status postfix
```

Test email:

```bash
echo "Test message" | mail -s "Test" $USER
```

Read:

```bash
mail
```

---

## ❌ Issue 4 — Grafana Alerts Not Triggering

Check:

1. Prometheus data source connectivity
2. Panel query results
3. Alert conditions
4. Evaluation interval
5. Alert duration
6. Contact point configuration
7. Grafana logs

Check logs:

```bash
sudo journalctl -u grafana-server -f
```

---

## ❌ Issue 5 — Alertmanager Not Receiving Alerts

Check:

```bash
sudo systemctl status alertmanager
```

Check Prometheus configuration:

```bash
sudo -u prometheus /usr/local/bin/promtool \
check config /etc/prometheus/prometheus.yml
```

Test Alertmanager endpoint:

```bash
curl http://localhost:9093/api/v1/status
```

---

# 🔎 Final Verification

## Service Health

```bash
sudo systemctl status prometheus
sudo systemctl status node_exporter
sudo systemctl status grafana-server
sudo systemctl status alertmanager
```

---

## Prometheus Targets

```bash
curl -s http://localhost:9090/api/v1/targets
```

---

## Alert Rules

```bash
curl -s http://localhost:9090/api/v1/rules
```

---

## Active Alerts

```bash
curl -s http://localhost:9090/api/v1/alerts
```

---

## Alertmanager Alerts

```bash
curl -s http://localhost:9093/api/v1/alerts
```

---

## Grafana Health

```bash
curl -s http://admin:admin@localhost:3000/api/health
```

> 🔐 Avoid using default Grafana credentials outside a temporary lab environment.

---

# 🧪 Automated Alert Test

Create:

```bash
cat > /tmp/alert_test.sh << 'EOF'
#!/bin/bash

echo "=== Alert System Test ==="

echo "1. Testing CPU alert..."

timeout 120s yes > /dev/null &
CPU_PID=$!

echo "2. Waiting for alert to trigger..."
sleep 120

echo "3. Stopping CPU stress..."
kill $CPU_PID 2>/dev/null

echo "4. Checking Prometheus alerts..."
curl -s http://localhost:9090/api/v1/alerts | \
grep -i "highcpuusage" || echo "No CPU alerts found"

echo "5. Checking Alertmanager..."

curl -s http://localhost:9093/api/v1/alerts | \
grep -c "firing" | xargs echo "Firing alerts:"

echo "=== Test Complete ==="
EOF
```

Make executable:

```bash
chmod +x /tmp/alert_test.sh
```

Run:

```bash
/tmp/alert_test.sh
```

---

# 📝 Final Verification Checklist

* [ ] Linux system updated
* [ ] Prometheus installed
* [ ] Prometheus configuration created
* [ ] Prometheus service running
* [ ] Node Exporter installed
* [ ] Node Exporter service running
* [ ] Grafana installed
* [ ] Grafana service running
* [ ] Prometheus added as Grafana data source
* [ ] Node Exporter dashboard imported
* [ ] CPU alert configured
* [ ] Memory alert configured
* [ ] Disk alert configured
* [ ] Service availability alert configured
* [ ] Alertmanager installed
* [ ] Alertmanager service running
* [ ] Email notification configured
* [ ] Slack notification configured
* [ ] Alert routing tested
* [ ] CPU stress test completed
* [ ] Memory stress test completed
* [ ] Alerts verified in Prometheus
* [ ] Alerts verified in Alertmanager
* [ ] Alerts verified in Grafana
* [ ] Alert resolution verified
* [ ] Alert silencing tested
* [ ] Alert grouping configured
* [ ] Troubleshooting commands tested

---

# 🎓 Skills Demonstrated

By completing this lab, you demonstrated practical experience with:

### 📊 Monitoring

* Prometheus
* Grafana
* Node Exporter
* PromQL
* System metrics

### 🚨 Alerting

* Prometheus alert rules
* Grafana alert rules
* Alertmanager
* Alert severity
* Alert routing
* Alert grouping
* Alert inhibition
* Alert silencing

### 📬 Notifications

* Email alerts
* Slack alerts
* Severity-based notification routing

### 🐧 Linux Administration

* Package management
* systemd services
* Linux users
* File permissions
* Service troubleshooting
* Journal logs

### 🧪 Testing

* CPU stress testing
* Memory stress testing
* Alert triggering
* Alert resolution
* API verification

---

# 🏁 Conclusion

This lab successfully builds a complete **monitoring and alerting ecosystem** using Prometheus, Grafana, Node Exporter, and Alertmanager.

You learned how to collect infrastructure metrics, visualize them through Grafana, define threshold-based alerts, route alerts according to severity, and deliver notifications through email and Slack.

More importantly, the lab demonstrates the complete alert lifecycle:

```text
📡 Metric Collection
       ↓
📊 Prometheus
       ↓
🧮 Rule Evaluation
       ↓
🚨 Alert Generated
       ↓
🔀 Alertmanager
       ↓
📦 Group / Route / Inhibit
       ↓
📧 Email + 💬 Slack
       ↓
👨‍💻 Incident Response
       ↓
✅ Alert Resolved
```

These skills are directly applicable to **Cloud Engineering, DevOps, SRE, Linux Administration, and Production Monitoring** environments.

## 🌟 Key Takeaways

> **Monitor → Detect → Alert → Notify → Respond → Resolve**

A properly designed alerting system allows engineering teams to detect infrastructure problems early, reduce incident response time, minimize alert noise, and improve overall system reliability.

---

## 🏆 Lab Outcome

**You have now built and tested a production-inspired monitoring and alerting workflow using open-source observability technologies.**

🚀 **Prometheus + Grafana + Node Exporter + Alertmanager = Powerful Infrastructure Observability**

---

### 👨‍💻 Author

**Hafiz Muhammad Salman**
**Cloud DevOps Engineer | Linux Administrator**

⭐ If this lab helped you, consider giving the repository a **Star** and sharing your learning journey.
