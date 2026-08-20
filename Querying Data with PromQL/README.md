# 📊 Querying Data with PromQL

![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?logo=prometheus\&logoColor=white)
![PromQL](https://img.shields.io/badge/PromQL-Query%20Language-E6522C?logo=prometheus\&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-Visualization-F46800?logo=grafana\&logoColor=white)
![Node Exporter](https://img.shields.io/badge/Node%20Exporter-System%20Metrics-1F8DD6?logo=prometheus\&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-E95420?logo=ubuntu\&logoColor=white)
![Systemd](https://img.shields.io/badge/Systemd-Service%20Management-1793D1?logo=linux\&logoColor=white)

> 🚀 **Hands-on DevOps & Observability Lab**
> Learn how to collect infrastructure metrics with **Prometheus + Node Exporter**, analyze them using **PromQL**, and build monitoring dashboards with **Grafana**.

---

## 📌 Overview

This lab provides a practical introduction to **Prometheus Query Language (PromQL)** and its use with Grafana for system monitoring.

You will build a complete monitoring environment consisting of:

```text
┌──────────────────────┐
│      Linux Host      │
│                      │
│   Node Exporter      │
│      :9100           │
└──────────┬───────────┘
           │ Metrics
           ▼
┌──────────────────────┐
│     Prometheus       │
│        :9090         │
│                      │
│   PromQL Queries     │
└──────────┬───────────┘
           │ Data Source
           ▼
┌──────────────────────┐
│       Grafana        │
│        :3000         │
│                      │
│ Dashboards & Panels  │
└──────────────────────┘
```

The lab covers PromQL fundamentals, system metrics, Grafana dashboards, aggregation, label matching, time-range functions, and optional alerting rules.

---

# 🎯 Lab Objectives

By completing this lab, you will be able to:

* 🔎 Understand the fundamentals of **PromQL**
* 📈 Write PromQL queries for system metrics
* 🖥️ Use Grafana's Query Editor
* 📊 Create system monitoring dashboards
* 🧮 Apply PromQL functions and operators
* 🚦 Create basic Prometheus alert rules
* 🛠️ Troubleshoot Prometheus, Node Exporter, and Grafana

---

# 🧰 Technology Stack

| Technology           | Purpose                                     |
| -------------------- | ------------------------------------------- |
| 🔥 **Prometheus**    | Metrics collection and time-series database |
| 📡 **Node Exporter** | Linux system metrics collection             |
| 📊 **Grafana**       | Metrics visualization and dashboards        |
| 🧠 **PromQL**        | Query and analyze Prometheus metrics        |
| 🐧 **Linux**         | Lab operating system                        |
| ⚙️ **systemd**       | Service management                          |
| 🛡️ **UFW**          | Firewall verification                       |

---

# 📋 Prerequisites

Before beginning, you should understand:

* 🐧 Basic Linux command-line operations
* 📊 System monitoring concepts
* 🧠 CPU and memory concepts
* 💾 Disk usage
* 🌐 Basic networking
* ⏱️ Time-series data concepts
* 🌍 Basic web-browser usage

The lab environment uses Linux-based cloud machines provided through **Al Nafi**. The machine starts without the required monitoring tools, so the components are installed during the lab.

---

# 🏗️ Lab Architecture

```text
                    ┌──────────────────────┐
                    │      Grafana         │
                    │      Port 3000       │
                    │                      │
                    │ Dashboards / Panels  │
                    └──────────┬───────────┘
                               │
                               │ PromQL
                               ▼
                    ┌──────────────────────┐
                    │     Prometheus       │
                    │      Port 9090       │
                    │                      │
                    │ Time-Series Database  │
                    └──────────┬───────────┘
                               │
                         Scrape Metrics
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Node Exporter     │
                    │      Port 9100       │
                    │                      │
                    │ CPU / RAM / Disk /   │
                    │ Network / Load       │
                    └──────────────────────┘
```

---

# 🚀 Task 1 — Environment Setup

## 🔹 Step 1: Update Linux

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y wget curl tar systemctl
```

---

## 🔹 Step 2: Create Prometheus User

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

Create Prometheus directories:

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

---

## 🔹 Step 3: Download Prometheus

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

Create the configuration file:

```bash
sudo tee /etc/prometheus/prometheus.yml > /dev/null <<EOF
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
EOF

sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

Prometheus will collect metrics from itself and Node Exporter using the configured scrape targets.

---

# 📡 Step 4: Install Node Exporter

Node Exporter provides Linux system metrics.

```bash
cd /tmp

wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz

tar xvf node_exporter-1.6.1.linux-amd64.tar.gz

sudo cp node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/

sudo chown prometheus:prometheus /usr/local/bin/node_exporter
```

Node Exporter exposes metrics on:

```text
http://localhost:9100/metrics
```

---

# ⚙️ Step 5: Create systemd Services

## Prometheus Service

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

## Node Exporter Service

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

# ▶️ Step 6: Start Monitoring Services

```bash
sudo systemctl daemon-reload

sudo systemctl enable prometheus
sudo systemctl enable node_exporter

sudo systemctl start prometheus
sudo systemctl start node_exporter
```

Verify:

```bash
sudo systemctl status prometheus
sudo systemctl status node_exporter
```

Expected result:

```text
Active: active (running)
```

---

# 📊 Step 7: Install Grafana

Install Grafana using the configured package repository:

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" | \
sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install -y grafana
```

Enable and start Grafana:

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

sudo systemctl status grafana-server
```

---

# 🔍 Task 2 — Writing PromQL Queries

## 🌐 Access Prometheus

Open:

```text
http://localhost:9090
```

Select **Graph** to access the query interface.

---

## 🧠 Query 1 — CPU Usage

```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

This query:

1. Calculates CPU idle rate
2. Uses a 5-minute range
3. Averages by instance
4. Converts idle percentage into CPU usage

---

## 🧠 Query 2 — Memory Usage

```promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

This calculates memory utilization as a percentage.

---

## 🧠 Query 3 — Disk Usage

```promql
100 - ((node_filesystem_avail_bytes{mountpoint="/",fstype!="rootfs"} / node_filesystem_size_bytes{mountpoint="/",fstype!="rootfs"}) * 100)
```

This calculates root filesystem usage.

---

# 🌐 Network & Load Queries

## 📡 Network Receive Rate

```promql
rate(node_network_receive_bytes_total{device!="lo"}[5m])
```

## 📈 Load Average

```promql
node_load1
```

## 🔥 Top CPU Activity

```promql
topk(5, rate(node_cpu_seconds_total{mode!="idle"}[5m]))
```

## 🧠 Memory Trend

```promql
increase(node_memory_MemTotal_bytes[1h])
```

These queries introduce time-based calculations and PromQL functions for analyzing system performance.

---

# 📊 Task 3 — Grafana Dashboard

## 🌐 Access Grafana

Open:

```text
http://localhost:3000
```

Default credentials specified by the lab:

```text
Username: admin
Password: admin
```

You will be prompted to change the password after the initial login.

---

# 🔌 Add Prometheus Data Source

Navigate to:

```text
Configuration → Data Sources → Add data source → Prometheus
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

A successful connection should display a green confirmation message.

---

# 📈 Create System Monitoring Dashboard

Create:

```text
Dashboard → Add new panel
```

The dashboard will contain:

* 🖥️ CPU Usage
* 🧠 Memory Usage
* 🌐 Network Traffic
* 💾 Disk Usage
* 📊 Load Average

---

## 🖥️ CPU Usage Panel

PromQL:

```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

Recommended configuration:

```text
Title: CPU Usage
Unit: Percent (0-100)
Min: 0
Max: 100
Visualization: Stat / Gauge
```

---

## 🧠 Memory Usage Panel

```promql
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

Configuration:

```text
Title: Memory Usage
Unit: Percent (0-100)
Min: 0
Max: 100
Visualization: Stat / Gauge
```

---

## 🌐 Network Traffic Panel

### Received

```promql
rate(node_network_receive_bytes_total{device!="lo"}[5m])
```

### Transmitted

```promql
rate(node_network_transmit_bytes_total{device!="lo"}[5m])
```

Configuration:

```text
Title: Network Traffic
Unit: Bytes/sec
Visualization: Time series

Query A: Received
Query B: Transmitted
```

---

## 💾 Disk Usage Panel

```promql
100 - ((node_filesystem_avail_bytes{mountpoint="/",fstype!="rootfs"} / node_filesystem_size_bytes{mountpoint="/",fstype!="rootfs"}) * 100)
```

Configuration:

```text
Title: Disk Usage (Root)
Unit: Percent (0-100)
Min: 0
Max: 100
Visualization: Gauge
```

---

## 📊 Load Average Panel

### 1 Minute

```promql
node_load1
```

### 5 Minutes

```promql
node_load5
```

### 15 Minutes

```promql
node_load15
```

Use a **Time series** visualization with legends for 1-minute, 5-minute, and 15-minute load averages.

---

# 💾 Save Dashboard

Save the dashboard with:

```text
Name: System Monitoring
Description: Basic system metrics dashboard
```

The completed dashboard provides a basic view of system performance metrics.

---

# 🧮 Task 4 — Advanced PromQL

## ➕ Aggregation Functions

### SUM

```promql
sum(rate(node_network_receive_bytes_total[5m]))
```

### AVG

```promql
avg(node_load1)
```

### MAX

```promql
max(node_filesystem_size_bytes)
```

---

# ➗ Mathematical Operations

## Memory Percentage

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) /
node_memory_MemTotal_bytes * 100
```

## Disk I/O Rate

```promql
rate(node_disk_read_bytes_total[5m]) +
rate(node_disk_written_bytes_total[5m])
```

---

# 🏷️ Label Matching

## Exact Match

```promql
node_filesystem_size_bytes{mountpoint="/"}
```

## Regex Match

```promql
node_filesystem_size_bytes{device=~"/dev/sd.*"}
```

## Negative Match

```promql
node_network_receive_bytes_total{device!="lo"}
```

---

# ⏱️ Time Range Functions

## Range Vector

```promql
rate(node_cpu_seconds_total[5m])
```

## Offset

```promql
node_load1 offset 1h
```

## Increase

```promql
increase(node_network_receive_bytes_total[1h])
```

These exercises demonstrate aggregation, mathematical operations, label matching, range vectors, offsets, and increase calculations.

---

# 🚨 Task 5 — Optional Alerting

Create:

```bash
sudo tee /etc/prometheus/alert_rules.yml > /dev/null <<EOF
groups:
- name: system_alerts
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage detected"
      description: "CPU usage is above 80% for more than 2 minutes"

  - alert: HighMemoryUsage
    expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High memory usage detected"
      description: "Memory usage is above 85% for more than 2 minutes"

  - alert: DiskSpaceLow
    expr: 100 - ((node_filesystem_avail_bytes{mountpoint="/",fstype!="rootfs"} / node_filesystem_size_bytes{mountpoint="/",fstype!="rootfs"}) * 100) > 90
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Disk space is running low"
      description: "Disk usage is above 90%"
EOF

sudo chown prometheus:prometheus /etc/prometheus/alert_rules.yml
```

The lab defines alerts for:

| Alert              |       Threshold | Severity |
| ------------------ | --------------: | -------- |
| 🔥 HighCPUUsage    | > 80% for 2 min | Warning  |
| 🧠 HighMemoryUsage | > 85% for 2 min | Warning  |
| 💾 DiskSpaceLow    | > 90% for 1 min | Critical |

---

# ⚙️ Enable Alert Rules

Update `/etc/prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

rule_files:
  - "alert_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
```

Restart Prometheus:

```bash
sudo systemctl restart prometheus
```

---

# 🛠️ Troubleshooting

## ❌ Services Not Starting

Check:

```bash
sudo systemctl status prometheus
sudo systemctl status node_exporter
sudo systemctl status grafana-server
```

Check logs:

```bash
sudo journalctl -u prometheus -f
sudo journalctl -u node_exporter -f
sudo journalctl -u grafana-server -f
```

---

## ❌ Cannot Access Web Interfaces

Check listening ports:

```bash
sudo netstat -tlnp | grep :9090
sudo netstat -tlnp | grep :3000
sudo netstat -tlnp | grep :9100
```

Check firewall:

```bash
sudo ufw status
```

---

## ❌ No Data in Grafana

Check:

* Prometheus data source configuration
* Node Exporter status
* PromQL syntax
* Prometheus target status

---

## ❌ PromQL Query Errors

Test queries directly in Prometheus:

```text
http://localhost:9090
```

Then:

* Check metric names
* Check labels
* Check label values
* Validate query syntax

The source lab specifically recommends testing PromQL in Prometheus before using it in Grafana.

---

# ✅ Verification Checklist

## Prometheus

```text
http://localhost:9090
```

Verify:

```text
Status → Targets
```

All configured targets should be:

```text
UP
```

Test:

```promql
up
```

---

## Node Exporter

Open:

```text
http://localhost:9100/metrics
```

Then test:

```promql
node_load1
```

---

## Grafana

Open:

```text
http://localhost:3000
```

Verify:

* Prometheus data source is connected
* Dashboard loads successfully
* `up` returns data
* CPU metrics appear
* Memory metrics appear
* Network metrics appear
* Disk metrics appear

These verification steps are part of the original lab workflow.

---

# 🎓 Learning Outcomes

After completing this lab, you have:

* ✅ Installed Prometheus
* ✅ Installed Node Exporter
* ✅ Installed Grafana
* ✅ Configured Prometheus scraping
* ✅ Created systemd services
* ✅ Learned PromQL fundamentals
* ✅ Queried CPU metrics
* ✅ Queried memory metrics
* ✅ Queried disk metrics
* ✅ Analyzed network traffic
* ✅ Analyzed system load
* ✅ Built Grafana dashboards
* ✅ Used aggregation functions
* ✅ Used mathematical operations
* ✅ Used label matching
* ✅ Used time-range functions
* ✅ Created optional alerting rules
* ✅ Practiced monitoring troubleshooting

---

# 🌟 Why PromQL Matters

PromQL is an important skill for **DevOps, Cloud Engineering, SRE, and Linux Administration** because it allows infrastructure teams to transform raw time-series metrics into meaningful operational information.

With Prometheus and Grafana, you can:

```text
Collect Metrics
      ↓
Store Time-Series Data
      ↓
Query with PromQL
      ↓
Visualize with Grafana
      ↓
Detect Problems
      ↓
Create Alerts
      ↓
Improve Infrastructure Reliability
```

The lab emphasizes that PromQL and Grafana skills support proactive system monitoring, visualization, alerting, trend analysis, capacity planning, and troubleshooting.

---

# 📁 Suggested Repository Structure

```text
querying-data-with-promql/
│
├── README.md
│
├── prometheus/
│   ├── prometheus.yml
│   └── alert_rules.yml
│
├── grafana/
│   └── dashboards/
│
└── screenshots/
    ├── prometheus-targets.png
    ├── prometheus-query.png
    └── grafana-dashboard.png
```

---

# 🏁 Conclusion

This lab builds a practical foundation in **Prometheus monitoring, PromQL query development, Node Exporter metrics collection, and Grafana visualization**.

The complete workflow is:

```text
🐧 Linux
  │
  ▼
📡 Node Exporter
  │
  ▼
🔥 Prometheus
  │
  ▼
🧠 PromQL
  │
  ▼
📊 Grafana
  │
  ▼
🚨 Alerts & Observability
```

> 💡 **Keep practicing different PromQL metrics, labels, functions, and query patterns to strengthen your time-series analysis and infrastructure monitoring skills.**

---

## 👨‍💻 Lab Focus

**Domain:** DevOps / Observability / Monitoring
**Primary Skills:** PromQL, Prometheus, Grafana, Node Exporter
**Environment:** Linux / Al Nafi Cloud Lab
**Level:** Beginner → Intermediate

---

⭐ **If this lab helped you understand PromQL and monitoring, consider documenting your results and adding your Grafana dashboard screenshots to your repository.**
