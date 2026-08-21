# ☁️ Monitoring Cloud Infrastructure with Grafana

> **Production-Pattern Cloud Observability Lab**
> Build a complete cloud infrastructure monitoring platform using **Grafana, Prometheus, Node Exporter, Alertmanager, Docker, Python, and a custom Cloud Metrics Simulator** on a bare Ubuntu AWS EC2 instance.

---

## 📌 Lab Overview

This hands-on lab demonstrates how to design and deploy a production-style observability platform from an empty Ubuntu system.

You will create a simulated cloud monitoring environment representing services such as:

* 🖥️ Amazon EC2
* 🗄️ Amazon RDS
* ⚖️ Elastic Load Balancer
* 🪣 Amazon S3
* 🐧 Linux host infrastructure

A Python-based metrics simulator exposes AWS-style Prometheus metrics. Prometheus collects and stores those metrics, Node Exporter provides host-level metrics, Grafana visualizes the data, and Alertmanager routes alerts generated from Prometheus rules.

The entire architecture demonstrates the separation of responsibilities commonly used in production observability platforms.

---

# 🎯 Objectives

By completing this lab, you will learn how to:

* 🚀 Deploy Grafana, Prometheus, Node Exporter, and Alertmanager.
* 🐧 Configure Linux services with systemd.
* 🐳 Install and configure Docker and Docker Compose.
* 🐍 Build a Python Prometheus metrics exporter.
* ☁️ Simulate AWS EC2, RDS, ELB, and S3 metrics.
* 📈 Configure Prometheus scraping.
* 🔎 Write PromQL queries.
* 📊 Build Grafana dashboards through the HTTP API.
* 🚨 Configure Prometheus alerting rules.
* 📬 Configure Alertmanager routing.
* 🔥 Trigger and validate production-style alerts.
* 🔄 Verify the complete Prometheus → Alertmanager alert pipeline.
* 🧩 Understand how simulated CloudWatch data can later be replaced with real AWS metrics.

---

# 🏗️ Architecture

```text
                         AWS EC2 Ubuntu
                    ┌──────────────────────┐
                    │                      │
                    │      Linux Host      │
                    │                      │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
      ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
      │ Cloud Metrics│  │ Node Exporter│  │    Docker    │
      │  Simulator   │  │    :9100     │  │              │
      │    :9106     │  │              │  │              │
      └──────┬───────┘  └──────┬───────┘  └──────────────┘
             │                  │
             │                  │
             └──────────┬───────┘
                        ▼
                ┌─────────────────┐
                │    Prometheus   │
                │      :9090      │
                │                 │
                │ Metrics + Rules │
                └────────┬────────┘
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
      ┌──────────────┐       ┌──────────────┐
      │ Alertmanager │       │    Grafana   │
      │    :9093     │       │     :3000    │
      │              │       │              │
      │ Alert Routing│       │ Dashboards   │
      └──────┬───────┘       └──────────────┘
             │
             ▼
       Notification
         Receiver
```

---

# 🔄 Observability Data Flow

```text
Cloud Simulator
      │
      │ AWS-style metrics
      ▼
  Prometheus
      │
      ├───────────────▶ Grafana
      │                     │
      │                     ▼
      │                 Dashboards
      │
      └───────────────▶ Alert Rules
                            │
                            ▼
                       Alertmanager
                            │
                            ▼
                        Receiver
```

---

# 🧰 Technology Stack

| Technology        | Purpose                        |
| ----------------- | ------------------------------ |
| 🐧 Ubuntu         | Operating system               |
| ☁️ AWS EC2        | Lab infrastructure             |
| 🐳 Docker         | Container runtime              |
| 🧩 Docker Compose | Container orchestration        |
| 📈 Prometheus     | Metrics collection and storage |
| 📊 Grafana        | Visualization and dashboards   |
| 🖥️ Node Exporter | Host-level metrics             |
| 🚨 Alertmanager   | Alert routing                  |
| 🐍 Python 3       | Cloud metrics simulator        |
| 🔎 PromQL         | Metrics querying               |
| ⚙️ systemd        | Service management             |
| 🌐 HTTP API       | Automation and validation      |
| 📝 YAML           | Configuration                  |

---

# 📋 Prerequisites

You should be comfortable with:

* Linux command-line operations.
* `systemctl`.
* Linux file permissions.
* Editing configuration files.
* Docker fundamentals.
* YAML syntax.
* Cloud infrastructure concepts.
* Compute instances.
* Managed databases.
* Load balancers.
* Object storage.
* Time-series metrics.
* Prometheus scrape intervals.
* Metric labels.
* PromQL `rate()` calculations.

---

# 🖥️ Lab Environment

The lab is performed on a dedicated **AWS EC2 Ubuntu instance provided through Al Nafi**.

The machine starts with a basic Ubuntu installation.

You will install:

```text
Docker
Docker Compose
Python 3
Grafana
Prometheus
Node Exporter
Alertmanager
```

---

# 📁 Recommended Project Structure

Create a working directory:

```bash
mkdir -p ~/cloud-grafana-monitoring
cd ~/cloud-grafana-monitoring
```

Recommended structure:

```text
cloud-grafana-monitoring/
│
├── simulator/
│   └── cloud_metrics_simulator.py
│
├── grafana/
│   └── dashboards/
│       ├── ec2.json
│       ├── rds-elb.json
│       └── infrastructure.json
│
├── prometheus/
│   ├── prometheus.yml
│   └── alert_rules.yml
│
├── alertmanager/
│   └── alertmanager.yml
│
└── scripts/
    └── provision_grafana.py
```

---

# 🛠️ Task 1 — Environment Setup

## 1.1 Install System Dependencies

Update Ubuntu:

```bash
sudo apt-get update -y
```

Install required packages:

```bash
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release \
    python3 \
    python3-pip \
    net-tools
```

Verify Python:

```bash
python3 --version
```

Verify networking tools:

```bash
ifconfig --version 2>/dev/null || ip addr
```

---

# 🐳 Install Docker

Create Docker keyring:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Import Docker's signing key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
    | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Set permissions:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Add repository:

```bash
sudo tee /etc/apt/sources.list.d/docker.list <<'EOF'
deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable
EOF
```

Update:

```bash
sudo apt-get update -y
```

Install Docker:

```bash
sudo apt-get install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-compose-plugin
```

Add the current user to Docker:

```bash
sudo usermod -aG docker "$USER"
```

Refresh the group:

```bash
newgrp docker
```

Verify:

```bash
docker version
docker compose version
```

---

# 🔧 Docker Repository Troubleshooting

If you see:

```text
E: Malformed entry 1 in list file
```

Inspect:

```bash
cat /etc/apt/sources.list.d/docker.list
```

The repository must be a single unbroken line.

If necessary:

```bash
sudo rm /etc/apt/sources.list.d/docker.list
```

Recreate the repository file using the heredoc above.

Then:

```bash
sudo apt-get update
```

---

# 📊 Install Grafana

Create the Grafana keyring:

```bash
sudo mkdir -p /etc/apt/keyrings
```

Download the signing key:

```bash
curl -fsSL https://apt.grafana.com/gpg.key \
    | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
```

Create the repository:

```bash
sudo tee /etc/apt/sources.list.d/grafana.list <<'EOF'
deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main
EOF
```

Update:

```bash
sudo apt-get update -y
```

Install Grafana:

```bash
sudo apt-get install -y grafana
```

Verify:

```bash
grafana-server --version
```

---

# 🔐 Grafana Repository Troubleshooting

If you receive:

```text
NO_PUBKEY
```

Check:

```bash
ls -lh /etc/apt/keyrings/grafana.gpg
```

If the file is empty, recreate it:

```bash
curl -fsSL https://apt.grafana.com/gpg.key \
    | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
```

Then:

```bash
sudo apt-get update
```

---

# 📈 Task 1.2 — Install Prometheus

Choose a stable Prometheus release.

Example:

```bash
PROM_VERSION="2.51.2"
```

Download:

```bash
cd /tmp

curl -fsSL \
"https://github.com/prometheus/prometheus/releases/download/v${PROM_VERSION}/prometheus-${PROM_VERSION}.linux-amd64.tar.gz" \
-o prometheus.tar.gz
```

Extract:

```bash
tar -xzf prometheus.tar.gz
```

Create service account:

```bash
sudo useradd \
    --no-create-home \
    --shell /bin/false \
    prometheus
```

Create directories:

```bash
sudo mkdir -p \
    /etc/prometheus \
    /var/lib/prometheus
```

Install binaries:

```bash
sudo cp \
"prometheus-${PROM_VERSION}.linux-amd64/prometheus" \
/usr/local/bin/prometheus

sudo cp \
"prometheus-${PROM_VERSION}.linux-amd64/promtool" \
/usr/local/bin/promtool
```

Copy console resources:

```bash
sudo cp -r \
"prometheus-${PROM_VERSION}.linux-amd64/consoles" \
/etc/prometheus/

sudo cp -r \
"prometheus-${PROM_VERSION}.linux-amd64/console_libraries" \
/etc/prometheus/
```

Set ownership:

```bash
sudo chown -R prometheus:prometheus \
    /etc/prometheus \
    /var/lib/prometheus \
    /usr/local/bin/prometheus \
    /usr/local/bin/promtool
```

Verify:

```bash
prometheus --version
promtool --version
```

---

# 🖥️ Install Node Exporter

Set the version:

```bash
NODE_VERSION="1.8.0"
```

Download:

```bash
cd /tmp

curl -fsSL \
"https://github.com/prometheus/node_exporter/releases/download/v${NODE_VERSION}/node_exporter-${NODE_VERSION}.linux-amd64.tar.gz" \
-o node_exporter.tar.gz
```

Extract:

```bash
tar -xzf node_exporter.tar.gz
```

Install:

```bash
sudo cp \
"node_exporter-${NODE_VERSION}.linux-amd64/node_exporter" \
/usr/local/bin/node_exporter
```

Set ownership:

```bash
sudo chown prometheus:prometheus \
    /usr/local/bin/node_exporter
```

Verify:

```bash
node_exporter --version
```

---

# ⚙️ Task 1.3 — Configure Prometheus

Create the configuration:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Example:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - /etc/prometheus/alert_rules.yml

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - localhost:9093

scrape_configs:

  - job_name: prometheus
    static_configs:
      - targets:
          - localhost:9090

  - job_name: node-exporter
    static_configs:
      - targets:
          - localhost:9100

  - job_name: cloud-simulator
    static_configs:
      - targets:
          - localhost:9106
```

Set ownership:

```bash
sudo chown prometheus:prometheus \
    /etc/prometheus/prometheus.yml
```

Create an initial empty rules file:

```bash
sudo tee /etc/prometheus/alert_rules.yml <<'EOF'
groups: []
EOF
```

Set ownership:

```bash
sudo chown prometheus:prometheus \
    /etc/prometheus/alert_rules.yml
```

---

# ⚙️ Create Prometheus systemd Service

Create:

```bash
sudo tee /etc/systemd/system/prometheus.service <<'EOF'
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple

ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

---

# ⚙️ Create Node Exporter systemd Service

```bash
sudo tee /etc/systemd/system/node_exporter.service <<'EOF'
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple

ExecStart=/usr/local/bin/node_exporter \
  --collector.systemd \
  --collector.processes

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

---

# 🚀 Start the Services

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable and start:

```bash
sudo systemctl enable --now \
    prometheus \
    node_exporter \
    grafana-server
```

Check status:

```bash
sudo systemctl status prometheus
sudo systemctl status node_exporter
sudo systemctl status grafana-server
```

Or:

```bash
sudo systemctl is-active \
    prometheus \
    node_exporter \
    grafana-server
```

Expected:

```text
active
active
active
```

---

# 🔎 Verify HTTP Endpoints

Prometheus:

```bash
curl -fsSL \
-o /dev/null \
-w "Prometheus: %{http_code}\n" \
http://localhost:9090/-/healthy
```

Node Exporter:

```bash
curl -fsSL \
-o /dev/null \
-w "NodeExporter: %{http_code}\n" \
http://localhost:9100/metrics
```

Grafana:

```bash
curl -fsSL \
-o /dev/null \
-w "Grafana: %{http_code}\n" \
http://localhost:3000/api/health
```

Expected:

```text
Prometheus: 200
NodeExporter: 200
Grafana: 200
```

---

# 🐍 Task 2 — Build Cloud Metrics Simulator

The simulator represents telemetry normally collected from cloud services.

It exposes:

```text
http://localhost:9106/metrics
```

The simulator must produce:

```text
EC2 metrics
RDS metrics
ELB metrics
S3 metrics
```

---

# ☁️ EC2 Metrics

Required metric families:

```text
aws_ec2_cpu_utilization
aws_ec2_memory_utilization
aws_ec2_network_in_bytes
aws_ec2_network_out_bytes
aws_ec2_disk_utilization
```

Minimum labels:

```text
instance_id
region
```

Example:

```text
aws_ec2_cpu_utilization{instance_id="i-demo001",region="us-east-1"} 42.5
```

---

# 🗄️ RDS Metrics

Required:

```text
aws_rds_cpu_utilization
aws_rds_connections
aws_rds_free_storage_bytes
```

Minimum labels:

```text
db_instance
engine
```

Example:

```text
aws_rds_cpu_utilization{db_instance="database-01",engine="postgres"} 37.2
```

---

# ⚖️ ELB Metrics

Required:

```text
aws_elb_request_count
aws_elb_latency_seconds
aws_elb_healthy_hosts
```

Minimum labels:

```text
load_balancer
region
```

---

# 🪣 S3 Metrics

Required:

```text
aws_s3_bucket_size_bytes
aws_s3_number_of_objects
```

Minimum labels:

```text
bucket_name
region
```

---

# 🐍 Example Simulator Implementation

Create:

```bash
mkdir -p ~/cloud-grafana-monitoring/simulator
nano ~/cloud-grafana-monitoring/simulator/cloud_metrics_simulator.py
```

Example implementation:

```python
#!/usr/bin/env python3

import random
from http.server import BaseHTTPRequestHandler, HTTPServer


class CloudMetricsSimulator:

    def get_ec2_metrics(self):
        instance = 'i-demo001'
        region = 'us-east-1'

        return [
            f'aws_ec2_cpu_utilization{{instance_id="{instance}",region="{region}"}} {random.uniform(20, 90):.2f}',
            f'aws_ec2_memory_utilization{{instance_id="{instance}",region="{region}"}} {random.uniform(30, 85):.2f}',
            f'aws_ec2_network_in_bytes{{instance_id="{instance}",region="{region}"}} {random.uniform(100000, 10000000):.2f}',
            f'aws_ec2_network_out_bytes{{instance_id="{instance}",region="{region}"}} {random.uniform(100000, 10000000):.2f}',
            f'aws_ec2_disk_utilization{{instance_id="{instance}",region="{region}"}} {random.uniform(20, 80):.2f}',
        ]

    def get_rds_metrics(self):
        db = 'database-01'
        engine = 'postgres'

        return [
            f'aws_rds_cpu_utilization{{db_instance="{db}",engine="{engine}"}} {random.uniform(10, 80):.2f}',
            f'aws_rds_connections{{db_instance="{db}",engine="{engine}"}} {random.randint(10, 400)}',
            f'aws_rds_free_storage_bytes{{db_instance="{db}",engine="{engine}"}} {random.uniform(2000000, 10000000000):.2f}',
        ]

    def get_elb_metrics(self):
        lb = 'app-lb'
        region = 'us-east-1'

        return [
            f'aws_elb_request_count{{load_balancer="{lb}",region="{region}"}} {random.randint(100, 5000)}',
            f'aws_elb_latency_seconds{{load_balancer="{lb}",region="{region}"}} {random.uniform(0.01, 4.0):.3f}',
            f'aws_elb_healthy_hosts{{load_balancer="{lb}",region="{region}"}} {random.randint(1, 10)}',
        ]

    def get_s3_metrics(self):
        bucket = 'demo-monitoring-bucket'
        region = 'us-east-1'

        return [
            f'aws_s3_bucket_size_bytes{{bucket_name="{bucket}",region="{region}"}} {random.uniform(1000000, 10000000000):.2f}',
            f'aws_s3_number_of_objects{{bucket_name="{bucket}",region="{region}"}} {random.randint(100, 100000)}',
        ]

    def render_all(self):
        metrics = (
            self.get_ec2_metrics()
            + self.get_rds_metrics()
            + self.get_elb_metrics()
            + self.get_s3_metrics()
        )

        return "\n".join(metrics) + "\n"


simulator = CloudMetricsSimulator()


class MetricsHandler(BaseHTTPRequestHandler):

    def do_GET(self):
        if self.path != "/metrics":
            self.send_response(404)
            self.end_headers()
            return

        body = simulator.render_all().encode()

        self.send_response(200)
        self.send_header(
            "Content-Type",
            "text/plain; version=0.0.4"
        )
        self.send_header(
            "Content-Length",
            str(len(body))
        )
        self.end_headers()

        self.wfile.write(body)


server = HTTPServer(
    ("0.0.0.0", 9106),
    MetricsHandler
)

print("Cloud Metrics Simulator listening on port 9106")

server.serve_forever()
```

Make executable:

```bash
chmod +x \
~/cloud-grafana-monitoring/simulator/cloud_metrics_simulator.py
```

---

# 🧪 Test the Simulator

Run:

```bash
python3 \
~/cloud-grafana-monitoring/simulator/cloud_metrics_simulator.py
```

From another terminal:

```bash
curl -fsSL \
http://localhost:9106/metrics
```

Filter AWS metrics:

```bash
curl -fsSL \
http://localhost:9106/metrics | grep -E "^aws_"
```

You should see:

```text
aws_ec2_
aws_rds_
aws_elb_
aws_s3_
```

---

# ⚙️ Run Simulator with systemd

Create:

```bash
sudo tee /etc/systemd/system/cloud-metrics-simulator.service <<'EOF'
[Unit]
Description=Cloud Metrics Simulator
After=network-online.target
Wants=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple

ExecStart=/usr/bin/python3 \
  /home/$USER/cloud-grafana-monitoring/simulator/cloud_metrics_simulator.py

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

> **Important:** Replace `/home/$USER` with the actual absolute path to the simulator before starting the service. systemd does not expand `$USER` in `ExecStart` the way an interactive shell does.

For example:

```text
/home/ubuntu/cloud-grafana-monitoring/simulator/cloud_metrics_simulator.py
```

Reload:

```bash
sudo systemctl daemon-reload
```

Enable:

```bash
sudo systemctl enable --now \
cloud-metrics-simulator
```

Check:

```bash
sudo systemctl status \
cloud-metrics-simulator
```

---

# 🔄 Reload Prometheus

Once the simulator is running:

```bash
curl -fsSL \
-X POST \
http://localhost:9090/-/reload
```

Verify targets:

```bash
curl -fsSL \
http://localhost:9090/api/v1/targets
```

Extract target health:

```bash
curl -fsSL \
http://localhost:9090/api/v1/targets \
| python3 -c '
import sys
import json

targets = json.load(sys.stdin)["data"]["activeTargets"]

for target in targets:
    print(
        target["labels"].get("job"),
        target["health"]
    )
'
```

Expected:

```text
prometheus up
node-exporter up
cloud-simulator up
```

---

# 📊 Task 2.2 — Configure Grafana

Open:

```text
http://<EC2-PUBLIC-IP>:3000
```

Log in using the configured Grafana administrator account.

> Change the default password immediately in a real environment.

---

# 🔌 Prometheus Data Source

Prometheus URL:

```text
http://localhost:9090
```

For Grafana running directly on the same host, use:

```text
http://localhost:9090
```

The data source should be configured as the default Prometheus source.

---

# 📈 Dashboard 1 — EC2 Instance Monitoring

Dashboard title:

```text
EC2 Instance Monitoring
```

Required panels:

| Panel              | Metric                       | Unit    |
| ------------------ | ---------------------------- | ------- |
| CPU Utilization    | `aws_ec2_cpu_utilization`    | Percent |
| Memory Utilization | `aws_ec2_memory_utilization` | Percent |
| Network I/O        | Network in/out metrics       | Bps     |
| Disk Utilization   | `aws_ec2_disk_utilization`   | Percent |

---

## CPU Query

```promql
aws_ec2_cpu_utilization
```

## Memory Query

```promql
aws_ec2_memory_utilization
```

## Network Input

```promql
aws_ec2_network_in_bytes
```

## Network Output

```promql
aws_ec2_network_out_bytes
```

## Disk

```promql
aws_ec2_disk_utilization
```

---

# 🗄️ Dashboard 2 — RDS and ELB Monitoring

Dashboard title:

```text
RDS and ELB Monitoring
```

Required panels:

| Panel            | Metric                       | Unit    |
| ---------------- | ---------------------------- | ------- |
| DB CPU           | `aws_rds_cpu_utilization`    | Percent |
| DB Connections   | `aws_rds_connections`        | Short   |
| Free Storage     | `aws_rds_free_storage_bytes` | Bytes   |
| ELB Request Rate | `aws_elb_request_count`      | Short   |
| ELB Latency      | `aws_elb_latency_seconds`    | Seconds |

---

## RDS CPU

```promql
aws_rds_cpu_utilization
```

## RDS Connections

```promql
aws_rds_connections
```

## RDS Storage

```promql
aws_rds_free_storage_bytes
```

## ELB Requests

```promql
aws_elb_request_count
```

## ELB Latency

```promql
aws_elb_latency_seconds
```

---

# 🖥️ Dashboard 3 — Infrastructure Overview

Dashboard title:

```text
Infrastructure Overview
```

Required panels:

```text
Host CPU
Host Memory
Host Disk
Service Status
```

---

## Host CPU

```promql
100 -
(
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

## Host Memory

```promql
(
  1 -
  node_memory_MemAvailable_bytes /
  node_memory_MemTotal_bytes
) * 100
```

## Host Disk

```promql
(
  1 -
  node_filesystem_avail_bytes{mountpoint="/"}
  /
  node_filesystem_size_bytes{mountpoint="/"}
) * 100
```

## Service Status

```promql
up
```

---

# 🤖 Grafana HTTP API Automation

The dashboards should be created through the Grafana HTTP API rather than manually importing them.

Important endpoints:

```text
POST /api/datasources
POST /api/dashboards/db
```

Example:

```bash
curl -u admin:admin \
-X POST \
-H "Content-Type: application/json" \
http://localhost:3000/api/datasources
```

For dashboards:

```bash
curl -u admin:admin \
-X POST \
-H "Content-Type: application/json" \
-d @dashboard.json \
http://localhost:3000/api/dashboards/db
```

---

# 🔁 Idempotent Dashboard Provisioning

Your provisioning script should use a stable dashboard UID.

Example:

```json
{
  "dashboard": {
    "uid": "ec2-monitoring",
    "title": "EC2 Instance Monitoring"
  },
  "overwrite": true
}
```

This ensures repeated executions update the existing dashboard instead of creating duplicates.

---

# 🚨 Task 3 — Alertmanager

Alertmanager receives firing alerts from Prometheus and determines where those alerts should be routed.

Architecture:

```text
Prometheus
    │
    │ Firing Alert
    ▼
Alertmanager
    │
    ├── Warning
    ├── Critical
    └── Other Routes
         │
         ▼
     Receiver
```

---

# 📋 Required Alert Rules

Create:

```bash
sudo nano /etc/prometheus/alert_rules.yml
```

Use:

```yaml
groups:

  - name: cloud-infrastructure-alerts

    rules:

      - alert: HighEC2CPU
        expr: aws_ec2_cpu_utilization > 85
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "EC2 instance CPU above 85% for 1 minute"

      - alert: HighEC2Memory
        expr: aws_ec2_memory_utilization > 80
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "EC2 instance memory above 80% for 1 minute"

      - alert: LowRDSStorage
        expr: aws_rds_free_storage_bytes < 2000000
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "RDS free storage below 2 MB for 2 minutes"

      - alert: ELBHighLatency
        expr: aws_elb_latency_seconds > 2
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "ELB latency above 2 seconds for 1 minute"

      - alert: TargetDown
        expr: up == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "A monitored target has been unreachable for 30 seconds"
```

---

# 🧪 Validate Alert Rules

Run:

```bash
sudo /usr/local/bin/promtool check rules \
/etc/prometheus/alert_rules.yml
```

Expected:

```text
SUCCESS: X rules found
```

If validation fails, fix the YAML before reloading Prometheus.

---

# 📬 Configure Alertmanager

Create:

```bash
sudo mkdir -p /etc/alertmanager
```

Create:

```bash
sudo nano /etc/alertmanager/alertmanager.yml
```

Example:

```yaml
global:
  resolve_timeout: 5m

route:
  group_by:
    - alertname

  group_wait: 10s
  group_interval: 30s
  repeat_interval: 1h

  receiver: local-webhook

receivers:

  - name: local-webhook

    webhook_configs:
      - url: "http://localhost:8080/alerts"
```

> The webhook receiver can be replaced with Slack, email, PagerDuty, or another notification platform in a production environment.

---

# ⚙️ Alertmanager systemd Service

Install Alertmanager from an official release.

After installing the binary, create:

```bash
sudo tee /etc/systemd/system/alertmanager.service <<'EOF'
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple

ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager \
  --web.listen-address=0.0.0.0:9093

Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

Create storage:

```bash
sudo mkdir -p /var/lib/alertmanager
sudo chown -R prometheus:prometheus \
    /etc/alertmanager \
    /var/lib/alertmanager
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable:

```bash
sudo systemctl enable --now alertmanager
```

Verify:

```bash
sudo systemctl status alertmanager
```

---

# 🔎 Verify Alertmanager

Check:

```bash
curl -fsSL \
http://localhost:9093/-/healthy
```

Open:

```text
http://<EC2-PUBLIC-IP>:9093
```

---

# 🔄 Reload Prometheus

After configuring Alertmanager and alert rules:

```bash
curl -fsSL \
-X POST \
http://localhost:9090/-/reload
```

Verify Prometheus sees Alertmanager:

```bash
curl -fsSL \
http://localhost:9090/api/v1/alertmanagers
```

---

# 🔥 Force High CPU Alert

The simulator normally generates values within realistic ranges.

For alert testing, temporarily modify the EC2 CPU metric so it produces a value above:

```text
85%
```

For example:

```text
aws_ec2_cpu_utilization{instance_id="i-demo001",region="us-east-1"} 95
```

Keep the value above the threshold for more than one minute.

---

# 🚨 Check Active Alerts

Run:

```bash
curl -fsSL \
http://localhost:9090/api/v1/alerts
```

Or:

```bash
curl -fsSL \
http://localhost:9090/api/v1/alerts \
| python3 -c '
import sys
import json

alerts = json.load(sys.stdin)["data"]["alerts"]

for alert in alerts:
    print(
        alert["labels"].get("alertname"),
        alert["state"]
    )
'
```

Expected:

```text
HighEC2CPU firing
```

---

# 🔄 Restore Normal Metrics

After validating the alert:

1. Restore the simulator's normal random CPU range.
2. Allow Prometheus to scrape the new values.
3. Wait for the alert to resolve.
4. Verify the alert becomes inactive.

This demonstrates the complete lifecycle:

```text
Normal
  │
  ▼
Threshold Crossed
  │
  ▼
Pending
  │
  ▼
Firing
  │
  ▼
Condition Cleared
  │
  ▼
Inactive
```

---

# 📊 Grafana Unified Alerting

Prometheus/Alertmanager alerting and Grafana's unified alerting are separate systems.

For the Grafana portion of the lab, create an alert rule using:

```promql
aws_ec2_cpu_utilization
```

Example condition:

```text
CPU > 85%
```

Configure a Grafana contact point and verify that an alert state transition appears in Grafana's alert history.

This demonstrates two common alerting architectures:

```text
Prometheus Alerting
       │
       ▼
 Alertmanager
       │
       ▼
 Notification

Grafana Alerting
       │
       ▼
 Grafana Contact Point
       │
       ▼
 Notification
```

---

# 🧪 End-to-End Validation

## Prometheus

Check:

```bash
curl http://localhost:9090/-/healthy
```

Expected:

```text
Prometheus Server is Healthy.
```

---

## Node Exporter

```bash
curl -fsSL \
http://localhost:9100/metrics \
| grep node_cpu_seconds_total
```

---

## Cloud Simulator

```bash
curl -fsSL \
http://localhost:9106/metrics \
| grep "^aws_"
```

---

## Grafana

```bash
curl -fsSL \
http://localhost:3000/api/health
```

---

## Alertmanager

```bash
curl -fsSL \
http://localhost:9093/-/healthy
```

---

## Prometheus Targets

Open:

```text
http://<EC2-PUBLIC-IP>:9090/targets
```

Expected:

```text
prometheus       UP
node-exporter    UP
cloud-simulator  UP
```

---

# 🧪 Troubleshooting

## ❌ Grafana Shows "No Data"

Check Prometheus targets:

```bash
curl -fsSL \
http://localhost:9090/api/v1/targets
```

Look for:

```text
health: up
```

Then test the metric directly:

```bash
curl -fsSL \
'http://localhost:9090/api/v1/query?query=aws_ec2_cpu_utilization'
```

If Prometheus returns data but Grafana does not, check:

* Grafana data source URL.
* Prometheus data source selection.
* Dashboard PromQL expression.
* Metric labels.
* Query time range.

---

# ❌ Target Is DOWN

Check service status:

```bash
sudo systemctl status cloud-metrics-simulator
```

Check logs:

```bash
sudo journalctl \
-u cloud-metrics-simulator \
-n 50 \
--no-pager
```

Test manually:

```bash
curl http://localhost:9106/metrics
```

---

# ❌ Alert Remains Pending

Check the alert expression:

```bash
curl -fsSL \
http://localhost:9090/api/v1/query \
--get \
--data-urlencode \
'query=aws_ec2_cpu_utilization > 85'
```

If no series is returned, the condition is not currently true.

Check:

* Simulator metric value.
* Prometheus scrape interval.
* Alert evaluation interval.
* `for` duration.
* Metric labels.
* PromQL syntax.

Remember:

```text
for: 1m
```

means the condition must remain continuously true for one minute before the alert becomes firing.

---

# 📊 Useful PromQL Queries

## EC2 CPU

```promql
aws_ec2_cpu_utilization
```

## EC2 Memory

```promql
aws_ec2_memory_utilization
```

## EC2 Network

```promql
aws_ec2_network_in_bytes
```

```promql
aws_ec2_network_out_bytes
```

## EC2 Disk

```promql
aws_ec2_disk_utilization
```

## RDS CPU

```promql
aws_rds_cpu_utilization
```

## RDS Connections

```promql
aws_rds_connections
```

## RDS Storage

```promql
aws_rds_free_storage_bytes
```

## ELB Requests

```promql
aws_elb_request_count
```

## ELB Latency

```promql
aws_elb_latency_seconds
```

## Healthy ELB Hosts

```promql
aws_elb_healthy_hosts
```

## S3 Bucket Size

```promql
aws_s3_bucket_size_bytes
```

## S3 Objects

```promql
aws_s3_number_of_objects
```

## Host CPU

```promql
100 -
(
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

## Host Memory

```promql
(
  1 -
  node_memory_MemAvailable_bytes /
  node_memory_MemTotal_bytes
) * 100
```

## Host Filesystem

```promql
(
  1 -
  node_filesystem_avail_bytes{mountpoint="/"}
  /
  node_filesystem_size_bytes{mountpoint="/"}
) * 100
```

## Service Availability

```promql
up
```

---

# 🧪 Lab Validation Checklist

## Environment

* [ ] Ubuntu instance prepared.
* [ ] Python installed.
* [ ] Docker installed.
* [ ] Docker Compose installed.
* [ ] Grafana installed.
* [ ] Prometheus installed.
* [ ] Node Exporter installed.
* [ ] Alertmanager installed.

## Services

* [ ] Prometheus is `active`.
* [ ] Node Exporter is `active`.
* [ ] Grafana is `active`.
* [ ] Cloud simulator is `active`.
* [ ] Alertmanager is `active`.

## Metrics

* [ ] Prometheus endpoint responds.
* [ ] Node Exporter exposes host metrics.
* [ ] Cloud simulator exposes AWS-style metrics.
* [ ] Prometheus sees the simulator as `UP`.
* [ ] EC2 metrics are available.
* [ ] RDS metrics are available.
* [ ] ELB metrics are available.
* [ ] S3 metrics are available.

## Grafana

* [ ] Prometheus configured as a data source.
* [ ] EC2 dashboard created.
* [ ] RDS and ELB dashboard created.
* [ ] Infrastructure dashboard created.
* [ ] All panels display data.
* [ ] Dashboards can be provisioned repeatedly without duplicates.

## Alerting

* [ ] Alert rules validated with `promtool`.
* [ ] Alertmanager running.
* [ ] Prometheus connected to Alertmanager.
* [ ] `HighEC2CPU` tested.
* [ ] `HighEC2Memory` configured.
* [ ] `LowRDSStorage` configured.
* [ ] `ELBHighLatency` configured.
* [ ] `TargetDown` configured.
* [ ] Alert reaches `firing`.
* [ ] Alert reaches `inactive` after recovery.
* [ ] Grafana alert history contains a state transition.

---

# 🔐 Production Security Recommendations

The lab uses simplified credentials and local endpoints for learning.

For production:

* 🔑 Change default Grafana credentials immediately.
* 🔐 Use HTTPS for Grafana.
* 🛡️ Restrict Prometheus and Alertmanager access.
* 🔒 Never expose administrative APIs publicly.
* 🔑 Store secrets outside source-controlled files.
* 🌐 Restrict AWS Security Group rules.
* 📌 Pin software versions.
* 💾 Configure persistent Prometheus storage.
* 🗃️ Back up Grafana dashboards and configuration.
* 🔔 Configure reliable Alertmanager receivers.
* 🧑‍💻 Apply least-privilege service accounts.
* 📜 Monitor service logs.
* 🔎 Add authentication to internal monitoring endpoints where appropriate.

---

# ☁️ Moving from Simulation to Real AWS

The cloud simulator is intentionally designed to resemble a real cloud metrics exporter.

In a production environment, the following:

```text
Python Cloud Metrics Simulator
```

could be replaced by:

```text
AWS CloudWatch Exporter
```

The rest of the architecture can remain conceptually similar:

```text
AWS CloudWatch
      │
      ▼
CloudWatch Exporter
      │
      ▼
Prometheus
      │
      ├───────────────┐
      ▼               ▼
   Grafana       Alertmanager
      │               │
      ▼               ▼
Dashboards       Notifications
```

This separation makes the lab a useful foundation for understanding real AWS observability architectures.

---

# 📈 Expected Outcomes

At the end of the lab, you should have **three live Grafana dashboards**:

### 🖥️ EC2 Instance Monitoring

Displays:

```text
CPU
Memory
Network I/O
Disk
```

### 🗄️ RDS and ELB Monitoring

Displays:

```text
Database CPU
Database connections
Free storage
ELB requests
ELB latency
```

### 🌐 Infrastructure Overview

Displays:

```text
Host CPU
Host Memory
Host Disk
Service Status
```

You should also be able to demonstrate:

```text
Metric Collection
      ↓
Prometheus
      ↓
PromQL
      ↓
Grafana
      ↓
Alert Rules
      ↓
Alertmanager
      ↓
Notification Receiver
```

---

# 🧠 Key Learning Points

### Prometheus

Prometheus provides the time-series storage and querying layer.

### Node Exporter

Node Exporter exposes Linux host metrics for CPU, memory, filesystem, processes, and other operating-system resources.

### Cloud Metrics Simulator

The Python application demonstrates how cloud-service metrics can be exposed in Prometheus's text exposition format.

### Grafana

Grafana provides visualization and operational dashboards on top of Prometheus data.

### Alertmanager

Alertmanager handles grouping, routing, silencing, and notification of Prometheus alerts.

### PromQL

PromQL allows operators to transform raw time-series data into meaningful operational measurements.

### systemd

Running observability components as systemd services ensures they automatically restart and survive system reboots.

---

# 🧹 Cleanup

Stop the simulator:

```bash
sudo systemctl disable --now \
cloud-metrics-simulator
```

Stop Alertmanager:

```bash
sudo systemctl disable --now \
alertmanager
```

Stop Node Exporter:

```bash
sudo systemctl disable --now \
node_exporter
```

Stop Prometheus:

```bash
sudo systemctl disable --now \
prometheus
```

Stop Grafana:

```bash
sudo systemctl disable --now \
grafana-server
```

---

# 🚀 Future Enhancements

After completing this lab, consider extending the platform with:

* ☁️ Real AWS CloudWatch integration.
* 🔔 Slack Alertmanager notifications.
* 📧 Email alerting.
* 📱 PagerDuty integration.
* 🐳 Container monitoring with cAdvisor.
* 🔥 Blackbox Exporter for endpoint monitoring.
* 📜 Loki for centralized logs.
* 🔭 Tempo for distributed tracing.
* 🧩 Terraform-based deployment.
* ☸️ Kubernetes monitoring.
* 🔐 OAuth/SSO for Grafana.
* 📊 SLO and SLA dashboards.
* 🏷️ Standardized cloud resource labels.

---

# 🏆 Conclusion

This lab builds a complete cloud infrastructure observability platform from a blank Ubuntu instance.

The final architecture separates the major responsibilities of a production monitoring system:

```text
              ┌────────────────────┐
              │ Cloud Infrastructure│
              └──────────┬─────────┘
                         │
                         ▼
                 Metrics Exporters
                         │
                         ▼
                  ┌─────────────┐
                  │ Prometheus  │
                  └──────┬──────┘
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
         ┌─────────┐          ┌─────────────┐
         │ Grafana │          │ Alertmanager│
         └────┬────┘          └──────┬──────┘
              │                      │
              ▼                      ▼
         Dashboards              Alerts
```

You have demonstrated the ability to:

* Build an observability stack from scratch.
* Collect cloud and host metrics.
* Create PromQL-based dashboards.
* Automate Grafana configuration through its API.
* Define threshold-based Prometheus alerts.
* Route alerts through Alertmanager.
* Test the complete alert lifecycle.
* Design an architecture that can later transition from simulated metrics to real AWS CloudWatch telemetry.

> ⭐ **Lab Achievement:** You have implemented a production-pattern cloud monitoring architecture combining **Grafana + Prometheus + Node Exporter + Alertmanager + Python metrics simulation**, providing a strong foundation for real-world Cloud DevOps and observability engineering.
