# 📊 Using Grafana for Kubernetes Monitoring

![Kubernetes](https://img.shields.io/badge/Kubernetes-Monitoring-326CE5?logo=kubernetes\&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C?logo=prometheus\&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-Visualization-F46800?logo=grafana\&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?logo=docker\&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-Local%20Kubernetes-FF6F00?logo=kubernetes\&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-FCC624?logo=linux\&logoColor=black)
![PromQL](https://img.shields.io/badge/PromQL-Metrics%20Queries-E6522C)
![Observability](https://img.shields.io/badge/Observability-Monitoring-8A2BE2)

> 🚀 **Hands-on Kubernetes observability lab using Minikube, Prometheus, and Grafana**

---

## 📌 Overview

This lab demonstrates how to build a complete **Kubernetes monitoring and observability stack** using **Minikube, Prometheus, and Grafana**.

You will create a local Kubernetes cluster, deploy Prometheus for metrics collection, configure Grafana for visualization, connect Grafana to Prometheus, and build dashboards that monitor Kubernetes nodes, pods, CPU, memory, disk, and cluster health.

The lab also introduces **PromQL**, Kubernetes service discovery, RBAC permissions, dashboard creation, and basic alerting.

### 🔄 Monitoring Architecture

```text
                    ┌─────────────────────────┐
                    │      Kubernetes         │
                    │       Minikube          │
                    │                         │
                    │  ┌───────┐  ┌────────┐ │
                    │  │ Nodes │  │  Pods  │ │
                    │  └───┬───┘  └───┬────┘ │
                    └──────┼───────────┼──────┘
                           │ Metrics
                           ▼
                 ┌─────────────────────┐
                 │     Prometheus      │
                 │                     │
                 │ • Scraping          │
                 │ • Storage           │
                 │ • PromQL            │
                 │ • Service Discovery │
                 └──────────┬──────────┘
                            │
                            │ Prometheus API
                            ▼
                 ┌─────────────────────┐
                 │       Grafana       │
                 │                     │
                 │ • Dashboards        │
                 │ • Visualization     │
                 │ • Alerts            │
                 └─────────────────────┘
```

---

# 🎯 Lab Objectives

By completing this lab, you will be able to:

* ☸️ Install and configure a local Kubernetes cluster using Minikube
* 🐳 Install and configure Docker as the Minikube container runtime
* 📊 Deploy Prometheus inside Kubernetes
* 🔐 Configure Kubernetes RBAC for Prometheus
* 🔎 Configure Kubernetes service discovery
* 📈 Install and configure Grafana
* 🔗 Integrate Prometheus with Grafana
* 📊 Import pre-built Kubernetes dashboards
* 🛠️ Build custom Grafana dashboards
* 🔥 Monitor Kubernetes pod CPU and memory usage
* 🖥️ Monitor node CPU, memory, and disk utilization
* 🧠 Write and understand PromQL queries
* 🚨 Configure basic Grafana alerting
* 🧪 Test monitoring using a sample NGINX deployment
* 🔧 Troubleshoot common Prometheus and Grafana problems

---

# 🧰 Prerequisites

Before starting, you should have:

* 🐧 Basic Linux command-line knowledge
* 🐳 Basic Docker knowledge
* ☸️ Basic Kubernetes concepts
* 📦 Understanding of containers, pods, services, and deployments
* 📊 Basic monitoring and observability knowledge
* 📝 Familiarity with YAML configuration files
* 🔐 Basic understanding of Kubernetes RBAC
* 🔎 Basic understanding of metrics and monitoring

---

# 💻 Lab Environment

The lab is designed for a single Linux machine provided through the **Al Nafi cloud laboratory environment**.

The machine starts without the required monitoring tools, so you will install:

| Technology         | Purpose                        |
| ------------------ | ------------------------------ |
| 🐧 Linux           | Operating system               |
| 🐳 Docker          | Container runtime              |
| ☸️ Minikube        | Local Kubernetes cluster       |
| ⚙️ kubectl         | Kubernetes CLI                 |
| 📊 Prometheus      | Metrics collection and storage |
| 📈 Grafana         | Metrics visualization          |
| 🔎 PromQL          | Prometheus query language      |
| 🔐 Kubernetes RBAC | Prometheus permissions         |
| 📡 Metrics Server  | Kubernetes resource metrics    |

---

# 🏗️ Lab Architecture

```text
┌──────────────────────────────────────────────────────┐
│                  Linux Machine                       │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │                 Minikube                       │  │
│  │                                                │  │
│  │  ┌───────────────┐      ┌──────────────────┐  │  │
│  │  │ Kubernetes    │      │ Monitoring       │  │  │
│  │  │ Workloads     │─────►│ Namespace        │  │  │
│  │  │               │      │                  │  │  │
│  │  │ • Nodes       │      │ Prometheus       │  │  │
│  │  │ • Pods        │      │ Grafana           │  │  │
│  │  │ • Services    │      │ RBAC              │  │  │
│  │  └───────────────┘      └──────────────────┘  │  │
│  │                                                │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

# 🚀 Task 1 — Set Up Prometheus in Kubernetes

## 1.1 Install Required Dependencies

### 🐳 Step 1 — Install Docker

```bash
# Update package repository
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io

# Start Docker
sudo systemctl start docker

# Enable Docker at boot
sudo systemctl enable docker

# Add current user to Docker group
sudo usermod -aG docker $USER

# Apply group changes
newgrp docker

# Verify installation
docker --version
```

### ✅ Verification

```bash
docker --version
```

Expected output should display the installed Docker version.

---

## ⚙️ Step 2 — Install kubectl

```bash
# Download the latest stable kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make executable
chmod +x kubectl

# Move to PATH
sudo mv kubectl /usr/local/bin/

# Verify installation
kubectl version --client
```

---

## ☸️ Step 3 — Install Minikube

```bash
# Download Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# Install Minikube
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Verify
minikube version
```

---

# ☸️ 1.2 Start the Kubernetes Cluster

## 🚀 Step 1 — Start Minikube

```bash
minikube start --driver=docker --memory=4096 --cpus=2
```

Verify the cluster:

```bash
minikube status
```

Check Kubernetes:

```bash
kubectl cluster-info
```

Check nodes:

```bash
kubectl get nodes
```

Expected result:

```text
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   ...   ...
```

---

## 📊 Step 2 — Enable Metrics Server

```bash
minikube addons enable metrics-server
```

Verify:

```bash
minikube addons list
```

Check the metrics server:

```bash
kubectl get pods -n kube-system | grep metrics-server
```

---

# 📦 1.3 Deploy Prometheus

## 🏷️ Step 1 — Create Monitoring Namespace

```bash
kubectl create namespace monitoring
```

Verify:

```bash
kubectl get namespaces
```

---

# ⚙️ Step 2 — Create Prometheus Configuration

Create the configuration file:

```bash
cat << 'EOF' > prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s

    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']

      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels:
          - __meta_kubernetes_namespace
          - __meta_kubernetes_service_name
          - __meta_kubernetes_endpoint_port_name
          action: keep
          regex: default;kubernetes;https

      - job_name: 'kubernetes-nodes'
        kubernetes_sd_configs:
        - role: node
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics

      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
        - role: pod
        relabel_configs:
        - source_labels:
          - __meta_kubernetes_pod_annotation_prometheus_io_scrape
          action: keep
          regex: true
        - source_labels:
          - __meta_kubernetes_pod_annotation_prometheus_io_path
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels:
          - __address__
          - __meta_kubernetes_pod_annotation_prometheus_io_port
          action: replace
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
          target_label: __address__
        - action: labelmap
          regex: __meta_kubernetes_pod_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_pod_name]
          action: replace
          target_label: kubernetes_pod_name
EOF
```

Apply:

```bash
kubectl apply -f prometheus-config.yaml
```

Verify:

```bash
kubectl get configmap -n monitoring
```

---

# 🔐 Step 3 — Configure Prometheus RBAC

Create:

```bash
cat << 'EOF' > prometheus-rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: monitoring
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: monitoring
EOF
```

Apply:

```bash
kubectl apply -f prometheus-rbac.yaml
```

Verify:

```bash
kubectl get clusterrole prometheus
kubectl get clusterrolebinding prometheus
kubectl get serviceaccount -n monitoring
```

---

# 📦 Step 4 — Deploy Prometheus

Create:

```bash
cat << 'EOF' > prometheus-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
  labels:
    app: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      serviceAccountName: prometheus
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        args:
          - '--config.file=/etc/prometheus/prometheus.yml'
          - '--storage.tsdb.path=/prometheus/'
          - '--web.console.libraries=/etc/prometheus/console_libraries'
          - '--web.console.templates=/etc/prometheus/consoles'
          - '--storage.tsdb.retention.time=200h'
          - '--web.enable-lifecycle'
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: prometheus-config-volume
          mountPath: /etc/prometheus/
        - name: prometheus-storage-volume
          mountPath: /prometheus/
      volumes:
      - name: prometheus-config-volume
        configMap:
          defaultMode: 420
          name: prometheus-config
      - name: prometheus-storage-volume
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
  namespace: monitoring
spec:
  selector:
    app: prometheus
  type: NodePort
  ports:
    - port: 8080
      targetPort: 9090
      nodePort: 30000
EOF
```

Deploy:

```bash
kubectl apply -f prometheus-deployment.yaml
```

---

# 🔍 Step 5 — Verify Prometheus

Check deployment:

```bash
kubectl get deployments -n monitoring
```

Check pods:

```bash
kubectl get pods -n monitoring
```

Check service:

```bash
kubectl get services -n monitoring
```

Wait for Prometheus:

```bash
kubectl wait \
  --for=condition=ready \
  pod -l app=prometheus \
  -n monitoring \
  --timeout=300s
```

Check logs:

```bash
kubectl logs -n monitoring deployment/prometheus
```

---

# 📈 Task 2 — Integrate Prometheus with Grafana

## 2.1 Deploy Grafana

Create:

```bash
cat << 'EOF' > grafana-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - name: grafana
          containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_USER
          value: admin
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: admin123
        volumeMounts:
        - mountPath: /var/lib/grafana
          name: grafana-storage
      volumes:
      - name: grafana-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: grafana-service
  namespace: monitoring
spec:
  selector:
    app: grafana
  type: NodePort
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 32000
EOF
```

Deploy:

```bash
kubectl apply -f grafana-deployment.yaml
```

---

# 🔎 Verify Grafana

```bash
kubectl get deployments -n monitoring
kubectl get pods -n monitoring
```

Wait for Grafana:

```bash
kubectl wait \
  --for=condition=ready \
  pod -l app=grafana \
  -n monitoring \
  --timeout=300s
```

Get Minikube IP:

```bash
minikube ip
```

---

# 🌐 2.2 Access Grafana

Get the service URL:

```bash
MINIKUBE_IP=$(minikube ip)

echo "Grafana: http://$MINIKUBE_IP:32000"
echo "Prometheus: http://$MINIKUBE_IP:30000"
```

### Alternative: Port Forwarding

Grafana:

```bash
kubectl port-forward \
  -n monitoring \
  service/grafana-service \
  3000:3000
```

Prometheus:

```bash
kubectl port-forward \
  -n monitoring \
  service/prometheus-service \
  9090:8080
```

Access:

```text
Grafana:    http://localhost:3000
Prometheus: http://localhost:9090
```

### 🔑 Grafana Credentials

```text
Username: admin
Password: admin123
```

> ⚠️ **Security Note:** The credentials above are intended for this lab only. Production environments should use secure credentials, Kubernetes Secrets, persistent storage, and appropriate authentication controls.

---

# 🔗 2.3 Configure Prometheus as Grafana Data Source

In Grafana:

1. ⚙️ Open **Connections / Data Sources**
2. Click **Add data source**
3. Select **Prometheus**
4. Configure:

```text
Name: Prometheus

URL:
http://prometheus-service.monitoring.svc.cluster.local:8080
```

5. Click **Save & Test**

A successful connection should display a confirmation that the data source is working.

---

# 📊 Task 3 — Create Kubernetes Monitoring Dashboards

## 3.1 Import a Pre-Built Dashboard

From Grafana:

1. Click **+**
2. Select **Import**
3. Enter dashboard ID:

```text
3119
```

4. Click **Load**
5. Select **Prometheus**
6. Click **Import**

The dashboard can provide visibility into areas such as:

* 🖥️ Cluster CPU usage
* 🧠 Memory utilization
* 📦 Pod status
* ☸️ Node information
* 🌐 Network traffic

---

# 📦 3.2 Create Pod Monitoring Dashboard

Create a new dashboard:

**+ → Dashboard → Add new panel**

## 🔥 Panel 1 — Pod CPU Usage

PromQL:

```promql
sum(
  rate(container_cpu_usage_seconds_total{
    container!="POD",
    container!=""
  }[5m])
) by (pod)
```

Panel title:

```text
Pod CPU Usage
```

---

## 🧠 Panel 2 — Pod Memory Usage

PromQL:

```promql
sum(
  container_memory_usage_bytes{
    container!="POD",
    container!=""
  }
) by (pod) / 1024 / 1024
```

Panel title:

```text
Pod Memory Usage (MB)
```

---

## 📦 Panel 3 — Pod Status

PromQL:

```promql
kube_pod_status_phase
```

Panel title:

```text
Pod Status
```

Visualization:

```text
Stat
```

---

## 💾 Save Dashboard

Name:

```text
Custom Kubernetes Pod Monitoring
```

---

# 🖥️ 3.3 Create Node Monitoring Dashboard

Create another dashboard.

## 🔥 Node CPU Usage

```promql
100 - (
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

Panel title:

```text
Node CPU Usage (%)
```

---

## 🧠 Node Memory Usage

```promql
(
  1 -
  (
    node_memory_MemAvailable_bytes /
    node_memory_MemTotal_bytes
  )
) * 100
```

Panel title:

```text
Node Memory Usage (%)
```

---

## 💽 Node Disk Usage

```promql
100 - (
  (
    node_filesystem_avail_bytes{
      mountpoint="/",
      fstype!="rootfs"
    }
    /
    node_filesystem_size_bytes{
      mountpoint="/",
      fstype!="rootfs"
    }
  ) * 100
)
```

Panel title:

```text
Node Disk Usage (%)
```

---

## 💾 Save Dashboard

Dashboard name:

```text
Kubernetes Node Monitoring
```

---

# 🚨 3.4 Optional — Configure Grafana Alerting

Create a new alert rule:

**Alerting → Alert Rules → New Rule**

PromQL:

```promql
up{job="kubernetes-nodes"} == 0
```

Example evaluation:

```text
Evaluate every: 1m
For: 5m
```

Alert name:

```text
Node Down Alert
```

Message:

```text
Kubernetes node is down
```

Save the rule.

---

# 🧪 Verification and Testing

## 🔎 Verify Prometheus Targets

Open:

```text
http://<MINIKUBE_IP>:30000
```

Navigate to:

```text
Status → Targets
```

Verify that the Kubernetes targets are being scraped successfully.

---

# 🔬 Test PromQL Queries

Try:

```promql
up
```

```promql
kube_pod_info
```

```promql
node_cpu_seconds_total
```

These queries help verify that Prometheus is collecting Kubernetes and node-related metrics.

---

# 📈 Verify Grafana Dashboards

Check that:

* ✅ Panels display metric data
* ✅ CPU metrics are updating
* ✅ Memory metrics are updating
* ✅ Pod information is visible
* ✅ Node information is visible
* ✅ Dashboard time ranges work
* ✅ Data refreshes correctly
* ✅ Prometheus queries return results

---

# 🧪 Generate Load with an NGINX Application

Deploy a test application:

```bash
kubectl create deployment nginx-test \
  --image=nginx \
  --replicas=3
```

Expose it:

```bash
kubectl expose deployment nginx-test \
  --port=80 \
  --type=NodePort
```

Check the deployment:

```bash
kubectl get deployments
```

Check pods:

```bash
kubectl get pods
```

Check services:

```bash
kubectl get services
```

Return to Grafana and observe the newly created workload.

---

# 🔧 Troubleshooting

## ❌ Problem 1 — Prometheus Is Not Scraping Targets

Check Prometheus logs:

```bash
kubectl logs \
  -n monitoring \
  deployment/prometheus
```

Check RBAC:

```bash
kubectl get clusterrolebinding prometheus
```

Check endpoints:

```bash
kubectl get endpoints -n monitoring
```

---

## ❌ Problem 2 — Grafana Cannot Connect to Prometheus

Check Prometheus service:

```bash
kubectl get svc -n monitoring prometheus-service
```

Test connectivity from Grafana:

```bash
kubectl exec \
  -n monitoring \
  deployment/grafana \
  -- wget -qO- \
  "http://prometheus-service.monitoring.svc.cluster.local:8080/api/v1/query?query=up"
```

If connectivity works, Prometheus should return a JSON API response.

---

## ❌ Problem 3 — Grafana Dashboard Has No Data

Check Metrics Server:

```bash
kubectl get pods \
  -n kube-system | grep metrics-server
```

Check Prometheus configuration:

```bash
kubectl get configmap \
  -n monitoring \
  prometheus-config \
  -o yaml
```

Check Prometheus targets:

```bash
curl \
  http://<MINIKUBE_IP>:30000/api/v1/targets
```

---

# 🧹 Cleanup

Delete the monitoring namespace:

```bash
kubectl delete namespace monitoring
```

Delete the test application:

```bash
kubectl delete deployment nginx-test
kubectl delete service nginx-test
```

Stop Minikube:

```bash
minikube stop
```

Delete the cluster completely if no longer needed:

```bash
minikube delete
```

---

# 📚 Key Concepts Learned

| Concept              | Description                                |
| -------------------- | ------------------------------------------ |
| ☸️ Minikube          | Runs Kubernetes locally                    |
| ⚙️ kubectl           | Manages Kubernetes resources               |
| 📊 Prometheus        | Collects and stores metrics                |
| 🔎 PromQL            | Queries Prometheus metrics                 |
| 📈 Grafana           | Visualizes monitoring data                 |
| 🔐 RBAC              | Controls Prometheus Kubernetes permissions |
| 📡 Service Discovery | Automatically discovers Kubernetes targets |
| 📦 Kubernetes Pods   | Application workloads being monitored      |
| 🖥️ Node Metrics     | CPU, memory, and disk monitoring           |
| 🚨 Alerting          | Detects operational problems               |
| 📊 Dashboards        | Provides visual observability              |

---

# 🎓 Learning Outcomes

After completing this lab, you should understand the complete monitoring workflow:

```text
Kubernetes Workloads
        │
        ▼
Metrics Endpoints
        │
        ▼
Prometheus Service Discovery
        │
        ▼
Prometheus Scraping
        │
        ▼
Prometheus Time-Series Database
        │
        ▼
PromQL
        │
        ▼
Grafana Data Source
        │
        ▼
Grafana Dashboards
        │
        ▼
Alerts & Observability
```

---

# 🏆 Conclusion

Congratulations! 🎉

You have successfully built a Kubernetes monitoring environment using **Minikube, Prometheus, and Grafana**.

During this lab, you:

* ☸️ Created a local Kubernetes cluster
* 🐳 Configured Docker as the container runtime
* 📊 Deployed Prometheus inside Kubernetes
* 🔐 Configured Prometheus RBAC permissions
* 🔎 Implemented Kubernetes service discovery
* 📈 Deployed Grafana
* 🔗 Connected Grafana to Prometheus
* 📊 Imported a Kubernetes monitoring dashboard
* 🛠️ Created custom pod and node dashboards
* 🔥 Monitored CPU and memory utilization
* 💽 Monitored node disk utilization
* 🔬 Practiced PromQL queries
* 🚨 Configured a basic alert
* 🧪 Tested monitoring with an NGINX workload
* 🔧 Troubleshot common monitoring issues

---

# 🌍 Real-World Applications

The skills from this lab are directly applicable to real-world DevOps and Cloud environments.

### 🚨 Incident Detection

Monitoring allows engineers to identify failing nodes, unhealthy pods, resource exhaustion, and abnormal workloads before they become major incidents.

### 📊 Resource Optimization

CPU, memory, disk, and network metrics help teams identify inefficient workloads and optimize resource allocation.

### 📈 Capacity Planning

Historical metrics can reveal resource consumption trends and help teams plan future Kubernetes capacity.

### 🔧 Troubleshooting

Prometheus metrics and Grafana dashboards provide valuable information during application and infrastructure troubleshooting.

### ☁️ Production Observability

The Prometheus + Grafana ecosystem is widely used as part of Kubernetes observability architectures and can be extended with alerting, exporters, service discovery, recording rules, and long-term metrics storage.

---

# 🛡️ Production Considerations

This lab uses simplified settings for learning. Before deploying a similar stack in production, consider:

* 🔐 Store Grafana credentials in Kubernetes Secrets
* 💾 Use PersistentVolumes for Grafana and Prometheus
* 🏷️ Pin container image versions instead of relying on `latest`
* 🔒 Apply least-privilege RBAC
* 🌐 Use secure ingress/TLS configuration
* 📊 Configure appropriate Prometheus retention
* 🚨 Configure notification channels for alerts
* 💾 Implement backup and disaster recovery
* 📈 Consider high availability for critical monitoring systems
* 🔎 Monitor Prometheus and Grafana themselves
* 🛡️ Restrict access to monitoring endpoints

---

# 🧠 Skills Demonstrated

```text
Linux Administration
        │
        ├── Docker
        │
        ├── Kubernetes
        │
        ├── Minikube
        │
        ├── kubectl
        │
        ├── Kubernetes RBAC
        │
        ├── Prometheus
        │
        ├── PromQL
        │
        ├── Grafana
        │
        ├── Dashboard Engineering
        │
        ├── Kubernetes Monitoring
        │
        └── Observability
```

---

## ⭐ Final Result

```text
┌──────────────────────────────────────────┐
│       Kubernetes Monitoring Stack        │
├──────────────────────────────────────────┤
│                                          │
│  ☸️ Kubernetes / Minikube                │
│             ↓                            │
│  📊 Prometheus                            │
│             ↓                            │
│  🔎 PromQL                                │
│             ↓                            │
│  📈 Grafana                               │
│             ↓                            │
│  🚨 Alerts + Dashboards                   │
│                                          │
└──────────────────────────────────────────┘
```

> 💡 **Key Takeaway:** Prometheus provides the metrics collection and querying layer, while Grafana turns those metrics into actionable dashboards and visualizations. Together, they form a powerful foundation for Kubernetes monitoring and observability.

---

## 👨‍💻 Author

**Hafiz Muhammad Salman**
Cloud DevOps Engineer | Linux Administrator

⭐ If this lab helped you, consider giving the repository a star and sharing your learning journey!

---

## 📜 License

This project is intended for educational and training purposes.
