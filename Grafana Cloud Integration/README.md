# 🌐 Grafana Cloud Integration with LocalStack CloudWatch

> **Hands-on Observability Lab**
> **Platform:** Ubuntu Linux + AWS EC2 + LocalStack + Grafana OSS
> **Focus:** CloudWatch Simulation • Dashboards • API Automation • Alerting

---

## 📋 Lab Overview

This lab demonstrates how to build a complete **cloud monitoring and alerting pipeline** on a bare Ubuntu instance by integrating **Grafana** with a **LocalStack-simulated AWS CloudWatch environment**.

Instead of connecting to a real AWS CloudWatch service, LocalStack provides a local CloudWatch-compatible endpoint. Custom EC2 metrics are generated with Python and `boto3`, visualized through Grafana dashboards, and monitored using Grafana Unified Alerting.

The entire environment is provisioned and validated through **CLI commands, scripts, and APIs**, without relying on manual Grafana UI configuration.

---

## 🎯 Objectives

By completing this lab, you will be able to:

* 🐧 Install and configure Grafana on Ubuntu.
* 🐳 Install Docker Engine and Docker Compose v2.
* ☁️ Run LocalStack as a simulated AWS CloudWatch environment.
* 🛠️ Configure AWS CLI to communicate with LocalStack.
* 🐍 Generate custom CloudWatch metrics using Python and `boto3`.
* 📊 Configure Grafana CloudWatch data sources through the HTTP API.
* 📈 Create automated Grafana dashboards.
* 🚨 Configure Grafana Unified Alerting rules through APIs.
* 🔔 Configure webhook-based contact points.
* 🧪 Trigger and validate alert state transitions.
* 🤖 Automate observability infrastructure without manual UI interaction.

---

## 🏗️ Architecture

```text
                    ┌─────────────────────────┐
                    │     Ubuntu EC2 Host     │
                    │                         │
                    │  ┌───────────────────┐  │
                    │  │ Python + boto3    │  │
                    │  │ Metric Generator  │  │
                    │  └─────────┬─────────┘  │
                    │            │             │
                    │            ▼             │
                    │  ┌───────────────────┐  │
                    │  │    LocalStack     │  │
                    │  │                   │  │
                    │  │  CloudWatch       │  │
                    │  │  CloudWatch Logs  │  │
                    │  └─────────┬─────────┘  │
                    │            │             │
                    │            ▼             │
                    │  ┌───────────────────┐  │
                    │  │      Grafana      │  │
                    │  │                   │  │
                    │  │ Dashboards        │  │
                    │  │ Alerting          │  │
                    │  │ CloudWatch DS     │  │
                    │  └─────────┬─────────┘  │
                    │            │             │
                    │            ▼             │
                    │      Webhook Alert       │
                    └─────────────────────────┘
```

---

# 🧰 Technology Stack

| Technology                  | Purpose                        |
| --------------------------- | ------------------------------ |
| 🐧 Ubuntu                   | Lab operating system           |
| ☁️ AWS EC2                  | Lab infrastructure             |
| 🐳 Docker                   | Container runtime              |
| 🔗 Docker Compose v2        | Container orchestration        |
| 🧪 LocalStack               | AWS service simulation         |
| ☁️ CloudWatch               | Simulated metrics backend      |
| 🖥️ AWS CLI v2              | CloudWatch command-line access |
| 🐍 Python 3                 | Metric generation              |
| 📦 boto3                    | AWS SDK for Python             |
| 📊 Grafana OSS              | Visualization and alerting     |
| 🔌 Grafana HTTP API         | Automated provisioning         |
| 🚨 Grafana Unified Alerting | Metric-based alerting          |
| 🪝 Webhook                  | Alert notification endpoint    |

---

# 📚 Prerequisites

Before starting, ensure you understand:

* Basic Ubuntu command-line operations.
* Linux services and `systemctl`.
* Package installation using APT.
* Docker fundamentals.
* Python basics.
* AWS CloudWatch concepts.
* Metrics, namespaces, dimensions, and thresholds.
* Basic REST API concepts.
* JSON and `curl`.

---

# 🖥️ Lab Environment

The lab uses a dedicated **AWS EC2 Ubuntu instance provided by Al Nafi**.

The base environment should contain a clean Ubuntu installation. Required tools are installed during the lab.

### Required Components

```text
Docker Engine
Docker Compose v2
AWS CLI v2
Python 3
boto3
LocalStack
Grafana OSS
```

### Important Ports

|   Port | Service               |
| -----: | --------------------- |
| `3000` | Grafana               |
| `4566` | LocalStack            |
| `9999` | Test webhook endpoint |

---

# 🚀 Task 1 — Install and Verify the Full Toolchain

## 1️⃣ Update Ubuntu

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

If Docker packages are unavailable, enable the Ubuntu Universe repository:

```bash
sudo add-apt-repository universe
sudo apt-get update
```

---

## 🐳 2️⃣ Install Docker Engine

Install Docker using the official Docker repository and installation method.

After installation:

```bash
docker --version
```

Verify Docker works without `sudo`:

```bash
docker run hello-world
```

If permission is denied:

```bash
sudo usermod -aG docker $USER
```

Then reload the session:

```bash
newgrp docker
```

Verify:

```bash
docker ps
```

### ✅ Expected Result

Your user should be able to execute:

```bash
docker ps
```

without `sudo`.

---

## 🔗 3️⃣ Verify Docker Compose v2

```bash
docker compose version
```

Expected output should report Docker Compose v2.

> ⚠️ Use `docker compose`, not the legacy `docker-compose`, for this lab.

---

# ☁️ 4️⃣ Install AWS CLI v2

Install AWS CLI v2 using the official AWS distribution.

Verify:

```bash
aws --version
```

### Troubleshooting

If you encounter:

```text
curl: (22) The requested URL returned error: 403
```

check the current AWS CLI v2 installation instructions and update the download URL.

---

# 🐍 5️⃣ Install Python and Required Packages

Check Python:

```bash
python3 --version
```

Install supporting packages:

```bash
sudo apt-get install -y python3 python3-pip python3-venv
```

### Recommended: Virtual Environment

```bash
python3 -m venv ~/labenv
source ~/labenv/bin/activate
```

Install dependencies:

```bash
pip install boto3 localstack
```

Verify:

```bash
python3 -c "import boto3; print(boto3.__version__)"
```

---

## ⚠️ Ubuntu `externally-managed-environment`

On newer Ubuntu releases, you may encounter:

```text
error: externally-managed-environment
```

A virtual environment is recommended:

```bash
python3 -m venv ~/labenv
source ~/labenv/bin/activate
pip install boto3 localstack
```

Alternatively:

```bash
pip3 install --break-system-packages localstack boto3
```

---

# 📊 6️⃣ Install Grafana OSS

Install Grafana using the official Grafana repository.

Verify:

```bash
grafana-server --version
```

Enable and start Grafana:

```bash
sudo systemctl enable --now grafana-server
```

Check status:

```bash
sudo systemctl status grafana-server
```

Or:

```bash
systemctl is-active grafana-server
```

### ✅ Expected Result

```text
active
```

Verify port `3000`:

```bash
ss -lntp | grep 3000
```

---

# 🧪 7️⃣ Verify the Complete Toolchain

Run:

```bash
docker --version
docker compose version
aws --version
python3 --version
grafana-server --version
systemctl is-active grafana-server
```

All components should respond successfully.

---

# 🐳 Task 1.2 — Deploy LocalStack

Create a LocalStack container:

```bash
docker run -d \
  --name localstack \
  -p 4566:4566 \
  -e SERVICES=cloudwatch,logs \
  -e AWS_DEFAULT_REGION=us-east-1 \
  localstack/localstack
```

Check the container:

```bash
docker ps
```

### 🔍 Check LocalStack Health

```bash
curl -fsSL http://localhost:4566/_localstack/health
```

The response should indicate that CloudWatch is:

```text
available
```

or:

```text
running
```

---

# 🔐 Configure AWS CLI

Configure dummy credentials:

```bash
aws configure
```

Use:

```text
AWS Access Key ID: test
AWS Secret Access Key: test
Default region name: us-east-1
Default output format: json
```

Because LocalStack does not require real AWS credentials, the values are intentionally dummy credentials.

---

## 🔎 Test CloudWatch

Run:

```bash
aws --endpoint-url=http://localhost:4566 \
  cloudwatch list-metrics \
  --output json
```

### ✅ Expected Result

The command should return:

```json
{
  "Metrics": []
}
```

or a JSON object containing metric definitions.

The important requirement is that the command completes successfully.

---

# 🧪 Task 1 Acceptance Test

Run:

```bash
docker ps
```

```bash
curl -fsSL http://localhost:4566/_localstack/health
```

```bash
systemctl is-active grafana-server
```

```bash
aws --endpoint-url=http://localhost:4566 \
  cloudwatch list-metrics \
  --output json
```

### ✅ Success Criteria

* 🐳 LocalStack container is `Up`.
* ☁️ CloudWatch is available/running.
* 📊 Grafana is `active`.
* 🔌 CloudWatch API returns HTTP 200.
* 📄 Response contains the `Metrics` key.

---

# 📈 Task 2 — Populate CloudWatch Metrics

Create a Python metric generator.

The script must publish:

| Namespace      | Metric              | Dimension                        |
| -------------- | ------------------- | -------------------------------- |
| `AWS/EC2`      | `CPUUtilization`    | `InstanceId=i-1234567890abcdef0` |
| `System/Linux` | `MemoryUtilization` | `InstanceId=i-1234567890abcdef0` |
| `AWS/EC2`      | `NetworkIn`         | `InstanceId=i-1234567890abcdef0` |

---

## 🐍 Metric Generator

Create:

```bash
nano ~/cloudwatch_metrics.py
```

Example implementation:

```python
import random
import time
import boto3
from botocore.exceptions import EndpointConnectionError

ENDPOINT = "http://localhost:4566"
REGION = "us-east-1"
INSTANCE_ID = "i-1234567890abcdef0"

cloudwatch = boto3.client(
    "cloudwatch",
    endpoint_url=ENDPOINT,
    region_name=REGION,
    aws_access_key_id="test",
    aws_secret_access_key="test",
)

def wait_for_localstack():
    while True:
        try:
            cloudwatch.list_metrics()
            print("LocalStack CloudWatch is ready.")
            return
        except Exception as exc:
            print(f"Waiting for LocalStack: {exc}")
            time.sleep(5)

def publish_metric(namespace, name, value, unit):
    cloudwatch.put_metric_data(
        Namespace=namespace,
        MetricData=[
            {
                "MetricName": name,
                "Dimensions": [
                    {
                        "Name": "InstanceId",
                        "Value": INSTANCE_ID,
                    }
                ],
                "Value": value,
                "Unit": unit,
            }
        ],
    )

wait_for_localstack()

while True:
    cpu = random.uniform(10, 90)
    memory = random.uniform(20, 85)
    network = random.uniform(1000, 100000)

    publish_metric(
        "AWS/EC2",
        "CPUUtilization",
        cpu,
        "Percent",
    )

    publish_metric(
        "System/Linux",
        "MemoryUtilization",
        memory,
        "Percent",
    )

    publish_metric(
        "AWS/EC2",
        "NetworkIn",
        network,
        "Bytes",
    )

    print(
        f"CPU={cpu:.2f}% "
        f"Memory={memory:.2f}% "
        f"NetworkIn={network:.2f}"
    )

    time.sleep(15)
```

---

# ▶️ Run the Metric Generator

Make it executable:

```bash
chmod +x ~/cloudwatch_metrics.py
```

Start it in the background:

```bash
nohup python3 ~/cloudwatch_metrics.py \
  > ~/cloudwatch_metrics.log 2>&1 &
```

Find the process:

```bash
ps aux | grep cloudwatch_metrics
```

Monitor the log:

```bash
tail -f ~/cloudwatch_metrics.log
```

---

# 📊 Verify Published Metrics

After sufficient data has been generated:

```bash
aws --endpoint-url=http://localhost:4566 \
  cloudwatch list-metrics \
  --namespace AWS/EC2 \
  --output json
```

Verify:

```text
CPUUtilization
NetworkIn
```

Then:

```bash
aws --endpoint-url=http://localhost:4566 \
  cloudwatch list-metrics \
  --namespace System/Linux \
  --output json
```

Verify:

```text
MemoryUtilization
```

At least **20 data points per metric** should exist before dashboard verification.

---

# 🔌 Troubleshooting boto3

If you encounter:

```text
EndpointResolutionError
```

or:

```text
Could not connect to the endpoint URL
```

make sure LocalStack is healthy:

```bash
curl -fsSL http://localhost:4566/_localstack/health
```

The metric generator should wait until CloudWatch is available before publishing data.

---

# 📊 Task 2.2 — Configure Grafana CloudWatch Data Source

Grafana must be configured through the **Grafana HTTP API**.

No manual browser configuration should be used as the deliverable.

Grafana API endpoint:

```text
http://localhost:3000
```

Default lab credentials:

```text
Username: admin
Password: admin
```

---

## 🔌 Create the LocalStack CloudWatch Data Source

Create a JSON payload similar to:

```json
{
  "name": "LocalStack CloudWatch",
  "type": "cloudwatch",
  "access": "proxy",
  "url": "http://localhost:4566",
  "jsonData": {
    "authType": "keys",
    "defaultRegion": "us-east-1"
  },
  "secureJsonData": {
    "accessKey": "test",
    "secretKey": "test"
  }
}
```

Send it to Grafana:

```bash
curl -X POST \
  -u admin:admin \
  -H "Content-Type: application/json" \
  http://localhost:3000/api/datasources \
  -d @datasource.json
```

---

# 🔍 Verify Data Source

```bash
curl -fsSL \
  -u admin:admin \
  http://localhost:3000/api/datasources
```

Look for:

```text
LocalStack CloudWatch
```

and:

```text
"type": "cloudwatch"
```

---

# 📈 Dashboard 1 — EC2 Instance Metrics

Create a dashboard titled:

```text
EC2 Instance Metrics
```

It must contain:

### CPU Panel

```text
CPUUtilization
```

Visualization:

```text
Time Series
```

### Memory Panel

```text
MemoryUtilization
```

Visualization:

```text
Stat
```

### Network Panel

```text
NetworkIn
```

Visualization:

```text
Time Series
```

---

# 📊 Dashboard 2 — Infrastructure Overview

Create:

```text
Infrastructure Overview
```

The dashboard must contain a panel comparing:

```text
CPUUtilization
MemoryUtilization
```

on the same time-series axes.

---

## ⚙️ Dashboard Settings

Both dashboards must use:

```text
Refresh: 30s
Time Range: Last 1 hour
```

Example dashboard configuration:

```json
{
  "refresh": "30s",
  "time": {
    "from": "now-1h",
    "to": "now"
  }
}
```

---

# 🔎 Dashboard Verification

Check all dashboards:

```bash
curl -fsSL \
  -u admin:admin \
  "http://localhost:3000/api/search?type=dash-db"
```

The response should contain:

```text
EC2 Instance Metrics
```

and:

```text
Infrastructure Overview
```

---

# ☁️ Verify CloudWatch Metrics

Run:

```bash
aws --endpoint-url=http://localhost:4566 \
  cloudwatch list-metrics \
  --namespace AWS/EC2 \
  --output json
```

Verify:

```text
CPUUtilization
NetworkIn
InstanceId
```

Then:

```bash
aws --endpoint-url=http://localhost:4566 \
  cloudwatch list-metrics \
  --namespace System/Linux \
  --output json
```

Verify:

```text
MemoryUtilization
InstanceId
```

---

# 🚨 Task 3 — Configure Grafana Unified Alerting

The lab uses **Grafana Unified Alerting**, not the legacy notification API.

Create an alert group:

```text
ec2-alerts
```

inside the:

```text
default
```

folder.

---

# 🔥 Alert Rule 1 — High CPU

Rule name:

```text
High CPU
```

Condition:

```text
Average CPUUtilization over 5 minutes > 75%
```

Label:

```text
severity=warning
```

Annotation:

```text
summary=High CPU utilization detected
```

---

# 🧠 Alert Rule 2 — High Memory

Rule name:

```text
High Memory
```

Condition:

```text
Average MemoryUtilization over 5 minutes > 80%
```

Label:

```text
severity=warning
```

Annotation:

```text
summary=High memory utilization detected
```

---

# 🪝 Configure Webhook Contact Point

Create:

```text
lab-webhook
```

Type:

```text
webhook
```

Endpoint:

```text
http://localhost:9999/alert
```

The endpoint does not need to actually exist for this lab.

The objective is to demonstrate correct alert contact-point configuration.

---

# 🔍 Verify Unified Alerting

Check the provisioning API:

```bash
curl -fsSL \
  -u admin:admin \
  http://localhost:3000/api/v1/provisioning/alert-rules
```

If this endpoint returns `404`, check Unified Alerting.

Edit:

```bash
sudo nano /etc/grafana/grafana.ini
```

Ensure:

```ini
[unified_alerting]
enabled = true
```

Restart Grafana:

```bash
sudo systemctl restart grafana-server
```

Check:

```bash
systemctl is-active grafana-server
```

---

# 🔎 Verify Alert Rules

```bash
curl -fsSL \
  -u admin:admin \
  http://localhost:3000/api/v1/provisioning/alert-rules
```

Verify:

```text
High CPU
High Memory
severity=warning
```

---

# 🔔 Verify Contact Point

```bash
curl -fsSL \
  -u admin:admin \
  http://localhost:3000/api/v1/provisioning/contact-points
```

Verify:

```text
lab-webhook
```

and:

```text
webhook
```

---

# 🔥 Task 3.2 — Trigger the High CPU Alert

Modify the metric generator so CPU values remain between:

```text
85% - 98%
```

Example:

```python
cpu = random.uniform(85, 98)
```

Allow the metric generator to continue publishing values for a sustained period.

Monitor the generated metrics:

```bash
tail -f ~/cloudwatch_metrics.log
```

---

# 🚦 Monitor Alert State

Query Grafana:

```bash
curl -fsSL \
  -u admin:admin \
  "http://localhost:3000/api/prometheus/grafana/api/v1/rules"
```

The `High CPU` rule should eventually transition from:

```text
Normal
```

to:

```text
Pending
```

or:

```text
Firing
```

depending on the configured evaluation and pending periods.

---

# 💾 Save Alert Validation

Capture the API response:

```bash
curl -fsSL \
  -u admin:admin \
  "http://localhost:3000/api/prometheus/grafana/api/v1/rules" \
  | tee ~/alert_validation.txt
```

Verify:

```bash
ls -lh ~/alert_validation.txt
```

Inspect:

```bash
cat ~/alert_validation.txt
```

The file must contain the raw JSON response showing the triggered state.

---

# 🧪 Final Acceptance Tests

## LocalStack

```bash
docker ps
```

Expected:

```text
localstack ... Up
```

---

## LocalStack Health

```bash
curl -fsSL http://localhost:4566/_localstack/health
```

CloudWatch must report:

```text
available
```

or:

```text
running
```

---

## Grafana

```bash
systemctl is-active grafana-server
```

Expected:

```text
active
```

---

## CloudWatch Data Source

```bash
curl -fsSL \
  -u admin:admin \
  http://localhost:3000/api/datasources
```

Expected:

```text
LocalStack CloudWatch
```

---

## Dashboards

```bash
curl -fsSL \
  -u admin:admin \
  "http://localhost:3000/api/search?type=dash-db"
```

Expected:

```text
EC2 Instance Metrics
Infrastructure Overview
```

---

## EC2 Metrics

```bash
aws --endpoint-url=http://localhost:4566 \
  cloudwatch list-metrics \
  --namespace AWS/EC2 \
  --output json
```

Expected:

```text
CPUUtilization
NetworkIn
```

---

## Linux Memory Metric

```bash
aws --endpoint-url=http://localhost:4566 \
  cloudwatch list-metrics \
  --namespace System/Linux \
  --output json
```

Expected:

```text
MemoryUtilization
```

---

## Alert Rules

```bash
curl -fsSL \
  -u admin:admin \
  http://localhost:3000/api/v1/provisioning/alert-rules
```

Expected:

```text
High CPU
High Memory
```

---

## Contact Point

```bash
curl -fsSL \
  -u admin:admin \
  http://localhost:3000/api/v1/provisioning/contact-points
```

Expected:

```text
lab-webhook
```

---

## Triggered Alert

```bash
curl -fsSL \
  -u admin:admin \
  "http://localhost:3000/api/prometheus/grafana/api/v1/rules"
```

Expected:

```text
High CPU → pending
```

or:

```text
High CPU → firing
```

---

# 🗂️ Suggested Lab Files

A clean implementation can use the following structure:

```text
grafana-cloud-integration/
│
├── README.md
├── scripts/
│   ├── cloudwatch_metrics.py
│   ├── datasource.json
│   ├── ec2-dashboard.json
│   ├── infrastructure-dashboard.json
│   └── provision-alerts.sh
│
├── logs/
│   └── cloudwatch_metrics.log
│
└── validation/
    └── alert_validation.txt
```

---

# 🧯 Troubleshooting Guide

## Docker Package Not Found

Error:

```text
E: Package 'docker.io' has no installation candidate
```

Solution:

```bash
sudo add-apt-repository universe
sudo apt-get update
```

Then retry the installation.

---

## AWS CLI Download Returns 403

Error:

```text
curl: (22) The requested URL returned error: 403
```

The AWS CLI download URL may have changed.

Check the current AWS CLI v2 installation instructions before downloading again.

---

## Grafana Repository Error

Error:

```text
Malformed entry 1 in list file
```

Inspect:

```bash
cat /etc/apt/sources.list.d/grafana.list
```

Ensure the repository entry is a single valid line without accidental backslashes.

---

## LocalStack Connection Failure

Error:

```text
Could not connect to the endpoint URL
```

Check:

```bash
docker ps
```

Then:

```bash
curl -fsSL http://localhost:4566/_localstack/health
```

Restart if necessary:

```bash
docker restart localstack
```

---

## Python Package Installation Error

Error:

```text
externally-managed-environment
```

Use:

```bash
python3 -m venv ~/labenv
source ~/labenv/bin/activate
pip install boto3 localstack
```

---

## Grafana Unified Alerting API Returns 404

Check:

```bash
curl -fsSL \
  -u admin:admin \
  http://localhost:3000/api/v1/provisioning/alert-rules
```

If unavailable, enable:

```ini
[unified_alerting]
enabled = true
```

Then:

```bash
sudo systemctl restart grafana-server
```

---

# 🔐 Security Considerations

This lab intentionally uses:

```text
access key = test
secret key = test
```

These credentials are **only dummy credentials for LocalStack**.

Do not use them in a production AWS environment.

For real AWS deployments:

* Use IAM roles where possible.
* Avoid hard-coded credentials.
* Apply least-privilege permissions.
* Protect Grafana credentials.
* Secure CloudWatch endpoints.
* Use HTTPS for external services.
* Restrict Grafana network access.
* Protect webhook endpoints.

---

# 📊 What This Lab Demonstrates

The completed lab creates an end-to-end monitoring workflow:

```text
Simulated EC2 Metrics
        │
        ▼
Python + boto3
        │
        ▼
LocalStack CloudWatch
        │
        ▼
Grafana CloudWatch Data Source
        │
        ├───────────────┐
        ▼               ▼
   Dashboards       Alert Rules
        │               │
        │               ▼
        │         lab-webhook
        │
        ▼
   Visualization
```

---

# 🧠 Key Learning Outcomes

After completing this lab, you should understand how to:

### ☁️ Cloud Monitoring

* Work with CloudWatch namespaces.
* Create custom metrics.
* Use dimensions such as `InstanceId`.
* Query CloudWatch-compatible APIs.

### 🐍 Automation

* Use `boto3` to publish metrics.
* Build continuous metric generators.
* Implement service readiness checks.
* Run monitoring scripts in the background.

### 📊 Grafana

* Configure CloudWatch data sources.
* Provision dashboards through REST APIs.
* Create time-series visualizations.
* Configure automatic refresh and time ranges.

### 🚨 Alerting

* Use Grafana Unified Alerting.
* Create threshold-based alert rules.
* Apply labels and annotations.
* Configure webhook contact points.
* Validate alert state transitions programmatically.

### 🤖 Infrastructure as Code Principles

The lab demonstrates a key DevOps principle:

> **Monitoring infrastructure should be reproducible, automated, and API-driven rather than dependent on manual configuration.**

---

# 🌍 Real-World AWS Extension

LocalStack acts as the simulated CloudWatch backend:

```text
Lab:

Grafana → LocalStack → Simulated CloudWatch
```

A production implementation can replace LocalStack with AWS:

```text
Production:

Grafana → AWS CloudWatch → Real EC2 Infrastructure
```

The monitoring concepts remain largely the same:

* Metrics
* Namespaces
* Dimensions
* Dashboards
* Thresholds
* Alert rules
* Notifications

This makes the lab directly relevant to real-world **AWS observability and DevOps engineering**.

---

# 🏆 Expected Final State

At the end of the lab, you should have:

```text
✅ Ubuntu EC2 instance
✅ Docker Engine
✅ Docker Compose v2
✅ AWS CLI v2
✅ Python 3
✅ boto3
✅ LocalStack
✅ CloudWatch simulation
✅ Grafana OSS
✅ LocalStack CloudWatch data source
✅ EC2 Instance Metrics dashboard
✅ Infrastructure Overview dashboard
✅ High CPU alert
✅ High Memory alert
✅ lab-webhook contact point
✅ Triggered High CPU alert
✅ alert_validation.txt
```

---

# 🎓 Conclusion

This lab builds a complete **cloud observability pipeline** from a bare Ubuntu server.

You begin by installing the monitoring toolchain, deploy LocalStack as a CloudWatch simulator, generate realistic EC2 metrics with Python, connect Grafana to the simulated CloudWatch endpoint, provision dashboards through the Grafana API, and finally implement Unified Alerting rules.

The most important outcome is not simply creating Grafana dashboards. It is learning how to build a **fully automated monitoring workflow** where data ingestion, visualization, alerting, and validation can all be controlled programmatically.

The same architecture can be adapted to real AWS environments by replacing the LocalStack endpoint with genuine AWS CloudWatch infrastructure and appropriate IAM authentication.

---

## ⭐ Lab Completion Checklist

* [ ] Ubuntu environment prepared
* [ ] Docker Engine installed
* [ ] Docker works without `sudo`
* [ ] Docker Compose v2 verified
* [ ] AWS CLI v2 verified
* [ ] Python 3 installed
* [ ] `boto3` installed
* [ ] LocalStack installed/running
* [ ] Grafana installed/running
* [ ] CloudWatch enabled in LocalStack
* [ ] AWS CLI connected to LocalStack
* [ ] EC2 metrics generated
* [ ] At least 20 data points published per metric
* [ ] LocalStack CloudWatch data source created
* [ ] `EC2 Instance Metrics` dashboard created
* [ ] `Infrastructure Overview` dashboard created
* [ ] 30-second dashboard refresh configured
* [ ] One-hour default time range configured
* [ ] `ec2-alerts` rule group created
* [ ] `High CPU` rule created
* [ ] `High Memory` rule created
* [ ] `severity=warning` configured
* [ ] `lab-webhook` contact point configured
* [ ] High CPU alert deliberately triggered
* [ ] Alert state verified through API
* [ ] `~/alert_validation.txt` created
* [ ] End-to-end monitoring pipeline validated

---

## 🚀 Final Result

**LocalStack + CloudWatch Simulation + Python + Grafana + Unified Alerting**

### → A complete API-driven cloud observability laboratory.

**Built for practical Cloud DevOps, Monitoring, and Observability Engineering.**

---

### 🏅 Skills Demonstrated

`AWS CloudWatch` `LocalStack` `Grafana` `Docker` `Python` `boto3` `AWS CLI` `Linux` `REST API` `Grafana Alerting` `Observability` `Monitoring` `DevOps` `Automation`
