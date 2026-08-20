# 📊 Introduction to Grafana

> 🚀 **Hands-On Monitoring & Observability Lab**
> Learn how to install Grafana, configure the Grafana server, explore its web interface, create dashboards, configure data sources, and understand the fundamentals of modern observability.

---

## 🏷️ Technology Stack

![Grafana](https://img.shields.io/badge/Grafana-Visualization-F46800?style=for-the-badge\&logo=grafana\&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-E95420?style=for-the-badge\&logo=ubuntu\&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-Scripting-121011?style=for-the-badge\&logo=gnubash\&logoColor=white)
![Systemd](https://img.shields.io/badge/Systemd-Service_Management-1793D1?style=for-the-badge\&logo=linux\&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C?style=for-the-badge\&logo=prometheus\&logoColor=white)
![HTTP](https://img.shields.io/badge/HTTP-Web_Protocol-005571?style=for-the-badge\&logo=httpie\&logoColor=white)

---

## 🎯 Lab Objectives

By completing this lab, you will learn how to:

* 🏗️ Understand Grafana architecture and core components
* 📦 Install Grafana on Linux using package management
* ⚙️ Configure and manage the Grafana server
* 🔄 Enable Grafana to start automatically at boot
* 🌐 Access and navigate the Grafana web interface
* 📊 Create your first Grafana dashboard
* 🧩 Understand dashboards, panels, and visualizations
* 🔌 Configure Grafana data sources
* 🧪 Use the built-in TestData DB data source
* 📈 Create a visualization using test data
* 🛠️ Troubleshoot common Grafana installation and service issues

---

## 📋 Prerequisites

Before starting, you should have:

* 🐧 Basic Linux command-line knowledge
* 🌐 Basic understanding of HTTP and web browsers
* ⚙️ Familiarity with Linux services and `systemctl`
* 🌐 Basic networking knowledge
* 🔐 Basic knowledge of `sudo`
* 💻 Access to an Al Nafi Linux cloud machine

---

# ☁️ Lab Environment

Al Nafi provides Linux-based cloud machines for this lab.

The machine is initially bare and required software will be installed during the exercises.

### 🖥️ Environment

| Component            | Details       |
| -------------------- | ------------- |
| Operating System     | Ubuntu/Linux  |
| Monitoring Platform  | Grafana       |
| Service Manager      | systemd       |
| Default Grafana Port | `3000`        |
| Access Method        | Web Browser   |
| Training Environment | Al Nafi Cloud |

---

# 🚀 Task 1 — Install Grafana

## 🔹 Step 1.1 — Update System Packages

First, update the local package index and upgrade installed packages.

### 💻 Command

```bash
sudo apt update
sudo apt upgrade -y
```

### ✅ Verification

```bash
apt list --upgradable
```

> 🎯 **Goal:** Ensure your Linux system has the latest available package information and updates.

---

## 🔹 Step 1.2 — Install Required Dependencies

Install the packages required to retrieve and install Grafana.

### 💻 Command

```bash
sudo apt install -y software-properties-common apt-transport-https wget curl
```

### 🧰 Technologies

![APT](https://img.shields.io/badge/APT-Package_Manager-E95420?style=flat-square\&logo=ubuntu)
![Curl](https://img.shields.io/badge/cURL-Network_Tool-073551?style=flat-square\&logo=curl)
![Wget](https://img.shields.io/badge/Wget-Downloader-000000?style=flat-square\&logo=gnu)

> 🎯 **Goal:** Prepare Ubuntu for downloading and installing Grafana packages.

---

## 🔹 Step 1.3 — Add the Grafana Repository

Add the Grafana signing key and package repository.

### 💻 Commands

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Then add the repository:

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | \
sudo tee -a /etc/apt/sources.list.d/grafana.list
```

### 🔎 Verify Repository

```bash
cat /etc/apt/sources.list.d/grafana.list
```

> 🔐 **Security Note:** Package repositories use signing keys to help verify package authenticity.

---

## 🔹 Step 1.4 — Install Grafana

Update the package index and install Grafana.

### 💻 Commands

```bash
sudo apt update
sudo apt install -y grafana
```

### 🏷️ Technology

![Grafana](https://img.shields.io/badge/Grafana-Installed-F46800?style=flat-square\&logo=grafana\&logoColor=white)

---

## 🔹 Step 1.5 — Verify Installation

Check the installed Grafana version.

### 💻 Command

```bash
grafana-server --version
```

### ✅ Expected Result

You should see output containing the installed Grafana version.

```text
Version: Grafana ...
```

> 🎉 **Checkpoint:** Grafana is now installed successfully.

---

# ⚙️ Task 2 — Set Up Grafana Server

## 🔹 Step 2.1 — Enable Grafana Service

Configure Grafana to start automatically when the system boots.

### 💻 Command

```bash
sudo systemctl enable grafana-server
```

### 🔄 Technology

![Systemd](https://img.shields.io/badge/systemd-Service_Management-1793D1?style=flat-square\&logo=linux)

> 🎯 **Goal:** Make Grafana persistent across system reboots.

---

## 🔹 Step 2.2 — Start Grafana

Start the Grafana server.

### 💻 Command

```bash
sudo systemctl start grafana-server
```

---

## 🔹 Step 2.3 — Check Grafana Status

Verify that the service is running.

### 💻 Command

```bash
sudo systemctl status grafana-server
```

### ✅ Expected Status

Look for:

```text
Active: active (running)
```

> 🟢 **Success:** Grafana is running correctly.

---

## 🔹 Step 2.4 — Verify Port 3000

Grafana normally listens on TCP port `3000`.

### 💻 Command

```bash
sudo netstat -tlnp | grep 3000
```

If `netstat` is unavailable, install it:

```bash
sudo apt install -y net-tools
```

Then run:

```bash
sudo netstat -tlnp | grep 3000
```

### 🌐 Port Information

|   Port | Protocol | Purpose               |
| -----: | -------- | --------------------- |
| `3000` | TCP      | Grafana Web Interface |

---

## 🔹 Step 2.5 — Test Local Connectivity

Test whether Grafana responds locally.

### 💻 Command

```bash
curl -I http://localhost:3000
```

### ✅ Expected Result

You should receive an HTTP response such as:

```text
HTTP/1.1 302 Found
```

or another valid HTTP response.

> 🌐 **Checkpoint:** The Grafana web server is responding.

---

# 🌐 Task 3 — Explore Grafana Web UI

## 🔹 Step 3.1 — Access Grafana

Open your browser and navigate to:

```text
http://localhost:3000
```

For a cloud machine, use the machine's accessible IP address:

```text
http://<SERVER-IP>:3000
```

### 🌐 Technology

![HTTP](https://img.shields.io/badge/HTTP-Web_Access-005571?style=flat-square\&logo=httpie)

> ⚠️ **Cloud Note:** If port `3000` is not publicly accessible, check the cloud firewall/security rules and local firewall configuration.

---

## 🔹 Step 3.2 — Initial Login

Use the default credentials provided by the Grafana installation.

```text
Username: admin
Password: admin
```

Grafana should prompt you to change the default password.

### 🔐 Security Best Practice

Immediately replace the default password with a strong unique password.

> 🔒 **Never use default administrator credentials in a production environment.**

---

# 🧭 Step 3.3 — Explore the Grafana Interface

After logging in, inspect the major areas of the Grafana interface.

### 📌 Main Components

```text
┌──────────────────────────────────────────────┐
│              Grafana Web UI                  │
├───────────────┬──────────────────────────────┤
│ Sidebar       │ Main Content Area            │
│               │                              │
│ Dashboards    │ Dashboards                   │
│ Explore       │ Panels                       │
│ Alerting      │ Visualizations               │
│ Connections   │ Configuration                │
│ Administration│                              │
└───────────────┴──────────────────────────────┘
```

### 🔎 Explore

* 📊 Dashboards
* 🔍 Explore
* 🚨 Alerting
* 🔌 Connections/Data Sources
* ⚙️ Administration
* 👤 User Preferences

---

# 📊 Task 4 — Explore Dashboards

## 🔹 Step 4.1 — Open Dashboards

Navigate to:

**Dashboards → Browse**

Explore:

* 📊 Existing dashboards
* 📁 Dashboard folders
* ▶️ Playlists
* 📸 Snapshots

### 🎯 Objective

Understand how Grafana organizes monitoring dashboards.

---

# ⚙️ Task 5 — Explore Configuration

Navigate through Grafana's administration/configuration areas.

Explore:

* 🔌 Data Sources
* 👥 Users
* 👨‍👩‍👧 Teams
* 🧩 Plugins
* 🎨 Preferences
* ⚙️ Server configuration

> 💡 **Concept:** Grafana is not primarily a database. It connects to external data sources and turns their data into dashboards and visualizations.

---

# 📈 Task 6 — Create Your First Dashboard

## 🔹 Step 6.1 — Create Dashboard

Click:

**+ → Dashboard**

Then select:

**Add new panel**

---

## 🔹 Step 6.2 — Explore Panel Editor

The panel editor contains several important areas.

### 🔎 Query

Used to define what data Grafana should retrieve.

### 📊 Visualization

Used to select how data should be displayed.

Examples:

* Time series
* Stat
* Gauge
* Bar chart
* Table

### ⚙️ Panel Options

Used to configure:

* Title
* Description
* Display settings
* Panel behavior

---

## 🔹 Step 6.3 — Configure Panel Title

Set:

```text
My First Panel
```

Then click:

**Apply**

---

## 🔹 Step 6.4 — Save Dashboard

Click the **Save dashboard** icon.

Set the dashboard name:

```text
My First Dashboard
```

Then save it.

### 🎉 Checkpoint

You have now created your first Grafana dashboard.

---

# 🔌 Task 7 — Configure a Data Source

Grafana requires a data source to retrieve real monitoring or application data.

Navigate to:

**Connections → Data Sources**

Depending on your Grafana version, the menu may appear under an administration/configuration section.

---

## 🔹 Step 7.1 — Explore Data Source Types

Explore available data sources such as:

![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C?style=flat-square\&logo=prometheus)
![InfluxDB](https://img.shields.io/badge/InfluxDB-Time_Series-22ADF6?style=flat-square\&logo=influxdb)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-4169E1?style=flat-square\&logo=postgresql)
![MySQL](https://img.shields.io/badge/MySQL-Database-4479A1?style=flat-square\&logo=mysql)
![Elasticsearch](https://img.shields.io/badge/Elasticsearch-Logs-005571?style=flat-square\&logo=elasticsearch)

Common examples include:

* 📈 Prometheus
* 📊 InfluxDB
* 🐘 PostgreSQL
* 🐬 MySQL
* 🔎 Elasticsearch
* 🧪 TestData DB

---

## 🔹 Step 7.2 — Configure TestData DB

Select:

**TestData DB**

Explore its configuration.

Then use:

**Save & Test**

> 🧪 **Purpose:** TestData DB allows you to experiment with Grafana visualizations without requiring an external monitoring system.

---

# 📉 Task 8 — Create a Panel Using Test Data

## 🔹 Step 8.1 — Open Your Dashboard

Navigate to:

**Dashboards → Browse → My First Dashboard**

---

## 🔹 Step 8.2 — Add Panel

Select:

**Add panel → Add new panel**

---

## 🔹 Step 8.3 — Configure Query

Select:

```text
Data source: TestData DB
Scenario: Random Walk
```

You should see generated test data.

### 📈 Expected Visualization

A time-series graph should appear.

```text
Value
  │
  │      ╭──╮
  │  ╭───╯  ╰──╮
  │──╯         ╰────
  └────────────────── Time
```

---

## 🔹 Step 8.4 — Configure Panel

Set the title:

```text
Test Data Visualization
```

Click:

**Apply**

Then save the dashboard.

### 🎉 Checkpoint

You have successfully created a Grafana visualization using a data source.

---

# 📊 Task 9 — Explore Visualization Types

Edit your test data panel.

Click:

**Panel title → Edit**

Explore the visualization options.

### 📈 Time Series

Best for time-based metrics.

Examples:

* CPU usage
* Memory usage
* Network traffic
* Request rate

### 🔢 Stat

Displays a single important value.

Examples:

* Current CPU usage
* Number of active users
* Request count

### 🎯 Gauge

Useful for values compared against thresholds.

Examples:

* Disk utilization
* CPU utilization
* Temperature

### 📊 Bar Chart

Useful for comparing categories.

### 📋 Table

Useful for structured records and detailed data.

---

## 🔹 Step 9.1 — Change to Stat

Change the visualization to:

```text
Stat
```

Observe how the panel changes from a graph to a single numerical value.

Then switch it back to:

```text
Time series
```

Click:

**Apply**

---

# 🏗️ Task 10 — Understand Grafana Architecture

Grafana can be understood through several major components.

```text
                  ┌─────────────────┐
                  │      Users      │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │   Grafana UI    │
                  │   Dashboards    │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Grafana Backend │
                  └────────┬────────┘
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
       ┌──────────┐  ┌──────────┐  ┌──────────┐
       │Prometheus│  │ InfluxDB │  │ Database │
       └──────────┘  └──────────┘  └──────────┘
```

### 🧩 Core Components

| Component       | Purpose                               |
| --------------- | ------------------------------------- |
| 🖥️ Frontend    | Provides the web interface            |
| ⚙️ Backend      | Processes requests and serves Grafana |
| 🔌 Data Sources | Provide metrics/logs/data             |
| 📊 Dashboards   | Organize monitoring views             |
| 📈 Panels       | Individual visualizations             |
| 🧩 Plugins      | Extend Grafana functionality          |

---

# 🎨 Task 11 — Explore User Preferences

Click your user avatar and open:

**Preferences**

Explore:

### 🌙 UI Theme

* Light
* Dark

### 🏠 Home Dashboard

Choose a default dashboard.

### 🕐 Timezone

Configure how timestamps should be displayed.

> 💡 **Best Practice:** In distributed environments, consistent timezone configuration makes monitoring and incident investigation easier.

---

# ❓ Task 12 — Explore Help & Documentation

Click the **?** icon.

Explore:

* ⌨️ Keyboard shortcuts
* 📚 Documentation
* 🆘 Support
* 📖 Learning resources

---

# 🛠️ Troubleshooting

## ❌ Issue 1 — Grafana Service Fails to Start

Check service logs:

```bash
sudo journalctl -u grafana-server -f
```

Check status:

```bash
sudo systemctl status grafana-server
```

### 🔍 Possible Causes

* Invalid configuration
* Port conflict
* Permission problem
* Missing dependency
* Configuration syntax error

---

## ❌ Issue 2 — Cannot Access Grafana

Check the service:

```bash
sudo systemctl status grafana-server
```

Check firewall:

```bash
sudo ufw status
```

Check port:

```bash
sudo netstat -tlnp | grep 3000
```

Test locally:

```bash
curl -I http://localhost:3000
```

---

## ❌ Issue 3 — Forgot Admin Password

Use the Grafana CLI available for your installed version.

```bash
sudo grafana-cli admin reset-admin-password NEW_PASSWORD
```

> 🔐 Replace `NEW_PASSWORD` with a strong password and avoid exposing it in shell history where possible.

---

## ❌ Issue 4 — Port 3000 Already in Use

Identify the process using port `3000`:

```bash
sudo netstat -tlnp | grep 3000
```

Or:

```bash
sudo ss -tlnp | grep :3000
```

You can then stop the conflicting service or configure Grafana to use another port through:

```text
/etc/grafana/grafana.ini
```

---

# 🧪 Lab Verification Checklist

Use this checklist to confirm that the lab has been completed successfully.

* [ ] Grafana packages installed successfully
* [ ] Grafana version verified
* [ ] Grafana service enabled
* [ ] Grafana service started
* [ ] Service status shows `active (running)`
* [ ] Port `3000` verified
* [ ] Local HTTP connection tested
* [ ] Grafana Web UI accessed
* [ ] Admin password changed
* [ ] Grafana navigation explored
* [ ] First dashboard created
* [ ] TestData DB configured
* [ ] Test data panel created
* [ ] Multiple visualization types explored
* [ ] Grafana architecture understood
* [ ] Troubleshooting commands tested

---

# 🏆 Lab Conclusion

Congratulations! 🎉

You have completed the **Introduction to Grafana** lab and established the foundation required for Grafana-based monitoring and observability.

### ✅ What You Accomplished

**1. 📦 Installed Grafana**

You installed Grafana on a Linux system using package management.

**2. ⚙️ Configured Grafana**

You enabled, started, and verified the Grafana server service.

**3. 🌐 Accessed Grafana**

You connected to Grafana through its web interface.

**4. 📊 Created a Dashboard**

You created your first dashboard and added visualization panels.

**5. 🔌 Configured a Data Source**

You explored data source configuration and used TestData DB.

**6. 📈 Created Visualizations**

You created a panel using generated test data and explored different visualization types.

**7. 🏗️ Learned Grafana Architecture**

You learned how Grafana's frontend, backend, dashboards, panels, plugins, and external data sources work together.

---

# 🌍 Why Grafana Matters

Grafana is widely used for observability and visualization across modern infrastructure.

### 🖥️ Infrastructure Monitoring

Visualize:

* CPU utilization
* Memory consumption
* Disk usage
* Network traffic
* System availability

### 🚀 Application Monitoring

Monitor:

* Request rates
* Response times
* Error rates
* Application performance
* Service health

### 📊 Business Intelligence

Create dashboards for:

* KPIs
* Business metrics
* Customer activity
* Operational performance

### 🌐 IoT Monitoring

Visualize:

* Sensor measurements
* Device telemetry
* Environmental data
* Device health

### 📝 Log Analytics

Combine Grafana with suitable log data sources to visualize:

* Error trends
* Application logs
* Security events
* Operational activity

---

# 🚀 Next Steps

Now that Grafana is installed, continue with more advanced monitoring projects.

### 🔥 Recommended Learning Path

```text
Introduction to Grafana
        │
        ▼
Prometheus Integration
        │
        ▼
Node Exporter
        │
        ▼
Grafana Dashboards
        │
        ▼
PromQL Queries
        │
        ▼
Recording Rules
        │
        ▼
Grafana Alerting
        │
        ▼
Alertmanager
        │
        ▼
Advanced Observability
```

### 🎯 Future Projects

* 🔌 Connect Grafana to Prometheus
* 🖥️ Monitor Linux servers with Node Exporter
* 📊 Build production monitoring dashboards
* 🔎 Write advanced PromQL queries
* 🚨 Configure Grafana alerting
* 📬 Integrate Alertmanager
* 🐳 Monitor Docker containers
* ☸️ Monitor Kubernetes clusters
* ☁️ Build cloud infrastructure dashboards
* 📈 Implement complete observability stacks

---

# 🏅 Skills Gained

| Skill                      | Level       |
| -------------------------- | ----------- |
| Grafana Installation       | 🟢 Beginner |
| Linux Package Management   | 🟢 Beginner |
| systemd Service Management | 🟢 Beginner |
| Grafana Web UI             | 🟢 Beginner |
| Dashboard Creation         | 🟢 Beginner |
| Panel Configuration        | 🟢 Beginner |
| Data Source Configuration  | 🟢 Beginner |
| Visualization              | 🟢 Beginner |
| Grafana Architecture       | 🟢 Beginner |
| Troubleshooting            | 🟢 Beginner |

---

# 👨‍💻 Author

**Hafiz Muhammad Salman**

🚀 **Cloud DevOps Engineer | Linux Administrator**

Focused on:

`AWS` • `Linux` • `Docker` • `Kubernetes` • `Terraform` • `Ansible` • `Jenkins` • `Prometheus` • `Grafana` • `DevOps` • `Cloud`

---

## ⭐ Repository Support

If this lab helped you understand Grafana and monitoring fundamentals:

⭐ **Star the repository**
🍴 **Fork the repository**
🐛 **Report issues**
💡 **Suggest improvements**
🤝 **Contribute to the project**

---

<div align="center">

### 🚀 Learn • Build • Monitor • Automate • Improve

**Grafana + Prometheus + Linux = Powerful Observability**

⭐ **Keep Learning. Keep Building. Keep Monitoring.** ⭐

</div>
