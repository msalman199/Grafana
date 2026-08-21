# 📊 Templating Dashboards in Grafana

> **Hands-on Grafana Lab | Dynamic Dashboards | Prometheus | Docker | PromQL**

This lab demonstrates how to build **dynamic, reusable, and user-driven Grafana dashboards** using dashboard variables and templating. You will deploy a complete monitoring stack with Grafana, Prometheus, Node Exporter, and sample applications, then create variables that allow users to dynamically filter environments, regions, datacenters, jobs, instances, services, and metrics.

---

## 🎯 Lab Objectives

By the end of this lab, you will be able to:

* Understand dashboard templating in Grafana
* Create and configure template variables
* Implement Query, Custom, Constant, Textbox, and Interval variables
* Build dependent and chained variables
* Create dropdown and multi-select filters
* Build panels that dynamically respond to user selections
* Use regex-based filtering
* Create repeating panels
* Configure dashboard links and annotations
* Optimize variable queries for better performance
* Apply Grafana templating best practices
* Export and validate dashboard configurations

---

## 🧰 Technology Stack

| Technology            | Purpose                                |
| --------------------- | -------------------------------------- |
| 🟠 **Grafana**        | Dashboard visualization and templating |
| 🔵 **Prometheus**     | Metrics collection and PromQL queries  |
| 🐳 **Docker**         | Container runtime                      |
| 🧩 **Docker Compose** | Multi-container orchestration          |
| 🖥️ **Node Exporter** | Linux system metrics                   |
| 🐍 **Python**         | Sample metrics application             |
| 📈 **PromQL**         | Prometheus query language              |
| 🐧 **Ubuntu Linux**   | Lab operating system                   |

---

## 📋 Prerequisites

Before starting, you should have:

* Basic Linux command-line knowledge
* Familiarity with Grafana dashboards and panels
* Basic PromQL knowledge
* Understanding of time-series data
* Basic system monitoring knowledge
* Access to the Al Nafi Linux cloud lab environment

---

# 🏗️ Lab Architecture

The completed environment uses the following architecture:

```text
                    ┌─────────────────────┐
                    │       Browser       │
                    │   Grafana :3000     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      Grafana        │
                    │ Dashboard Templates │
                    │     Variables       │
                    └──────────┬──────────┘
                               │
                         PromQL Queries
                               │
                               ▼
                    ┌─────────────────────┐
                    │     Prometheus      │
                    │      :9090          │
                    └─────┬──────┬────────┘
                          │      │
             ┌────────────┘      └─────────────┐
             ▼                                  ▼
    ┌─────────────────┐                ┌─────────────────┐
    │  Node Exporter  │                │ Sample Apps     │
    │      :9100      │                │  :8080          │
    └─────────────────┘                └─────────────────┘
```

---

# 🚀 Task 1: Environment Setup

## 1.1 Update Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y wget curl software-properties-common apt-transport-https
```

---

## 1.2 Install Docker

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

sudo systemctl start docker
sudo systemctl enable docker

sudo usermod -aG docker $USER
```

Reload the Docker group:

```bash
newgrp docker
```

Verify:

```bash
docker --version
```

---

## 1.3 Install Docker Compose

```bash
sudo curl -L \
"https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
-o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
```

---

## 1.4 Create Lab Directory

```bash
mkdir -p ~/grafana-templating-lab/{prometheus,grafana,node-exporter,sample-apps}

cd ~/grafana-templating-lab
```

Expected structure:

```text
grafana-templating-lab/
├── prometheus/
├── grafana/
├── node-exporter/
├── sample-apps/
└── docker-compose.yml
```

---

# 🔵 Task 2: Deploy Prometheus

Create the Prometheus configuration:

```bash
cat > prometheus/prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
        labels:
          environment: 'production'
          region: 'us-east-1'
          datacenter: 'dc1'

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          environment: 'production'
          region: 'us-east-1'
          datacenter: 'dc1'
          server_type: 'web'

      - targets: ['node-exporter:9100']
        labels:
          environment: 'staging'
          region: 'us-west-2'
          datacenter: 'dc2'
          server_type: 'database'

  - job_name: 'sample-app-prod'
    static_configs:
      - targets: ['sample-app-1:8080', 'sample-app-2:8080']
        labels:
          environment: 'production'
          region: 'us-east-1'
          datacenter: 'dc1'
          app: 'web-service'

  - job_name: 'sample-app-staging'
    static_configs:
      - targets: ['sample-app-3:8080']
        labels:
          environment: 'staging'
          region: 'us-west-2'
          datacenter: 'dc2'
          app: 'web-service'
EOF
```

---

# 🐍 Task 3: Create Sample Metrics Application

Create the Python application:

```bash
cat > sample-apps/app.py << 'EOF'
from prometheus_client import start_http_server, Counter, Histogram, Gauge
import random
import time
import os

REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['endpoint']
)

ACTIVE_CONNECTIONS = Gauge(
    'active_connections',
    'Active connections'
)

CPU_USAGE = Gauge(
    'cpu_usage_percent',
    'CPU usage percentage'
)

MEMORY_USAGE = Gauge(
    'memory_usage_bytes',
    'Memory usage in bytes'
)

def simulate_metrics():
    endpoints = [
        '/api/users',
        '/api/orders',
        '/api/products',
        '/health'
    ]

    methods = ['GET', 'POST', 'PUT', 'DELETE']
    statuses = ['200', '201', '400', '404', '500']

    while True:
        endpoint = random.choice(endpoints)
        method = random.choice(methods)
        status = random.choice(statuses)

        REQUEST_COUNT.labels(
            method=method,
            endpoint=endpoint,
            status=status
        ).inc()

        latency = random.uniform(0.1, 2.0)

        REQUEST_LATENCY.labels(
            endpoint=endpoint
        ).observe(latency)

        ACTIVE_CONNECTIONS.set(
            random.randint(10, 100)
        )

        CPU_USAGE.set(
            random.uniform(20, 80)
        )

        MEMORY_USAGE.set(
            random.randint(1000000000, 4000000000)
        )

        time.sleep(random.uniform(1, 5))

if __name__ == '__main__':
    port = int(os.environ.get('METRICS_PORT', 8080))

    start_http_server(port)

    print(f"Metrics server started on port {port}")

    simulate_metrics()
EOF
```

Create the Dockerfile:

```bash
cat > sample-apps/Dockerfile << 'EOF'
FROM python:3.9-slim

WORKDIR /app

RUN pip install prometheus_client

COPY app.py .

EXPOSE 8080

CMD ["python", "app.py"]
EOF
```

---

# 🐳 Task 4: Create Docker Compose Stack

Create `docker-compose.yml`:

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-storage:/var/lib/grafana
    networks:
      - monitoring

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
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    networks:
      - monitoring

  sample-app-1:
    build: ./sample-apps
    container_name: sample-app-1
    environment:
      - METRICS_PORT=8080
    networks:
      - monitoring

  sample-app-2:
    build: ./sample-apps
    container_name: sample-app-2
    environment:
      - METRICS_PORT=8080
    networks:
      - monitoring

  sample-app-3:
    build: ./sample-apps
    container_name: sample-app-3
    environment:
      - METRICS_PORT=8080
    networks:
      - monitoring

volumes:
  grafana-storage:

networks:
  monitoring:
    driver: bridge
EOF
```

Deploy:

```bash
docker-compose up -d
```

Check services:

```bash
docker-compose ps
```

Verify Prometheus:

```bash
curl http://localhost:9090/api/v1/targets
```

Verify Grafana:

```bash
curl http://localhost:3000
```

---

# 📈 Task 5: Configure Grafana

Open:

```text
http://localhost:3000
```

Login:

```text
Username: admin
Password: admin123
```

Add Prometheus:

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
Name: Prometheus
URL: http://prometheus:9090
Access: Server
```

Click:

**Save & Test**

A successful connection confirms that Grafana can communicate with Prometheus.

---

# 🎛️ Task 6: Create Template Variables

Navigate to:

```text
Dashboard
 → Dashboard Settings
 → Variables
 → Add variable
```

## 6.1 Environment Variable

```text
Name: environment
Type: Query
Label: Environment
Data source: Prometheus
```

Query:

```promql
label_values(up, environment)
```

Enable:

```text
Multi-value: Yes
Include All: Yes
Custom all value: .*
```

---

## 6.2 Region Variable

```text
Name: region
Type: Query
Label: Region
Data source: Prometheus
```

Query:

```promql
label_values(up{environment=~"$environment"}, region)
```

Enable multi-select and **All**.

---

## 6.3 Datacenter Variable

Query:

```promql
label_values(
  up{
    environment=~"$environment",
    region=~"$region"
  },
  datacenter
)
```

---

## 6.4 Job Variable

Query:

```promql
label_values(
  up{
    environment=~"$environment",
    region=~"$region",
    datacenter=~"$datacenter"
  },
  job
)
```

---

# 🧩 Task 7: Custom and Constant Variables

## Custom Time Range

Create:

```text
Name: time_range
Type: Custom
Label: Time Range
```

Values:

```text
5m : 5 minutes,
15m : 15 minutes,
1h : 1 hour,
6h : 6 hours,
24h : 24 hours
```

---

## Constant Variable

Create:

```text
Name: dashboard_version
Type: Constant
Value: v1.0.0
```

This can be used for dashboard metadata and version identification.

---

# 📊 Task 8: Build Templated Panels

## 8.1 System Overview

Panel title:

```text
System Overview - $environment ($region)
```

PromQL:

```promql
up{
  environment=~"$environment",
  region=~"$region",
  datacenter=~"$datacenter",
  job=~"$job"
}
```

Panel type:

```text
Stat
```

---

## 8.2 CPU Usage

PromQL:

```promql
avg by (environment, region, datacenter) (
  100 - (
    avg by (
      instance,
      environment,
      region,
      datacenter
    ) (
      rate(
        node_cpu_seconds_total{
          mode="idle",
          environment=~"$environment",
          region=~"$region",
          datacenter=~"$datacenter"
        }[5m]
      )
    ) * 100
  )
)
```

Panel:

```text
Type: Time series
Unit: Percent (0-100)
```

Legend:

```text
{{environment}} - {{region}} - {{datacenter}}
```

---

# 🧠 Task 9: Memory Usage

PromQL:

```promql
(
  node_memory_MemTotal_bytes{
    environment=~"$environment",
    region=~"$region",
    datacenter=~"$datacenter"
  }
  -
  node_memory_MemAvailable_bytes{
    environment=~"$environment",
    region=~"$region",
    datacenter=~"$datacenter"
  }
)
/
node_memory_MemTotal_bytes{
  environment=~"$environment",
  region=~"$region",
  datacenter=~"$datacenter"
}
* 100
```

Configure:

```text
Panel Type: Gauge
Unit: Percent (0-100)
Min: 0
Max: 100
```

Recommended thresholds:

```text
0-70     → Normal
70-85    → Warning
85-100   → Critical
```

---

# 🌐 Task 10: HTTP Request Rate

PromQL:

```promql
sum by (
  environment,
  region,
  method,
  status
) (
  rate(
    http_requests_total{
      environment=~"$environment",
      region=~"$region",
      datacenter=~"$datacenter"
    }[$time_range]
  )
)
```

Configure:

```text
Panel Type: Time series
Unit: Requests/sec
```

Legend:

```text
{{environment}} - {{method}} {{status}}
```

---

# 📋 Task 11: Dynamic Service Status Table

PromQL:

```promql
up{
  environment=~"$environment",
  region=~"$region",
  datacenter=~"$datacenter",
  job=~"$job"
}
```

Panel:

```text
Type: Table
```

Use **Organize fields** to display:

```text
Time
job
environment
region
datacenter
Value
```

Create value mappings:

```text
1 → Up
0 → Down
```

This creates a user-driven service status table.

---

# 🔗 Task 12: Chained Variables

Create an `instance` variable.

Query:

```promql
label_values(
  up{
    environment=~"$environment",
    region=~"$region",
    datacenter=~"$datacenter",
    job=~"$job"
  },
  instance
)
```

Enable:

```text
Multi-value: Yes
Include All: Yes
Custom all value: .*
```

Now the selection chain becomes:

```text
Environment
      ↓
   Region
      ↓
 Datacenter
      ↓
     Job
      ↓
  Instance
```

This is known as **variable chaining** or **cascading variables**.

---

# 🔍 Task 13: Regex-Based Variable

Create:

```text
Name: service_type
Type: Query
Label: Service Type
```

Query:

```promql
label_values(up, job)
```

Regex:

```regex
/(.*-app.*|node-exporter)/
```

This limits the variable to matching services.

---

# ⏱️ Task 14: Interval Variable

Create:

```text
Name: interval
Type: Interval
Label: Interval
```

Values:

```text
1m,5m,10m,30m,1h
```

Enable:

```text
Auto option: Yes
Min interval: 1m
```

Update the HTTP query:

```promql
sum by (
  environment,
  region,
  method,
  status
) (
  rate(
    http_requests_total{
      environment=~"$environment",
      region=~"$region",
      datacenter=~"$datacenter",
      job=~"$service_type",
      instance=~"$instance"
    }[$interval]
  )
)
```

---

# 🔤 Task 15: Textbox Filtering

Create:

```text
Name: custom_filter
Type: Textbox
Label: Custom Filter (Regex)
Default value: .*
```

Example query:

```promql
{
  __name__=~"$custom_filter",
  environment=~"$environment",
  region=~"$region",
  datacenter=~"$datacenter"
}
```

Example filters:

```text
http_.*
node_.*
.*memory.*
```

This allows users to enter their own metric filters.

---

# 🔗 Task 16: Dashboard Links

Navigate to:

```text
Dashboard Settings
 → Links
 → Add link
```

Configure:

```text
Title: Environment Overview
Type: Dashboard
Tooltip: View detailed environment metrics
Icon: External link
```

Example URL:

```text
/d/dashboard-id/environment-overview?var-environment=$environment&var-region=$region
```

Dashboard links allow users to move between related dashboards while preserving variable selections.

---

# 📦 Task 17: Application Version Variable

Create:

```text
Name: app_version
Type: Query
Label: App Version
Data source: Prometheus
```

Query:

```promql
label_values(
  http_requests_total{
    environment=~"$environment",
    job=~"$service_type"
  },
  version
)
```

Enable multi-select and **All**.

> **Note:** The sample application in this lab does not currently expose a `version` label. If the query returns no values, add a `version` label to the metric definition before using this variable.

---

# 🔄 Task 18: Repeating Panels

Create a panel using:

```promql
rate(
  http_requests_total{
    environment=~"$environment",
    region=~"$region"
  }[5m]
)
```

Configure:

```text
Repeat by variable: environment
Max per row: 2
```

Grafana will automatically create one panel for each selected environment.

Conceptually:

```text
┌───────────────────┐ ┌───────────────────┐
│ Production        │ │ Staging           │
│ HTTP Metrics      │ │ HTTP Metrics       │
└───────────────────┘ └───────────────────┘
```

---

# 📝 Task 19: Dashboard Annotations

Create an annotation:

```text
Name: Deployments
Data source: Prometheus
```

Query:

```promql
increase(http_requests_total[1m]) > 100
```

Title:

```text
High traffic detected
```

Text:

```text
Traffic spike in {{environment}}
```

Tags:

```text
deployment
traffic
```

Annotations help correlate events with metric behavior.

---

# ⏰ Task 20: Time Range Controls

Create:

```text
Name: time_from
Type: Textbox
Default: now-1h
```

Create another:

```text
Name: time_to
Type: Textbox
Default: now
```

These can be used as dashboard controls where supported by the dashboard/query configuration.

---

# 💾 Task 21: Export Dashboard JSON

Open:

```text
Dashboard Settings
 → JSON Model
```

Copy the dashboard JSON and save it:

```bash
nano ~/grafana-templating-lab/dashboard-backup.json
```

Or:

```bash
cat > ~/grafana-templating-lab/dashboard-backup.json << 'EOF'
# Paste exported Grafana dashboard JSON here
EOF
```

Verify:

```bash
ls -lh ~/grafana-templating-lab/dashboard-backup.json
```

Dashboard JSON provides a portable backup of the dashboard configuration.

---

# ⚡ Task 22: Performance Optimization

Large dashboards can become slow when they contain many variables and expensive queries.

## Optimize Variable Queries

Instead of:

```promql
label_values(up, environment)
```

Consider:

```promql
label_values(up{job!=""}, environment)
```

Use specific label filters whenever possible.

---

## Recommended Optimization Practices

```text
✓ Use narrow query scopes
✓ Avoid unnecessarily expensive regex
✓ Reduce the number of variables
✓ Avoid excessive multi-select combinations
✓ Use reasonable dashboard time ranges
✓ Configure appropriate refresh intervals
✓ Use caching where supported
✓ Avoid unnecessary high-cardinality queries
```

---

# 📈 Task 23: Monitor Query Performance

Create a panel using:

```promql
prometheus_engine_query_duration_seconds{
  quantile="0.9"
}
```

Use this panel to observe Prometheus query execution performance.

---

# 🛡️ Task 24: Variable Validation

For the `custom_filter` textbox, use:

```regex
^[a-zA-Z0-9_.*]+$
```

This restricts user input to expected metric/regex characters.

Validation is especially useful when dashboards are shared with many users.

---

# 🔐 Task 25: Dashboard Permissions

Configure dashboard permissions according to user responsibilities.

| Role       | Capabilities                     |
| ---------- | -------------------------------- |
| 👁️ Viewer | View dashboard and use variables |
| ✏️ Editor  | Modify panels and variables      |
| 👑 Admin   | Full dashboard management        |

A good production design separates dashboard consumers from dashboard administrators.

---

# 🧪 Task 26: Test Variable Interactions

Test the dependency chain:

```text
Environment → Region → Datacenter → Job → Instance
```

Verify:

* Environment selections change Region options
* Region selections change Datacenter options
* Datacenter selections affect Job options
* Job selections affect Instance options
* Panels respond to variable changes
* Panel titles update dynamically
* Legends update correctly

---

# 🔀 Task 27: Test Multi-Select

Enable multi-select for `environment`.

Select:

```text
production
staging
```

Verify that the dashboard displays data for both environments.

For multi-value variables, use:

```promql
=~"$environment"
```

rather than:

```promql
="$environment"
```

The regex matcher allows multiple selected values to work correctly.

---

# 🔎 Task 28: Test Custom Filtering

Test:

```text
http_.*
```

Then:

```text
node_.*
```

Then:

```text
.*memory.*
```

Confirm that the custom-filtered panel changes according to the selected metric pattern.

---

# 🔬 Verification

## Prometheus Target Test

```bash
curl http://localhost:9090/api/v1/targets
```

---

## Prometheus Query Test

```bash
curl "http://localhost:9090/api/v1/query?query=up"
```

---

## Environment Label Test

```bash
curl "http://localhost:9090/api/v1/label/environment/values"
```

---

## Docker Service Test

```bash
docker-compose ps
```

---

## Container Resource Test

```bash
docker stats
```

---

## System Resource Test

```bash
htop
```

---

# 🧪 Validation Checklist

* [ ] Grafana is accessible on port `3000`
* [ ] Prometheus is accessible on port `9090`
* [ ] Node Exporter is running
* [ ] Sample applications are running
* [ ] Prometheus targets are available
* [ ] Grafana successfully connects to Prometheus
* [ ] Environment variable works
* [ ] Region variable is dependent on Environment
* [ ] Datacenter variable is dependent on Region
* [ ] Job variable works
* [ ] Instance variable works
* [ ] Multi-select works
* [ ] All option works
* [ ] Custom regex filtering works
* [ ] Interval variable changes query behavior
* [ ] Panels respond to variable changes
* [ ] Repeating panels work
* [ ] Dashboard links work
* [ ] Dashboard annotations work
* [ ] Dashboard JSON exports successfully
* [ ] Dashboard performance is acceptable

---

# 🛠️ Troubleshooting

## ❌ Issue 1: Variables Show "No Data"

Check Prometheus:

```bash
curl http://localhost:9090/api/v1/targets
```

Then verify:

```text
✓ Prometheus data source
✓ Variable query
✓ Label names
✓ Prometheus targets
✓ Query syntax
```

---

## ❌ Issue 2: Panels Do Not Update

Check that the variable name matches exactly.

Correct:

```promql
environment=~"$environment"
```

Also verify that the variable is actually referenced by the panel query.

Refresh the dashboard after changing variable configuration.

---

## ❌ Issue 3: Multi-Select Does Not Work

Use:

```promql
environment=~"$environment"
```

instead of:

```promql
environment="$environment"
```

Also verify:

```text
Multi-value: Enabled
Include All: Enabled
Custom All Value: .*
```

---

## ❌ Issue 4: Dashboard Is Slow

Investigate:

```text
• Number of variables
• Variable refresh frequency
• Query complexity
• Regex usage
• Dashboard time range
• Number of panels
• Prometheus query duration
```

Use:

```promql
prometheus_engine_query_duration_seconds{quantile="0.9"}
```

to help identify query performance issues.

---

# 🏆 Best Practices

## 1. Use Meaningful Variable Names

Good:

```text
environment
region
datacenter
instance
service_type
```

Avoid unclear names such as:

```text
var1
temp
x
```

---

## 2. Build Variables Hierarchically

A scalable dashboard can use:

```text
Environment
    ↓
Region
    ↓
Datacenter
    ↓
Job
    ↓
Instance
```

This prevents users from selecting irrelevant combinations.

---

## 3. Use Regex Correctly

For multi-select variables:

```promql
label=~"$variable"
```

For a single-value variable:

```promql
label="$variable"
```

---

## 4. Provide an "All" Option

For broad operational dashboards:

```text
Include All: Enabled
Custom All Value: .*
```

This lets users quickly switch from a focused view to a global view.

---

## 5. Avoid Excessive Variables

Variables are powerful, but too many variables can make dashboards:

* Difficult to understand
* Slow to load
* Expensive to query
* Difficult to maintain

Only expose filters that provide meaningful operational value.

---

# 🌍 Real-World Use Cases

Grafana templating is particularly useful for:

### ☁️ Cloud Monitoring

```text
Environment
   ↓
AWS Region
   ↓
Availability Zone
   ↓
Instance
```

### ☸️ Kubernetes Monitoring

```text
Cluster
   ↓
Namespace
   ↓
Workload
   ↓
Pod
   ↓
Container
```

### 🧩 Microservices

```text
Environment
   ↓
Region
   ↓
Service
   ↓
Instance
```

### 🏢 Enterprise Monitoring

A single dashboard can serve:

```text
Production
Staging
Development
```

without requiring three separate dashboard copies.

---

# 📚 Key Concepts Learned

| Concept              | Description                                |
| -------------------- | ------------------------------------------ |
| 🎛️ Query Variable   | Gets values dynamically from a data source |
| 🧾 Custom Variable   | Uses predefined values                     |
| 🔒 Constant Variable | Stores a fixed value                       |
| 🔤 Textbox Variable  | Allows user-entered values                 |
| ⏱️ Interval Variable | Controls query intervals                   |
| 🔗 Chained Variable  | Depends on another variable                |
| 🔍 Regex Variable    | Filters variable values                    |
| 🔄 Repeating Panel   | Creates panels dynamically                 |
| 📝 Annotation        | Marks events on dashboards                 |
| 🔗 Dashboard Link    | Connects related dashboards                |

---

# 🏁 Final Result

After completing this lab, you will have a reusable Grafana dashboard capable of dynamically filtering monitoring data through:

```text
┌─────────────────────────────────────────────┐
│        GRAFANA TEMPLATED DASHBOARD          │
├─────────────────────────────────────────────┤
│ Environment ▼ │ Region ▼ │ Datacenter ▼     │
│ Job ▼         │ Instance ▼ │ Interval ▼     │
├─────────────────────────────────────────────┤
│                                             │
│       System Availability                   │
│                                             │
├─────────────────────────────────────────────┤
│       CPU Usage                             │
│                                             │
├─────────────────────────────────────────────┤
│       Memory Usage                          │
│                                             │
├─────────────────────────────────────────────┤
│       HTTP Request Rate                     │
│                                             │
├─────────────────────────────────────────────┤
│       Service Status Table                  │
│                                             │
└─────────────────────────────────────────────┘
```

---

# 🎓 Conclusion

This lab demonstrates how Grafana templating transforms static dashboards into **dynamic monitoring platforms**.

You created multiple variable types, implemented chained filtering, configured multi-select controls, introduced regex filtering, built dynamic panels, configured repeating panels, added annotations and dashboard links, and learned how to optimize dashboard performance.

The major advantage is **reuse**. Instead of maintaining separate dashboards for every environment, region, application, or service, one templated dashboard can provide multiple operational views.

For example:

```text
One Dashboard
      │
      ├── Production
      │     ├── US-East
      │     └── US-West
      │
      ├── Staging
      │     ├── US-East
      │     └── US-West
      │
      └── Development
            ├── US-East
            └── US-West
```

This approach reduces dashboard duplication, improves consistency, simplifies maintenance, and gives operations teams a powerful way to explore metrics interactively.

> 🚀 **Final Outcome:** You have built a scalable, user-driven Grafana monitoring dashboard that can adapt to changing infrastructure, environments, regions, services, and operational requirements.
