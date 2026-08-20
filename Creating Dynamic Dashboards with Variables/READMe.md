# 📊 Creating Dynamic Dashboards with Variables

![Grafana](https://img.shields.io/badge/Grafana-Dashboard-orange?style=for-the-badge\&logo=grafana)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?style=for-the-badge\&logo=prometheus)
![Docker](https://img.shields.io/badge/Docker-Containers-2496ED?style=for-the-badge\&logo=docker)
![PromQL](https://img.shields.io/badge/PromQL-Query%20Language-red?style=for-the-badge)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-FCC624?style=for-the-badge\&logo=linux)

> 🚀 **Hands-on Grafana observability lab focused on dynamic dashboards, variables, cascading filters, PromQL, and interactive monitoring.**

---

## 📌 Overview

This lab demonstrates how to build **dynamic and reusable Grafana dashboards** using dashboard variables backed by Prometheus.

Instead of creating separate dashboards for every environment, region, service, or instance, Grafana variables allow users to dynamically change filters and immediately update multiple panels.

During this lab, you will build a complete monitoring environment using:

* 🐧 Ubuntu/Linux
* 🐳 Docker
* 📈 Prometheus
* 📊 Grafana
* 🖥️ Node Exporter
* 🔎 PromQL

You will create query, custom, constant, and textbox variables, implement multi-value selections, build cascading variables, use regular expressions, and dynamically update panel queries and titles.

---

## 🎯 Lab Objectives

By completing this lab, you will be able to:

* Understand Grafana dashboard variables and their benefits.
* Create query, custom, constant, and textbox variables.
* Configure refresh behavior for variables.
* Enable multi-value selections.
* Enable and use the **All** option.
* Use variables inside PromQL queries.
* Use variables in panel titles and descriptions.
* Build cascading variables.
* Implement variable filtering with regular expressions.
* Create hierarchical variable dependencies.
* Validate dashboard behavior.
* Troubleshoot variable-related issues.
* Export and import Grafana dashboards.
* Optimize dashboards for better performance.

---

## 🏗️ Architecture

```text
                    ┌──────────────────────┐
                    │      User / Browser  │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │       Grafana        │
                    │   Dynamic Dashboard  │
                    └──────────┬───────────┘
                               │ PromQL
                               ▼
                    ┌──────────────────────┐
                    │     Prometheus       │
                    │   Metrics Storage    │
                    └───────┬──────┬───────┘
                            │      │
             ┌──────────────┘      └──────────────┐
             ▼                                     ▼
      ┌─────────────┐                       ┌─────────────┐
      │ Node Exporter│                       │ Sample Apps │
      └─────────────┘                       └─────────────┘
```

---

## 🧰 Technology Stack

| Technology        | Purpose                        |
| ----------------- | ------------------------------ |
| 🐧 Ubuntu Linux   | Lab operating system           |
| 🐳 Docker         | Container runtime              |
| 📈 Prometheus     | Metrics collection and storage |
| 📊 Grafana        | Visualization and dashboarding |
| 🖥️ Node Exporter | System metrics                 |
| 🔎 PromQL         | Metrics querying               |
| 📦 Docker Compose | Multi-container orchestration  |

---

## 📋 Prerequisites

Before beginning, you should have:

* Basic Linux command-line knowledge.
* Familiarity with Grafana dashboards.
* Basic PromQL knowledge.
* Understanding of time-series metrics.
* Basic monitoring concepts.
* Access to an Al Nafi Linux cloud machine.

---

# 🚀 Task 1 — Environment Preparation

## 1.1 Update the System

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

Apply the Docker group without logging out:

```bash
newgrp docker
```

Verify:

```bash
docker --version
docker ps
```

---

## 1.3 Create the Project Structure

```bash
mkdir -p ~/grafana-variables-lab/{prometheus,grafana,data}
cd ~/grafana-variables-lab
```

Expected structure:

```text
grafana-variables-lab/
├── data/
├── grafana/
├── prometheus/
└── docker-compose.yml
```

---

# 📈 Task 2 — Configure Prometheus

Create the configuration file:

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
    scrape_interval: 5s

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
    scrape_interval: 5s

  - job_name: 'sample-app-1'
    static_configs:
      - targets: ['sample-app-1:8080']
    scrape_interval: 5s
    labels:
      environment: 'production'
      region: 'us-east-1'
      team: 'backend'

  - job_name: 'sample-app-2'
    static_configs:
      - targets: ['sample-app-2:8080']
    scrape_interval: 5s
    labels:
      environment: 'staging'
      region: 'us-west-2'
      team: 'frontend'

  - job_name: 'sample-app-3'
    static_configs:
      - targets: ['sample-app-3:8080']
    scrape_interval: 5s
    labels:
      environment: 'development'
      region: 'eu-west-1'
      team: 'backend'
EOF
```

---

# 🐳 Task 3 — Create Docker Compose Environment

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
      - prometheus_data:/prometheus
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
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
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
    image: prom/prometheus:latest
    container_name: sample-app-1
    ports:
      - "8081:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.listen-address=0.0.0.0:8080'
    networks:
      - monitoring

  sample-app-2:
    image: prom/prometheus:latest
    container_name: sample-app-2
    ports:
      - "8082:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.listen-address=0.0.0.0:8080'
    networks:
      - monitoring

  sample-app-3:
    image: prom/prometheus:latest
    container_name: sample-app-3
    ports:
      - "8083:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.listen-address=0.0.0.0:8080'
    networks:
      - monitoring

volumes:
  prometheus_data:
  grafana_data:

networks:
  monitoring:
    driver: bridge
EOF
```

Start the monitoring stack:

```bash
docker-compose up -d
```

Check containers:

```bash
docker-compose ps
```

Allow services time to initialize:

```bash
sleep 180
```

---

# 📊 Task 4 — Configure Grafana

Open:

```text
http://localhost:3000
```

Credentials:

```text
Username: admin
Password: admin123
```

Add Prometheus:

```text
Configuration
→ Data Sources
→ Add data source
→ Prometheus
```

Use:

```text
URL: http://prometheus:9090
```

Click:

```text
Save & Test
```

---

# 📌 Task 5 — Create the Dynamic Dashboard

Create:

```text
+ → Dashboard → Add new panel
```

Save the dashboard as:

```text
Dynamic Variables Dashboard
```

---

# 🔽 Task 6 — Create Grafana Variables

## Variable 1 — Job

Navigate to:

```text
Dashboard Settings → Variables → Add variable
```

Configure:

```text
Name: job
Type: Query
Label: Job
Data Source: Prometheus
Query: label_values(up, job)
Sort: Alphabetical (asc)
Multi-value: Enabled
Include All: Enabled
```

---

## Variable 2 — Environment

```text
Name: environment
Type: Query
Label: Environment
Data Source: Prometheus
Query: label_values(up, environment)
Sort: Alphabetical (asc)
Multi-value: Enabled
Include All: Enabled
```

---

## Variable 3 — Region

```text
Name: region
Type: Query
Label: Region
Data Source: Prometheus
Query: label_values(up, region)
Sort: Alphabetical (asc)
Multi-value: Enabled
Include All: Enabled
```

---

## Variable 4 — Cascading Instance

```text
Name: instance
Type: Query
Label: Instance
Data Source: Prometheus
Query: label_values(up{job=~"$job"}, instance)
Sort: Alphabetical (asc)
Multi-value: Enabled
Include All: Enabled
```

This creates a dependency:

```text
Job Selection
      ↓
Instance Options
```

Changing the selected job changes the available instances.

---

## Variable 5 — Interval

Create a custom variable:

```text
Name: interval
Type: Custom
Label: Interval
Values: 5m,10m,30m,1h,6h,12h,1d
Multi-value: Disabled
Include All: Disabled
```

---

# 📈 Task 7 — Build Dynamic Panels

## CPU Usage Panel

Use:

```promql
rate(prometheus_tsdb_head_samples_appended_total{job=~"$job", instance=~"$instance"}[$interval])
```

Title:

```text
CPU Usage - Job: $job, Instance: $instance
```

Description:

```text
Dynamic CPU usage metrics filtered by selected job and instance.
```

---

## Memory Series Panel

Query:

```promql
prometheus_tsdb_head_series{job=~"$job", instance=~"$instance"}
```

Title:

```text
Memory Series - $job ($region)
```

Description:

```text
Memory metrics for job $job in region $region.
```

---

## HTTP Request Rate Panel

Query:

```promql
rate(prometheus_http_requests_total{job=~"$job", instance=~"$instance"}[$interval])
```

Title:

```text
HTTP Requests Rate - $environment Environment
```

Legend:

```text
{{job}} - {{instance}}
```

---

# 🟢 Task 8 — Create Active Services Panel

Use a **Stat** visualization:

```promql
count(up{job=~"$job", environment=~"$environment", region=~"$region"} == 1)
```

Title:

```text
Active Services
```

Description:

```text
Number of active services matching current filters.
```

Recommended value settings:

```text
Unit: Short
Decimals: 0
```

---

# 📋 Task 9 — Create Service Status Table

Change visualization to:

```text
Table
```

### Query A

```promql
up{job=~"$job", environment=~"$environment", region=~"$region"}
```

### Query B

```promql
prometheus_build_info{job=~"$job", instance=~"$instance"}
```

Title:

```text
Service Status Table - $job
```

---

# 🧪 Task 10 — Test Variable Functionality

Test the dashboard variables systematically.

### Job Variable

* Select different jobs.
* Verify panels update.
* Select multiple jobs.
* Test **All**.

### Environment Variable

* Select one environment.
* Select multiple environments.
* Verify queries update.

### Region Variable

* Select different regions.
* Combine region selections with job and environment filters.

### Cascading Instance

Change:

```text
Job
```

Then verify:

```text
Instance
```

changes accordingly.

---

# 🔎 Task 11 — Advanced Regex Variable

Create:

```text
Name: filtered_job
Type: Query
Label: Filtered Jobs
Data Source: Prometheus
Query: label_values(up, job)
Regex: /.*app.*/
Sort: Alphabetical (asc)
Multi-value: Enabled
Include All: Enabled
```

Use it in a panel:

```promql
up{job=~"$filtered_job", environment=~"$environment"}
```

Dynamic title:

```text
Filtered Services - $filtered_job
```

---

# 🎚️ Task 12 — Constant Variable

Create:

```text
Name: cpu_threshold
Type: Constant
Label: CPU Threshold
Value: 80
```

Example query:

```promql
prometheus_tsdb_head_samples_appended_total > $cpu_threshold
```

Panel title:

```text
Services Above $cpu_threshold Threshold
```

> 💡 **Note:** The metric above is used as a demonstration of threshold substitution; choose a metric whose units and semantics match the threshold when implementing a real CPU alert/dashboard.

---

# 🔄 Task 13 — Variable Refresh

Edit the `job` variable.

Set:

```text
Refresh: On Dashboard Load
```

Save the dashboard and reload the page.

The variable should refresh automatically.

---

# 📝 Task 14 — Textbox Variable

Create:

```text
Name: custom_filter
Type: Textbox
Label: Custom Filter
Default value: .*
```

Use:

```promql
up{job=~"$job", instance=~"$custom_filter"}
```

This allows users to enter a custom regular expression.

---

# 🔗 Task 15 — Variable Chaining

Create a country variable:

```text
Name: country
Type: Custom
Values: US,EU,ASIA
```

Modify the region variable:

```promql
label_values(up{region=~"${country:regex}"}, region)
```

This demonstrates hierarchical variable relationships.

---

# ✅ Task 16 — Variable Validation

Create a Stat panel:

```promql
count(up{job=~"$job"}) or vector(0)
```

Title:

```text
Selected Jobs Count: $job
```

This provides visual feedback about the current variable selection.

---

# 🩺 Task 17 — Troubleshooting

## Empty Variable Results

Check Prometheus:

```bash
curl http://localhost:9090/api/v1/label/job/values
```

Check available metrics:

```bash
curl "http://localhost:9090/api/v1/query?query=up"
```

Verify:

* Prometheus is running.
* The Grafana data source is correct.
* The requested label exists.
* The variable query is valid.

---

## Panels Not Updating

Check:

```text
$variable_name
```

Make sure:

* Variable names match exactly.
* Variables are saved.
* PromQL uses the appropriate matcher.
* Multi-value variables generally use `=~` rather than `=`.
* The dashboard is refreshed when necessary.

---

## Performance Problems

If the dashboard becomes slow:

* Reduce unnecessary multi-value selections.
* Limit large label sets with regex.
* Optimize PromQL queries.
* Use appropriate variable refresh settings.
* Avoid excessively expensive queries.
* Use reasonable dashboard time ranges.

---

# 💾 Task 18 — Export and Import Dashboard

Open:

```text
Dashboard Settings → JSON Model
```

Copy the JSON and save it:

```bash
cat > ~/grafana-variables-lab/dashboard-export.json << 'EOF'
# Paste exported Grafana JSON here
EOF
```

To test the export:

```text
Create Dashboard
→ Import
→ Upload/Enter JSON
```

This demonstrates dashboard portability and backup.

---

# 🔍 Verification Checklist

Use the following checklist before completing the lab:

* [ ] Grafana is accessible.
* [ ] Prometheus data source is working.
* [ ] Job variable displays available jobs.
* [ ] Environment variable displays environments.
* [ ] Region variable displays regions.
* [ ] Instance variable cascades from Job.
* [ ] Interval variable provides time options.
* [ ] Multi-value selection works.
* [ ] All option works.
* [ ] Variables update panel queries.
* [ ] Variables update panel titles.
* [ ] Regex filtering works.
* [ ] Constant variables work.
* [ ] Textbox filtering works.
* [ ] Variable refresh works.
* [ ] Dashboard export works.
* [ ] Dashboard import works.

---

# ⚡ Performance Targets

Use these targets as practical lab goals:

```text
Dashboard load time: < 5 seconds
Variable query time: < 2 seconds
Panel refresh time:  < 3 seconds
```

Actual performance depends on the number of series, query complexity, dashboard time range, and available system resources.

---

# 🧹 Cleanup

Stop and remove the lab:

```bash
cd ~/grafana-variables-lab
docker-compose down -v
```

Remove the project:

```bash
cd ~
rm -rf ~/grafana-variables-lab
```

---

# 🧠 Key Concepts Learned

| Concept            | What You Learned                            |
| ------------------ | ------------------------------------------- |
| Query Variable     | Dynamically obtains options from Prometheus |
| Custom Variable    | Provides predefined values                  |
| Constant Variable  | Stores reusable fixed values                |
| Textbox Variable   | Allows user-provided filtering              |
| Multi-value        | Enables multiple selections                 |
| All Option         | Selects all available values                |
| Cascading Variable | Makes one variable depend on another        |
| Regex              | Filters available variable values           |
| Variable Refresh   | Controls when variable values update        |
| Dynamic Titles     | Displays selected values in panel titles    |
| PromQL Variables   | Makes queries reusable and interactive      |

---

# 🌍 Real-World Use Cases

Grafana variables are especially useful for organizations monitoring:

* ☁️ Multiple cloud environments
* 🌎 Multiple geographic regions
* 🏭 Production and staging systems
* 🖥️ Multiple servers
* 📦 Kubernetes clusters
* 🚀 Microservices
* 🧑‍💻 Development environments
* 🔧 Application instances
* 📊 Infrastructure metrics

For example, instead of maintaining:

```text
Production Dashboard
Staging Dashboard
Development Dashboard
```

you can create one dashboard:

```text
Environment: [Production ▼]
Region:      [US-East ▼]
Service:     [Backend ▼]
Instance:    [All ▼]
```

This significantly reduces dashboard duplication and maintenance.

---

# 🏆 Best Practices

### 1. Use Meaningful Names

Prefer:

```text
environment
region
instance
service
```

instead of:

```text
var1
var2
var3
```

### 2. Use Regex for Multi-Value Variables

For multi-select variables:

```promql
job=~"$job"
```

is generally appropriate.

### 3. Avoid Excessive Cardinality

Do not expose thousands of values in a dropdown unless there is a clear operational need.

### 4. Use Cascading Variables

For large infrastructures, use relationships such as:

```text
Environment
    ↓
Region
    ↓
Cluster
    ↓
Namespace
    ↓
Service
    ↓
Instance
```

### 5. Make Titles Dynamic

For example:

```text
HTTP Requests - $environment / $region
```

helps users understand what they are viewing.

### 6. Optimize Queries

Avoid unnecessarily expensive queries and use appropriate time ranges and aggregation.

---

# 🎓 Learning Outcomes

After completing this lab, you should understand how Grafana variables transform static dashboards into interactive observability tools.

You have learned how to:

```text
Create Variables
      ↓
Connect Variables to Prometheus
      ↓
Use Variables in PromQL
      ↓
Build Dynamic Panels
      ↓
Implement Cascading Filters
      ↓
Add Regex & Custom Filtering
      ↓
Validate Dashboard Behavior
      ↓
Optimize Dashboard Performance
```

---

# 🚀 Conclusion

This lab provided hands-on experience with **dynamic Grafana dashboards using Prometheus variables**.

You created multiple variable types, including query, custom, constant, and textbox variables. You also implemented multi-value selections, the **All** option, regular-expression filtering, variable refresh behavior, and cascading dependencies.

The resulting dashboard is more flexible than a collection of static dashboards because users can dynamically select environments, regions, jobs, and instances while the underlying PromQL queries automatically adapt.

These techniques are fundamental to modern **observability and monitoring platforms**, where a single reusable dashboard can serve multiple teams, environments, applications, and infrastructure components.

> ⭐ **A well-designed Grafana dashboard should not only display metrics—it should help engineers quickly filter, investigate, compare, and understand those metrics.**

---

## ✨ Lab Completion

```text
╔════════════════════════════════════════════════════════════╗
║              🎉 LAB COMPLETED SUCCESSFULLY 🎉             ║
║                                                            ║
║        Creating Dynamic Dashboards with Variables          ║
║                                                            ║
║   📊 Grafana  +  📈 Prometheus  +  🐳 Docker  + PromQL   ║
║                                                            ║
║              Build • Filter • Visualize • Monitor          ║
╚════════════════════════════════════════════════════════════╝
```

**Author:** Hafiz Muhammad Salman
**Role:** Cloud DevOps Engineer & Linux Administrator
**Lab Platform:** Al Nafi Cloud
**Focus:** Grafana • Prometheus • PromQL • Observability • Dynamic Dashboards
