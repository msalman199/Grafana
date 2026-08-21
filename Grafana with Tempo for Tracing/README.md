# 🔭 Grafana with Tempo for Distributed Tracing

> **Hands-on Observability Lab**
> Build a production-style distributed tracing platform using **Grafana Tempo, OpenTelemetry Collector, Prometheus, Loki, Grafana, and a traced HTTP application**.

---

## 🏷️ Technology Stack

| Technology                     | Purpose                                    |
| ------------------------------ | ------------------------------------------ |
| 🟠 **Grafana**                 | Visualization and observability dashboards |
| 🟣 **Grafana Tempo**           | Distributed trace storage and querying     |
| 🔵 **OpenTelemetry Collector** | Receives, processes, and exports telemetry |
| 🟢 **Prometheus**              | Metrics collection and storage             |
| 🟡 **Loki**                    | Centralized log aggregation                |
| 🐳 **Docker**                  | Container runtime                          |
| 🧩 **Docker Compose**          | Multi-service orchestration                |
| 🐧 **Ubuntu**                  | Lab operating system                       |
| 📡 **OpenTelemetry**           | Application instrumentation and telemetry  |
| 🐚 **Bash**                    | Automation and validation scripts          |
| 🔎 **jq**                      | JSON processing                            |
| ☁️ **AWS EC2**                 | Lab infrastructure                         |

---

# 🎯 Lab Objectives

By completing this lab, you will learn how to:

* 🚀 Deploy Grafana Tempo on Ubuntu using Docker Compose.
* 📡 Configure OpenTelemetry Collector for OTLP telemetry.
* 📊 Integrate Tempo, Prometheus, and Loki with Grafana.
* 🔗 Understand distributed trace context and parent-child spans.
* 🧩 Instrument a multi-endpoint HTTP application.
* ⚡ Generate application traces and metrics automatically.
* ❌ Capture and classify application errors in traces.
* 📈 Build a Grafana dashboard for tracing and performance analysis.
* 🔄 Correlate trace latency with Prometheus metrics.
* 🧪 Generate continuous application traffic.
* 🔍 Query Tempo directly through its HTTP API.
* 🤖 Automate observability deployment and validation without relying on browser configuration.

---

# 📚 Prerequisites

Before starting, you should have:

* Basic Linux command-line knowledge.
* Experience editing YAML files.
* Familiarity with Bash scripts.
* Basic Docker and Docker Compose knowledge.
* Understanding of:

  * Trace
  * Span
  * Trace ID
  * Span ID
  * Parent-child relationships
  * Trace context
  * Metrics vs. logs vs. traces

---

# 🏗️ Lab Architecture

```text
                         ┌─────────────────────┐
                         │    AWS EC2 Ubuntu   │
                         │                     │
                         │  Docker Compose     │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    │       observability-net       │
                    │         Docker Network        │
                    └───────────────┬───────────────┘
                                    │
       ┌────────────────────────────┼────────────────────────────┐
       │                            │                            │
       ▼                            ▼                            ▼
┌──────────────┐            ┌─────────────────┐          ┌──────────────┐
│ sample-app   │──OTLP─────▶│ OpenTelemetry   │          │   Grafana    │
│              │            │   Collector     │          │              │
│ HTTP Routes  │            └───────┬─────────┘          └──────┬───────┘
│ Traces       │                    │                           │
│ Metrics      │                    │                           │
└──────────────┘                    │                           │
                                    │                           │
                    ┌───────────────┼───────────────┐           │
                    │               │               │           │
                    ▼               ▼               ▼           │
             ┌────────────┐  ┌────────────┐  ┌────────────┐    │
             │   Tempo    │  │ Prometheus │  │    Loki    │◀───┘
             │   Traces   │  │  Metrics   │  │    Logs    │
             └────────────┘  └────────────┘  └────────────┘
                    ▲               ▲               ▲
                    │               │               │
                    └───────────────┴───────────────┘
                             Grafana Queries
```

### 🔄 Telemetry Flow

```text
Application
     │
     ├── Traces ──────▶ OpenTelemetry Collector ──────▶ Tempo
     │
     ├── Metrics ─────▶ OpenTelemetry Collector ──────▶ Prometheus
     │
     └── Logs ────────▶ OpenTelemetry Collector ──────▶ Loki
                                                        │
                                                        ▼
                                                     Grafana
```

---

# 📦 Lab Services

The completed environment contains **six Docker Compose services**:

1. 🔭 Grafana Tempo
2. 📊 Grafana
3. 📈 Prometheus
4. 📝 Loki
5. 📡 OpenTelemetry Collector
6. 🚀 Sample traced application

All services communicate through a single user-defined Docker bridge network.

---

# 🚀 Task 1 — Install the Observability Toolchain

## 1.1 Update Ubuntu

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 1.2 Install Required Utilities

```bash
sudo apt install -y curl wget bc
```

Verify:

```bash
curl --version
wget --version
bc --version
```

---

## 1.3 Install Docker Engine

Configure the official Docker repository and install Docker Engine and the Compose plugin.

Verify:

```bash
docker --version
docker compose version
```

Enable Docker:

```bash
sudo systemctl enable --now docker
```

Verify:

```bash
sudo systemctl status docker
```

### ⚠️ Troubleshooting — Docker Repository

If you encounter:

```text
E: Package 'docker-ce' has no installation candidate
```

check:

```bash
cat /etc/apt/sources.list.d/docker.list
```

The repository configuration must be a valid single line.

If the repository file is malformed, remove and recreate it using a heredoc.

For the official installation procedure, see:

[Docker Engine installation documentation](https://docs.docker.com/engine/install/ubuntu/?utm_source=chatgpt.com)

---

# 🐳 1.4 Deploy the Docker Compose Stack

Create the lab directory:

```bash
mkdir -p ~/grafana-tempo-lab
cd ~/grafana-tempo-lab
```

The Compose deployment should contain:

```text
grafana
tempo
prometheus
loki
otel-collector
sample-app
```

All services must use the same Docker network:

```text
observability-net
```

---

# 🟣 1.5 Configure Tempo

Tempo should:

* Store traces on local disk.
* Receive OTLP telemetry from the Collector.
* Enable the metrics generator.
* Enable:

  * `service-graphs`
  * `span-metrics`
* Send generated metrics to Prometheus through remote write.

### ⚠️ Important Port Configuration

Only the OpenTelemetry Collector should expose OTLP ports to the host.

Avoid binding Tempo and the Collector to the same host ports:

```text
4317
4318
```

Tempo can receive these ports internally through the Docker network.

For example:

```text
Application
    │
    ▼
OTel Collector:4317
    │
    ▼
Tempo:4317
```

### ⚠️ Troubleshooting

If you see:

```text
Ports are not available: exposing port TCP 0.0.0.0:4317
```

check whether both Tempo and the Collector are publishing port `4317`.

Remove the host mapping from Tempo and keep the Collector's host mapping.

See the official:

[Grafana Tempo configuration documentation](https://grafana.com/docs/tempo/latest/configuration/?utm_source=chatgpt.com)

---

# 🔵 1.6 Configure OpenTelemetry Collector

The Collector must expose:

```text
OTLP gRPC
4317
```

and:

```text
OTLP HTTP
4318
```

Create separate pipelines for:

```text
traces
metrics
logs
```

### Trace Pipeline

```text
OTLP Receiver
     │
     ▼
Trace Processors
     │
     ▼
Tempo Exporter
```

### Metrics Pipeline

```text
OTLP Receiver
     │
     ▼
Metric Processors
     │
     ▼
Prometheus
```

### Logs Pipeline

```text
OTLP Receiver
     │
     ▼
Log Processors
     │
     ▼
Loki
```

---

# 🟢 1.7 Configure Prometheus

Prometheus must have remote-write receiving enabled because Tempo's metrics generator sends generated metrics through Prometheus remote write.

The Prometheus health endpoint is:

```text
http://localhost:9090/-/healthy
```

Expected response:

```text
Prometheus Server is Healthy.
```

---

# 🟡 1.8 Configure Loki

Loki provides centralized log storage.

The Collector forwards log telemetry to Loki.

Grafana will use Loki as a provisioned data source.

---

# 📊 1.9 Configure Grafana

Grafana should automatically provision:

* Tempo
* Prometheus
* Loki

No manual UI configuration should be required.

Health endpoint:

```bash
curl -sf http://localhost:3000/api/health
```

Expected JSON should contain:

```json
"database": "ok"
```

---

# 🔍 1.10 Verify the Stack

Start everything:

```bash
docker compose up -d
```

Check containers:

```bash
docker compose ps
```

Expected services:

```text
grafana
tempo
prometheus
loki
otel-collector
sample-app
```

All should show:

```text
running
```

---

# 🧪 1.11 Verify Service Health

### Tempo

```bash
curl -sf http://localhost:3200/ready
```

Expected:

```text
ready
```

### Grafana

```bash
curl -sf http://localhost:3000/api/health
```

Expected JSON containing:

```text
"database": "ok"
```

### Prometheus

```bash
curl -sf http://localhost:9090/-/healthy
```

Expected:

```text
Prometheus Server is Healthy.
```

---

# 🩺 1.12 Create Health Verification Script

Create:

```bash
nano health-check.sh
```

The script should:

* Check every required service.
* Use `curl -sf`.
* Print PASS or FAIL.
* Display HTTP status codes.
* Return exit code `0` only when every check succeeds.
* Complete within 10 seconds during normal operation.

Example output:

```text
[PASS] Tempo       HTTP 200
[PASS] Grafana     HTTP 200
[PASS] Prometheus  HTTP 200
[PASS] Loki        HTTP 200
[PASS] Collector   HTTP 200
```

Make executable:

```bash
chmod +x health-check.sh
```

Run:

```bash
./health-check.sh
```

Verify exit code:

```bash
echo $?
```

Expected:

```text
0
```

---

# 🚀 Task 2 — Build the Trace-Emitting Application

Create a containerized HTTP application named:

```text
sample-app
```

The application should support at least five routes.

---

# 🌐 2.1 Application Routes

| Route         | Purpose                       |
| ------------- | ----------------------------- |
| `/`           | Root endpoint                 |
| `/users/<id>` | User lookup                   |
| `/orders`     | Simulates downstream services |
| `/slow`       | Variable 1–4 second latency   |
| `/error`      | Always returns HTTP 500       |

---

# 🔗 2.2 Trace Structure

Every request must create a root span.

The `/orders` route should create nested child spans.

Example:

```text
orders
│
└── order-processing
    │
    ├── payment
    │
    └── inventory
```

The resulting trace must provide meaningful parent-child relationships.

For example:

```text
HTTP Request
└── Order Processing
    ├── Payment Service
    └── Inventory Service
```

---

# 🏷️ 2.3 Span Attributes

Every span should contain at least three semantic attributes.

Examples:

```text
service.name
http.method
http.route
```

Other useful attributes include:

```text
http.status_code
db.operation
server.address
```

---

# ❌ 2.4 Error Classification

The `/error` endpoint must:

```text
HTTP 500
```

and the associated span should have:

```text
Status = ERROR
```

with a descriptive error message.

---

# 📈 2.5 Application Metrics

Expose:

```text
/metrics
```

The endpoint must provide Prometheus-compatible metrics.

Required metric:

```text
http_requests_total
```

Required histogram:

```text
http_request_duration_seconds_bucket
```

Test:

```bash
curl -sf http://localhost:8080/metrics
```

---

# 🔭 2.6 Export Traces through OpenTelemetry

The application must export traces using:

```text
OTLP/gRPC
```

The application should send telemetry to the Collector using its Docker service name.

Example architecture:

```text
sample-app
    │
    │ OTLP/gRPC
    ▼
otel-collector:4317
```

### ⚠️ Troubleshooting OTLP

If application logs contain:

```text
Failed to export spans: rpc error: code = Unavailable
```

check the network:

```bash
docker inspect <app-container> | grep -A 10 Networks
```

Verify:

* Application is connected to `observability-net`.
* Collector is connected to `observability-net`.
* The Collector hostname is correct.
* Port `4317` is reachable internally.

See:

[OpenTelemetry OTLP specification](https://opentelemetry.io/docs/specs/otlp/?utm_source=chatgpt.com)

---

# 🧪 2.7 Generate Test Requests

Test each route:

```bash
curl http://localhost:8080/
curl http://localhost:8080/users/100
curl http://localhost:8080/orders
curl http://localhost:8080/slow
curl http://localhost:8080/error
```

Generate additional traffic:

```bash
for i in {1..50}; do
    curl -s http://localhost:8080/ > /dev/null
    curl -s http://localhost:8080/users/$i > /dev/null
    curl -s http://localhost:8080/orders > /dev/null
    curl -s http://localhost:8080/slow > /dev/null
    curl -s http://localhost:8080/error > /dev/null
done
```

---

# 🔄 Task 2.2 — Continuous Load Generator

Create:

```bash
nano load.sh
```

The load generator should:

* Run indefinitely.
* Default to **2 requests per second**.
* Randomly select all five routes.
* Use a **5% error probability**.
* Use a **10% slow-route probability**.
* Record HTTP status codes.
* Record response times.
* Write to:

```text
load.log
```

* Store its PID in:

```text
load.pid
```

---

# 📝 Example Log Format

```text
2026-08-21T07:30:01Z GET /users/25 200 42ms
2026-08-21T07:30:02Z GET /orders 200 185ms
2026-08-21T07:30:03Z GET /error 500 17ms
2026-08-21T07:30:04Z GET /slow 200 2317ms
```

---

# ▶️ Start the Load Generator

```bash
chmod +x load.sh
./load.sh &
```

Or implement the script so it backgrounds itself and writes its PID automatically.

Check:

```bash
cat load.pid
```

Monitor:

```bash
tail -f load.log
```

---

# 🛑 Stop the Load Generator

```bash
kill $(cat load.pid)
```

Confirm:

```bash
ps aux | grep load
```

The process should terminate cleanly without leaving zombie processes.

---

# 📊 Validate Generated Load

After approximately five minutes:

```bash
wc -l load.log
```

At 2 RPS, the expected volume is roughly:

```text
600 requests
```

The acceptance requirement is at least:

```text
550 lines
```

Check errors:

```bash
grep " 500 " load.log
```

Check slow responses:

```bash
awk '$NF ~ /ms/ && ($NF+0) > 1000' load.log
```

---

# 📊 Task 3 — Build the Grafana Tracing Dashboard

The dashboard must contain **exactly four panels**.

## Dashboard Layout

```text
┌──────────────────────────┬──────────────────────────┐
│                          │                          │
│   🔭 Tempo Trace Search  │  📈 Latency Percentiles │
│                          │                          │
├──────────────────────────┼──────────────────────────┤
│                          │                          │
│   ❌ Error Rate          │  🚀 Request Rate        │
│                          │                          │
└──────────────────────────┴──────────────────────────┘
```

---

# 🔭 Panel 1 — Tempo Trace Search

Configure the panel to:

* Use Tempo.
* Filter for:

```text
service.name = sample-app
```

* Display the most recent 50 traces.

Users should be able to select a trace and open its complete waterfall.

---

# 📈 Panel 2 — Request Duration

Use Prometheus histogram data to calculate:

* P50
* P95
* P99

Example PromQL structure:

```promql
histogram_quantile(
  0.50,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

P95:

```promql
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

P99:

```promql
histogram_quantile(
  0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

---

# ❌ Panel 3 — Error Rate

Calculate:

```text
4xx + 5xx responses
------------------- × 100
total responses
```

The resulting Grafana stat panel should display the current error percentage.

---

# 🚀 Panel 4 — Request Rate

Display request rate per route.

Example:

```promql
sum(
  rate(http_requests_total[5m])
) by (route)
```

The panel should show requests per second for each application route.

---

# ⚙️ Dashboard Configuration

The dashboard must:

```text
Refresh: 10 seconds
Time range: Last 15 minutes
Panels: Exactly 4
```

Save the dashboard as:

```text
tempo-tracing-dashboard.json
```

---

# 🚀 3.1 Import Dashboard through Grafana API

The dashboard must be imported without browser interaction.

Example API workflow:

```bash
curl -u admin:password \
  -H "Content-Type: application/json" \
  -X POST \
  http://localhost:3000/api/dashboards/db \
  -d @tempo-tracing-dashboard.json
```

A successful response should contain:

```json
"status": "success"
```

Expected HTTP status:

```text
200
```

---

# ⚠️ Troubleshooting Dashboard Import

If you receive:

```json
{"message":"invalid API key"}
```

or:

```text
HTTP 401
```

check Grafana's configured security environment:

```bash
docker compose exec grafana env | grep GF_SECURITY
```

Verify the credentials used by the API request.

For API documentation, see:

[Grafana Dashboard HTTP API documentation](https://grafana.com/docs/grafana/latest/developers/http_api/dashboard/?utm_source=chatgpt.com)

---

# 🔎 Task 3.2 — Automated Tempo Trace Analysis

Create:

```bash
nano analyze-traces.sh
```

The script must use only:

```text
curl
jq
POSIX utilities
```

It should:

1. Query Tempo.
2. Restrict results to `sample-app`.
3. Search the previous 10 minutes.
4. Identify traces whose root span exceeds one second.
5. Print:

```text
TRACE_ID    DURATION
```

6. Identify the longest trace.
7. Fetch the complete span tree.
8. Print every span name and duration.

---

# 📋 Expected Analysis Output

```text
TRACE_ID                             DURATION
a91f7c3e...                          3.214s
bc72a91d...                          2.671s
```

Longest trace:

```text
Longest Trace
--------------------------------
TRACE ID: a91f7c3e...

Span Name                 Duration
-----------------------------------
HTTP GET /slow            3.214s
slow-operation            3.208s
database-operation        1.102s
```

---

# 🧪 Error Handling

The analysis script must return:

```text
0
```

when successful.

It must return:

```text
1
```

when:

* Tempo returns a non-2xx response.
* `jq` cannot parse the response.
* Required trace data cannot be processed.

Test the failure path:

```bash
docker compose stop tempo
./analyze-traces.sh
echo $?
```

Expected:

```text
1
```

Restart Tempo:

```bash
docker compose start tempo
```

---

# 🔍 Observability Validation

Once the load generator is running, open Grafana and verify:

### Tempo

```text
sample-app traces
```

### Prometheus

```text
http_requests_total
```

### Loki

```text
application logs
```

### Grafana

```text
Trace latency
Error rate
Request rate
Trace search
```

---

# 🔗 Trace Correlation

A successful implementation provides this complete observability chain:

```text
HTTP Request
     │
     ▼
sample-app
     │
     ├───────────────┐
     │               │
     ▼               ▼
  Metrics          Traces
     │               │
     ▼               ▼
Prometheus         Tempo
     │               │
     └───────┬───────┘
             ▼
          Grafana
```

Logs can additionally flow through:

```text
sample-app
     │
     ▼
OTel Collector
     │
     ▼
Loki
     │
     ▼
Grafana
```

---

# ✅ Final Acceptance Checklist

## Infrastructure

* [ ] Ubuntu instance prepared.
* [ ] Docker Engine installed.
* [ ] Docker Compose installed.
* [ ] `curl`, `wget`, and `bc` installed.
* [ ] Six-service Compose stack deployed.
* [ ] All services share one Docker bridge network.

## Tempo

* [ ] Tempo is running.
* [ ] Tempo uses local storage.
* [ ] Metrics generator enabled.
* [ ] `service-graphs` enabled.
* [ ] `span-metrics` enabled.
* [ ] Metrics forwarded to Prometheus.

## OpenTelemetry

* [ ] OTLP/gRPC configured.
* [ ] OTLP/HTTP configured.
* [ ] Trace pipeline configured.
* [ ] Metrics pipeline configured.
* [ ] Logs pipeline configured.

## Application

* [ ] Root route implemented.
* [ ] User route implemented.
* [ ] Orders route implemented.
* [ ] Slow route implemented.
* [ ] Error route implemented.
* [ ] Parent-child spans implemented.
* [ ] Semantic span attributes added.
* [ ] Error status configured.
* [ ] Prometheus metrics exposed.

## Load Generator

* [ ] Default rate is 2 RPS.
* [ ] Random routes generated.
* [ ] 5% error probability configured.
* [ ] 10% slow-route probability configured.
* [ ] `load.log` generated.
* [ ] `load.pid` generated.
* [ ] Clean shutdown implemented.

## Grafana

* [ ] Tempo data source provisioned.
* [ ] Prometheus data source provisioned.
* [ ] Loki data source provisioned.
* [ ] Dashboard contains exactly four panels.
* [ ] Dashboard refreshes every 10 seconds.
* [ ] Default range is 15 minutes.
* [ ] Dashboard imported through API.
* [ ] No manual UI configuration required.

## Trace Analysis

* [ ] Tempo HTTP API queried.
* [ ] Last 10 minutes analyzed.
* [ ] Slow traces identified.
* [ ] Longest trace selected.
* [ ] Span tree displayed.
* [ ] Error handling implemented.

---

# 🧪 Expected Outcomes

After completing the lab, you should have a fully operational distributed tracing environment:

```text
┌──────────────────────────────────────────────┐
│          Distributed Observability           │
├──────────────────────────────────────────────┤
│                                              │
│  Application                                │
│       │                                      │
│       ▼                                      │
│  OpenTelemetry Collector                    │
│       │                                      │
│       ├──────────────▶ Tempo                 │
│       │                   │                  │
│       │                   ▼                  │
│       │               Distributed            │
│       │                 Traces               │
│       │                                      │
│       ├──────────────▶ Prometheus            │
│       │                   │                  │
│       │                   ▼                  │
│       │                 Metrics              │
│       │                                      │
│       └──────────────▶ Loki                  │
│                           │                  │
│                           ▼                  │
│                          Logs                │
│                                              │
│                 ┌──────────────┐             │
│                 │   Grafana    │             │
│                 │ Visualization│             │
│                 └──────────────┘             │
└──────────────────────────────────────────────┘
```

You should be able to:

* 🔭 Search distributed traces.
* 🌳 Inspect parent-child span relationships.
* ⚡ Identify slow requests.
* ❌ Identify failed requests.
* 📊 Monitor request rates.
* 📈 Analyze P50/P95/P99 latency.
* 🔗 Correlate metrics with traces.
* 📝 Query centralized logs.
* 🤖 Deploy and validate the complete environment through scripts.

---

# 🧠 Key Concepts Learned

### Trace

A trace represents the complete journey of a request through a distributed system.

### Span

A span represents one operation within that trace.

### Parent-Child Relationship

Parent-child relationships show how individual operations contribute to a larger request.

### Trace Context

Trace context allows telemetry generated by different services to belong to the same distributed trace.

### Span Metrics

Tempo can derive metrics from collected spans, allowing trace information to contribute to service-level monitoring.

### Service Graphs

Service graphs provide a dependency-oriented view of service interactions and performance.

---

# 🏆 Conclusion

This lab demonstrates how to construct a complete distributed tracing platform from the ground up.

You deployed:

```text
Grafana
Tempo
Prometheus
Loki
OpenTelemetry Collector
Sample Application
```

You then connected these components into a unified observability pipeline:

```text
Application
     │
     ▼
OpenTelemetry
     │
     ├──▶ Tempo       → Traces
     ├──▶ Prometheus  → Metrics
     └──▶ Loki        → Logs
              │
              ▼
           Grafana
```

The lab also introduced production-oriented practices such as automated provisioning, health verification, continuous load generation, API-based dashboard deployment, trace analysis, error classification, span metrics, and service graphs.

The most important concept to revisit is the relationship between **raw spans and derived metrics**. Understanding how Tempo's metrics generator produces span metrics and service graphs provides a strong foundation for building **SLOs, latency monitoring, service dependency analysis, and alerting based on distributed traces**.

---

## ⭐ Skills Demonstrated

```text
🐧 Linux Administration
🐳 Docker & Docker Compose
🔭 Grafana Tempo
📡 OpenTelemetry
📊 Prometheus
📝 Loki
📈 Grafana Dashboards
🔗 Distributed Tracing
🧩 Application Instrumentation
📡 OTLP
🐚 Bash Automation
🔎 REST API / JSON Analysis
📉 Histogram Quantiles
🚨 Error Monitoring
☁️ AWS EC2
🛠️ Observability Engineering
```

---

## 👨‍💻 Lab Platform

**Environment:** AWS EC2 Ubuntu instance
**Training Platform:** Al Nafi Cloud Lab
**Focus:** Cloud DevOps / Observability / Distributed Tracing

---

> 🚀 **Build it. Trace it. Measure it. Understand it.**
>
> Modern DevOps isn't only about deploying applications — it's about understanding what happens inside them.
