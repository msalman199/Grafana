# 🚀 Automating Grafana Dashboard Provisioning with Terraform

![Terraform](https://img.shields.io/badge/Terraform-1.6+-623CE4?style=for-the-badge\&logo=terraform\&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-Monitoring-F46800?style=for-the-badge\&logo=grafana\&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C?style=for-the-badge\&logo=prometheus\&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Container-2496ED?style=for-the-badge\&logo=docker\&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-FCC624?style=for-the-badge\&logo=linux\&logoColor=black)
![HCL](https://img.shields.io/badge/HCL-Infrastructure%20as%20Code-623CE4?style=for-the-badge)

> 🏗️ **Infrastructure as Code Lab** — Automate Grafana organizations, data sources, dashboards, users, teams, permissions, and alerting with Terraform.

---

## 📌 Overview

This hands-on lab demonstrates how to manage **Grafana as Infrastructure as Code (IaC)** using Terraform.

Instead of manually creating Grafana resources through the web interface, Terraform is used to define the desired monitoring infrastructure in code. The configuration can then be planned, deployed, version-controlled, modified, and recreated consistently.

### 🔧 What This Lab Builds

```text
                    ┌─────────────────────┐
                    │      Terraform      │
                    │   Infrastructure    │
                    │       as Code       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  Grafana Provider   │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
       ┌────────────┐   ┌────────────┐   ┌────────────┐
       │ Organizations│  │ Data Sources│  │ Dashboards │
       └────────────┘   └─────┬──────┘   └────────────┘
                               │
                         ┌─────▼──────┐
                         │ Prometheus │
                         └────────────┘
```

---

# 🎯 Learning Objectives

By completing this lab, you will learn how to:

* 🐧 Install and configure Terraform on Linux
* 📊 Deploy Grafana using Docker
* 🔐 Configure Grafana authentication
* 🏗️ Use Terraform to provision Grafana resources
* 📡 Configure Prometheus as a Grafana data source
* 📈 Provision Grafana dashboards automatically
* 👥 Manage Grafana users and teams
* 🔑 Configure dashboard permissions
* 🚨 Configure Grafana alert notification resources
* 🌿 Manage Grafana configuration through version control
* 💾 Back up Terraform state
* ♻️ Test infrastructure recovery
* 🔍 Troubleshoot Terraform/Grafana integration problems

---

# 🧰 Technology Stack

| Technology          | Purpose                        |
| ------------------- | ------------------------------ |
| 🐧 Ubuntu/Linux     | Lab operating system           |
| 🏗️ Terraform       | Infrastructure as Code         |
| 📊 Grafana          | Monitoring and visualization   |
| 🔥 Prometheus       | Metrics collection and storage |
| 🐳 Docker           | Grafana container runtime      |
| 📄 JSON             | Dashboard definitions          |
| 📝 HCL              | Terraform configuration        |
| 🔗 Grafana Provider | Terraform/Grafana integration  |
| 🌿 Git              | Configuration version control  |
| 🛠️ cURL            | API testing                    |
| 🔎 jq               | JSON inspection                |

---

# ✅ Prerequisites

Before beginning, you should have:

* Basic Linux command-line knowledge
* Familiarity with `sudo`, `systemctl`, and shell commands
* Basic understanding of JSON
* Basic understanding of HCL
* Basic monitoring knowledge
* Understanding of Infrastructure as Code concepts
* Familiarity with `nano`, `vim`, or another text editor

---

# ☁️ Lab Environment

The lab can be performed on an **Al Nafi Linux cloud machine**.

The environment starts with a bare Linux system, so the required tools are installed during the exercise.

### Expected Components

```text
Linux Host
   │
   ├── Terraform
   │
   ├── Docker
   │     └── Grafana :3000
   │
   └── Prometheus :9090
```

---

# 🏁 Task 1 — Environment Setup

## 🔄 Step 1.1 — Update Linux

Update the package repositories and install common dependencies.

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y \
  curl \
  wget \
  unzip \
  software-properties-common \
  apt-transport-https \
  ca-certificates \
  gnupg \
  lsb-release \
  jq
```

### ✅ Verification

```bash
curl --version
wget --version
jq --version
```

---

## 🏗️ Step 1.2 — Install Terraform

Download Terraform:

```bash
wget https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_linux_amd64.zip
```

Extract the archive:

```bash
unzip terraform_1.6.6_linux_amd64.zip
```

Move Terraform into the system path:

```bash
sudo mv terraform /usr/local/bin/
```

Verify:

```bash
terraform version
```

Expected output should show Terraform `v1.6.6`.

---

## 🐳 Step 1.3 — Install Docker

Install Docker:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Add the current user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Start Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

Verify:

```bash
sudo docker --version
sudo systemctl status docker
```

> ⚠️ **Note:** You may need to log out and back in for the Docker group membership to become active.

---

# 📊 Task 2 — Deploy Grafana

## 📁 Step 2.1 — Create Grafana Storage

```bash
mkdir -p ~/grafana-data
```

---

## 🚀 Step 2.2 — Start Grafana

```bash
sudo docker run -d \
  --name grafana \
  -p 3000:3000 \
  -v ~/grafana-data:/var/lib/grafana \
  -e "GF_SECURITY_ADMIN_PASSWORD=admin123" \
  -e "GF_USERS_ALLOW_SIGN_UP=false" \
  grafana/grafana:latest
```

Verify the container:

```bash
sudo docker ps
```

---

## 🔎 Step 2.3 — Verify Grafana

Allow Grafana time to initialize:

```bash
sleep 30
```

Test the HTTP endpoint:

```bash
curl -I http://localhost:3000
```

Inspect logs:

```bash
sudo docker logs grafana
```

### 🌐 Grafana

```text
URL:      http://localhost:3000
Username: admin
Password: admin123
```

> 🔐 **Production Note:** Never use `admin123` in a production environment. Store credentials in Terraform variables, environment variables, a secrets manager, or another secure mechanism.

---

# 🏗️ Task 3 — Create Terraform Project

## 📂 Step 3.1 — Create Project Structure

```bash
mkdir -p ~/terraform-grafana-lab
cd ~/terraform-grafana-lab

mkdir -p dashboards
mkdir -p datasources
mkdir -p terraform
```

The resulting structure:

```text
terraform-grafana-lab/
├── dashboards/
│   ├── system-metrics.json
│   ├── test-dashboard.json
│   └── simple-dashboard.json
│
├── datasources/
│
└── terraform/
    ├── main.tf
    └── variables.tf
```

---

# 🔌 Task 4 — Configure Terraform and Grafana

## 📝 Step 4.1 — Create `main.tf`

```bash
cat > ~/terraform-grafana-lab/terraform/main.tf << 'EOF'
terraform {
  required_providers {
    grafana = {
      source  = "grafana/grafana"
      version = "~> 2.9.0"
    }
  }
}

provider "grafana" {
  url  = "http://localhost:3000"
  auth = "admin:admin123"
}

resource "grafana_organization" "lab_org" {
  name         = "Lab Organization"
  admin_user   = "admin"
  create_users = true
  admins       = ["admin"]
}

resource "grafana_folder" "monitoring_folder" {
  title  = "Monitoring Dashboards"
  org_id = grafana_organization.lab_org.id
}
EOF
```

---

## ⚙️ Step 4.2 — Initialize Terraform

```bash
cd ~/terraform-grafana-lab/terraform

terraform init
```

Verify:

```bash
terraform version
terraform providers
```

---

# 🔐 Task 5 — Configure Terraform Variables

Create `variables.tf`:

```bash
cat > variables.tf << 'EOF'
variable "grafana_url" {
  description = "Grafana server URL"
  type        = string
  default     = "http://localhost:3000"
}

variable "grafana_admin_password" {
  description = "Grafana admin password"
  type        = string
  default     = "admin123"
  sensitive   = true
}

variable "organization_name" {
  description = "Name of the Grafana organization"
  type        = string
  default     = "Lab Organization"
}
EOF
```

> 💡 **Best Practice:** For production, avoid storing credentials directly in `.tf` files. Use environment variables, a secure secrets manager, or Terraform Cloud/Enterprise variable storage.

---

# 🔥 Task 6 — Install Prometheus

## 📥 Step 6.1 — Download Prometheus

```bash
cd ~

wget https://github.com/prometheus/prometheus/releases/download/v2.48.0/prometheus-2.48.0.linux-amd64.tar.gz
```

Extract:

```bash
tar xvfz prometheus-2.48.0.linux-amd64.tar.gz
```

Enter the directory:

```bash
cd prometheus-2.48.0.linux-amd64
```

---

## ⚙️ Step 6.2 — Configure Prometheus

```bash
cat > prometheus.yml << 'EOF'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
EOF
```

Start Prometheus:

```bash
nohup ./prometheus \
  --config.file=prometheus.yml \
  --storage.tsdb.path=./data \
  > prometheus.log 2>&1 &
```

Verify:

```bash
sleep 10

curl http://localhost:9090/api/v1/status/config
```

---

# 📡 Task 7 — Manage Grafana Data Sources with Terraform

Add the following to `main.tf`:

```bash
cat >> ~/terraform-grafana-lab/terraform/main.tf << 'EOF'

resource "grafana_data_source" "prometheus" {
  type       = "prometheus"
  name       = "Prometheus"
  url        = "http://localhost:9090"
  is_default = true
  org_id     = grafana_organization.lab_org.id

  json_data_encoded = jsonencode({
    httpMethod   = "POST"
    queryTimeout = "60s"
  })
}

resource "grafana_data_source" "testdata" {
  type   = "testdata"
  name   = "TestData"
  org_id = grafana_organization.lab_org.id
}
EOF
```

---

## 🚀 Step 7.1 — Apply Data Sources

```bash
terraform fmt
terraform validate
terraform plan
terraform apply -auto-approve
```

Verify:

```bash
curl -u admin:admin123 \
  http://localhost:3000/api/datasources | jq '.'
```

---

# 📈 Task 8 — Create Dashboard JSON

## 🖥️ Step 8.1 — System Metrics Dashboard

```bash
cd ~/terraform-grafana-lab

cat > dashboards/system-metrics.json << 'EOF'
{
  "dashboard": {
    "id": null,
    "title": "System Metrics Dashboard",
    "tags": [
      "terraform",
      "system",
      "monitoring"
    ],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "CPU Usage",
        "type": "stat",
        "targets": [
          {
            "expr": "100 - (avg(irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "min": 0,
            "max": 100
          }
        },
        "gridPos": {
          "h": 8,
          "w": 12,
          "x": 0,
          "y": 0
        }
      },
      {
        "id": 2,
        "title": "Memory Usage",
        "type": "stat",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "min": 0,
            "max": 100
          }
        },
        "gridPos": {
          "h": 8,
          "w": 12,
          "x": 12,
          "y": 0
        }
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "30s"
  }
}
EOF
```

Validate the JSON:

```bash
jq '.' dashboards/system-metrics.json
```

---

# 🧪 Task 9 — Create TestData Dashboard

```bash
cat > dashboards/test-dashboard.json << 'EOF'
{
  "dashboard": {
    "id": null,
    "title": "Test Data Dashboard",
    "tags": [
      "terraform",
      "test"
    ],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Random Walk",
        "type": "timeseries",
        "targets": [
          {
            "datasource": {
              "type": "testdata",
              "uid": "testdata"
            },
            "refId": "A",
            "scenarioId": "random_walk"
          }
        ],
        "gridPos": {
          "h": 8,
          "w": 24,
          "x": 0,
          "y": 0
        }
      },
      {
        "id": 2,
        "title": "Linear Growth",
        "type": "timeseries",
        "targets": [
          {
            "datasource": {
              "type": "testdata",
              "uid": "testdata"
            },
            "refId": "A",
            "scenarioId": "slow_query"
          }
        ],
        "gridPos": {
          "h": 8,
          "w": 24,
          "x": 0,
          "y": 8
        }
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "5s"
  }
}
EOF
```

Validate:

```bash
jq '.' dashboards/test-dashboard.json
```

---

# 🏗️ Task 10 — Provision Dashboards with Terraform

Add dashboard resources:

```bash
cat >> ~/terraform-grafana-lab/terraform/main.tf << 'EOF'

resource "grafana_dashboard" "test_dashboard" {
  org_id      = grafana_organization.lab_org.id
  folder      = grafana_folder.monitoring_folder.id
  config_json = file("../dashboards/test-dashboard.json")
}

resource "grafana_dashboard" "system_dashboard" {
  org_id      = grafana_organization.lab_org.id
  folder      = grafana_folder.monitoring_folder.id
  config_json = file("../dashboards/system-metrics.json")
}
EOF
```

Format and validate:

```bash
terraform fmt
terraform validate
```

Review:

```bash
terraform plan
```

Deploy:

```bash
terraform apply -auto-approve
```

Verify:

```bash
curl -u admin:admin123 \
  http://localhost:3000/api/search | jq '.'
```

---

# 👥 Task 11 — Configure Users and Teams

Add a monitoring team:

```bash
cat >> ~/terraform-grafana-lab/terraform/main.tf << 'EOF'

resource "grafana_team" "monitoring_team" {
  name   = "Monitoring Team"
  email  = "monitoring@company.com"
  org_id = grafana_organization.lab_org.id
}

resource "grafana_user" "monitoring_user" {
  email    = "monitor@company.com"
  name     = "Monitor User"
  login    = "monitor"
  password = "monitor123"
  is_admin = false
}

resource "grafana_organization_user" "monitoring_user_org" {
  org_id  = grafana_organization.lab_org.id
  user_id = grafana_user.monitoring_user.id
  role    = "Editor"
}

resource "grafana_team_membership" "monitoring_user_team" {
  team_id = grafana_team.monitoring_team.id
  user_id = grafana_user.monitoring_user.id
}
EOF
```

Apply:

```bash
terraform fmt
terraform plan
terraform apply -auto-approve
```

---

# 🔑 Task 12 — Configure Dashboard Permissions

```bash
cat >> ~/terraform-grafana-lab/terraform/main.tf << 'EOF'

resource "grafana_dashboard_permission" "test_dashboard_permissions" {
  dashboard_id = grafana_dashboard.test_dashboard.id
  org_id       = grafana_organization.lab_org.id

  permissions {
    role       = "Editor"
    permission = "Edit"
  }

  permissions {
    role       = "Viewer"
    permission = "View"
  }

  permissions {
    team_id    = grafana_team.monitoring_team.id
    permission = "Edit"
  }
}
EOF
```

Apply:

```bash
terraform fmt
terraform plan
terraform apply -auto-approve
```

---

# 🚨 Task 13 — Configure Alert Notifications

Create an email contact point:

```bash
cat >> ~/terraform-grafana-lab/terraform/main.tf << 'EOF'

resource "grafana_contact_point" "email_contact" {
  org_id = grafana_organization.lab_org.id
  name   = "email-notifications"

  email {
    addresses = ["admin@company.com"]
    subject   = "Grafana Alert"
  }
}

resource "grafana_notification_policy" "alert_policy" {
  org_id          = grafana_organization.lab_org.id
  contact_point   = grafana_contact_point.email_contact.name
  group_by        = ["alertname"]
  group_wait      = "10s"
  group_interval  = "5m"
  repeat_interval = "12h"
}
EOF
```

Deploy:

```bash
terraform fmt
terraform validate
terraform plan
terraform apply -auto-approve
```

> 📌 **Important:** An email contact point normally also requires Grafana SMTP to be configured for actual email delivery.

---

# 🧪 Task 14 — Validate the Deployment

## 🔍 Step 14.1 — Verify Dashboards

```bash
curl -u admin:admin123 \
  http://localhost:3000/api/search | jq '.'
```

Retrieve a dashboard UID:

```bash
DASHBOARD_UID=$(curl -s -u admin:admin123 \
  http://localhost:3000/api/search \
  | grep -o '"uid":"[^"]*"' \
  | head -1 \
  | cut -d'"' -f4)

echo "Dashboard UID: $DASHBOARD_UID"
```

Retrieve dashboard JSON:

```bash
curl -u admin:admin123 \
  "http://localhost:3000/api/dashboards/uid/$DASHBOARD_UID" | jq '.'
```

---

# 📡 Task 15 — Test Data Source Connectivity

Test Prometheus:

```bash
curl -u admin:admin123 \
  "http://localhost:3000/api/datasources/proxy/1/api/v1/query?query=up"
```

List data sources:

```bash
curl -u admin:admin123 \
  http://localhost:3000/api/datasources \
  | jq '.[] | {id: .id, name: .name, type: .type}'
```

---

# 🌳 Task 16 — Inspect Terraform State

Display the complete state:

```bash
terraform show
```

List resources:

```bash
terraform state list
```

Inspect a dashboard:

```bash
terraform state show grafana_dashboard.test_dashboard
```

This demonstrates that Terraform maintains a record of the resources it manages.

---

# 🧪 Task 17 — Test Configuration Changes

Create another dashboard:

```bash
cd ~/terraform-grafana-lab

cat > dashboards/simple-dashboard.json << 'EOF'
{
  "dashboard": {
    "id": null,
    "title": "Simple Monitoring Dashboard",
    "tags": [
      "terraform",
      "simple"
    ],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Server Uptime",
        "type": "stat",
        "targets": [
          {
            "datasource": {
              "type": "testdata",
              "uid": "testdata"
            },
            "refId": "A",
            "scenarioId": "random_walk"
          }
        ],
        "gridPos": {
          "h": 6,
          "w": 12,
          "x": 0,
          "y": 0
        }
      }
    ],
    "time": {
      "from": "now-6h",
      "to": "now"
    },
    "refresh": "1m"
  }
}
EOF
```

Add it to Terraform:

```bash
cat >> terraform/main.tf << 'EOF'

resource "grafana_dashboard" "simple_dashboard" {
  org_id      = grafana_organization.lab_org.id
  folder      = grafana_folder.monitoring_folder.id
  config_json = file("../dashboards/simple-dashboard.json")
}
EOF
```

Deploy:

```bash
cd terraform

terraform fmt
terraform validate
terraform plan
terraform apply -auto-approve
```

---

# 📤 Task 18 — Create Terraform Outputs

Add useful outputs:

```bash
cat >> ~/terraform-grafana-lab/terraform/main.tf << 'EOF'

output "grafana_url" {
  description = "Grafana server URL"
  value       = "http://localhost:3000"
}

output "organization_id" {
  description = "Created organization ID"
  value       = grafana_organization.lab_org.id
}

output "dashboard_urls" {
  description = "Dashboard URLs"
  value = {
    test_dashboard   = "http://localhost:3000/d/${grafana_dashboard.test_dashboard.uid}"
    system_dashboard = "http://localhost:3000/d/${grafana_dashboard.system_dashboard.uid}"
    simple_dashboard = "http://localhost:3000/d/${grafana_dashboard.simple_dashboard.uid}"
  }
}

output "data_source_ids" {
  description = "Data source IDs"
  value = {
    prometheus = grafana_data_source.prometheus.id
    testdata   = grafana_data_source.testdata.id
  }
}
EOF
```

Apply:

```bash
terraform fmt
terraform validate
terraform apply -auto-approve
```

Display outputs:

```bash
terraform output
```

---

# 💾 Task 19 — Back Up Terraform State

Create a backup directory:

```bash
mkdir -p ~/terraform-backups
```

Back up the state:

```bash
cp terraform.tfstate \
  ~/terraform-backups/terraform.tfstate.backup
```

Export state as JSON:

```bash
terraform show -json \
  > ~/terraform-backups/terraform-state.json
```

Verify:

```bash
ls -lh ~/terraform-backups/
```

> 🔐 **Security Warning:** Terraform state may contain sensitive values. Do not commit `terraform.tfstate` or state backups to a public Git repository.

---

# ♻️ Task 20 — Disaster Recovery Test

Create a Terraform plan:

```bash
terraform plan -out=grafana-plan
```

Inspect it:

```bash
terraform show grafana-plan
```

The plan demonstrates that the infrastructure can be recreated from the Terraform configuration.

---

# 🌿 Task 21 — Version Control

Initialize Git:

```bash
cd ~/terraform-grafana-lab

git init
```

Create a `.gitignore`:

```bash
cat > .gitignore << 'EOF'
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
*.tfvars
*.tfvars.json
crash.log
crash.*.log
grafana-plan
terraform-provider-*
EOF
```

Check the repository:

```bash
git status
```

Add the infrastructure files:

```bash
git add .
```

Create the first commit:

```bash
git commit -m "Add Grafana Terraform infrastructure"
```

### 🌳 Recommended Workflow

```text
feature/dashboard-change
          │
          ▼
       git commit
          │
          ▼
      Pull Request
          │
          ▼
       Code Review
          │
          ▼
        main
          │
          ▼
 terraform plan/apply
          │
          ▼
      Grafana
```

This workflow allows Grafana configuration changes to be reviewed and tracked before deployment.

---

# 🧹 Task 22 — Cleanup

When the lab is complete, Terraform-managed Grafana resources can be removed with:

```bash
terraform destroy
```

Confirm the operation when prompted.

To remove the Grafana container:

```bash
sudo docker stop grafana
sudo docker rm grafana
```

Optional data cleanup:

```bash
rm -rf ~/grafana-data
```

Stop Prometheus:

```bash
pkill prometheus
```

> ⚠️ **Warning:** Removing `~/grafana-data` permanently deletes the Grafana container's persisted data.

---

# 🛠️ Troubleshooting

## ❌ Issue 1 — Grafana Connection Failed

Check the container:

```bash
sudo docker ps | grep grafana
```

Check logs:

```bash
sudo docker logs grafana
```

Restart:

```bash
sudo docker restart grafana
```

Test:

```bash
sleep 30
curl -I http://localhost:3000
```

---

## ❌ Issue 2 — Dashboard Creation Failed

Validate the JSON:

```bash
jq '.' dashboards/test-dashboard.json
```

Check Grafana API:

```bash
curl -u admin:admin123 \
  -X POST \
  -H "Content-Type: application/json" \
  -d @dashboards/test-dashboard.json \
  http://localhost:3000/api/dashboards/db
```

Also inspect Terraform errors:

```bash
terraform plan
```

---

## ❌ Issue 3 — Prometheus Is Not Available

Check Prometheus:

```bash
curl http://localhost:9090/api/v1/status/config
```

Check the process:

```bash
ps aux | grep prometheus
```

Inspect logs:

```bash
tail -f ~/prometheus-2.48.0.linux-amd64/prometheus.log
```

Restart:

```bash
pkill prometheus

cd ~/prometheus-2.48.0.linux-amd64

nohup ./prometheus \
  --config.file=prometheus.yml \
  --storage.tsdb.path=./data \
  > prometheus.log 2>&1 &
```

---

## ❌ Issue 4 — Terraform State Problem

First inspect the state:

```bash
terraform state list
```

Back up the current state before attempting recovery.

Restore the lab backup if appropriate:

```bash
cp ~/terraform-backups/terraform.tfstate.backup \
  terraform.tfstate
```

Then inspect:

```bash
terraform show
```

For resources that already exist outside Terraform, use `terraform import` where supported and appropriate.

> 💡 **Best Practice:** Avoid manually editing Terraform state unless you fully understand the consequences.

---

# 🔎 Useful Validation Commands

### Terraform

```bash
terraform fmt
terraform validate
terraform plan
terraform apply
terraform show
terraform state list
```

### Docker

```bash
sudo docker ps
sudo docker logs grafana
sudo docker restart grafana
```

### Grafana API

```bash
curl -u admin:admin123 http://localhost:3000/api/search
```

### Prometheus

```bash
curl http://localhost:9090/api/v1/status/config
```

### JSON

```bash
jq '.' dashboards/system-metrics.json
```

---

# 📁 Final Project Structure

After completing the lab, your project should resemble:

```text
terraform-grafana-lab/
│
├── .gitignore
├── README.md
│
├── dashboards/
│   ├── system-metrics.json
│   ├── test-dashboard.json
│   └── simple-dashboard.json
│
├── datasources/
│
└── terraform/
    ├── main.tf
    ├── variables.tf
    ├── .terraform/
    ├── .terraform.lock.hcl
    └── terraform.tfstate
```

---

# 🏆 Lab Completion Checklist

* [ ] Install Terraform
* [ ] Install Docker
* [ ] Deploy Grafana
* [ ] Verify Grafana API
* [ ] Create Terraform project
* [ ] Install Grafana Terraform provider
* [ ] Create Grafana organization
* [ ] Create Grafana folder
* [ ] Install Prometheus
* [ ] Configure Prometheus data source
* [ ] Configure TestData data source
* [ ] Create dashboard JSON
* [ ] Provision dashboards with Terraform
* [ ] Create Grafana team
* [ ] Create Grafana user
* [ ] Assign organization permissions
* [ ] Configure dashboard permissions
* [ ] Configure alert contact point
* [ ] Configure notification policy
* [ ] Validate Terraform state
* [ ] Create additional dashboard
* [ ] Configure Terraform outputs
* [ ] Back up Terraform state
* [ ] Test disaster recovery
* [ ] Initialize Git repository
* [ ] Create `.gitignore`
* [ ] Commit infrastructure configuration
* [ ] Test cleanup

---

# 🎓 Key Concepts Learned

## 🏗️ Infrastructure as Code

Terraform allows infrastructure to be described declaratively:

```text
Desired State
     │
     ▼
Terraform Configuration
     │
     ▼
terraform plan
     │
     ▼
terraform apply
     │
     ▼
Grafana Infrastructure
```

---

## 📊 Dashboard as Code

Grafana dashboards can be represented as JSON and managed through Terraform:

```text
Dashboard JSON
      │
      ▼
Terraform
      │
      ▼
Grafana Provider
      │
      ▼
Grafana Dashboard
```

This makes dashboard configuration easier to reproduce and version.

---

## 🌿 Version Control

Using Git provides:

* Change tracking
* Collaboration
* Code review
* Rollback capability
* Audit history
* Branch-based development

---

## 🔐 Access Control

Terraform can manage:

```text
Organization
     │
     ├── Users
     │
     ├── Teams
     │
     ├── Roles
     │
     └── Dashboard Permissions
```

---

# 🚀 Production Recommendations

For a production implementation, improve this lab with:

### 🔐 Secrets Management

Do not hard-code credentials:

```hcl
auth = "admin:admin123"
```

Instead, use secure variables or a secrets manager.

### 🗄️ Remote Terraform State

Use a remote backend rather than local state for team environments.

### 🔒 HTTPS

Use HTTPS/TLS for Grafana rather than plain HTTP.

### 🐳 Production Containers

Pin Grafana image versions instead of using:

```text
grafana/grafana:latest
```

### 📦 Provider Pinning

Keep provider versions controlled and commit:

```text
.terraform.lock.hcl
```

### 🔄 CI/CD

Automate:

```text
Git Push
   │
   ▼
CI Pipeline
   │
   ├── terraform fmt
   ├── terraform validate
   ├── terraform plan
   └── Security checks
   │
   ▼
Approval
   │
   ▼
terraform apply
```

---

# 🌟 Conclusion

This lab demonstrates how to transform Grafana administration from a manual configuration process into a repeatable **Infrastructure as Code workflow**.

You have built a Terraform-managed Grafana environment containing:

* 🏢 Organizations
* 📁 Dashboard folders
* 📡 Prometheus data sources
* 🧪 TestData sources
* 📊 Automated dashboards
* 👥 Users
* 👨‍👩‍👧 Teams
* 🔑 Dashboard permissions
* 🚨 Alert notification resources
* 💾 Terraform state backups
* 🌿 Git-based configuration management

The major advantage is that Grafana configuration is now represented as code rather than depending entirely on manual UI operations.

```text
              🚀 Grafana as Code 🚀

       ┌──────────────────────────────┐
       │        Git Repository        │
       └──────────────┬───────────────┘
                      │
                      ▼
       ┌──────────────────────────────┐
       │          Terraform           │
       └──────────────┬───────────────┘
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
      Dashboards  Data Sources  RBAC
          │           │           │
          └───────────┼───────────┘
                      ▼
       ┌──────────────────────────────┐
       │            Grafana           │
       │      📊 Observability        │
       └──────────────────────────────┘
```

## 💡 Final Takeaway

**Grafana + Terraform + Git = repeatable, version-controlled, automated observability infrastructure.**

These skills are directly applicable to modern DevOps, Cloud Engineering, SRE, and Platform Engineering environments where monitoring infrastructure must be **consistent, auditable, scalable, and reproducible**.
