<div align="center">

# 📊 Building Your First Dashboard
### Grafana & Prometheus System Monitoring Lab

![Grafana](https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Node Exporter](https://img.shields.io/badge/Node_Exporter-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)
![systemd](https://img.shields.io/badge/systemd-000000?style=for-the-badge&logo=linux&logoColor=white)

**Level:** Beginner &nbsp;|&nbsp; **Track:** Observability & Monitoring

</div>

---

## 📋 Table of Contents

- [🎯 Lab Objectives](#-lab-objectives)
- [✅ Prerequisites](#-prerequisites)
- [🖥️ Lab Environment](#️-lab-environment)
- [🧭 Lab Overview](#-lab-overview)
- [🚀 Task 1: Environment Setup and Grafana Installation](#-task-1-environment-setup-and-grafana-installation)
- [📊 Task 2: Create a New Dashboard](#-task-2-create-a-new-dashboard)
- [📈 Task 3: Add Panels with Visualizations](#-task-3-add-panels-with-visualizations)
- [🎨 Task 4: Customize Layout and Design](#-task-4-customize-layout-and-design)
- [🧪 Verification and Testing](#-verification-and-testing)
- [🛠️ Troubleshooting Common Issues](#️-troubleshooting-common-issues)
- [🔑 Key Concepts](#-key-concepts)
- [🏁 Lab Conclusion](#-lab-conclusion)

---

## 🎯 Lab Objectives

By the end of this lab, you will be able to:

| # | Objective |
|---|---|
| 1 | Install and configure Grafana on a Linux system |
| 2 | Create a new dashboard from scratch |
| 3 | Add and configure visualization panels — graphs, tables, and gauges |
| 4 | Customize dashboard layout and design elements |
| 5 | Connect data sources to visualizations |
| 6 | Apply best practices for dashboard organization and user experience |

---

## ✅ Prerequisites

| # | Requirement |
|---|---|
| 1 | Basic understanding of Linux command line operations |
| 2 | Familiarity with web browsers and basic web navigation |
| 3 | Understanding of basic data visualization concepts |
| 4 | Knowledge of system monitoring fundamentals |
| 5 | Basic understanding of time-series data |

---

## 🖥️ Lab Environment

> Al Nafi provides a bare-metal Linux cloud machine for this lab, with no tools pre-installed. Simply click **Start Lab** to access your dedicated environment — every piece of software used below is installed from scratch as part of the exercises.

---

## 🧭 Lab Overview

This lab walks through building your first comprehensive dashboard using **Grafana**, an open-source analytics and monitoring platform. You will create several types of visualizations and organize them into a professional-looking, production-style dashboard.

---

## 🚀 Task 1: Environment Setup and Grafana Installation

### 🔄 Subtask 1.1: Update System and Install Dependencies

```bash
# 🔃 refresh package index and upgrade existing packages
sudo apt update && sudo apt upgrade -y

# 📦 install required dependencies
sudo apt install -y wget curl software-properties-common apt-transport-https
```

### 📥 Subtask 1.2: Install Grafana

```bash
# 🔑 add the grafana gpg key
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# 📋 register the grafana apt repository
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# 🔃 refresh package list
sudo apt update

# ⬇️ install grafana
sudo apt install -y grafana
```

### ▶️ Subtask 1.3: Start and Enable Grafana Service

```bash
# ▶️ start the grafana service
sudo systemctl start grafana-server

# 🔁 enable grafana to launch automatically on boot
sudo systemctl enable grafana-server

# 🔍 confirm the service is healthy
sudo systemctl status grafana-server
```

### 🔌 Subtask 1.4: Install and Configure Data Source (Prometheus)

```bash
# 👤 create a dedicated, unprivileged prometheus user
sudo useradd --no-create-home --shell /bin/false prometheus

# 📁 create prometheus directories
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus

# ⬇️ download and unpack prometheus
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.40.0/prometheus-2.40.0.linux-amd64.tar.gz
tar xvf prometheus-2.40.0.linux-amd64.tar.gz
cd prometheus-2.40.0.linux-amd64

# 📋 install the prometheus binaries
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool

# 🗂️ install the console templates
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
```

### ⚙️ Subtask 1.5: Configure Prometheus

```bash
# 📝 write the prometheus scrape configuration
sudo tee /etc/prometheus/prometheus.yml > /dev/null <<EOF
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

# 🔐 fix ownership of the config file
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

### 📡 Subtask 1.6: Install Node Exporter

```bash
# ⬇️ download and unpack node exporter
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xvf node_exporter-1.5.0.linux-amd64.tar.gz

# 📋 install the node exporter binary
sudo cp node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/node_exporter
```

### 🧩 Subtask 1.7: Create Systemd Services

```bash
# 🧩 prometheus systemd unit
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
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
EOF

# 🧩 node exporter systemd unit
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF
```

### ▶️ Subtask 1.8: Start Services

```bash
# 🔄 reload systemd so it picks up the new units
sudo systemctl daemon-reload

# ▶️ start prometheus and node exporter
sudo systemctl start prometheus
sudo systemctl start node_exporter

# 🔁 enable both on boot
sudo systemctl enable prometheus
sudo systemctl enable node_exporter

# 🔍 verify every service is running
sudo systemctl status prometheus
sudo systemctl status node_exporter
sudo systemctl status grafana-server
```

---

## 📊 Task 2: Create a New Dashboard

### 🌐 Subtask 2.1: Access Grafana Web Interface

1. Open your browser and navigate to `http://localhost:3000`
2. Login with the default credentials:
   - **Username:** `admin`
   - **Password:** `admin`
3. Change the password when prompted (use a secure password)

### 🔗 Subtask 2.2: Add Prometheus Data Source

1. Click **Configuration** (gear icon) in the left sidebar
2. Select **Data Sources**
3. Click **Add data source**
4. Select **Prometheus**
5. Configure the data source:
   - **Name:** `Prometheus`
   - **URL:** `http://localhost:9090`
   - Leave other settings as default
6. Click **Save & Test**
7. Confirm you see the **"Data source is working"** message

### 🆕 Subtask 2.3: Create New Dashboard

1. Click the **+** icon in the left sidebar
2. Select **Dashboard**
3. Click **Add new panel** — this opens the panel editor interface

### ⚙️ Subtask 2.4: Configure Dashboard Settings

1. Click **Dashboard settings** (gear icon) at the top
2. In the **General** tab, configure:
   - **Title:** `My First System Monitoring Dashboard`
   - **Description:** `A comprehensive dashboard for system monitoring`
   - **Tags:** `system`, `monitoring`, `beginner`
3. Click **Save** to apply the settings

---

## 📈 Task 3: Add Panels with Visualizations

### 📉 Subtask 3.1: Create a Graph Panel for CPU Usage

1. Click **Add panel** → **Add new panel**
2. **Query** section:
   - Data source: **Prometheus**
   ```promql
   100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
   ```
   - **Legend:** `CPU Usage %`
3. **Panel** section:
   - **Title:** `CPU Usage Over Time`
   - **Description:** `Shows CPU utilization percentage over time`
4. **Visualization** section:
   - Select **Time series (graph)**
   - **Style:** Line, **Line width:** `2`, **Fill opacity:** `10`
5. **Field** section:
   - **Unit:** Percent (0–100), **Min:** `0`, **Max:** `100`
6. Click **Apply** to save the panel

### 📋 Subtask 3.2: Create a Table Panel for System Information

1. Click **Add panel** → **Add new panel**
2. **Query** section — add multiple queries:
   - Query A: `node_uname_info`
   - Query B: `node_boot_time_seconds`
   - Query C: `node_load1`
   - Query D: `node_load5`
   - Query E: `node_load15`
3. **Panel** section:
   - **Title:** `System Information`
   - **Description:** `Key system metrics and information`
4. **Visualization** section:
   - Select **Table**
   - Open the **Field** tab and add an **Override** per field to customize display names
5. Click **Apply** to save the panel

### ⏱️ Subtask 3.3: Create Gauge Panels for Key Metrics

**🧠 Memory Usage Gauge**
1. Click **Add panel** → **Add new panel**
2. Query:
   ```promql
   (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
   ```
   **Legend:** `Memory Usage`
3. **Title:** `Memory Usage` — **Description:** `Current memory utilization`
4. **Visualization:** Gauge — **Min:** `0`, **Max:** `100`, **Unit:** Percent (0–100)
5. **Thresholds:** 🟢 Green `0–70` · 🟡 Yellow `70–85` · 🔴 Red `85–100`
6. Click **Apply** to save

**💾 Disk Usage Gauge**
1. Click **Add panel** → **Add new panel**
2. Query:
   ```promql
   (1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100
   ```
   **Legend:** `Disk Usage`
3. **Title:** `Root Disk Usage` — **Description:** `Root filesystem utilization`
4. **Visualization:** Gauge, configured like the memory gauge, with thresholds appropriate for disk usage
5. Click **Apply** to save

**🌐 Network Traffic Gauge**
1. Click **Add panel** → **Add new panel**
2. Queries:
   ```promql
   irate(node_network_receive_bytes_total{device!="lo"}[5m]) * 8
   irate(node_network_transmit_bytes_total{device!="lo"}[5m]) * 8
   ```
3. **Title:** `Network Traffic` — **Description:** `Network receive and transmit rates`
4. **Visualization:** Stat or Gauge — **Unit:** bits/sec
5. Click **Apply** to save

### 📊 Subtask 3.4: Create Additional Visualization Panels

**📶 System Load Bar Chart**
- Add a new panel with visualization type **Bar chart**
- Query: `node_load1`, `node_load5`, `node_load15`
- **Title:** `System Load Average` — show the 1, 5, and 15-minute load averages

**🔢 Process Count Stat Panel**
- Add a new panel with visualization type **Stat**
- Query: `node_procs_running`
- **Title:** `Running Processes` — style as appropriate

---

## 🎨 Task 4: Customize Layout and Design

### 🧩 Subtask 4.1: Organize Panel Layout

- Drag and resize panels into a logical layout:
  - CPU usage graph at the top (full width)
  - Gauges (Memory, Disk, Network) in a row below
  - System information table on the left
  - Additional charts on the right
- Resize panels appropriately:
  - Enlarge the most important metrics (CPU, Memory)
  - Keep gauges easily readable
  - Give the table enough space for its data

### ⏲️ Subtask 4.2: Configure Time Range and Refresh

1. Click the time range selector at the top right
2. Set the default time range to **Last 1 hour**
3. Configure auto-refresh:
   - Open the refresh dropdown
   - Set it to **30s** for real-time monitoring
4. Save these settings as dashboard defaults

### 🧮 Subtask 4.3: Add Dashboard Variables

1. Go to **Dashboard settings → Variables**
2. Click **Add variable**
3. Create a `server` variable:
   - **Name:** `server`
   - **Type:** Query
   - **Data source:** Prometheus
   - **Query:** `label_values(node_uname_info, instance)`
   - **Multi-value:** Enable
   - **Include All option:** Enable
4. Click **Add** to save the variable

### 🔧 Subtask 4.4: Apply Variables to Panels

1. Edit each panel and update its queries to use the `$server` variable, e.g.:
   - Original: `node_cpu_seconds_total`
   - Modified: `node_cpu_seconds_total{instance=~"$server"}`
2. Apply this pattern to every relevant query
3. Test the variable by selecting different values from the dropdown

### 🎨 Subtask 4.5: Customize Visual Appearance

- **Panel titles and descriptions:** clear, descriptive titles; helpful descriptions for complex metrics; consistent naming
- **Color schemes:** consistent colors across panels; meaningful colors (🔴 red for alerts, 🟢 green for normal); thresholds with matching colors
- **Spacing and alignment:** consistent spacing between panels; proper alignment; grid snapping for precise positioning

### 📝 Subtask 4.6: Add Text and Annotation Panels

- Add a **Text panel** at the top:
  - **Title:** `Dashboard Overview`
  - **Content:** brief description of what the dashboard monitors, using markdown formatting
- Configure panel links:
  - Links to related dashboards
  - Links to documentation
  - Drill-down links where appropriate

### 💾 Subtask 4.7: Save and Share Dashboard

1. **Save the dashboard:**
   - Click the **Save** icon (disk icon)
   - Confirm the title and folder location
   - Add save notes describing your changes
2. **Export dashboard:**
   - Go to **Dashboard settings → JSON Model**
   - Copy the JSON for backup purposes
   - Save it to a file for version control
3. **Set up sharing (optional):**
   - Configure dashboard permissions
   - Create shareable links if needed
   - Set up snapshot sharing for external users

---

## 🧪 Verification and Testing

### ✅ Verify Dashboard Functionality

**Check data flow**
- [ ] All panels display data correctly
- [ ] Metrics are updating in real-time
- [ ] Time range changes affect all panels

**Test interactivity**
- [ ] Variable dropdowns work correctly
- [ ] Panel zoom and drill-down features work
- [ ] Refresh functionality works

**Validate visualizations**
- [ ] Gauges show correct ranges and thresholds
- [ ] Table data is formatted properly
- [ ] Graphs display trends accurately

### ⚡ Performance Testing

**Load testing**
- [ ] Dashboard opens correctly across different time ranges
- [ ] Tested with multiple concurrent users (if applicable)
- [ ] Query performance monitored in Prometheus

**Optimization checks**
- [ ] Query efficiency reviewed
- [ ] No unnecessary data requests
- [ ] Refresh intervals optimized

---

## 🛠️ Troubleshooting Common Issues

<details>
<summary>🔌 Data Source Connection Issues</summary>

If the Prometheus data source fails:

```bash
# 🔍 check prometheus service status
sudo systemctl status prometheus

# 📜 check prometheus logs
sudo journalctl -u prometheus -f

# ✅ verify prometheus is reachable
curl http://localhost:9090/api/v1/query?query=up
```

</details>

<details>
<summary>🖼️ Panel Display Issues</summary>

If panels don't show data:

- Check the query syntax in the panel editor
- Verify the time range covers the data's availability
- Confirm the correct data source is selected
- Review Prometheus targets at `http://localhost:9090/targets`

</details>

<details>
<summary>🐢 Performance Issues</summary>

If the dashboard loads slowly:

- Optimize queries by reducing time ranges
- Increase refresh intervals for less critical panels
- Limit the number of series returned by queries
- Use recording rules in Prometheus for complex calculations

</details>

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Grafana** | Open-source platform for querying, visualizing, and alerting on metrics |
| **Prometheus** | Time-series database and monitoring system that scrapes and stores metrics |
| **Node Exporter** | Exposes host-level hardware and OS metrics for Prometheus to scrape |
| **PromQL** | Prometheus's query language, used to select and aggregate time-series data |
| **Data Source** | The backend (e.g., Prometheus) a Grafana dashboard queries for data |
| **Panel** | A single visualization (graph, table, gauge, stat) within a dashboard |
| **Dashboard Variable** | A dropdown-driven placeholder (e.g., `$server`) that dynamically filters panel queries |
| **Threshold** | A configured value range (green/yellow/red) that colors a gauge or stat by severity |
| **Systemd Unit** | A service definition that manages a Linux background process (start, stop, boot behavior) |

---

## 🏁 Lab Conclusion

Congratulations! You have successfully completed **Lab 4: Building Your First Dashboard**.

### 🏆 Key Accomplishments

- Installed and configured Grafana on a Linux system from scratch
- Set up Prometheus and Node Exporter as data sources for system monitoring
- Created a comprehensive dashboard with multiple visualization types
- Implemented time series graphs, tables, gauges, and stat panels
- Customized dashboard layout and design for an optimal user experience
- Applied dashboard variables for dynamic filtering and interactivity
- Configured proper time ranges and refresh intervals for real-time monitoring

### 🧠 Skills Developed

- Dashboard design principles and best practices
- Data visualization techniques using different chart types
- Query writing and optimization for time-series data
- User interface customization and professional presentation
- System monitoring fundamentals using open-source tools

### 🌍 Real-World Applications

The dashboard you built reflects monitoring practices used in:

- **DevOps and Site Reliability Engineering** — system health monitoring
- **Infrastructure management** — capacity planning and performance optimization
- **Incident response** — quick identification of system issues
- **Business intelligence** — operational metrics and KPI tracking

### ⏭️ Next Steps

- Add alerting rules to notify when thresholds are exceeded
- Create additional dashboards for specific services or applications
- Implement advanced visualizations like heatmaps and histograms
- Set up dashboard automation using Infrastructure as Code tools
- Explore Grafana plugins for extended functionality

---

<div align="center">

![Al Nafi](https://img.shields.io/badge/Al%20Nafi-Cybersecurity%20Training-1e3a8a?style=for-the-badge)

</div>
