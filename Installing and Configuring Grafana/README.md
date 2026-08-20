# 📊 Installing and Configuring Grafana

<p align="center">

<img src="https://grafana.com/static/assets/img/grafana_icon.svg" width="100" alt="Grafana Logo">

</p>

<h1 align="center">🚀 Installing and Configuring Grafana</h1>

<p align="center">
  <b>Linux Monitoring • Prometheus • Node Exporter • Grafana Dashboards</b>
</p>

<p align="center">

![Grafana](https://img.shields.io/badge/Grafana-Monitoring-orange?style=for-the-badge\&logo=grafana)
![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-orange?style=for-the-badge\&logo=prometheus)
![Linux](https://img.shields.io/badge/Linux-Server-black?style=for-the-badge\&logo=linux)
![Ubuntu](https://img.shields.io/badge/Ubuntu-Linux-E95420?style=for-the-badge\&logo=ubuntu)
![Node Exporter](https://img.shields.io/badge/Node%20Exporter-System%20Metrics-blue?style=for-the-badge)
![Systemd](https://img.shields.io/badge/Systemd-Service%20Management-4EAA25?style=for-the-badge)

</p>

---

## 📌 About This Lab

This lab demonstrates how to build a basic **Linux monitoring and visualization stack** using:

* 🎨 **Grafana** — Visualization and dashboard platform
* 🔥 **Prometheus** — Time-series metrics collection
* 🖥️ **Node Exporter** — Linux system metrics exporter
* 🐧 **Linux** — Monitoring server environment
* ⚙️ **systemd** — Service management

The lab starts from a bare Linux machine and walks through installation, configuration, service management, Prometheus integration, metric exploration, dashboard creation, verification, and troubleshooting.

---

# 🎯 Lab Objectives

By completing this lab, you will learn how to:

* ✅ Install Grafana on a Linux server
* ✅ Configure Grafana for initial use and security
* ✅ Access and navigate the Grafana web interface
* ✅ Install Prometheus as a local metrics data source
* ✅ Install and configure Node Exporter
* ✅ Connect Prometheus with Grafana
* ✅ Query system metrics
* ✅ Build a basic Grafana dashboard
* ✅ Troubleshoot common monitoring-stack problems

---

# 🧰 Technology Stack

| Technology            | Purpose                              |
| --------------------- | ------------------------------------ |
| 🐧 **Linux / Ubuntu** | Server operating system              |
| 📊 **Grafana**        | Metrics visualization and dashboards |
| 🔥 **Prometheus**     | Time-series metrics collection       |
| 🖥️ **Node Exporter** | Linux system metrics                 |
| ⚙️ **systemd**        | Service management                   |
| 🔐 **UFW**            | Firewall management                  |
| 📦 **APT**            | Package management                   |
| 📝 **YAML**           | Prometheus configuration             |
| 📄 **INI**            | Grafana configuration                |
| 🔎 **PromQL**         | Prometheus metric queries            |
| 🌐 **HTTP**           | Web access and API communication     |

---

# 🏗️ Monitoring Architecture

```text
                 ┌─────────────────────────┐
                 │      Linux Server       │
                 │                         │
                 │  ┌───────────────────┐  │
                 │  │   Node Exporter   │  │
                 │  │     :9100         │  │
                 │  └─────────┬─────────┘  │
                 │            │ metrics    │
                 │            ▼            │
                 │  ┌───────────────────┐  │
                 │  │    Prometheus     │  │
                 │  │      :9090        │  │
                 │  └─────────┬─────────┘  │
                 │            │             │
                 │            │ PromQL      │
                 │            ▼             │
                 │  ┌───────────────────┐  │
                 │  │      Grafana      │  │
                 │  │      :3000        │  │
                 │  └─────────┬─────────┘  │
                 └────────────┼────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │ Monitoring       │
                    │ Dashboards       │
                    └──────────────────┘
```

---

# 📋 Prerequisites

Before starting, you should understand:

* 🐧 Basic Linux command-line operations
* 🔐 Linux file permissions
* ⚙️ Linux system services
* 🌐 Basic networking concepts
* 📝 YAML configuration
* 📈 Basic monitoring and metrics concepts

---

# ☁️ Lab Environment

The lab uses an **Al Nafi Linux-based cloud machine**.

The environment provides:

* 🖥️ One Linux machine
* 🔑 Root/sudo access
* 🌐 Internet connectivity
* 📦 Ability to download required packages
* 🚫 No additional virtual machines required

---

# 🚀 Task 1 — Install Grafana

## 🔹 Step 1: Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

Install the required packages:

```bash
sudo apt install -y software-properties-common apt-transport-https wget curl gnupg2
```

---

## 🔹 Step 2: Add Grafana Repository

Add the Grafana GPG key:

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Add the Grafana repository:

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | \
sudo tee /etc/apt/sources.list.d/grafana.list
```

Update package repositories:

```bash
sudo apt update
```

---

## 🔹 Step 3: Install Grafana

```bash
sudo apt install -y grafana
```

Verify the installation:

```bash
grafana-server --version
```

Check the service:

```bash
systemctl status grafana-server
```

---

## 🔹 Step 4: Enable and Start Grafana

Enable Grafana at boot:

```bash
sudo systemctl enable grafana-server
```

Start Grafana:

```bash
sudo systemctl start grafana-server
```

Verify:

```bash
sudo systemctl status grafana-server
```

Check port `3000`:

```bash
sudo netstat -tlnp | grep 3000
```

---

# 🌐 Task 2 — Configure Grafana

## 🔹 Step 1: Find Server IP

```bash
ip addr show | grep inet
```

Grafana runs on port `3000` by default.

```text
http://YOUR_IP_ADDRESS:3000
```

---

## 🔐 Step 2: Initial Login

The lab initially uses:

```text
Username: admin
Password: admin
```

Immediately change the default password to a strong password.

> ⚠️ **Security Note:** Never keep default credentials in a production environment.

---

## 🔹 Step 3: Back Up Grafana Configuration

```bash
sudo cp /etc/grafana/grafana.ini \
/etc/grafana/grafana.ini.backup
```

Open the configuration:

```bash
sudo nano /etc/grafana/grafana.ini
```

Example settings:

```ini
[server]
http_port = 3000
domain = localhost
root_url = http://localhost:3000

[security]
admin_user = admin
admin_password = your_secure_password

[users]
allow_sign_up = false
allow_org_create = false

[auth.anonymous]
enabled = false
```

---

## 🔄 Step 4: Restart Grafana

```bash
sudo systemctl restart grafana-server
```

Verify:

```bash
sudo systemctl status grafana-server
```

View live logs:

```bash
sudo journalctl -u grafana-server -f
```

---

# 🔥 Task 3 — Install Prometheus

Prometheus will collect and store the metrics that Grafana will visualize.

## 🔹 Step 1: Create Prometheus User

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

Create directories:

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```

Set ownership:

```bash
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

---

## 🔹 Step 2: Download Prometheus

```bash
cd /tmp

wget https://github.com/prometheus/prometheus/releases/download/v2.40.0/prometheus-2.40.0.linux-amd64.tar.gz
```

Extract:

```bash
tar xvf prometheus-2.40.0.linux-amd64.tar.gz
```

Enter the directory:

```bash
cd prometheus-2.40.0.linux-amd64
```

Install the binaries:

```bash
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
```

---

# 📝 Configure Prometheus

Create the configuration file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Use the basic scrape configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

Set ownership:

```bash
sudo chown prometheus:prometheus \
/etc/prometheus/prometheus.yml
```

---

# ⚙️ Create Prometheus systemd Service

Create:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Configuration:

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
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable and start Prometheus:

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

Verify:

```bash
sudo systemctl status prometheus
```

Test metrics:

```bash
curl http://localhost:9090/metrics
```

---

# 🖥️ Task 4 — Install Node Exporter

Node Exporter exposes Linux system metrics for Prometheus.

Create the user:

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

Download:

```bash
cd /tmp

wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
```

Extract:

```bash
tar xvf node_exporter-1.5.0.linux-amd64.tar.gz
```

Install:

```bash
sudo cp node_exporter-1.5.0.linux-amd64/node_exporter \
/usr/local/bin/
```

Set ownership:

```bash
sudo chown node_exporter:node_exporter \
/usr/local/bin/node_exporter
```

---

# ⚙️ Create Node Exporter Service

Create:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Add:

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

Start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

Verify:

```bash
sudo systemctl status node_exporter
```

Test:

```bash
curl http://localhost:9100/metrics
```

---

# 🔗 Task 5 — Connect Prometheus to Grafana

Open Grafana:

```text
http://localhost:3000
```

Navigate to:

```text
Configuration
    ↓
Data Sources
    ↓
Add data source
    ↓
Prometheus
```

Configure:

```text
Name:   Prometheus
URL:    http://localhost:9090
Access: Server
```

Click:

```text
Save & Test
```

---

# 🔎 Task 6 — Verify Prometheus Metrics

Check Prometheus targets:

```bash
curl http://localhost:9090/api/v1/targets
```

List available metrics:

```bash
curl http://localhost:9090/api/v1/label/__name__/values
```

In Grafana, open:

```text
Explore → Prometheus
```

Try these queries:

```promql
up
```

```promql
node_cpu_seconds_total
```

```promql
node_memory_MemAvailable_bytes
```

---

# 📊 Task 7 — Create Your First Grafana Dashboard

Create a new dashboard:

```text
+ Create
    ↓
Dashboard
    ↓
Add new panel
```

Use this PromQL query:

```promql
rate(node_cpu_seconds_total[5m])
```

Panel configuration:

```text
Panel Title: CPU Usage Rate
Visualization: Time series
```

Click:

```text
Apply
```

Save the dashboard as:

```text
System Monitoring
```

---

# 🛠️ Troubleshooting

## ❌ Grafana Service Won't Start

Check logs:

```bash
sudo journalctl -u grafana-server -n 50
```

Test configuration:

```bash
sudo grafana-server \
-config /etc/grafana/grafana.ini -test
```

Check permissions:

```bash
ls -la /etc/grafana/
```

Fix ownership if required:

```bash
sudo chown -R grafana:grafana /etc/grafana/
```

---

## ❌ Cannot Access Grafana Web Interface

Check port:

```bash
sudo netstat -tlnp | grep 3000
```

Check firewall:

```bash
sudo ufw status
```

Allow port:

```bash
sudo ufw allow 3000
```

Check Grafana configuration:

```bash
grep -n "http_port" /etc/grafana/grafana.ini
```

---

## ❌ Prometheus Connection Failed

Test Prometheus:

```bash
curl -I http://localhost:9090
```

Check logs:

```bash
sudo journalctl -u prometheus -n 20
```

Validate configuration:

```bash
sudo promtool check config \
/etc/prometheus/prometheus.yml
```

---

## ❌ No Metrics Available

Check Node Exporter:

```bash
curl http://localhost:9100/metrics | head -20
```

Check Prometheus targets:

```bash
curl http://localhost:9090/api/v1/targets | jq '.'
```

Review scrape configuration:

```bash
cat /etc/prometheus/prometheus.yml
```

---

# ✅ Verification Checklist

## Grafana

```bash
grafana-server --version

sudo systemctl is-active grafana-server

sudo systemctl is-enabled grafana-server

sudo ss -tlnp | grep 3000
```

## Prometheus

```bash
sudo systemctl is-active prometheus

curl -s http://localhost:9090/-/healthy
```

## Node Exporter

```bash
sudo systemctl is-active node_exporter

curl -s http://localhost:9100/metrics | wc -l
```

---

# 🔍 Final Integration Verification

Verify the complete monitoring pipeline:

```text
Node Exporter
      │
      │ System Metrics
      ▼
 Prometheus
      │
      │ PromQL
      ▼
   Grafana
      │
      ▼
 Dashboards
```

In Grafana:

1. Open **Configuration → Data Sources**
2. Verify **Prometheus** is connected
3. Open **Explore**
4. Execute PromQL queries
5. Confirm metrics are returned
6. Open **System Monitoring**
7. Confirm the dashboard displays system metrics

---

# 🎓 Skills Developed

After completing this lab, you will have practiced:

### 🐧 Linux Administration

* Package installation
* User management
* File permissions
* systemd service management
* Log analysis

### 📊 Monitoring

* Metrics collection
* Prometheus configuration
* Node Exporter
* Prometheus targets
* Metric querying

### 🎨 Visualization

* Grafana configuration
* Data source integration
* Dashboard creation
* Time-series visualization

### 🔐 Security

* Changing default credentials
* Disabling anonymous access
* Disabling user registration
* Basic firewall configuration

### 🔧 Troubleshooting

* Service failures
* Port connectivity
* Configuration validation
* Prometheus target problems
* Missing metrics

---

# 🏆 Lab Outcome

By the end of the exercise, you will have created a working local monitoring stack:

```text
┌─────────────────────────────────────────────┐
│             Linux Monitoring Stack          │
├─────────────────────────────────────────────┤
│                                             │
│  🖥️ Node Exporter                           │
│       │                                     │
│       ▼                                     │
│  🔥 Prometheus                              │
│       │                                     │
│       ▼                                     │
│  📊 Grafana                                 │
│       │                                     │
│       ▼                                     │
│  📈 System Monitoring Dashboard             │
│                                             │
└─────────────────────────────────────────────┘
```

The completed lab establishes a foundation for infrastructure monitoring, application monitoring, business dashboards, and alerting systems.

---

# 🚀 Next Steps

After completing this lab, continue with advanced Grafana capabilities:

* 🚨 Grafana Alerting
* 📊 Advanced dashboard design
* 👥 User and organization management
* 🔗 Additional data sources
* 📈 Advanced PromQL
* 🔥 Prometheus Alertmanager
* ☁️ Cloud monitoring integrations
* 🏗️ Production monitoring architecture

The source lab specifically identifies advanced alerting, advanced visualizations, user management, and additional data sources such as InfluxDB, Elasticsearch, and cloud monitoring services as possible next steps.

---

# 📚 Technology Badges

<p align="center">

![Linux](https://img.shields.io/badge/Linux-Administration-black?style=for-the-badge\&logo=linux)
![Ubuntu](https://img.shields.io/badge/Ubuntu-Server-E95420?style=for-the-badge\&logo=ubuntu)
![Grafana](https://img.shields.io/badge/Grafana-Dashboard-F46800?style=for-the-badge\&logo=grafana)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?style=for-the-badge\&logo=prometheus)
![YAML](https://img.shields.io/badge/YAML-Configuration-CB171E?style=for-the-badge\&logo=yaml)
![Bash](https://img.shields.io/badge/Bash-Scripting-4EAA25?style=for-the-badge\&logo=gnubash)
![systemd](https://img.shields.io/badge/systemd-Service%20Management-blue?style=for-the-badge)

</p>

---

# 👨‍💻 Author

**Hafiz Muhammad Salman**

💼 Cloud DevOps Engineer | Linux Administrator

<p align="center">

⭐ If this lab helped you, consider giving the repository a star!

</p>

---

# 📜 Learning Disclaimer

This repository is intended for **educational and laboratory purposes**. Commands and configurations should be reviewed and hardened appropriately before being used in a production environment.

---

<p align="center">

### 🚀 Learn • Build • Monitor • Automate • Improve

**Happy Monitoring! 📊🔥**

</p>
