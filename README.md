# Grafana — Monitoring & Observability Labs

![Grafana](https://img.shields.io/badge/Grafana-Observability-orange?logo=grafana)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-orange?logo=prometheus)
![Loki](https://img.shields.io/badge/Loki-Logging-blue?logo=grafana)
![Tempo](https://img.shields.io/badge/Tempo-Tracing-purple?logo=grafana)
![Terraform](https://img.shields.io/badge/Terraform-IaC-623CE4?logo=terraform)
![Docker](https://img.shields.io/badge/Docker-Containers-2496ED?logo=docker)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Monitoring-326CE5?logo=kubernetes)
![Linux](https://img.shields.io/badge/Linux-Administration-FCC624?logo=linux)

## 📌 About This Repository

This repository is a **hands-on Grafana monitoring and observability learning project** developed as part of the **Al-Razzaq Programme**.

The purpose of this repository is to build practical expertise in **Grafana, monitoring, observability, visualization, alerting, logging, distributed tracing, cloud monitoring, infrastructure automation, and security**.

Grafana is an open-source observability platform that can visualize and analyze metrics, logs, and traces from multiple data sources. It supports dashboards, dynamic variables, alerting, and integration with tools such as Prometheus and Loki.

This repository applies those concepts through practical labs and real-world DevOps scenarios.

---

## 🎯 Repository Objectives

The main objectives of this repository are to:

* Learn Grafana from fundamentals to advanced concepts
* Install and configure Grafana on Linux
* Build professional monitoring dashboards
* Connect Grafana with different data sources
* Query monitoring data using PromQL
* Create dynamic dashboards using variables
* Configure Grafana alerting
* Integrate Grafana with Alertmanager
* Monitor Docker containers
* Monitor Kubernetes environments
* Integrate Grafana with cloud monitoring platforms
* Collect and visualize logs using Loki
* Implement distributed tracing using Tempo
* Automate Grafana resources with Terraform
* Implement Grafana RBAC and security controls
* Configure Single Sign-On (SSO)
* Manage Grafana dashboards through version control
* Develop practical production-oriented observability skills

---

## 📚 Labs Included

The repository contains a collection of progressive Grafana labs covering different areas of monitoring and observability.

### 🔰 Grafana Fundamentals

* Introduction to Grafana
* Installing and Configuring Grafana
* Building Your First Dashboard
* Connecting Grafana to Data Sources
* Querying Data with PromQL
* Exploring Advanced Grafana Panels
* Templating Dashboards in Grafana
* Creating Dynamic Dashboards with Variables

### 🚨 Monitoring & Alerting

* Setting Up Alerts in Grafana
* Integrating Grafana with Alertmanager
* Monitoring Cloud Infrastructure with Grafana
* Monitoring Docker Containers with Grafana
* Using Grafana for Kubernetes Monitoring

### 📊 Logs & Distributed Tracing

* Grafana with Loki for Logs
* Grafana with Tempo for Tracing

These labs demonstrate how metrics, logs, and traces can work together to provide a more complete observability solution.

### ☁️ Cloud & Infrastructure

* Grafana Cloud Integration
* Cloud infrastructure monitoring
* EC2/RDS-style monitoring scenarios
* Infrastructure and application metrics visualization

### 🔐 Security

* Implementing RBAC in Grafana
* Securing Grafana with SSO (Single Sign-On)

These exercises focus on controlling access to Grafana and integrating it with external authentication systems.

### ⚙️ Automation & DevOps

* Automating Grafana Dashboard Provisioning with Terraform
* Version Control for Grafana Dashboards

These labs demonstrate Infrastructure as Code and Git-based management practices for Grafana resources.

---

## 🏗️ Observability Architecture

The repository explores an observability architecture built around the following components:

```text
                    ┌─────────────────────┐
                    │   Applications      │
                    │   Infrastructure     │
                    │   Containers         │
                    │   Kubernetes         │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
        ┌──────────┐     ┌──────────┐     ┌──────────┐
        │Prometheus│     │   Loki   │     │  Tempo   │
        │ Metrics  │     │   Logs   │     │  Traces  │
        └────┬─────┘     └────┬─────┘     └────┬─────┘
             │                │                │
             └────────────────┼────────────────┘
                              ▼
                       ┌─────────────┐
                       │   Grafana   │
                       │ Dashboards  │
                       │  Alerting   │
                       │ Visualization│
                       └──────┬──────┘
                              │
                              ▼
                       ┌─────────────┐
                       │ Alertmanager│
                       │ Notifications│
                       └─────────────┘
```

---

## 🛠️ Technologies Covered

| Technology          | Purpose                               |
| ------------------- | ------------------------------------- |
| **Grafana**         | Visualization, dashboards, alerting   |
| **Prometheus**      | Metrics collection and monitoring     |
| **PromQL**          | Metrics querying                      |
| **Alertmanager**    | Alert routing and notifications       |
| **Loki**            | Log aggregation                       |
| **Tempo**           | Distributed tracing                   |
| **Docker**          | Container monitoring                  |
| **Kubernetes**      | Container orchestration monitoring    |
| **Terraform**       | Infrastructure as Code and automation |
| **Linux**           | Server and monitoring environment     |
| **Cloud Platforms** | Cloud infrastructure monitoring       |
| **Git/GitHub**      | Version control and collaboration     |

---

## 📈 What You Can Learn

By working through this repository, you can develop practical skills in:

### Monitoring

* CPU monitoring
* Memory monitoring
* Disk monitoring
* Network monitoring
* Application metrics
* Infrastructure metrics
* Container metrics
* Kubernetes metrics

### Visualization

* Time-series panels
* Gauge panels
* Stat panels
* Tables
* Logs panels
* Variables
* Dashboard filters
* Multi-panel dashboards

### Alerting

* Grafana alert rules
* Threshold-based alerts
* Alert evaluation
* Alert routing
* Alertmanager integration
* Notification workflows

### Observability

* Metrics
* Logs
* Traces
* Service performance
* Infrastructure health
* Application troubleshooting
* Correlation between metrics, logs, and traces

### DevOps Automation

* Terraform-based Grafana management
* Dashboard provisioning
* Git-based dashboard version control
* Reproducible monitoring configurations

### Security

* Grafana authentication
* RBAC
* SSO
* Access control
* Secure monitoring environments

---

## 📂 Repository Structure

```text
Grafana/
│
├── Automating Grafana Dashboard Provisioning with Terraform/
├── Building Your First Dashboard/
├── Connecting Grafana to Data Sources/
├── Creating Dynamic Dashboards with Variables/
├── Exploring Advanced Grafana Panels/
├── Grafana Cloud Integration/
├── Grafana with Loki for Logs/
├── Grafana with Tempo for Tracing/
├── Implementing RBAC in Grafana/
├── Installing and Configuring Grafana/
├── Integrating Grafana with Alertmanager/
├── Introduction to Grafana/
├── Monitoring Cloud Infrastructure with Grafana/
├── Monitoring Docker Containers with Grafana/
├── Querying Data with PromQL/
├── Securing Grafana with SSO/
├── Setting Up Alerts in Grafana/
├── Templating Dashboards in Grafana/
├── Using Grafana for Kubernetes Monitoring/
├── Version Control for Grafana Dashboards/
│
└── README.md
```

---

## 🚀 Learning Path

A recommended order for completing the repository is:

```text
1. Introduction to Grafana
             ↓
2. Installing and Configuring Grafana
             ↓
3. Connecting Grafana to Data Sources
             ↓
4. Building Your First Dashboard
             ↓
5. Querying Data with PromQL
             ↓
6. Exploring Advanced Grafana Panels
             ↓
7. Creating Dynamic Dashboards
             ↓
8. Templating Dashboards
             ↓
9. Setting Up Alerts
             ↓
10. Alertmanager Integration
             ↓
11. Docker Monitoring
             ↓
12. Kubernetes Monitoring
             ↓
13. Cloud Infrastructure Monitoring
             ↓
14. Loki for Logs
             ↓
15. Tempo for Tracing
             ↓
16. RBAC & SSO
             ↓
17. Terraform Automation
             ↓
18. Dashboard Version Control
```

---

## 🎓 Programme

This repository is part of the **Al-Razzaq Programme** and is intended to demonstrate practical learning and implementation of Grafana-based monitoring and observability.

The repository progresses from basic Grafana concepts toward more advanced **Cloud, DevOps, Security, Monitoring, Logging, Tracing, and Infrastructure as Code** practices.

---

## 💡 Real-World Use Cases

The skills demonstrated in this repository can be applied to:

* Cloud infrastructure monitoring
* Production server monitoring
* Kubernetes observability
* Docker container monitoring
* Application performance monitoring
* Centralized log visualization
* Distributed tracing
* Infrastructure alerting
* DevOps operations
* SRE workflows
* Cloud operations
* Incident investigation
* Capacity planning

---

## 🔗 Repository

**GitHub:**
https://github.com/msalman199/Grafana

---

## 📖 Official Grafana Resources

* [Grafana Documentation](https://grafana.com/docs/)
* [Grafana GitHub](https://github.com/grafana/grafana)
* [Prometheus](https://prometheus.io/)
* [Grafana Loki](https://grafana.com/oss/loki/)
* [Grafana Tempo](https://grafana.com/oss/tempo/)
* [Terraform](https://developer.hashicorp.com/terraform)

---

## 👨‍💻 Author

**Hafiz Muhammad Salman**

Cloud DevOps Engineer | Linux Administrator

GitHub: https://github.com/msalman199

---

## ⭐ Purpose of the Repository

The ultimate purpose of this repository is to provide a **practical, structured, and reusable Grafana learning environment** that demonstrates how modern observability solutions are designed and implemented.

Rather than focusing only on Grafana theory, the repository emphasizes **hands-on implementation**, allowing learners to build dashboards, connect data sources, monitor infrastructure, configure alerts, analyze logs, visualize traces, secure Grafana, and automate monitoring configurations.

> **Learn → Build → Monitor → Alert → Automate → Secure → Observe**

---

## 📜 License

This repository is intended primarily for educational and training purposes as part of the Al-Razzaq Programme.
