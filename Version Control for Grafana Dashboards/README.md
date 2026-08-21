# 📊 Version Control for Grafana Dashboards

> 🚀 **Production-Style DevOps Lab**
> Learn how to manage Grafana dashboards as version-controlled infrastructure using **Git, Docker, Grafana REST APIs, Bash, JSON, and automated backup/rollback workflows**.

---

## 🏆 Lab Overview

This lab demonstrates how to build a **Git-backed lifecycle for Grafana dashboards**.

Instead of treating dashboards as configuration that exists only inside Grafana, you will export them as JSON files, store them in Git, create releases with Git tags, compare changes between versions, and restore historical versions when required.

### 🎯 Objectives

By completing this lab, you will be able to:

* 🔄 Export and import Grafana dashboards through the REST API
* 🌿 Manage dashboards using Git branches
* 🏷️ Create tagged dashboard releases
* 🔍 Compare dashboard JSON across Git versions
* ⏪ Roll back dashboards to historical versions
* 💾 Automate dashboard backups
* 📋 Generate structured backup manifests
* 🛠️ Build scriptable Grafana dashboard management workflows
* 🚨 Troubleshoot Grafana API and JSON-related failures

---

## 🧰 Technology Stack

| Technology      | Purpose                                |
| --------------- | -------------------------------------- |
| 🟠 **Grafana**  | Dashboard visualization and management |
| 🐧 **Ubuntu**   | Lab operating system                   |
| 🐳 **Docker**   | Grafana container runtime              |
| 🔀 **Git**      | Version control                        |
| 🐚 **Bash**     | Automation scripts                     |
| 🌐 **REST API** | Grafana dashboard communication        |
| 🧩 **jq**       | JSON processing                        |
| 📡 **curl**     | HTTP API requests                      |
| 📁 **JSON**     | Dashboard configuration format         |
| ☁️ **AWS EC2**  | Lab infrastructure                     |

---

## 🔰 Prerequisites

Before starting, you should have:

* 🐧 Basic Linux command-line knowledge
* 🔐 Understanding of Linux permissions
* 🔀 Familiarity with Git branches and tags
* 📦 Basic Docker knowledge
* 🌐 Understanding of REST APIs
* 🧩 Basic JSON knowledge
* 🐚 Basic Bash scripting skills
* ☁️ Access to an AWS EC2 Ubuntu instance provided by Al Nafi

---

# 🏗️ Lab Environment

The lab uses an **AWS EC2 Ubuntu instance**.

Grafana will run inside Docker and its data will be persisted using a host-mounted directory.

### 📁 Directory Structure

```text
~/grafana-vc/
├── data/
└── repo/
    ├── dashboards/
    │   ├── production/
    │   ├── staging/
    │   └── development/
    ├── backups/
    ├── README.md
    ├── .gitignore
    ├── export-dashboard.sh
    ├── import-dashboard.sh
    ├── diff-dashboards.sh
    ├── rollback-dashboard.sh
    └── backup-all-dashboards.sh
```

---

# 🚀 Task 1 — Provision the Environment

## 🔹 Step 1.1 — Install Runtime Dependencies

Install Git, Docker, curl, jq, and tree.

### 💻 Technology: Ubuntu + APT

```bash
sudo apt-get update -y
sudo apt-get install -y git docker.io curl jq tree

sudo systemctl enable --now docker
sudo usermod -aG docker "$USER"
newgrp docker
```

### ✅ Verify Installation

```bash
git --version && docker --version && curl --version | head -1 && jq --version
```

All commands should return version information successfully.

### 🔎 Troubleshooting

If Docker cannot be installed:

```bash
sudo apt-get update -y
```

Then retry the installation.

For official Docker documentation, see:

https://docs.docker.com/engine/install/ubuntu/

---

# 🐳 Step 1.2 — Deploy Grafana

Create persistent Grafana storage and the Git repository.

### 💻 Technology: Docker + Grafana

```bash
mkdir -p "$HOME/grafana-vc/data" "$HOME/grafana-vc/repo"

sudo chown -R 472:472 "$HOME/grafana-vc/data"
```

Start Grafana:

```bash
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -v "$HOME/grafana-vc/data:/var/lib/grafana" \
  -e GF_SECURITY_ADMIN_PASSWORD=LabPass99 \
  grafana/grafana:latest
```

### ❤️ Check Grafana Health

```bash
until curl -fsSL http://localhost:3000/api/health | \
jq -e '.database == "ok"' > /dev/null 2>&1; do
  echo "Waiting for Grafana..."
  sleep 5
done

echo "Grafana is ready"
```

Expected:

```text
Grafana is ready
```

### 🔍 Check Container

```bash
docker ps
```

You should see the Grafana container running.

If Grafana fails:

```bash
docker logs grafana
```

Check directory ownership:

```bash
ls -ln "$HOME/grafana-vc/data"
```

Grafana normally runs inside the container using UID **472**.

---

# 🔀 Step 1.3 — Initialize Git Repository

Move into the repository:

```bash
cd "$HOME/grafana-vc/repo"
```

Initialize Git:

```bash
git init

git config user.name "Dashboard Engineer"
git config user.email "engineer@lab.local"
```

Create the environment structure:

```bash
mkdir -p dashboards/production
mkdir -p dashboards/staging
mkdir -p dashboards/development
mkdir -p backups
```

Create the README:

```bash
cat > README.md << 'EOF'
# Grafana Dashboard Version Control

## Layout
- dashboards/production/  — live dashboards
- dashboards/staging/     — pre-release candidates
- dashboards/development/ — experimental work

## Workflow
Export from Grafana -> commit to branch -> tag release -> merge to production
EOF
```

Create `.gitignore`:

```bash
cat > .gitignore << 'EOF'
*.bak
*.tmp
backups/
EOF
```

Commit the initial structure:

```bash
git add .
git commit -m "chore: initialise dashboard repository structure"
```

Create branches:

```bash
git checkout -b development
git checkout -b staging
git checkout main
```

### ✅ Verify Branches

```bash
git branch
```

Expected:

```text
  development
* main
  staging
```

Check the commit:

```bash
git log --oneline
```

---

# ⚙️ Task 2 — Build the Dashboard Version Control Pipeline

The pipeline contains five automation scripts:

```text
export-dashboard.sh
import-dashboard.sh
diff-dashboards.sh
rollback-dashboard.sh
backup-all-dashboards.sh
```

These scripts form the core of the dashboard lifecycle.

---

# 📤 Step 2.1 — Export, Import & Diff Contracts

## 📤 `export-dashboard.sh`

### Purpose

The export script:

1. Receives a dashboard UID.
2. Queries Grafana.
3. Removes mutable `.id` and `.version` fields.
4. Generates a normalized filename.
5. Saves the dashboard into the selected environment.

Example:

```bash
./export-dashboard.sh <uid> production
```

Output structure:

```text
dashboards/production/infrastructure-overview.json
```

---

## 📥 `import-dashboard.sh`

### Purpose

Imports a bare dashboard JSON file into Grafana.

The script must construct:

```json
{
  "dashboard": {},
  "folderId": 0,
  "overwrite": true
}
```

Then POST the payload to:

```text
/api/dashboards/db
```

Example:

```bash
./import-dashboard.sh dashboards/production/infrastructure-overview.json
```

Successful execution should print the resulting Grafana dashboard URL.

---

## 🔍 `diff-dashboards.sh`

### Purpose

Compare dashboard versions stored in different Git references.

Example:

```bash
./diff-dashboards.sh \
  v1.0.0 \
  HEAD \
  dashboards/development/infrastructure-overview.json
```

The output should include:

* 📊 Panel count delta
* 🏷️ Tag delta
* 📝 Unified JSON diff

Temporary files must be cleaned automatically using a shell `trap`.

---

## 🔐 Make Scripts Executable

```bash
chmod +x export-dashboard.sh
chmod +x import-dashboard.sh
chmod +x diff-dashboards.sh
```

Verify:

```bash
ls -l *.sh
```

Expected permission pattern:

```text
-rwxr-xr-x
```

---

# 🌱 Step 2.2 — Seed Production Dashboards

Create two Grafana dashboards:

### 📊 Dashboard 1

```text
Infrastructure Overview
```

### 📊 Dashboard 2

```text
Application Latency
```

Each dashboard should contain at least **two different panel types**.

Example:

```text
Infrastructure Overview
├── Stat
└── Time series

Application Latency
├── Stat
└── Time series
```

PromQL queries can be used even when no live metrics are available.

---

## 🔎 Retrieve Dashboard UIDs

```bash
curl -fsSL \
  -u admin:LabPass99 \
  http://localhost:3000/api/search |
  jq -r '.[] | [.uid, .title] | @tsv'
```

Example:

```text
abc123    Infrastructure Overview
def456    Application Latency
```

---

# 📦 Export Production Dashboards

Use the export script:

```bash
./export-dashboard.sh <infrastructure-uid> production
./export-dashboard.sh <latency-uid> production
```

Verify:

```bash
tree dashboards/
```

Expected:

```text
dashboards/
├── development/
├── production/
│   ├── application-latency.json
│   └── infrastructure-overview.json
└── staging/
```

---

# 📝 Commit Production Release

```bash
git add dashboards/production/
git commit -m "feat: add initial production dashboards"
```

Create release tag:

```bash
git tag v1.0.0
```

Verify:

```bash
git tag
```

Expected:

```text
v1.0.0
```

Inspect release:

```bash
git show v1.0.0 --stat
```

---

# 🌿 Step 2.3 — Evolve Dashboards on Development

Switch to the development branch:

```bash
git checkout development
```

Modify **Infrastructure Overview**:

### Changes

* ➕ Add a third panel
* 🕐 Change default time range from:

  ```text
  now-1h
  ```

  to:

  ```text
  now-6h
  ```

Export the updated dashboard:

```bash
./export-dashboard.sh <infrastructure-uid> development
```

Commit:

```bash
git add dashboards/development/
git commit -m "feat: expand infrastructure dashboard"
```

---

# 🔎 Review the Dashboard Diff

```bash
./diff-dashboards.sh \
  v1.0.0 \
  HEAD \
  dashboards/development/infrastructure-overview.json
```

You should observe:

```text
Panel count:
2 → 3

Time range:
now-1h → now-6h
```

This simulates a dashboard code review before production promotion.

---

# 🚀 Merge Development into Production

Switch to main:

```bash
git checkout main
```

Merge using `--no-ff`:

```bash
git merge --no-ff development \
  -m "feat: promote infrastructure-overview v2 to production"
```

Export the final production dashboard:

```bash
./export-dashboard.sh <infrastructure-uid> production
```

Commit:

```bash
git add dashboards/production/
git commit -m "release: promote infrastructure dashboard v2"
```

Create the second release:

```bash
git tag v2.0.0
```

Verify:

```bash
git tag
```

Expected:

```text
v1.0.0
v2.0.0
```

Inspect the Git history:

```bash
git log --oneline --graph main
```

The history should contain a merge commit.

---

# ⏪ Task 3 — Implement Rollback & Automated Backup

## 🔙 Step 3.1 — Rollback Contract

Create:

```text
rollback-dashboard.sh
```

The script must perform these operations:

```text
1. Validate JSON file
        ↓
2. Validate Git reference
        ↓
3. Backup current JSON
        ↓
4. Restore historical Git version
        ↓
5. Import restored dashboard into Grafana
        ↓
6. Commit restored version
```

The script accepts:

```bash
./rollback-dashboard.sh <json-file> <git-ref>
```

If a reason is not supplied as the third argument, read it from standard input.

---

# 🛡️ Rollback Safety

Before modifying the dashboard:

```text
Current Dashboard
       ↓
Timestamped Backup
       ↓
Git Historical Version
       ↓
Grafana Import
       ↓
Git Commit
```

If any operation fails:

```text
Failure
   ↓
Restore Backup
   ↓
Exit Non-Zero
```

This protects the working tree from partial rollback operations.

---

# 🔄 Demonstrate a Rollback

Rollback:

```text
Infrastructure Overview
v2.0.0 → v1.0.0
```

After rollback, verify the dashboard through Grafana's API.

```bash
curl -fsSL -u admin:LabPass99 \
  "http://localhost:3000/api/dashboards/uid/$(
    curl -fsSL -u admin:LabPass99 \
    http://localhost:3000/api/search |
    jq -r '.[] |
      select(.title=="Infrastructure Overview") |
      .uid
  )" |
  jq '.dashboard.panels | length'
```

Expected output:

```text
2
```

🎉 The historical version has been restored.

---

# 💾 Step 3.2 — Automated Dashboard Backup

Create:

```text
backup-all-dashboards.sh
```

The script should:

1. 🔎 Discover every dashboard.
2. 📥 Export each dashboard.
3. 🧹 Remove mutable fields.
4. 📁 Store backups by date.
5. 📋 Generate a manifest.
6. 🚫 Never commit backups to Git.

---

# 📁 Backup Structure

Example:

```text
backups/
└── 2026-08-21/
    ├── infrastructure-overview.json
    ├── application-latency.json
    └── manifest.json
```

---

# 📋 Manifest Structure

The manifest contains:

```json
[
  {
    "uid": "string",
    "title": "string",
    "slug": "string",
    "file": "string",
    "exported_at": "RFC-3339 timestamp"
  }
]
```

This creates an independent audit trail for disaster recovery.

---

# ▶️ Run Automated Backup

```bash
./backup-all-dashboards.sh
```

Expected output:

```text
Backed up N dashboards to backups/2026-08-21/
```

---

# 🔎 Validate Backup Manifest

```bash
BACKUP_DATE=$(date +%Y-%m-%d)

MANIFEST="$HOME/grafana-vc/repo/backups/${BACKUP_DATE}/manifest.json"

DASHBOARD_COUNT=$(curl -fsSL \
  -u admin:LabPass99 \
  "http://localhost:3000/api/search?type=dash-db" |
  jq 'length')

MANIFEST_COUNT=$(jq 'length' "$MANIFEST")

echo "API count: $DASHBOARD_COUNT | Manifest count: $MANIFEST_COUNT"

[ "$DASHBOARD_COUNT" -eq "$MANIFEST_COUNT" ] \
  && echo "PASS" \
  || echo "FAIL"
```

Expected:

```text
API count: 2 | Manifest count: 2
PASS
```

---

# 🧪 Final Validation

Verify all scripts exist:

```bash
ls -l *.sh
```

Expected:

```text
export-dashboard.sh
import-dashboard.sh
diff-dashboards.sh
rollback-dashboard.sh
backup-all-dashboards.sh
```

Verify Git branches:

```bash
git branch
```

Verify tags:

```bash
git tag
```

Expected:

```text
v1.0.0
v2.0.0
```

Verify history:

```bash
git log --oneline --graph --all
```

Verify repository:

```bash
git status
```

---

# 🧩 Troubleshooting

## ❌ Import Returns HTTP 422

If Grafana returns:

```text
Dashboard title cannot be empty
```

the JSON being imported may not contain the expected **bare dashboard object**.

Inspect the JSON:

```bash
jq '.' dashboards/production/infrastructure-overview.json
```

Check the title:

```bash
jq '.title' dashboards/production/infrastructure-overview.json
```

Expected:

```text
"Infrastructure Overview"
```

The import script should send:

```json
{
  "dashboard": {
    "...": "dashboard content"
  },
  "folderId": 0,
  "overwrite": true
}
```

It should **not** accidentally wrap an already wrapped Grafana API response.

---

# 🔬 Inspect Exact API Payload

Before sending the request, save or print the generated payload:

```bash
jq '.' payload.json
```

You can also inspect the JSON bytes:

```bash
cat payload.json
```

For deeper troubleshooting:

```bash
jq -c '.' payload.json
```

This helps identify malformed JSON, incorrect nesting, missing titles, and unexpected fields.

---

# ⚠️ Rollback Succeeds but Grafana Does Not Change

Two common causes should be investigated.

### 🐚 Script-Side Cause

The import script may be:

* Reading the wrong file
* Building the wrong JSON envelope
* Failing to send the restored content
* Importing a different dashboard UID
* Silently ignoring an API error

Check the restored file:

```bash
git diff
```

Then inspect:

```bash
jq '.title, .panels | length' \
  dashboards/production/infrastructure-overview.json
```

---

### 📊 Grafana API Cause

Grafana may have rejected the update or the request may have targeted the wrong dashboard.

Inspect the API response directly:

```bash
curl -i -fsSL \
  -u admin:LabPass99 \
  http://localhost:3000/api/dashboards/uid/<UID>
```

The response allows you to determine whether Grafana currently contains the expected dashboard state.

---

# 🔐 Recommended Production Practices

For a real production environment, improve this lab by adding:

* 🔑 API tokens instead of passwords
* 🔒 HTTPS/TLS
* 👤 Dedicated Grafana service accounts
* 🌳 Protected Git branches
* 🔍 Pull-request review
* 🏷️ Semantic versioning
* 🤖 CI validation for dashboard JSON
* 🧪 Automated API tests
* 💾 Remote backup storage
* 📊 Dashboard linting
* 🚨 Monitoring for backup failures
* 🔐 Secret management through a secure vault
* 📜 Audit logging
* 🔄 Automated deployment pipelines

---

# 🏁 Expected Outcomes

After completing the lab, you should have:

### ✅ Git Lifecycle

```text
Development
     ↓
Staging
     ↓
Production
     ↓
Tagged Release
```

### ✅ Release History

```text
v1.0.0
   ↓
Development Changes
   ↓
v2.0.0
   ↓
Rollback
```

### ✅ Automation

Five working scripts:

```text
export-dashboard.sh
import-dashboard.sh
diff-dashboards.sh
rollback-dashboard.sh
backup-all-dashboards.sh
```

### ✅ Backup

A date-based backup directory containing:

```text
Dashboard JSON
+
manifest.json
```

### ✅ Recovery

Any historical dashboard version can be restored from Git and pushed back into Grafana.

---

# 🧠 Key Concepts Learned

| Concept               | What You Learned                      |
| --------------------- | ------------------------------------- |
| 📊 Grafana API        | Programmatically manage dashboards    |
| 🔀 Git Branching      | Isolate dashboard changes             |
| 🏷️ Git Tags          | Create dashboard releases             |
| 📝 JSON               | Store dashboards as code              |
| 🔍 Git Diff           | Review dashboard changes              |
| ⏪ Rollback            | Restore historical dashboard versions |
| 💾 Backup             | Protect dashboard configurations      |
| 📋 Manifest           | Track backup metadata                 |
| 🐚 Bash               | Automate dashboard operations         |
| 🐳 Docker             | Run persistent Grafana                |
| 🔐 API Authentication | Secure Grafana API access             |

---

# 🎯 Conclusion

This lab establishes a **Dashboard-as-Code** workflow for Grafana.

Instead of relying exclusively on the Grafana UI, dashboards become version-controlled artifacts that can be:

```text
Exported
   ↓
Versioned
   ↓
Reviewed
   ↓
Tagged
   ↓
Released
   ↓
Backed Up
   ↓
Rolled Back
```

The combination of **Grafana REST APIs, Git, Bash, Docker, JSON, curl, and jq** creates a repeatable dashboard lifecycle suitable for DevOps and observability environments.

🚀 **The final result is a production-oriented Grafana workflow where dashboard configuration is reproducible, auditable, reviewable, and recoverable.**

---

## 📚 Official Documentation

* Grafana Docker: https://hub.docker.com/r/grafana/grafana
* Grafana Docker Installation: https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/
* Grafana Dashboard HTTP API: https://grafana.com/docs/grafana/latest/developers/http_api/dashboard/
* Docker Engine on Ubuntu: https://docs.docker.com/engine/install/ubuntu/

---

## 🏆 Lab Completion Checklist

* [ ] Install Git, Docker, curl, jq, and tree
* [ ] Start Docker
* [ ] Deploy Grafana
* [ ] Verify Grafana health
* [ ] Initialize Git repository
* [ ] Create development, staging, and production directories
* [ ] Create Git branches
* [ ] Implement export script
* [ ] Implement import script
* [ ] Implement diff script
* [ ] Create production dashboards
* [ ] Export dashboards to Git
* [ ] Create `v1.0.0`
* [ ] Modify dashboard on development branch
* [ ] Review dashboard diff
* [ ] Merge development with `--no-ff`
* [ ] Create `v2.0.0`
* [ ] Implement rollback script
* [ ] Roll back to `v1.0.0`
* [ ] Verify Grafana panel count
* [ ] Implement automated backup
* [ ] Generate backup manifest
* [ ] Validate API and manifest counts
* [ ] Verify complete Git history

---

# ⭐ Final Status

**🎉 Lab Complete — Grafana Dashboards are now managed as version-controlled infrastructure!**

**Technologies:** `Grafana` • `Git` • `Docker` • `Ubuntu` • `Bash` • `REST API` • `JSON` • `curl` • `jq` • `AWS EC2`
