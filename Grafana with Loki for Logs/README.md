# 📊 Grafana with Loki for Logs

> 🚀 **Hands-On DevOps & Observability Lab**
> Learn how to build a complete centralized logging stack using **Grafana, Loki, and Promtail** on a Linux system.

---

## 🎯 Lab Objectives

By the end of this lab, you will be able to:

* 🔹 Install and configure **Grafana Loki**
* 🔹 Install and configure **Promtail** for log collection
* 🔹 Set up **Grafana** for log visualization
* 🔹 Connect Loki to Grafana as a data source
* 🔹 Collect logs from Linux system services
* 🔹 Create dashboards for centralized log monitoring
* 🔹 Write and execute **LogQL** queries
* 🔹 Filter and analyze application and system logs
* 🔹 Configure log-based alerts
* 🔹 Understand the architecture of modern log-management systems
* 🔹 Troubleshoot common Loki, Promtail, and Grafana issues

---

## 🏗️ Logging Architecture

```text
                    ┌─────────────────────┐
                    │    Linux System     │
                    │                     │
                    │ /var/log/*.log      │
                    │ syslog              │
                    │ auth.log             │
                    │ nginx logs           │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      Promtail       │
                    │    Log Collector    │
                    │                     │
                    │ Parse / Label / Ship│
                    └──────────┬──────────┘
                               │
                               │ HTTP :3100
                               ▼
                    ┌─────────────────────┐
                    │        Loki         │
                    │  Log Aggregation    │
                    │                     │
                    │   LogQL / Storage   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      Grafana        │
                    │ Visualization/UI    │
                    │                     │
                    │ Dashboards / Alerts │
                    └─────────────────────┘
```

### 🔗 Technology Flow

**Linux Logs → Promtail → Loki → Grafana → Dashboards & Alerts**

---

# 🧰 Technology Stack

| Technology  | Purpose                          |
| ----------- | -------------------------------- |
| 🐧 Linux    | Lab operating system             |
| 📦 Loki     | Log aggregation and storage      |
| 🚚 Promtail | Log collection and shipping      |
| 📊 Grafana  | Log visualization and dashboards |
| 🔎 LogQL    | Loki query language              |
| ⚙️ systemd  | Service management               |
| 🌐 HTTP     | Communication between components |

---

# 📋 Prerequisites

Before starting this lab, you should have:

* 🐧 Basic Linux command-line knowledge
* 📜 Familiarity with Linux system logs
* 📝 Basic understanding of YAML
* 🌐 Basic networking knowledge
* 🖥️ Familiarity with web-based interfaces
* 🔧 Basic knowledge of systemd services

---

# ☁️ Lab Environment

The lab is designed for an **Al Nafi Linux cloud machine**.

The provided machine is bare metal with no required monitoring tools pre-installed.

You will install and configure:

```text
Loki
Promtail
Grafana
```

### 🔌 Default Ports

| Component |   Port | Purpose                 |
| --------- | -----: | ----------------------- |
| Grafana   | `3000` | Web interface           |
| Loki      | `3100` | Log ingestion/query API |
| Promtail  | `9080` | Promtail HTTP server    |

---

# 🚀 Task 1: Install Loki and Connect It to Grafana

## 1.1 🔄 Prepare the System

Update the operating system and install required dependencies.

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y wget curl unzip software-properties-common apt-transport-https
```

### ✅ Verify

```bash
curl --version
wget --version
unzip -v
```

---

# 1.2 📦 Install Loki

Create a directory for Loki:

```bash
sudo mkdir -p /opt/loki
cd /opt/loki
```

Download Loki:

```bash
wget https://github.com/grafana/loki/releases/download/v2.9.2/loki-linux-amd64.zip
```

Extract the archive:

```bash
unzip loki-linux-amd64.zip
```

Make the binary executable:

```bash
sudo chmod +x loki-linux-amd64
```

Move Loki into the system binary path:

```bash
sudo mv loki-linux-amd64 /usr/local/bin/loki
```

Verify:

```bash
loki --version
```

### ✅ Expected Result

```text
loki, version 2.9.2
```

---

# 1.3 ⚙️ Configure Loki

Create the configuration directory:

```bash
sudo mkdir -p /etc/loki
```

Create the Loki configuration:

```bash
sudo tee /etc/loki/loki.yml > /dev/null <<EOF
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory

query_range:
  results_cache:
    cache:
      embedded_cache:
        enabled: true
        max_size_mb: 100

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093
EOF
```

### 🔍 Review Configuration

```bash
sudo cat /etc/loki/loki.yml
```

---

# 1.4 🛠️ Create Loki systemd Service

Create the service:

```bash
sudo tee /etc/systemd/system/loki.service > /dev/null <<EOF
[Unit]
Description=Loki service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/loki -config.file /etc/loki/loki.yml
Restart=on-failure
RestartSec=20
StandardOutput=journal
StandardError=journal
SyslogIdentifier=loki

[Install]
WantedBy=multi-user.target
EOF
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable Loki:

```bash
sudo systemctl enable loki
```

Start Loki:

```bash
sudo systemctl start loki
```

Check status:

```bash
sudo systemctl status loki
```

### ✅ Verify Loki Port

```bash
sudo ss -lntp | grep 3100
```

---

# 1.5 🚚 Install Promtail

Promtail collects logs and sends them to Loki.

Move to `/opt`:

```bash
cd /opt
```

Download Promtail:

```bash
sudo wget https://github.com/grafana/loki/releases/download/v2.9.2/promtail-linux-amd64.zip
```

Extract:

```bash
sudo unzip promtail-linux-amd64.zip
```

Make executable:

```bash
sudo chmod +x promtail-linux-amd64
```

Move the binary:

```bash
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
```

Verify:

```bash
promtail --version
```

---

# 1.6 ⚙️ Configure Promtail

Create the configuration directory:

```bash
sudo mkdir -p /etc/promtail
```

Create the configuration:

```bash
sudo tee /etc/promtail/promtail.yml > /dev/null <<EOF
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*log

  - job_name: syslog
    static_configs:
      - targets:
          - localhost
        labels:
          job: syslog
          __path__: /var/log/syslog

  - job_name: auth
    static_configs:
      - targets:
          - localhost
        labels:
          job: auth
          __path__: /var/log/auth.log

  - job_name: nginx
    static_configs:
      - targets:
          - localhost
        labels:
          job: nginx
          __path__: /var/log/nginx/*.log
EOF
```

Review:

```bash
sudo cat /etc/promtail/promtail.yml
```

---

# 1.7 🔧 Create Promtail Service

```bash
sudo tee /etc/systemd/system/promtail.service > /dev/null <<EOF
[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/promtail -config.file /etc/promtail/promtail.yml
Restart=on-failure
RestartSec=20
StandardOutput=journal
StandardError=journal
SyslogIdentifier=promtail

[Install]
WantedBy=multi-user.target
EOF
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable Promtail:

```bash
sudo systemctl enable promtail
```

Start Promtail:

```bash
sudo systemctl start promtail
```

Check:

```bash
sudo systemctl status promtail
```

---

# 1.8 📊 Install Grafana

Add the Grafana signing key:

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Add the repository:

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Update packages:

```bash
sudo apt update
```

Install Grafana:

```bash
sudo apt install -y grafana
```

Enable Grafana:

```bash
sudo systemctl enable grafana-server
```

Start Grafana:

```bash
sudo systemctl start grafana-server
```

Check:

```bash
sudo systemctl status grafana-server
```

---

# 1.9 🔍 Verify All Services

Check Loki:

```bash
sudo systemctl status loki --no-pager -l
```

Check Promtail:

```bash
sudo systemctl status promtail --no-pager -l
```

Check Grafana:

```bash
sudo systemctl status grafana-server --no-pager -l
```

Check ports:

```bash
sudo ss -lntp | grep -E "(3100|9080|3000)"
```

### ✅ Expected Ports

```text
3000  → Grafana
3100  → Loki
9080  → Promtail
```

---

# 1.10 🌐 Access Grafana

Find the machine IP:

```bash
ip addr show | grep "inet " | grep -v 127.0.0.1
```

Grafana:

```text
http://localhost:3000
```

Default credentials:

```text
Username: admin
Password: admin
```

On first login, Grafana will ask you to change the password.

> 🔐 **Security Note:** Never keep default credentials on a production Grafana installation.

---

# 🎨 Task 2: Create Dashboards to Visualize Log Data

## 2.1 🔗 Add Loki as a Grafana Data Source

In Grafana:

1. Open **Connections / Data Sources**
2. Select **Add data source**
3. Choose **Loki**
4. Set the name:

```text
Loki
```

5. Set the URL:

```text
http://localhost:3100
```

6. Click **Save & Test**

### ✅ Expected Result

You should receive a successful connection message.

---

# 2.2 🧪 Generate Sample Logs

Create a sample log generator:

```bash
sudo tee /opt/generate_logs.sh > /dev/null <<'EOF'
#!/bin/bash

generate_logs() {
    local services=("nginx" "apache2" "mysql" "redis" "mongodb")
    local levels=("INFO" "WARN" "ERROR" "DEBUG")
    local messages=(
        "User authentication successful"
        "Database connection established"
        "Cache miss for key"
        "Request processed successfully"
        "Configuration reloaded"
        "Service started"
        "Connection timeout"
        "Invalid request format"
        "Memory usage high"
        "Disk space low"
    )

    while true; do
        service=${services[$RANDOM % ${#services[@]}]}
        level=${levels[$RANDOM % ${#levels[@]}]}
        message=${messages[$RANDOM % ${#messages[@]}]}
        timestamp=$(date '+%Y-%m-%d %H:%M:%S')

        echo "[$timestamp] [$level] [$service] $message" >> /var/log/sample_app.log

        sleep $(($RANDOM % 5 + 1))
    done
}

generate_logs
EOF
```

Make it executable:

```bash
sudo chmod +x /opt/generate_logs.sh
```

Run it:

```bash
sudo nohup /opt/generate_logs.sh > /dev/null 2>&1 &
```

Verify:

```bash
sudo tail -f /var/log/sample_app.log
```

---

# 2.3 ➕ Add Sample Logs to Promtail

Append the following scrape configuration:

```bash
sudo tee -a /etc/promtail/promtail.yml > /dev/null <<EOF

  - job_name: sample_app
    static_configs:
      - targets:
          - localhost
        labels:
          job: sample_app
          __path__: /var/log/sample_app.log
EOF
```

Restart Promtail:

```bash
sudo systemctl restart promtail
```

Verify:

```bash
sudo systemctl status promtail
```

---

# 2.4 📜 Create Sample Application Logs Panel

Create a new Grafana dashboard.

Select:

**Dashboard → Add new panel**

Choose Loki as the data source.

Use:

```logql
{job="sample_app"}
```

Set:

```text
Title: Sample Application Logs
Visualization: Logs
```

Click **Apply**.

---

# 2.5 🖥️ Create System Logs Panel

Create another panel.

Query:

```logql
{job="syslog"}
```

Configure:

```text
Title: System Logs
Visualization: Logs
```

Click **Apply**.

---

# 2.6 📈 Create Log-Level Analysis

Use:

```logql
sum by (level) (
  count_over_time(
    {job="sample_app"} |~ "\\[(INFO|WARN|ERROR|DEBUG)\\]" [5m]
  )
)
```

Configure:

```text
Title: Log Levels Distribution
Visualization: Pie Chart
```

---

# 2.7 🔐 Authentication Logs

Query:

```logql
{job="auth"}
```

Configure:

```text
Title: Authentication Logs
Visualization: Logs
```

This panel can help identify authentication activity and potential login problems.

---

# 2.8 💾 Save the Dashboard

Save the dashboard with:

```text
Name:
Log Management Dashboard
```

Description:

```text
Comprehensive log monitoring and analysis
```

---

# 🔎 2.9 Advanced LogQL Queries

## ❌ Error Logs

```logql
{job="sample_app"} |~ "ERROR"
```

---

## ⏱️ Logs from the Last Hour

```logql
{job="sample_app"}[1h]
```

---

## 📊 Count Logs by Service

```logql
sum by (service) (
  count_over_time(
    {job="sample_app"} |
    regexp "\\[(?P<service>\\w+)\\]" [5m]
  )
)
```

---

## 🔍 Search for nginx

```logql
{job="sample_app"} |~ "nginx"
```

---

## 📈 Log Rate

```logql
rate({job="sample_app"}[5m])
```

---

# 🚨 2.10 Configure Log Alerts

Navigate to:

**Alerting → Alert Rules → New Rule**

Use:

```logql
count_over_time({job="sample_app"} |~ "ERROR" [5m])
```

Configure the threshold:

```text
Condition: IS ABOVE 5
Evaluation: Every 1m
For: 2m
```

Rule name:

```text
High Error Rate
```

Message:

```text
Too many errors detected in application logs
```

Save the alert rule.

---

# 🧪 Verification and Testing

## 1️⃣ Verify Loki API

```bash
curl -G -s \
"http://localhost:3100/loki/api/v1/query" \
--data-urlencode 'query={job="sample_app"}' | jq
```

---

## 2️⃣ Check Loki Metrics

```bash
curl -s http://localhost:3100/metrics | grep loki_ingester_streams
```

---

## 3️⃣ Watch Sample Logs

```bash
tail -f /var/log/sample_app.log
```

---

# 🔍 Test Queries in Grafana Explore

### All Logs

```logql
{job=~".+"}
```

### Error Logs

```logql
{job="sample_app"} |~ "ERROR"
```

### Specific Service

```logql
{job="sample_app"} |~ "nginx"
```

### Log Rate

```logql
rate({job="sample_app"}[5m])
```

---

# 🛠️ Troubleshooting

## ❌ Loki Is Not Starting

Check logs:

```bash
sudo journalctl -u loki -f
```

Check configuration:

```bash
loki -config.file /etc/loki/loki.yml -verify-config
```

Check service:

```bash
sudo systemctl status loki
```

---

## ❌ Promtail Is Not Collecting Logs

Check Promtail logs:

```bash
sudo journalctl -u promtail -f
```

Check log permissions:

```bash
ls -la /var/log/
```

Check configuration:

```bash
promtail -config.file /etc/promtail/promtail.yml -dry-run
```

Restart:

```bash
sudo systemctl restart promtail
```

---

## ❌ Grafana Connection Problems

Check Grafana logs:

```bash
sudo journalctl -u grafana-server -f
```

Check port:

```bash
sudo ss -lntp | grep 3000
```

Reset admin password if required:

```bash
sudo grafana-cli admin reset-admin-password newpassword
```

---

# 💾 Advanced Configuration

## Log Retention

For the lab configuration, a seven-day retention policy can be configured:

```yaml
limits_config:
  retention_period: 168h
```

> ⚠️ Retention settings depend on the Loki storage/configuration model and version. Validate the configuration against the Loki version being deployed before using retention settings in production.

Restart Loki after configuration changes:

```bash
sudo systemctl restart loki
```

---

# 🧩 Log Parsing with Pipeline Stages

Promtail can parse structured log entries.

Example:

```yaml
pipeline_stages:
  - regex:
      expression: '\[(?P<timestamp>.*?)\] \[(?P<level>.*?)\] \[(?P<service>.*?)\] (?P<message>.*)'
  - labels:
      level:
      service:
```

This extracts:

```text
timestamp
level
service
message
```

These extracted fields can then be used for improved filtering and analysis.

---

# ⚡ Performance Optimization

## Monitor System Resources

```bash
htop
```

---

## Monitor Loki

```bash
ps aux | grep loki
```

---

## Check Loki Storage

```bash
df -h /tmp/loki/
```

---

## Query Optimization

Prefer specific selectors:

```logql
{job="sample_app"}
```

instead of:

```logql
{job=~".+"}
```

### Best Practices

* 🎯 Use specific label selectors
* ⏱️ Limit query time ranges
* 🔎 Filter streams before applying pipeline operations
* 🏷️ Use meaningful labels
* 📦 Avoid unnecessarily high-cardinality labels
* 💾 Monitor Loki storage usage
* 📊 Monitor CPU and memory utilization

---

# 🧠 Key Concepts Learned

### Loki

Loki is responsible for centralized log aggregation and storage.

### Promtail

Promtail reads log files, attaches labels, and forwards logs to Loki.

### Grafana

Grafana provides dashboards, visualization, exploration, and alerting.

### LogQL

LogQL is Loki's query language for searching, filtering, parsing, and analyzing logs.

### Labels

Labels identify log streams and allow efficient filtering.

---

# 🧪 Lab Validation Checklist

* [ ] Loki installed successfully
* [ ] Loki configuration created
* [ ] Loki systemd service running
* [ ] Loki listening on port `3100`
* [ ] Promtail installed
* [ ] Promtail configuration created
* [ ] Promtail service running
* [ ] Grafana installed
* [ ] Grafana listening on port `3000`
* [ ] Loki added as Grafana data source
* [ ] Sample logs generated
* [ ] Sample logs visible in Loki
* [ ] Grafana log dashboard created
* [ ] System logs displayed
* [ ] Authentication logs displayed
* [ ] LogQL queries tested
* [ ] Log-level analysis configured
* [ ] Alert rule created
* [ ] Troubleshooting commands tested

---

# 🏆 Final Outcome

After completing this lab, you will have built a complete Linux log-monitoring pipeline:

```text
┌───────────────────┐
│    Linux Logs     │
│                   │
│ syslog            │
│ auth.log          │
│ application logs  │
│ nginx logs        │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│     Promtail      │
│                   │
│ Collect           │
│ Parse             │
│ Label             │
│ Ship              │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│       Loki        │
│                   │
│ Aggregate         │
│ Store             │
│ Query             │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│      Grafana      │
│                   │
│ Explore           │
│ Dashboards        │
│ Visualization     │
│ Alerting          │
└───────────────────┘
```

---

# 🎓 Skills Demonstrated

This lab demonstrates practical skills in:

* 🐧 Linux Administration
* 📊 Grafana Monitoring
* 📝 Centralized Logging
* 🔎 LogQL
* 🚚 Promtail
* 🗄️ Loki
* ⚙️ systemd
* 📈 Dashboard Development
* 🚨 Monitoring & Alerting
* 🔧 Troubleshooting
* ☁️ Cloud-Based Linux Lab Administration
* 🛡️ Observability Engineering

---

# 🌟 Conclusion

You have successfully implemented a **Grafana + Loki centralized logging stack**.

The completed environment provides:

* ✅ Centralized log management
* ✅ Real-time log monitoring
* ✅ Application and system log collection
* ✅ Powerful LogQL queries
* ✅ Grafana dashboards
* ✅ Log-based alerting
* ✅ Structured log parsing
* ✅ Efficient log aggregation
* ✅ A foundation for production observability

The same architecture can be extended to collect logs from **Docker containers, Kubernetes workloads, NGINX, microservices, cloud infrastructure, and distributed applications**.

> 🚀 **Next Step:** Extend this lab by integrating Loki with Kubernetes, collecting container logs, creating service-specific dashboards, and implementing production-grade alerting and retention policies.

---

## 👨‍💻 Author

**Hafiz Muhammad Salman**
**Cloud DevOps Engineer | Linux Administrator**

⭐ If this lab helped you, consider documenting your implementation and sharing your DevOps learning journey on GitHub.
