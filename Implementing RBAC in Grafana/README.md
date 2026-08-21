# 🔐 Implementing RBAC in Grafana

![Grafana](https://img.shields.io/badge/Grafana-RBAC-F46800?style=for-the-badge\&logo=grafana\&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-E95420?style=for-the-badge\&logo=ubuntu\&logoColor=white)
![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-E6522C?style=for-the-badge\&logo=prometheus\&logoColor=white)
![InfluxDB](https://img.shields.io/badge/Database-InfluxDB-22ADF6?style=for-the-badge\&logo=influxdb\&logoColor=white)
![Bash](https://img.shields.io/badge/Scripting-Bash-4EAA25?style=for-the-badge\&logo=gnubash\&logoColor=white)
![REST API](https://img.shields.io/badge/API-REST-009688?style=for-the-badge)

> 🛡️ **A hands-on Grafana security lab focused on Role-Based Access Control, teams, folders, custom permissions, API access, auditing, and least-privilege security.**

---

## 📚 Table of Contents

* [🎯 Lab Objectives](#-lab-objectives)
* [📋 Prerequisites](#-prerequisites)
* [🖥️ Lab Environment](#️-lab-environment)
* [🏗️ RBAC Architecture](#️-rbac-architecture)
* [🚀 Task 1 — Installing Grafana](#-task-1--installing-grafana)
* [👥 Task 2 — User Management](#-task-2--configuring-basic-user-management)
* [🔐 Task 3 — Understanding Grafana RBAC](#-task-3--understanding-grafana-rbac-structure)
* [⚙️ Task 4 — Custom Roles and Permissions](#️-task-4--creating-custom-roles-and-permissions)
* [📊 Task 5 — Sample Data and Dashboards](#-task-5--creating-sample-dashboards-for-testing)
* [🧪 Task 6 — Testing Access Controls](#-task-6--testing-access-controls)
* [🎛️ Task 7 — Advanced RBAC](#️-task-7--advanced-rbac-configuration)
* [📜 Task 8 — Monitoring and Auditing](#-task-8--monitoring-and-auditing-access)
* [🛠️ Troubleshooting](#️-troubleshooting-common-issues)
* [🛡️ Security Best Practices](#️-security-best-practices)
* [🏁 Conclusion](#-conclusion)

---

# 🎯 Lab Objectives

By completing this lab, you will learn how to:

* 🚀 Install and configure Grafana on Linux
* 🔐 Understand Role-Based Access Control
* 👤 Create and manage Grafana users
* 👥 Create teams for access management
* 📁 Configure folder-level permissions
* 🎛️ Create custom Grafana roles
* 🔑 Control API access
* 🧪 Test permissions for different users
* 📜 Monitor and audit access
* 🛡️ Apply least-privilege security principles
* 💾 Back up Grafana configuration and database

---

# 📋 Prerequisites

Before starting, you should have:

* 🐧 Basic Linux command-line knowledge
* 🌐 Basic networking knowledge
* 👤 Understanding of user management
* 📊 Basic monitoring knowledge
* 📝 Familiarity with JSON/configuration files
* 🔧 Basic Bash knowledge
* 🔐 Basic understanding of access-control concepts

---

# 🖥️ Lab Environment

This lab can be performed on an **Al Nafi Linux cloud machine**.

The environment starts with a bare Linux system, so Grafana and the required supporting components must be installed manually.

### 🧰 Main Technologies

| Technology      | Purpose                          |
| --------------- | -------------------------------- |
| 🟠 Grafana      | Visualization and RBAC           |
| 🐧 Ubuntu/Linux | Operating system                 |
| 🟦 InfluxDB     | Sample monitoring data source    |
| 🐚 Bash         | Automation and administration    |
| 🌐 REST API     | API access testing               |
| 🔐 RBAC         | Authorization and access control |
| 📜 Audit Logs   | Security monitoring              |

---

# 🏗️ RBAC Architecture

The lab implements the following logical structure:

```text
                         ┌─────────────────────┐
                         │      Grafana        │
                         │       Server       │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   RBAC / Access     │
                         │      Control        │
                         └──────────┬──────────┘
                                    │
             ┌──────────────────────┼──────────────────────┐
             │                      │                      │
      ┌──────▼──────┐       ┌──────▼──────┐       ┌──────▼──────┐
      │ Development │       │ Management  │       │ Operations  │
      │    Team     │       │    Team     │       │    Team     │
      └──────┬──────┘       └──────┬──────┘       └──────┬──────┘
             │                      │                      │
             ▼                      ▼                      ▼
       Development              Business                System
        Metrics                 Metrics                Monitoring
```

### 👤 Lab Users

| User         | Team             | Primary Role      |
| ------------ | ---------------- | ----------------- |
| `jdeveloper` | Development Team | Developer         |
| `smanager`   | Management Team  | Manager           |
| `bviewer`    | Operations Team  | Operations Viewer |

---

# 🚀 Task 1 — Installing and Setting Up Grafana

## 🔹 Step 1.1 — Update the System

✨ **Start by updating the package repository and installing dependencies.**

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y \
software-properties-common \
apt-transport-https \
wget \
curl \
gnupg2
```

✅ **Expected Result:** Required Linux packages are installed successfully.

---

## 🔹 Step 1.2 — Install Grafana

✨ **Add the Grafana repository and install Grafana.**

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" | \
sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt update

sudo apt install -y grafana
```

> ⚠️ **Note:** For new deployments, prefer Grafana's current official installation documentation because repository/key-management instructions can change between releases.

---

## 🔹 Step 1.3 — Start Grafana

✨ **Enable Grafana at boot and start the service.**

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

✅ You should see:

```text
Active: active (running)
```

---

## 🔹 Step 1.4 — Verify Grafana

✨ **Check whether Grafana is listening on port `3000`.**

```bash
sudo ss -tlnp | grep :3000
```

View Grafana logs:

```bash
sudo journalctl -u grafana-server -f
```

Open:

```text
http://localhost:3000
```

### 🔑 Default Login

```text
Username: admin
Password: admin
```

🔐 Change the default password immediately after the first login.

---

# 👥 Task 2 — Configuring Basic User Management

## 🔹 Step 2.1 — Change the Admin Password

✨ **Never leave the default administrator password enabled.**

1. Login to Grafana.
2. Use the default credentials.
3. Change the administrator password.
4. Store the new password securely.

🚨 **Security Rule:** Never commit Grafana credentials, API keys, or passwords to GitHub.

---

## 🔹 Step 2.2 — Optional User Registration

Edit:

```bash
sudo nano /etc/grafana/grafana.ini
```

Configure:

```ini
[users]
allow_sign_up = true
auto_assign_org_role = Viewer
```

Restart:

```bash
sudo systemctl restart grafana-server
```

### 🛡️ Production Recommendation

For production environments, avoid unrestricted public registration unless there is a specific business requirement.

---

## 🔹 Step 2.3 — Create Test Users

Create these users through the Grafana interface:

### 👨‍💻 Developer

```text
Name: John Developer
Email: john.developer@company.com
Username: jdeveloper
```

### 👩‍💼 Manager

```text
Name: Sarah Manager
Email: sarah.manager@company.com
Username: smanager
```

### 👨‍🔧 Viewer

```text
Name: Bob Viewer
Email: bob.viewer@company.com
Username: bviewer
```

⚠️ Use unique temporary lab passwords rather than publishing reusable passwords in documentation or repositories.

---

# 🔐 Task 3 — Understanding Grafana RBAC Structure

## 🔹 Step 3.1 — Default Organization Roles

Grafana commonly provides organization roles such as:

| Role       | General Capability                                            |
| ---------- | ------------------------------------------------------------- |
| 👑 Admin   | Administrative organization access                            |
| ✏️ Editor  | Create/edit dashboards and resources according to permissions |
| 👁️ Viewer | Read-only access                                              |

> 💡 Exact RBAC capabilities depend on the Grafana edition/version and configured permissions.

---

## 🔹 Step 3.2 — Important RBAC Resources

Grafana access control can involve:

```text
Organization
     │
     ├── Users
     │
     ├── Teams
     │
     ├── Folders
     │      └── Dashboards
     │
     └── Data Sources
```

### 🔑 Core Principle

> **Users should receive only the permissions required to perform their responsibilities.**

This is the **Principle of Least Privilege**.

---

# ⚙️ Task 4 — Creating Custom Roles and Permissions

## 🔹 Step 4.1 — Enable Access Control

Edit:

```bash
sudo nano /etc/grafana/grafana.ini
```

If your Grafana version supports the relevant feature toggle, configure:

```ini
[feature_toggles]
accesscontrol = true
```

Restart:

```bash
sudo systemctl restart grafana-server
```

Verify:

```bash
grep "accesscontrol" /etc/grafana/grafana.ini
```

> ⚠️ **Version Note:** Grafana RBAC capabilities and feature availability vary by Grafana version and edition. Always verify the configuration against the version installed in your lab.

---

## 🔹 Step 4.2 — Create Teams

Navigate to:

```text
Administration / Configuration → Teams
```

### 👨‍💻 Development Team

```text
Name: Development Team
Description: Developers who need dashboard editing access
Member: jdeveloper
```

### 👩‍💼 Management Team

```text
Name: Management Team
Description: Managers who need read access to dashboards
Member: smanager
```

### 👨‍🔧 Operations Team

```text
Name: Operations Team
Description: Operations staff with monitoring access
Member: bviewer
```

---

# 📁 Task 4.3 — Create Dashboard Folders

Create:

```text
Development Metrics
Business Metrics
System Monitoring
```

Folders provide an important boundary for dashboard organization and permissions.

---

# 🔒 Task 4.4 — Configure Folder Permissions

## 👨‍💻 Development Metrics

| Team             | Permission   |
| ---------------- | ------------ |
| Development Team | ✏️ Edit      |
| Management Team  | 👁️ View     |
| Operations Team  | 🚫 No Access |

---

## 👩‍💼 Business Metrics

| Team             | Permission |
| ---------------- | ---------- |
| Management Team  | ✏️ Edit    |
| Development Team | 👁️ View   |
| Operations Team  | 👁️ View   |

---

## 👨‍🔧 System Monitoring

| Team             | Permission |
| ---------------- | ---------- |
| Operations Team  | ✏️ Edit    |
| Development Team | 👁️ View   |
| Management Team  | 👁️ View   |

### ⭐ Result

```text
Development Team ──► Development Metrics
                   └──► View Business Metrics

Management Team ───► Business Metrics
                   ├──► View Development Metrics
                   └──► View System Monitoring

Operations Team ───► System Monitoring
                   └──► View Business Metrics
```

---

# 📊 Task 5 — Creating Sample Dashboards for Testing

## 🔹 Step 5.1 — Install InfluxDB

✨ **Use InfluxDB as a sample data source for the lab.**

```bash
sudo apt install -y influxdb influxdb-client
```

Start and enable:

```bash
sudo systemctl start influxdb
sudo systemctl enable influxdb
```

Create the sample database where supported by the installed InfluxDB version:

```bash
influx -execute "CREATE DATABASE sampledata"
```

> ⚠️ **Version Note:** InfluxDB 1.x and 2.x use different configuration and database concepts. If the command above is unavailable, follow the setup method for the installed InfluxDB release.

---

# 🔹 Step 5.2 — Add InfluxDB to Grafana

Navigate to:

```text
Configuration → Data Sources → Add data source
```

Select:

```text
InfluxDB
```

Configure:

```text
Name: SampleInfluxDB
URL: http://localhost:8086
Database: sampledata
```

Click:

```text
Save & Test
```

✅ **Expected Result:**

```text
Data source connected successfully
```

---

# 🔹 Step 5.3 — Create Sample Dashboards

### 📈 Application Performance

Folder:

```text
Development Metrics
```

Dashboard:

```text
Application Performance
```

---

### 💰 Revenue Tracking

Folder:

```text
Business Metrics
```

Dashboard:

```text
Revenue Tracking
```

---

### 🖥️ Server Health

Folder:

```text
System Monitoring
```

Dashboard:

```text
Server Health
```

---

# 🧪 Task 6 — Testing Access Controls

## 🔹 Step 6.1 — Test Developer

Login:

```text
Username: jdeveloper
```

Expected:

| Resource            | Result       |
| ------------------- | ------------ |
| Development Metrics | ✅ EDIT       |
| Business Metrics    | 👁️ VIEW     |
| System Monitoring   | 🚫 NO ACCESS |
| Administration      | 🚫 BLOCKED   |

---

## 🔹 Step 6.2 — Test Manager

Login:

```text
Username: smanager
```

Expected:

| Resource            | Result     |
| ------------------- | ---------- |
| Business Metrics    | ✅ EDIT     |
| Development Metrics | 👁️ VIEW   |
| System Monitoring   | 👁️ VIEW   |
| Administration      | 🚫 BLOCKED |

---

## 🔹 Step 6.3 — Test Operations User

Login:

```text
Username: bviewer
```

Expected:

| Resource            | Result       |
| ------------------- | ------------ |
| System Monitoring   | ✅ EDIT       |
| Business Metrics    | 👁️ VIEW     |
| Development Metrics | 🚫 NO ACCESS |
| Configuration       | 🚫 BLOCKED   |

---

# 📝 Step 6.4 — Document Test Results

Create:

```bash
nano rbac_test_results.txt
```

Example:

```text
Grafana RBAC Test Results
=========================

User: jdeveloper
Development Metrics: EDIT - PASS
Business Metrics: VIEW - PASS
System Monitoring: NO ACCESS - PASS
Admin Functions: BLOCKED - PASS

User: smanager
Business Metrics: EDIT - PASS
Development Metrics: VIEW - PASS
System Monitoring: VIEW - PASS
Admin Functions: BLOCKED - PASS

User: bviewer
System Monitoring: EDIT - PASS
Business Metrics: VIEW - PASS
Development Metrics: NO ACCESS - PASS
Admin Functions: BLOCKED - PASS

Overall RBAC Implementation: SUCCESS
```

---

# 🎛️ Task 7 — Advanced RBAC Configuration

## 🔹 Step 7.1 — Create a Custom Role

Navigate to the access-control/roles interface available in your Grafana version.

Create:

```text
Role Name: Dashboard Creator
Description:
Can create and edit dashboards without managing users.
```

Example permissions:

```text
dashboards:create
dashboards:write
folders:create
folders:write
```

> ⚠️ Permission names and availability can vary by Grafana version and edition.

---

# 🔹 Step 7.2 — Assign the Custom Role

1. Open the target user.
2. Open the **Roles** section.
3. Select **Dashboard Creator**.
4. Save the changes.
5. Login as the user.
6. Test dashboard and folder operations.
7. Verify administrative functions remain unavailable.

---

# 🔑 Step 7.3 — API Access Control

For current Grafana releases, API access is commonly handled through **service accounts and tokens** rather than legacy API keys.

Where supported by your deployment, create appropriately scoped credentials.

Example API test:

```bash
curl \
  -H "Authorization: Bearer YOUR_TOKEN" \
  http://localhost:3000/api/dashboards/home
```

For an API operation requiring write access:

```bash
curl \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  http://localhost:3000/api/dashboards/db
```

### 🛡️ API Security Rules

* 🔐 Use minimum required scopes.
* 🚫 Never commit tokens to Git.
* 🔄 Rotate credentials periodically.
* 🧹 Revoke unused tokens.
* 📜 Audit API activity.
* 🔒 Prefer HTTPS in production.

---

# 📜 Task 8 — Monitoring and Auditing Access

## 🔹 Step 8.1 — Configure Logging

Edit:

```bash
sudo nano /etc/grafana/grafana.ini
```

Example logging configuration:

```ini
[log]
mode = console file
level = info

[log.file]
log_rotate = true
max_lines = 1000000
max_size_shift = 28
daily_rotate = true
max_days = 7
```

If audit logging is available for your Grafana edition/version, configure it according to the corresponding Grafana documentation.

Restart:

```bash
sudo systemctl restart grafana-server
```

---

# 🔹 Step 8.2 — Monitor Grafana Logs

View recent logs:

```bash
sudo journalctl -u grafana-server -n 100
```

Monitor logs live:

```bash
sudo journalctl -u grafana-server -f
```

If file logging is enabled:

```bash
sudo tail -f /var/log/grafana/grafana.log
```

---

# 🔹 Step 8.3 — Create Access Monitoring Dashboard

Create:

```text
Access Monitoring
```

Recommended panels:

```text
🔐 Login Attempts
📊 Dashboard Access
🚨 Failed Authentication
👥 Activity by User
🛡️ Activity by Role
📁 Folder Access
```

This dashboard can help security and operations teams identify unusual activity.

---

# 🛠️ Troubleshooting Common Issues

## ❌ Issue 1 — Users Cannot Access Dashboards

### 🔍 Symptoms

```text
Access Denied
```

### 🔧 Troubleshooting

Check:

```text
✓ User role
✓ Team membership
✓ Folder permissions
✓ Dashboard permissions
✓ Organization membership
✓ Default permissions
```

Restart if necessary:

```bash
sudo systemctl restart grafana-server
```

---

# ❌ Issue 2 — RBAC Features Are Missing

Check:

```bash
grep "accesscontrol" /etc/grafana/grafana.ini
```

Check version:

```bash
grafana-server -v
```

Restart:

```bash
sudo systemctl restart grafana-server
```

Also verify that the required RBAC capability is supported by your installed Grafana edition/version.

---

# ❌ Issue 3 — Permission Changes Are Not Applied

Try:

```text
1. Log out
2. Log back in
3. Clear browser cache
4. Verify team membership
5. Verify folder permissions
6. Restart Grafana
```

Restart:

```bash
sudo systemctl restart grafana-server
```

---

# ❌ Issue 4 — Grafana Service Is Not Running

Check:

```bash
sudo systemctl status grafana-server
```

View errors:

```bash
sudo journalctl -u grafana-server -n 100 --no-pager
```

Restart:

```bash
sudo systemctl restart grafana-server
```

---

# 🛡️ Security Best Practices

## 🔐 1. Follow Least Privilege

Give users only the permissions required for their job.

```text
Viewer  → Read
Editor  → Modify dashboards
Admin   → Administrative tasks
Custom  → Specific capabilities
```

---

## 🔑 2. Use Strong Passwords

Configure appropriate security settings in:

```bash
sudo nano /etc/grafana/grafana.ini
```

Example:

```ini
[security]
min_password_length = 12
```

> 💡 Password-policy options differ between Grafana versions. Use the security settings supported by your installed release.

---

## 👥 3. Manage Access Through Teams

Prefer team-based authorization where practical:

```text
User
  ↓
Team
  ↓
Role / Permission
  ↓
Folder
  ↓
Dashboard
```

This makes access management easier to maintain as the organization grows.

---

## 🔍 4. Audit Permissions Regularly

Create an audit script:

```bash
cat > audit_permissions.sh << 'EOF'
#!/bin/bash

echo "=========================================="
echo "Grafana RBAC Audit Report"
echo "Date: $(date)"
echo "=========================================="

echo ""
echo "Active Users"
echo "------------"
echo "Review users from the Grafana administration interface."

echo ""
echo "Team Memberships"
echo "----------------"
echo "Review teams and membership assignments."

echo ""
echo "Folder Permissions"
echo "------------------"
echo "Review permissions configured on dashboard folders."

echo ""
echo "Audit completed."
EOF

chmod +x audit_permissions.sh
```

Run:

```bash
./audit_permissions.sh
```

---

# 💾 5. Back Up Grafana

Create:

```bash
cat > backup_grafana.sh << 'EOF'
#!/bin/bash

BACKUP_DIR="/backup/grafana/$(date +%Y%m%d)"

mkdir -p "$BACKUP_DIR"

cp /etc/grafana/grafana.ini "$BACKUP_DIR/"

if [ -f /var/lib/grafana/grafana.db ]; then
    cp /var/lib/grafana/grafana.db "$BACKUP_DIR/"
fi

echo "Grafana backup completed:"
echo "$BACKUP_DIR"
EOF

chmod +x backup_grafana.sh
```

Run:

```bash
sudo ./backup_grafana.sh
```

### 🔒 Backup Security

Protect backups because they may contain:

* Configuration
* User information
* Dashboard definitions
* Database contents
* Security-related settings

---

# 🚨 6. Protect Secrets

Never store:

```text
❌ Passwords
❌ API tokens
❌ Service-account tokens
❌ Database credentials
❌ Private keys
```

inside:

```text
GitHub repositories
Public documentation
Shell history
Screenshots
README files
```

---

# 🌐 7. Secure Grafana in Production

A production deployment should consider:

```text
HTTPS/TLS
   ↓
Authentication
   ↓
RBAC
   ↓
Least Privilege
   ↓
Audit Logging
   ↓
Monitoring
   ↓
Regular Backups
```

---

# 🧪 RBAC Validation Matrix

| User             | Development | Business |   System | Admin |
| ---------------- | ----------: | -------: | -------: | ----: |
| 👨‍💻 Developer  |     ✏️ Edit | 👁️ View |  🚫 None | 🚫 No |
| 👩‍💼 Manager    |    👁️ View |  ✏️ Edit | 👁️ View | 🚫 No |
| 👨‍🔧 Operations |     🚫 None | 👁️ View |  ✏️ Edit | 🚫 No |

### 🎯 Expected Outcome

```text
Developer  → Development-focused access
Manager    → Business-focused access
Operations → System-focused access
Admin      → Full administrative access
```

---

# 🧠 Key Learning Points

### 🔐 RBAC

Role-Based Access Control allows permissions to be assigned according to responsibilities rather than giving every user unrestricted access.

### 👥 Teams

Teams simplify management by grouping users with similar access requirements.

### 📁 Folders

Folder-level permissions provide an effective boundary for organizing dashboard access.

### 🎛️ Custom Roles

Custom roles can provide more granular authorization than broad organization roles when supported by the Grafana deployment.

### 📜 Auditing

Logging and auditing help identify unauthorized access attempts and suspicious activity.

### 🛡️ Least Privilege

Users should receive the minimum permissions necessary to perform their work.

---

# 🏁 Conclusion

🎉 **Congratulations!**

You have completed the **Implementing RBAC in Grafana** lab.

During this lab, you learned how to:

* 🚀 Install Grafana on Linux
* 👤 Create and manage users
* 👥 Organize users into teams
* 🔐 Understand Grafana RBAC
* 📁 Configure folder permissions
* 📊 Create test dashboards
* 🧪 Validate role-based access
* 🎛️ Configure granular permissions
* 🔑 Control API access
* 📜 Monitor Grafana logs
* 🔍 Audit access permissions
* 💾 Back up Grafana configuration
* 🛡️ Apply least-privilege security

---

# 🌍 Real-World Applications

### 🏢 Enterprise Monitoring

Separate monitoring access between:

```text
Development
Operations
Management
Security
```

### ☁️ Multi-Tenant Monitoring

Use access boundaries to prevent different customers or departments from viewing unauthorized dashboards.

### 🛡️ Security Operations

Restrict access to sensitive security dashboards and monitoring data.

### 👨‍💻 DevOps Teams

Allow developers to manage application dashboards while preventing unnecessary administrative access.

### 📋 Compliance

RBAC supports security governance by providing controlled and auditable access to monitoring resources.

---

# ⭐ Final Takeaways

```text
┌─────────────────────────────────────────────┐
│          🔐 GRAFANA RBAC PRINCIPLES         │
├─────────────────────────────────────────────┤
│                                             │
│  👤 Users       → Identity                  │
│  👥 Teams       → Grouping                  │
│  🎭 Roles       → Authorization             │
│  📁 Folders     → Resource Boundary         │
│  📊 Dashboards  → Monitoring Resources      │
│  🔑 API Tokens  → Programmatic Access       │
│  📜 Logs        → Auditing                  │
│  🛡️ Least Privilege → Security             │
│                                             │
└─────────────────────────────────────────────┘
```

> 💡 **The goal of Grafana RBAC is not simply to restrict users — it is to provide the right access to the right people at the right level.**

---

## 🏆 Lab Status

```text
╔══════════════════════════════════════════════╗
║        🎉 GRAFANA RBAC LAB COMPLETE 🎉      ║
╠══════════════════════════════════════════════╣
║                                              ║
║  ✅ Grafana Installation                     ║
║  ✅ User Management                          ║
║  ✅ Team Configuration                       ║
║  ✅ RBAC Permissions                         ║
║  ✅ Folder Security                          ║
║  ✅ Dashboard Testing                        ║
║  ✅ Custom Access Control                    ║
║  ✅ API Security                             ║
║  ✅ Logging & Auditing                       ║
║  ✅ Security Best Practices                  ║
║                                              ║
╚══════════════════════════════════════════════╝
```

### 🚀 Technologies Used

![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square\&logo=grafana\&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-E95420?style=flat-square\&logo=linux\&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=flat-square\&logo=ubuntu\&logoColor=white)
![InfluxDB](https://img.shields.io/badge/InfluxDB-22ADF6?style=flat-square\&logo=influxdb\&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat-square\&logo=gnubash\&logoColor=white)
![REST API](https://img.shields.io/badge/REST_API-009688?style=flat-square)
![RBAC](https://img.shields.io/badge/RBAC-Security-critical?style=flat-square)
![Monitoring](https://img.shields.io/badge/Monitoring-Observability-blue?style=flat-square)

---

⭐ **If this lab helped you understand Grafana security and RBAC, consider adding it to your DevOps/Cloud learning portfolio.**

**#Grafana #RBAC #DevOps #CloudDevOps #Linux #Monitoring #Observability #CyberSecurity #GrafanaSecurity #InfluxDB #Automation #CloudComputing**
