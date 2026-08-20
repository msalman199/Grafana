# 📊 Connecting Grafana to Data Sources

![Grafana](https://img.shields.io/badge/Grafana-Visualization-orange?logo=grafana)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-red?logo=prometheus)
![InfluxDB](https://img.shields.io/badge/InfluxDB-Time--Series%20DB-22ADF6?logo=influxdb)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-E95420?logo=ubuntu)
![DevOps](https://img.shields.io/badge/Domain-DevOps-blue)
![Monitoring](https://img.shields.io/badge/Focus-Observability-purple)

> 🚀 **Lab 3:** Learn how to connect Grafana with multiple monitoring and time-series data sources, including Prometheus and InfluxDB, and visualize their data through unified dashboards.

---

## 🎯 Lab Objectives

By completing this lab, you will learn how to:

* 🔹 Install and configure **Prometheus**
* 🔹 Install and configure **InfluxDB**
* 🔹 Install and configure **Grafana**
* 🔹 Connect Grafana to **Prometheus**
* 🔹 Connect Grafana to **InfluxDB**
* 🔹 Test and verify data-source connectivity
* 🔹 Understand different data-source architectures
* 🔹 Create queries using multiple data sources
* 🔹 Build a basic Grafana dashboard using data from different systems

The lab specifically focuses on integrating Prometheus and InfluxDB with Grafana.

---

## 🧰 Technology Stack

| Technology        | Purpose                           |
| ----------------- | --------------------------------- |
| 📊 **Grafana**    | Data visualization and dashboards |
| 🔥 **Prometheus** | Metrics collection and monitoring |
| 🗄️ **InfluxDB**  | Time-series data storage          |
| 🐧 **Linux**      | Lab operating system              |
| ⚙️ **systemd**    | Service management                |
| 🔎 **PromQL**     | Prometheus query language         |
| 📝 **InfluxQL**   | InfluxDB query language           |
| 🌐 **HTTP**       | Service and API communication     |

---

## 📋 Prerequisites

Before beginning, you should have:

* Basic Linux command-line knowledge
* Fundamental networking knowledge
* Understanding of ports and `localhost`
* Familiarity with `nano` or `vim`
* Basic understanding of databases and monitoring
* Previous Grafana installation experience or equivalent knowledge

---

## ☁️ Lab Environment

The lab is performed on an **Al Nafi Linux-based cloud machine**.

The environment provides a bare Linux machine, so the required monitoring and visualization components are installed during the lab. All tasks are performed on a single machine.

---

# 🏗️ Architecture

```text
                    ┌─────────────────────┐
                    │       Grafana       │
                    │      Port 3000      │
                    │   Visualization     │
                    └──────────┬──────────┘
                               │
                  ┌────────────┴────────────┐
                  │                         │
                  ▼                         ▼
        ┌──────────────────┐      ┌──────────────────┐
        │    Prometheus    │      │     InfluxDB     │
        │    Port 9090     │      │    Port 8086     │
        │ Monitoring/Metrics│      │ Time-Series DB   │
        └────────┬─────────┘      └────────┬─────────┘
                 │                         │
                 ▼                         ▼
          Node Exporter              Sample Metrics
             :9100                    grafana_lab
```

---

# 🚀 Task 1 — Install and Configure Prometheus

## 👤 1.1 Create Prometheus User

Create a dedicated system user:

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

Create Prometheus directories:

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

## 📥 1.2 Download Prometheus

Move to `/tmp`:

```bash
cd /tmp
```

Download the version specified in the lab:

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz
```

Extract the archive:

```bash
tar xvf prometheus-2.47.0.linux-amd64.tar.gz
cd prometheus-2.47.0.linux-amd64
```

---

## ⚙️ 1.3 Install Prometheus Binaries

Copy the binaries:

```bash
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
```

Set ownership:

```bash
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

Copy Prometheus console files:

```bash
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
```

Set ownership:

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

---

## 📝 1.4 Configure Prometheus

Create the configuration file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Use the lab configuration:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

Set ownership:

```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

---

## 🔧 1.5 Create Prometheus systemd Service

Create:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Add:

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

---

## ▶️ 1.6 Start Prometheus

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

Check status:

```bash
sudo systemctl status prometheus
```

Verify metrics:

```bash
curl http://localhost:9090/metrics
```

---

# 🗄️ Task 2 — Install and Configure InfluxDB

## 📦 2.1 Install InfluxDB

Add the repository key:

```bash
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
```

Add the repository:

```bash
echo "deb https://repos.influxdata.com/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/influxdb.list
```

Update packages:

```bash
sudo apt update
```

Install:

```bash
sudo apt install influxdb -y
```

---

## ⚙️ 2.2 Configure InfluxDB

Edit:

```bash
sudo nano /etc/influxdb/influxdb.conf
```

Configure the HTTP section:

```ini
[http]
  enabled = true
  bind-address = ":8086"
  auth-enabled = false
  log-enabled = true
  write-tracing = false
  pprof-enabled = true
  debug-pprof-enabled = false
  https-enabled = false
```

---

## ▶️ 2.3 Start InfluxDB

```bash
sudo systemctl enable influxdb
sudo systemctl start influxdb
```

Check:

```bash
sudo systemctl status influxdb
```

Test:

```bash
curl -i http://localhost:8086/ping
```

---

## 📊 2.4 Create Sample Database

Open the InfluxDB CLI:

```bash
influx
```

Create the database:

```sql
CREATE DATABASE grafana_lab
```

Select it:

```sql
USE grafana_lab
```

Insert CPU metrics:

```sql
INSERT cpu_usage,host=server1,region=us-west value=80.5
INSERT cpu_usage,host=server2,region=us-east value=65.2
```

Insert memory metrics:

```sql
INSERT memory_usage,host=server1,region=us-west value=75.8
INSERT memory_usage,host=server2,region=us-east value=82.1
```

Verify:

```sql
SELECT * FROM cpu_usage
SELECT * FROM memory_usage
```

Exit:

```sql
exit
```

---

# 📈 Task 3 — Install and Configure Grafana

## 📦 3.1 Install Grafana

Install required package:

```bash
sudo apt-get install -y software-properties-common
```

Add Grafana key:

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Add repository:

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | \
sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Install:

```bash
sudo apt-get update
sudo apt-get install grafana -y
```

Enable and start Grafana:

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Verify:

```bash
sudo systemctl status grafana-server
```

---

## 🌐 3.2 Access Grafana

Test locally:

```bash
curl http://localhost:3000
```

Find the machine IP:

```bash
ip addr show | grep inet
```

Open:

```text
http://localhost:3000
```

or:

```text
http://<SERVER-IP>:3000
```

The lab specifies the initial credentials as:

```text
Username: admin
Password: admin
```

You should change the password after the first login.

---

# 🔌 Task 4 — Connect Grafana to Prometheus

## 4.1 Add Prometheus Data Source

In Grafana:

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

| Setting     | Value                   |
| ----------- | ----------------------- |
| Name        | `Prometheus`            |
| URL         | `http://localhost:9090` |
| Access      | `Server (default)`      |
| HTTP Method | `GET`                   |

Click:

```text
Save & Test
```

A successful configuration should display:

```text
Data source is working
```

The lab specifies these connection settings and verification process.

---

## 🔍 Troubleshoot Prometheus Connection

Check service:

```bash
sudo systemctl status prometheus
```

Check port:

```bash
sudo netstat -tlnp | grep 9090
```

Test the API:

```bash
curl http://localhost:9090/api/v1/query?query=up
```

---

# 🔌 Task 5 — Connect Grafana to InfluxDB

## 5.1 Add InfluxDB Data Source

Navigate to:

```text
Configuration
      ↓
Data Sources
      ↓
Add data source
      ↓
InfluxDB
```

Configure:

| Setting  | Value                   |
| -------- | ----------------------- |
| Name     | `InfluxDB`              |
| URL      | `http://localhost:8086` |
| Access   | `Server (default)`      |
| Database | `grafana_lab`           |
| User     | Leave empty             |
| Password | Leave empty             |

Click:

```text
Save & Test
```

The expected result is:

```text
Data source is working
```

---

## 🔍 Troubleshoot InfluxDB Connection

Check service:

```bash
sudo systemctl status influxdb
```

Check port:

```bash
sudo netstat -tlnp | grep 8086
```

Test:

```bash
curl -i http://localhost:8086/ping
```

Check databases:

```bash
influx -execute "SHOW DATABASES"
```

---

# 📊 Task 6 — Create a Multi-Source Dashboard

## 6.1 Prometheus Panel

In Grafana:

```text
+
  ↓
Dashboard
  ↓
Add new panel
```

Select:

```text
Data Source → Prometheus
```

Use:

```promql
up
```

Apply the panel and save the dashboard as:

```text
Test Dashboard
```

---

## 6.2 InfluxDB Panel

Add another panel.

Select:

```text
Data Source → InfluxDB
```

Configure:

```text
FROM:       cpu_usage
SELECT:     field(value)
GROUP BY:   tag(host)
```

Apply the panel.

---

## 🎨 Expected Dashboard

Your dashboard should contain data from both systems:

```text
┌──────────────────────────────────────────────┐
│              Test Dashboard                  │
├──────────────────────────┬───────────────────┤
│                          │                   │
│   Prometheus "up"        │   CPU Usage       │
│                          │   InfluxDB         │
│                          │                   │
├──────────────────────────┴───────────────────┤
│             Additional Metrics               │
└──────────────────────────────────────────────┘
```

The lab expects one panel for Prometheus `up` metrics and another for CPU data from InfluxDB.

---

# 🔎 Query Examples

## PromQL

Check target availability:

```promql
up
```

Check Prometheus notifications:

```promql
prometheus_notifications_total
```

Check scrape duration:

```promql
prometheus_target_scrape_duration_seconds
```

Check configuration reload status:

```promql
prometheus_config_last_reload_successful
```

---

## InfluxQL

Query CPU data:

```sql
SELECT * FROM cpu_usage
```

Calculate average CPU usage:

```sql
SELECT mean("value")
FROM "cpu_usage"
WHERE time >= now() - 1h
GROUP BY time(5m), "host"
```

Query memory data:

```sql
SELECT mean(value)
FROM memory_usage
GROUP BY host
```

The lab uses PromQL for Prometheus and InfluxQL for InfluxDB queries.

---

# 🛠️ Troubleshooting

## 🔥 Prometheus

View logs:

```bash
sudo journalctl -u prometheus -f
```

Validate configuration:

```bash
sudo /usr/local/bin/promtool check config /etc/prometheus/prometheus.yml
```

Check firewall:

```bash
sudo ufw status
```

---

## 🗄️ InfluxDB

View logs:

```bash
sudo journalctl -u influxdb -f
```

Test response:

```bash
curl -sl -I http://localhost:8086/ping
```

Check users:

```bash
influx -execute "SHOW USERS"
```

---

## 📊 Grafana

View logs:

```bash
sudo journalctl -u grafana-server -f
```

Inspect configuration:

```bash
sudo cat /etc/grafana/grafana.ini | grep -v "^#" | grep -v "^$"
```

Check port:

```bash
sudo netstat -tlnp | grep 3000
```

The source lab provides these troubleshooting commands for diagnosing Prometheus, InfluxDB, and Grafana problems.

---

# ✅ Final Verification

## 1️⃣ Verify Services

```bash
sudo systemctl status prometheus grafana-server influxdb
```

Expected ports:

```text
Grafana       → 3000
InfluxDB      → 8086
Prometheus    → 9090
Node Exporter → 9100
```

Check:

```bash
sudo netstat -tlnp | grep -E "(3000|8086|9090)"
```

---

## 2️⃣ Verify Grafana Data Sources

In Grafana:

```text
Configuration
    ↓
Data Sources
```

Confirm:

* 🟢 Prometheus is working
* 🟢 InfluxDB is working
* 🟢 Both connections pass **Save & Test**

---

## 3️⃣ Verify Queries

### Prometheus

```promql
up
```

```promql
prometheus_config_last_reload_successful
```

### InfluxDB

```sql
SELECT * FROM cpu_usage
```

```sql
SELECT mean(value) FROM memory_usage GROUP BY host
```

The source lab identifies these queries as final verification examples.

---

# 🧠 What You Learned

After completing this lab, you have built a basic multi-source observability environment:

```text
          Linux Server
               │
       ┌───────┴────────┐
       │                │
       ▼                ▼
  Prometheus         InfluxDB
   Metrics          Time-Series
       │                │
       └───────┬────────┘
               ▼
            Grafana
               │
               ▼
         Unified Dashboard
```

### ⭐ Key Achievements

1. ✅ Installed and configured Prometheus
2. ✅ Installed and configured InfluxDB
3. ✅ Installed and configured Grafana
4. ✅ Connected Grafana to Prometheus
5. ✅ Connected Grafana to InfluxDB
6. ✅ Tested both data-source connections
7. ✅ Created dashboard panels using multiple sources
8. ✅ Practiced PromQL and InfluxQL queries

These achievements correspond directly to the lab's stated conclusion.

---

# 🌍 Why This Lab Matters

### 🔹 Multi-Source Monitoring

Production environments frequently contain different monitoring and telemetry systems. Grafana provides a unified visualization layer for multiple sources.

### 🔹 Flexible Observability

Understanding both Prometheus and InfluxDB gives you experience working with different approaches to time-series monitoring.

### 🔹 DevOps & SRE Skills

Grafana, Prometheus, and InfluxDB are valuable technologies for monitoring infrastructure, applications, and services.

### 🔹 Real-World Dashboards

The same concepts learned here can be extended into production dashboards containing CPU, memory, network, application, and service metrics.

---

# 🚀 Next Steps

After completing this lab, continue with:

* 📊 Advanced Grafana dashboards
* 🚨 Grafana alerting
* 📈 Advanced PromQL
* 📈 Advanced InfluxQL
* 🔢 Custom metrics collection
* 🖥️ Node Exporter monitoring
* 🔗 Multi-source dashboards
* ☁️ Cloud monitoring
* 🔐 Monitoring and security integration
* ⚙️ Production-grade observability architecture

The original lab also identifies advanced dashboards, alerting, custom metrics, and advanced query languages as natural next topics.

---

# 🏆 Lab Completion Checklist

* [ ] Prometheus installed
* [ ] Prometheus service running
* [ ] Prometheus configuration validated
* [ ] InfluxDB installed
* [ ] InfluxDB service running
* [ ] `grafana_lab` database created
* [ ] Sample metrics inserted
* [ ] Grafana installed
* [ ] Grafana service running
* [ ] Prometheus added as Grafana data source
* [ ] InfluxDB added as Grafana data source
* [ ] Both data sources tested successfully
* [ ] Prometheus dashboard panel created
* [ ] InfluxDB dashboard panel created
* [ ] PromQL queries tested
* [ ] InfluxQL queries tested
* [ ] Final services and ports verified

---

## 👨‍💻 Skills Demonstrated

```text
Linux Administration
       │
       ├── Service Management
       ├── Configuration Management
       └── Network/Port Verification
                │
                ▼
        Monitoring & Observability
                │
       ┌────────┼─────────┐
       ▼        ▼         ▼
 Prometheus  InfluxDB   Grafana
       │        │         │
       └────────┴─────────┘
                │
                ▼
        Unified Dashboards
```

---

## 📌 Lab Summary

**Lab:** Connecting Grafana to Data Sources
**Focus:** Multi-source monitoring and visualization
**Primary Tools:** Grafana, Prometheus, InfluxDB
**Environment:** Al Nafi Linux Cloud Machine
**Grafana Port:** `3000`
**Prometheus Port:** `9090`
**InfluxDB Port:** `8086`
**Node Exporter Port:** `9100`

---

## 🎉 Conclusion

Congratulations! 🎊

You have successfully completed **Lab 3 — Connecting Grafana to Data Sources**.

You now have practical experience installing monitoring components, configuring time-series databases, integrating multiple data sources with Grafana, validating connections, writing queries, and building a unified monitoring dashboard.

This provides a strong foundation for progressing toward **advanced monitoring, alerting, observability, DevOps, and SRE practices**.

> 💡 **Monitor → Collect → Store → Visualize → Analyze → Improve**

**Keep learning. Keep monitoring. Keep building. 🚀**
