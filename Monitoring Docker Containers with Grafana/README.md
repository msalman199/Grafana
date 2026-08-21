# 📊 Monitoring Docker Containers with Grafana

> **Hands-on DevOps & Observability Lab**
> Build a complete container monitoring platform using **Docker, Prometheus, Grafana, cAdvisor, and Node Exporter** on Ubuntu.

---

## 🚀 Lab Overview

This lab demonstrates how to design and deploy a production-pattern monitoring stack for Docker containers and the underlying Linux host.

You will configure Docker's native Prometheus metrics endpoint, collect container metrics with **cAdvisor**, collect host metrics with **Node Exporter**, store metrics in **Prometheus**, and visualize them through automatically provisioned **Grafana dashboards**.

The entire observability configuration is managed as code, making the environment reproducible and suitable for Git-based version control.

---

## 🎯 Objectives

By completing this lab, you will be able to:

* 🐳 Configure Docker Engine with a Prometheus metrics endpoint.
* 📈 Deploy Prometheus for centralized metric collection.
* 📦 Monitor Docker containers with cAdvisor.
* 🖥️ Monitor Linux host resources with Node Exporter.
* 📊 Deploy Grafana for metric visualization.
* 🔗 Configure Prometheus as Grafana's default data source automatically.
* 🎨 Create dashboards through Grafana file-based provisioning.
* 🚨 Configure Prometheus alerting rules.
* 🔥 Generate container CPU load and validate alert firing.
* 🧩 Correlate host-level and container-level performance metrics.
* 💾 Manage the complete monitoring configuration as code.

---

## 🏗️ Architecture

```text
                         ┌──────────────────────┐
                         │      Ubuntu EC2      │
                         │                      │
                         │      Docker Engine   │
                         │      Metrics :9323   │
                         └──────────┬───────────┘
                                    │
                                    │ Docker Metrics
                                    ▼
┌────────────────┐          ┌──────────────────────┐
│ Docker         │─────────▶│                     │
│ Workloads      │          │     Prometheus      │
│                │          │       :9090         │
└────────────────┘          │                     │
                            └──────────┬──────────┘
                                       │
                  ┌────────────────────┼────────────────────┐
                  │                    │                    │
                  ▼                    ▼                    ▼
           ┌────────────┐      ┌──────────────┐      ┌──────────────┐
           │  cAdvisor  │      │ Node Exporter│      │ Docker Engine│
           │    :8080   │      │     :9100    │      │    :9323     │
           └────────────┘      └──────────────┘      └──────────────┘
                  │                    │                    │
                  └────────────────────┼────────────────────┘
                                       │
                                       ▼
                            ┌──────────────────────┐
                            │       Grafana        │
                            │        :3000         │
                            │                      │
                            │ Dashboards + Alerts  │
                            └──────────────────────┘
```

### 🔄 Data Flow

```text
Docker Containers
       │
       ├──────────────▶ cAdvisor ──────────┐
       │                                   │
       └──────────────▶ Docker Metrics ────┤
                                           │
Ubuntu Host ──────────▶ Node Exporter ─────┤
                                           ▼
                                      Prometheus
                                           │
                                           ▼
                                       Grafana
                                           │
                                           ▼
                                  Dashboards / Alerts
```

---

# 🧰 Technology Stack

| Technology                   | Purpose                        |
| ---------------------------- | ------------------------------ |
| 🐧 Ubuntu                    | Host operating system          |
| ☁️ AWS EC2                   | Lab infrastructure             |
| 🐳 Docker Engine             | Container runtime              |
| 🧩 Docker Compose            | Multi-container orchestration  |
| 📈 Prometheus                | Metrics collection and storage |
| 📊 Grafana                   | Metrics visualization          |
| 📦 cAdvisor                  | Container-level metrics        |
| 🖥️ Node Exporter            | Linux host metrics             |
| 🔎 PromQL                    | Metrics querying               |
| 🚨 Prometheus Alerting Rules | Threshold-based alerts         |
| 📝 YAML                      | Configuration                  |
| 🔧 REST API                  | Validation and automation      |

---

# 📋 Prerequisites

Before starting, you should have:

* Basic Linux command-line knowledge.
* Experience navigating and editing files.
* Basic Docker knowledge.
* Understanding of:

  * Images
  * Containers
  * Volumes
  * Networks
* Basic YAML knowledge.
* Access to an Ubuntu AWS EC2 instance.
* Internet connectivity for downloading packages and container images.

---

# 🖥️ Lab Environment

The lab uses a dedicated **AWS EC2 Ubuntu instance provided through Al Nafi**.

The machine starts with a base Ubuntu installation.

You will install:

```text
Docker Engine
Docker Compose
Prometheus
Grafana
cAdvisor
Node Exporter
```

---

# 📁 Recommended Project Structure

Create a dedicated project directory:

```bash
mkdir -p ~/docker-grafana-monitoring
cd ~/docker-grafana-monitoring
```

Recommended structure:

```text
docker-grafana-monitoring/
│
├── docker-compose.yml
│
├── prometheus/
│   ├── prometheus.yml
│   └── rules/
│       └── alerts.yml
│
├── grafana/
│   ├── provisioning/
│   │   ├── datasources/
│   │   │   └── prometheus.yml
│   │   └── dashboards/
│   │       └── dashboards.yml
│   │
│   └── dashboards/
│       ├── container-performance.json
│       └── host-resources.json
│
└── workloads/
    └── docker-compose.yml
```

---

# 🛠️ Task 1 — Install and Verify the Full Tool Stack

## 🔹 Requirement 1 — Install Docker Engine

Update the Ubuntu package index:

```bash
sudo apt update
sudo apt upgrade -y
```

Install prerequisite packages:

```bash
sudo apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```

Create the Docker keyring directory:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

Add Docker's official GPG key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

Set the correct permissions:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

Add the Docker repository:

```bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update repositories:

```bash
sudo apt update
```

Install Docker Engine:

```bash
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

---

## 🔹 Verify Docker

Check the Docker version:

```bash
docker --version
```

Check the service:

```bash
sudo systemctl status docker
```

Enable Docker at boot:

```bash
sudo systemctl enable docker
```

Test Docker:

```bash
sudo docker run --rm hello-world
```

Expected output includes:

```text
Hello from Docker!
```

---

# 👤 Configure Docker Without sudo

Add the current user to the Docker group:

```bash
sudo usermod -aG docker "$USER"
```

Refresh group membership without logging out:

```bash
newgrp docker
```

Verify:

```bash
docker ps
```

The command should execute without:

```text
permission denied
```

---

# 📈 Enable Docker Prometheus Metrics

Create the Docker daemon configuration:

```bash
sudo nano /etc/docker/daemon.json
```

Use:

```json
{
  "experimental": true,
  "metrics-addr": "0.0.0.0:9323"
}
```

Validate the JSON:

```bash
python3 -m json.tool /etc/docker/daemon.json
```

Restart Docker:

```bash
sudo systemctl restart docker
```

Verify the service:

```bash
sudo systemctl status docker
```

Test the metrics endpoint:

```bash
curl http://localhost:9323/metrics
```

You should see metrics containing names such as:

```text
engine_daemon_container_actions_seconds_sum
engine_daemon_container_actions_seconds_count
engine_daemon_engine_cpus_cpus
```

---

## 🧪 Docker Metrics Validation

Run:

```bash
curl -s http://localhost:9323/metrics | grep '^engine_' | head
```

Successful output confirms that Docker is exposing Prometheus-compatible metrics.

---

# 🧩 Troubleshooting Docker Installation

### ❌ Error: `docker-ce has no installation candidate`

Check the repository:

```bash
cat /etc/apt/sources.list.d/docker.list
```

The file should contain one valid repository line.

Check the Ubuntu codename:

```bash
. /etc/os-release
echo "$VERSION_CODENAME"
```

Then refresh:

```bash
sudo apt update
```

---

### ❌ Error: Port 9323 Connection Refused

Check the configuration:

```bash
cat /etc/docker/daemon.json
```

Validate:

```bash
python3 -m json.tool /etc/docker/daemon.json
```

Restart Docker:

```bash
sudo systemctl restart docker
```

Check logs:

```bash
sudo journalctl -u docker --no-pager -n 100
```

Test again:

```bash
curl http://localhost:9323/metrics
```

---

# 🐳 Verify Docker Compose

Check Compose:

```bash
docker compose version
```

Expected format:

```text
Docker Compose version v2.x.x
```

Verify Docker access:

```bash
docker ps
```

---

# 📊 Task 2 — Deploy the Observability Stack

## 🔹 Create Prometheus Configuration

Create:

```bash
mkdir -p ~/docker-grafana-monitoring/prometheus/rules
nano ~/docker-grafana-monitoring/prometheus/prometheus.yml
```

Example configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: "prometheus"
    static_configs:
      - targets:
          - "prometheus:9090"

  - job_name: "docker"
    scrape_interval: 5s
    static_configs:
      - targets:
          - "host.docker.internal:9323"

  - job_name: "cadvisor"
    scrape_interval: 5s
    static_configs:
      - targets:
          - "cadvisor:8080"

  - job_name: "node-exporter"
    static_configs:
      - targets:
          - "node-exporter:9100"

rule_files:
  - "/etc/prometheus/rules/*.yml"
```

---

# 🌐 Configure Grafana Provisioning

Create the directories:

```bash
mkdir -p ~/docker-grafana-monitoring/grafana/provisioning/datasources
mkdir -p ~/docker-grafana-monitoring/grafana/provisioning/dashboards
mkdir -p ~/docker-grafana-monitoring/grafana/dashboards
```

Create the Prometheus data source configuration:

```bash
nano ~/docker-grafana-monitoring/grafana/provisioning/datasources/prometheus.yml
```

Example:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
```

---

# 🐳 Create the Monitoring Compose File

Create:

```bash
nano ~/docker-grafana-monitoring/docker-compose.yml
```

Example structure:

```yaml
services:

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./prometheus/rules:/etc/prometheus/rules:ro
    extra_hosts:
      - "host.docker.internal:host-gateway"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning:ro
      - ./grafana/dashboards:/var/lib/grafana/dashboards:ro
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on:
      - prometheus
    restart: unless-stopped

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - "--path.procfs=/host/proc"
      - "--path.sysfs=/host/sys"
      - "--path.rootfs=/rootfs"
    restart: unless-stopped

volumes:
  grafana-data:
```

> **Production note:** Pin tested image versions instead of using `latest` when implementing this architecture in a real environment.

---

# 🚀 Start the Monitoring Stack

Navigate to the project:

```bash
cd ~/docker-grafana-monitoring
```

Validate Compose:

```bash
docker compose config
```

Start the services:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

---

# 🔎 Verify Prometheus

Open:

```text
http://<EC2-IP>:9090
```

Check targets:

```text
http://<EC2-IP>:9090/targets
```

Or:

```bash
curl -s http://localhost:9090/api/v1/targets
```

Within approximately 60 seconds, the targets should report:

```json
"health": "up"
```

Expected targets:

```text
prometheus
docker
cadvisor
node-exporter
```

---

# 📦 Verify cAdvisor Container Metrics

Run:

```bash
curl -s \
"http://localhost:9090/api/v1/query?query=count(container_last_seen{name!=''})"
```

A numeric result greater than zero confirms cAdvisor is providing container metrics.

Prometheus expression:

```promql
count(container_last_seen{name!="“})
```

Use the correct PromQL expression:

```promql
count(container_last_seen{name!=""})
```

---

# 🧪 Task 2 — Create Workload Containers

Create the workload directory:

```bash
mkdir -p ~/docker-grafana-monitoring/workloads
```

Create:

```bash
nano ~/docker-grafana-monitoring/workloads/docker-compose.yml
```

Example:

```yaml
services:

  web:
    image: nginx:alpine
    container_name: workload-web
    restart: unless-stopped

  redis:
    image: redis:alpine
    container_name: workload-redis
    restart: unless-stopped

  busybox:
    image: busybox:latest
    container_name: workload-busybox
    command: >
      sh -c "while true; do
      echo monitoring workload;
      sleep 5;
      done"
    restart: unless-stopped
```

Start the workloads:

```bash
cd ~/docker-grafana-monitoring/workloads
docker compose up -d
```

Verify:

```bash
docker compose ps
```

All three workload containers should be running.

---

# 📊 Verify Total Container Metrics

Query Prometheus:

```bash
curl -s \
"http://localhost:9090/api/v1/query?query=count(container_last_seen{name!=''})"
```

The expected result should be **at least 7**:

```text
4 monitoring containers
+
3 workload containers
=
7 containers
```

---

# 🎨 Task 3 — Grafana Dashboards

## 🔹 Configure Dashboard Provider

Create:

```bash
nano ~/docker-grafana-monitoring/grafana/provisioning/dashboards/dashboards.yml
```

Use:

```yaml
apiVersion: 1

providers:
  - name: "Docker Monitoring"
    orgId: 1
    folder: ""
    type: file
    disableDeletion: false
    editable: false
    options:
      path: /var/lib/grafana/dashboards
      foldersFromFilesStructure: false
```

Grafana will automatically load dashboards from the mounted directory.

---

# 📦 Dashboard 1 — Container Performance

Create:

```bash
nano ~/docker-grafana-monitoring/grafana/dashboards/container-performance.json
```

The dashboard should contain at least these panels:

### Panel 1 — Running Containers

PromQL:

```promql
count(container_last_seen{name!=""})
```

Panel type:

```text
Stat
```

---

### Panel 2 — Container CPU Usage

PromQL:

```promql
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100
```

Panel type:

```text
Time series
```

---

### Panel 3 — Container Memory Usage

PromQL:

```promql
container_memory_usage_bytes{name!=""} / 1024 / 1024
```

Panel type:

```text
Time series
```

Unit:

```text
MiB
```

---

### Panel 4 — Container Network Receive

PromQL:

```promql
rate(container_network_receive_bytes_total{name!=""}[5m])
```

---

### Panel 5 — Container Network Transmit

PromQL:

```promql
rate(container_network_transmit_bytes_total{name!=""}[5m])
```

Use:

```text
Bytes/sec
```

for the unit.

---

# 🖥️ Dashboard 2 — Host Resources

Create:

```bash
nano ~/docker-grafana-monitoring/grafana/dashboards/host-resources.json
```

Include at least three panels.

---

## CPU Usage

Example PromQL:

```promql
100 - (
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

---

## Memory Usage

```promql
(
  1 -
  node_memory_MemAvailable_bytes /
  node_memory_MemTotal_bytes
) * 100
```

---

## Root Filesystem Usage

```promql
(
  1 -
  node_filesystem_avail_bytes{
    mountpoint="/",
    fstype!~"tmpfs|overlay"
  }
  /
  node_filesystem_size_bytes{
    mountpoint="/",
    fstype!~"tmpfs|overlay"
  }
) * 100
```

---

# 🔄 Restart Grafana

Restart the Grafana service:

```bash
cd ~/docker-grafana-monitoring
docker compose restart grafana
```

Check:

```bash
docker compose ps grafana
```

Query the Grafana API:

```bash
curl -s \
-u admin:admin \
"http://localhost:3000/api/search?type=dash-db"
```

Expected dashboards:

```text
Container Performance
Host Resources
```

The dashboards should appear automatically without manually importing JSON files.

---

# 🔌 Verify Grafana Data Source

Run:

```bash
curl -s \
-u admin:admin \
http://localhost:3000/api/datasources
```

Look for:

```json
{
  "name": "Prometheus",
  "isDefault": true
}
```

This confirms that Grafana was provisioned automatically.

---

# 🚨 Task 3 — Prometheus Alerting

Create the alert directory:

```bash
mkdir -p ~/docker-grafana-monitoring/prometheus/rules
```

Create:

```bash
nano ~/docker-grafana-monitoring/prometheus/rules/alerts.yml
```

Use:

```yaml
groups:
  - name: container-alerts

    rules:

      - alert: HighContainerCPU
        expr: |
          rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100 > 50
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High container CPU usage"
          description: "Container {{ $labels.name }} has CPU usage above 50%."

      - alert: HighContainerMemory
        expr: |
          container_memory_usage_bytes{name!=""} > 200 * 1024 * 1024
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High container memory usage"
          description: "Container {{ $labels.name }} is using more than 200 MiB."
```

---

# 🔍 Validate Prometheus Rules

Restart Prometheus:

```bash
docker compose restart prometheus
```

Check loaded rules:

```bash
curl -s http://localhost:9090/api/v1/rules
```

The response should contain:

```text
HighContainerCPU
HighContainerMemory
```

Both should have:

```json
"type": "alerting"
```

---

# 🔥 Generate CPU Load

Use an existing workload container rather than launching a new container.

For example:

```bash
docker exec -d workload-busybox sh -c \
"while true; do :; done"
```

Check the process:

```bash
docker exec workload-busybox ps
```

Monitor container CPU:

```bash
docker stats workload-busybox
```

---

# 🚨 Verify HighContainerCPU Alert

Wait for the configured evaluation period.

Query:

```bash
curl -s http://localhost:9090/api/v1/alerts
```

Search for:

```text
HighContainerCPU
```

The expected state is:

```json
"state": "firing"
```

You can also inspect the Prometheus UI:

```text
http://<EC2-IP>:9090/alerts
```

---

# 🔎 Useful PromQL Queries

## Running Containers

```promql
count(container_last_seen{name!=""})
```

## CPU Usage

```promql
rate(container_cpu_usage_seconds_total{name!=""}[5m]) * 100
```

## Memory Usage

```promql
container_memory_usage_bytes{name!=""} / 1024 / 1024
```

## Network Receive

```promql
rate(container_network_receive_bytes_total{name!=""}[5m])
```

## Network Transmit

```promql
rate(container_network_transmit_bytes_total{name!=""}[5m])
```

## Host CPU

```promql
100 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100
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

---

# 🧪 Validation Checklist

## Docker

* [ ] Docker Engine installed.
* [ ] Docker service running.
* [ ] User added to `docker` group.
* [ ] `docker ps` works without sudo.
* [ ] `docker run --rm hello-world` succeeds.
* [ ] Docker metrics endpoint responds on port `9323`.

## Docker Compose

* [ ] `docker compose version` works.
* [ ] Compose plugin is available.
* [ ] Monitoring Compose file validates successfully.

## Prometheus

* [ ] Prometheus starts successfully.
* [ ] Port `9090` is accessible.
* [ ] Prometheus target is `UP`.
* [ ] Docker target is `UP`.
* [ ] cAdvisor target is `UP`.
* [ ] Node Exporter target is `UP`.
* [ ] Container metrics are available.

## Grafana

* [ ] Grafana starts successfully.
* [ ] Port `3000` is accessible.
* [ ] Prometheus is automatically provisioned.
* [ ] Prometheus is the default data source.
* [ ] Container Performance dashboard loads automatically.
* [ ] Host Resources dashboard loads automatically.
* [ ] Dashboard panels display real data.

## Workloads

* [ ] Three workload containers are running.
* [ ] cAdvisor detects the workload containers.
* [ ] Container count reaches at least seven.

## Alerting

* [ ] `HighContainerCPU` is loaded.
* [ ] `HighContainerMemory` is loaded.
* [ ] CPU load is generated inside an existing container.
* [ ] `HighContainerCPU` reaches `firing`.
* [ ] Prometheus API reports the firing alert.

---

# 📈 Expected Outcomes

After completing this lab, you should have:

```text
                 ┌─────────────────────┐
                 │      Grafana        │
                 │                     │
                 │ Container Dashboard │
                 │ Host Dashboard      │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │     Prometheus      │
                 │                     │
                 │ Metrics + Rules     │
                 └──────────┬──────────┘
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
         cAdvisor      Node Exporter   Docker API
             │              │              │
             ▼              ▼              ▼
       Containers       Ubuntu Host    Docker Engine
```

The final environment provides:

* 📦 Container observability
* 🖥️ Host observability
* 📊 Automated dashboards
* 🔎 PromQL-based analysis
* 🚨 Automated alerting
* 🔄 Reproducible provisioning
* 📝 Configuration-as-code
* 🐳 Docker-native monitoring

---

# 🧹 Cleanup

Stop the workload containers:

```bash
cd ~/docker-grafana-monitoring/workloads
docker compose down
```

Stop the monitoring stack:

```bash
cd ~/docker-grafana-monitoring
docker compose down
```

To remove the Grafana persistent volume:

```bash
docker compose down -v
```

Verify remaining containers:

```bash
docker ps -a
```

---

# 🛡️ Production Considerations

This lab intentionally focuses on learning and reproducibility. For production deployment, consider:

* 🔐 Use strong Grafana credentials.
* 🔒 Avoid exposing Prometheus and Grafana directly to the public internet.
* 🌐 Place Grafana behind HTTPS and a reverse proxy.
* 📌 Pin container image versions.
* 💾 Configure durable Prometheus storage.
* 💾 Back up Grafana configuration and dashboards.
* 🚨 Add Alertmanager for notification routing.
* 📧 Integrate alerts with email, Slack, PagerDuty, or another notification system.
* 🔑 Restrict access to Docker's metrics endpoint where appropriate.
* 📊 Add recording rules for expensive PromQL queries.
* 🏷️ Use consistent container labels and metric naming.
* 🔍 Monitor Prometheus itself.
* 🧱 Apply appropriate firewall and security-group rules.

---

# 🔗 Useful Documentation

* [Docker Engine Installation](https://docs.docker.com/engine/install/ubuntu/)
* [Docker Prometheus Metrics](https://docs.docker.com/config/daemon/prometheus/)
* [Docker Compose Documentation](https://docs.docker.com/compose/)
* [Prometheus Documentation](https://prometheus.io/docs/)
* [Grafana Documentation](https://grafana.com/docs/grafana/latest/)
* [cAdvisor](https://github.com/google/cadvisor)
* [Node Exporter](https://github.com/prometheus/node_exporter)

---

# 🧠 Key Learning Points

### Docker Metrics

Docker Engine can expose internal runtime metrics in Prometheus format, allowing Prometheus to monitor Docker itself.

### cAdvisor

cAdvisor provides detailed container-level information including:

```text
CPU
Memory
Network
Filesystem
Container lifecycle
```

### Node Exporter

Node Exporter exposes operating-system-level metrics such as:

```text
CPU
Memory
Disk
Filesystem
Network
Load
```

### Prometheus

Prometheus periodically scrapes exporters and stores the resulting time-series data.

### Grafana

Grafana transforms Prometheus metrics into dashboards, graphs, statistics, and operational views.

### Provisioning as Code

Grafana data sources and dashboards can be loaded from files, eliminating repetitive manual configuration and making monitoring infrastructure easier to version and reproduce.

---

# 🏆 Conclusion

You have designed and deployed a complete **Docker container observability platform** from a blank Ubuntu system.

The architecture combines:

```text
Docker
   +
cAdvisor
   +
Node Exporter
   +
Prometheus
   +
Grafana
   +
Prometheus Alerting
```

The result is a reproducible monitoring environment capable of correlating **container performance with host-level resource utilization**.

The configuration is managed through files rather than manual UI operations, making it suitable for source control and automated deployment.

### 🚀 Recommended Next Step

Extend the platform by adding **Prometheus Alertmanager** and configure notification routing to services such as Slack or email. This transforms the monitoring stack from a visualization platform into a more complete operational alerting system.

---

## ⭐ Lab Skills Demonstrated

```text
🐧 Linux Administration
🐳 Docker
🧩 Docker Compose
📈 Prometheus
📊 Grafana
📦 cAdvisor
🖥️ Node Exporter
🔎 PromQL
🚨 Monitoring & Alerting
📝 YAML Configuration
⚙️ Infrastructure as Code Principles
☁️ AWS EC2
🔐 DevOps Observability
```

**Built as a hands-on Cloud DevOps & Linux Administration observability lab.**
