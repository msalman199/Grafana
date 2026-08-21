# 🔐 Securing Grafana with SSO (Single Sign-On)

![Grafana](https://img.shields.io/badge/Grafana-SSO-F46800?style=for-the-badge\&logo=grafana\&logoColor=white)
![OAuth 2.0](https://img.shields.io/badge/OAuth-2.0-2F80ED?style=for-the-badge)
![GitHub](https://img.shields.io/badge/GitHub-OAuth-181717?style=for-the-badge\&logo=github)
![Linux](https://img.shields.io/badge/Linux-Ubuntu-E95420?style=for-the-badge\&logo=ubuntu)
![Security](https://img.shields.io/badge/Security-SSO-success?style=for-the-badge)

> 🔒 **A practical hands-on lab for integrating Grafana with GitHub OAuth 2.0 to provide centralized, secure Single Sign-On authentication.**

---

## 📚 Table of Contents

* [🎯 Lab Objectives](#-lab-objectives)
* [📋 Prerequisites](#-prerequisites)
* [🖥️ Lab Environment](#️-lab-environment)
* [🏗️ Architecture](#️-architecture)
* [Task 1 - Environment Setup](#task-1---environment-setup-and-grafana-installation)
* [Task 2 - Configure GitHub OAuth](#task-2---configure-oauth-20-with-github)
* [Task 3 - Test SSO](#task-3---test-sso-authentication)
* [Task 4 - Advanced SSO](#task-4---advanced-sso-configuration)
* [Task 5 - Security Verification](#task-5---security-verification-and-monitoring)
* [🛠️ Troubleshooting](#️-troubleshooting-common-issues)
* [✅ Lab Verification](#-lab-verification)
* [🏁 Conclusion](#-conclusion)
* [🚀 Next Steps](#-next-steps)

---

## 🎯 Lab Objectives

By completing this lab, you will learn how to:

* 🔐 Understand Single Sign-On authentication
* 🖥️ Install and configure Grafana on Linux
* 🔑 Configure OAuth 2.0 authentication
* 🐙 Use GitHub as an external identity provider
* 🔗 Integrate Grafana with an OAuth provider
* 👤 Configure user registration and roles
* 🧪 Test the complete SSO authentication workflow
* 📊 Monitor authentication events
* 🛡️ Apply security best practices
* 🔧 Troubleshoot common OAuth configuration problems

---

## 📋 Prerequisites

Before beginning, ensure you have:

* Basic Linux command-line knowledge
* Familiarity with web browsers
* Basic networking knowledge
* Understanding of authentication and authorization
* A GitHub account
* Basic knowledge of Linux configuration files
* Familiarity with `systemctl`, `curl`, and `journalctl`

---

## 🖥️ Lab Environment

This lab is designed for the **Al Nafi Linux-based cloud environment**.

The environment consists of:

| Component               | Description    |
| ----------------------- | -------------- |
| 💻 Operating System     | Linux / Ubuntu |
| 📊 Monitoring           | Grafana        |
| 🔐 Authentication       | OAuth 2.0      |
| 🐙 Identity Provider    | GitHub         |
| 🌐 Grafana Port         | `3000`         |
| 🔑 Authentication Model | Single Sign-On |
| 🖥️ Machines Required   | 1              |

> 💡 All exercises can be completed on a single Linux machine.

---

# 🏗️ Architecture

The completed environment follows this authentication flow:

```text
                  ┌─────────────────────┐
                  │       User          │
                  │      Browser        │
                  └──────────┬──────────┘
                             │
                             │ Login
                             ▼
                  ┌─────────────────────┐
                  │      Grafana        │
                  │    Port 3000        │
                  └──────────┬──────────┘
                             │
                             │ OAuth Request
                             ▼
                  ┌─────────────────────┐
                  │       GitHub        │
                  │   OAuth Provider    │
                  └──────────┬──────────┘
                             │
                             │ Authorization
                             ▼
                  ┌─────────────────────┐
                  │      Grafana        │
                  │   Create Session    │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │  Authenticated User │
                  │   Grafana Dashboard │
                  └─────────────────────┘
```

---

# 🚀 Task 1 - Environment Setup and Grafana Installation

## 🔧 Subtask 1.1 - Update the System

Update the Linux package repositories and install required dependencies.

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y \
    wget \
    curl \
    gnupg2 \
    software-properties-common \
    apt-transport-https
```

### 🔎 Verify the installation

```bash
which wget
which curl
which gpg
```

---

## 📊 Subtask 1.2 - Install Grafana

Add the Grafana repository and install Grafana.

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" | \
sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install -y grafana
```

Enable and start Grafana:

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Check the service:

```bash
sudo systemctl status grafana-server
```

Expected result:

```text
Active: active (running)
```

---

## 🔍 Subtask 1.3 - Verify Grafana

Check whether Grafana is listening on port `3000`:

```bash
sudo netstat -tlnp | grep :3000
```

If `netstat` is unavailable:

```bash
sudo ss -tlnp | grep :3000
```

Check Grafana logs:

```bash
sudo journalctl -u grafana-server -f --no-pager -n 20
```

Open:

```text
http://localhost:3000
```

Default credentials:

```text
Username: admin
Password: admin
```

> ⚠️ Change the default administrator password immediately after the first login.

---

# 🔐 Task 2 - Configure OAuth 2.0 with GitHub

## 🐙 Subtask 2.1 - Create a GitHub OAuth Application

Log in to GitHub and navigate to:

```text
Settings
  └── Developer settings
        └── OAuth Apps
```

Create a new OAuth application.

Use:

| Setting                    | Value                                |
| -------------------------- | ------------------------------------ |
| Application name           | `Grafana SSO Lab`                    |
| Homepage URL               | `http://localhost:3000`              |
| Authorization callback URL | `http://localhost:3000/login/github` |

After registering the application:

* Copy the **Client ID**
* Generate a **Client Secret**
* Store both securely

> 🔒 **Never commit OAuth client secrets to GitHub or other public repositories.**

---

# ⚙️ Subtask 2.2 - Configure Grafana GitHub OAuth

Back up the Grafana configuration:

```bash
sudo cp /etc/grafana/grafana.ini \
/etc/grafana/grafana.ini.backup
```

Edit the configuration:

```bash
sudo nano /etc/grafana/grafana.ini
```

Configure the server:

```ini
[server]
http_port = 3000
domain = localhost
root_url = http://localhost:3000
```

Configure authentication:

```ini
[auth]
disable_login_form = false
disable_signout_menu = false
```

Configure GitHub OAuth:

```ini
[auth.github]
enabled = true
allow_sign_up = true

client_id = YOUR_GITHUB_CLIENT_ID
client_secret = YOUR_GITHUB_CLIENT_SECRET

scopes = user:email,read:org

auth_url = https://github.com/login/oauth/authorize
token_url = https://github.com/login/oauth/access_token
api_url = https://api.github.com/user

allowed_organizations =
team_ids =

allow_assign_grafana_admin = false
role_attribute_path =
```

Replace:

```text
YOUR_GITHUB_CLIENT_ID
YOUR_GITHUB_CLIENT_SECRET
```

with your actual credentials.

---

# 👥 Subtask 2.3 - Configure Users and Security

Add user configuration:

```ini
[users]
allow_sign_up = true
allow_org_create = true
auto_assign_org = true
auto_assign_org_id = 1
auto_assign_org_role = Viewer
verify_email_enabled = false
default_theme = dark
```

Configure security:

```ini
[security]
admin_user = admin
admin_password =
secret_key =
disable_gravatar = false
```

> 🔒 In production, use secure secret management instead of storing sensitive credentials directly in configuration files.

---

# 🔄 Subtask 2.4 - Restart Grafana

Apply the configuration:

```bash
sudo systemctl restart grafana-server
```

Verify:

```bash
sudo systemctl status grafana-server
```

Monitor logs:

```bash
sudo journalctl -u grafana-server -f --no-pager -n 30
```

---

# 🧪 Task 3 - Test SSO Authentication

## 🔑 Subtask 3.1 - Test GitHub Login

Open:

```text
http://localhost:3000
```

You should see:

```text
Sign in with GitHub
```

Click the button.

The authentication flow should be:

```text
Grafana
   ↓
GitHub
   ↓
User Authentication
   ↓
Authorization
   ↓
Grafana Callback
   ↓
Authenticated Session
```

---

## 👤 Subtask 3.2 - Verify User Information

After successful login:

1. Open the user menu.
2. Select **Profile**.
3. Verify the username.
4. Verify the email address.
5. Review organization information.

Confirm that the GitHub identity is correctly associated with the Grafana account.

---

# 📊 Subtask 3.3 - Test Permissions

Create a test dashboard:

```text
Dashboards
   ↓
New Dashboard
   ↓
Add Panel
   ↓
Save Dashboard
```

Name it:

```text
SSO Test Dashboard
```

Verify that the authenticated user has the expected permissions.

---

# 🚪 Subtask 3.4 - Test Logout and Re-Login

Inspect authentication events:

```bash
sudo grep -i "login\|logout\|oauth" \
/var/log/grafana/grafana.log | tail -10
```

Then:

1. Log out of Grafana.
2. Access a protected page.
3. Verify authentication is required.
4. Select **Sign in with GitHub**.
5. Authenticate again.
6. Confirm that a new Grafana session is created.

---

# 🛡️ Task 4 - Advanced SSO Configuration

## 👥 Subtask 4.1 - Configure Team Mapping

Edit:

```bash
sudo nano /etc/grafana/grafana.ini
```

Example configuration:

```ini
[auth.github]
enabled = true
allow_sign_up = true

client_id = YOUR_GITHUB_CLIENT_ID
client_secret = YOUR_GITHUB_CLIENT_SECRET

scopes = user:email,read:org

auth_url = https://github.com/login/oauth/authorize
token_url = https://github.com/login/oauth/access_token
api_url = https://api.github.com/user

allowed_organizations = YOUR_GITHUB_ORGANIZATION
team_ids =

allow_assign_grafana_admin = true

role_attribute_path = contains(groups[*], '@YOUR_ORG/admin') && 'Admin' || contains(groups[*], '@YOUR_ORG/editor') && 'Editor' || 'Viewer'
```

This concept allows external identity information to influence Grafana role assignment.

### Example mapping

```text
GitHub Admin Team
       │
       ▼
Grafana Admin

GitHub Editor Team
       │
       ▼
Grafana Editor

Other Users
       │
       ▼
Grafana Viewer
```

> ⚠️ Grant administrator privileges only when the external identity mapping has been carefully validated.

---

# 🍪 Subtask 4.2 - Configure Session Management

Configure session behavior:

```ini
[session]
provider = file
provider_config = sessions
cookie_name = grafana_sess
cookie_secure = false
session_life_time = 86400
gc_interval_time = 86400
```

For production deployments using HTTPS, secure cookies should be enabled.

---

# 📝 Subtask 4.3 - Configure Logging

Configure Grafana logging:

```ini
[log]
mode = console file
level = info
filters =
```

Console logging:

```ini
[log.console]
level = info
format = console
```

File logging:

```ini
[log.file]
level = info
format = text
log_rotate = true
max_lines = 1000000
max_size_shift = 28
daily_rotate = true
max_days = 7
```

Restart Grafana:

```bash
sudo systemctl restart grafana-server
```

Monitor logs:

```bash
sudo journalctl -u grafana-server -f --no-pager -n 20
```

---

# 🔎 Task 5 - Security Verification and Monitoring

## 🛡️ Subtask 5.1 - Verify Security Headers

Check HTTP response headers:

```bash
curl -I http://localhost:3000
```

Check common security headers:

```bash
curl -s -D- http://localhost:3000 | \
grep -i "x-frame-options\|x-content-type-options\|x-xss-protection"
```

Review the returned headers and identify which protections are enabled.

---

# 📡 Subtask 5.2 - Monitor Authentication Logs

Create an SSO monitoring script:

```bash
cat > ~/monitor_sso.sh << 'EOF'
#!/bin/bash

echo "Monitoring Grafana SSO authentication events..."
echo "Press Ctrl+C to stop monitoring"
echo "----------------------------------------"

tail -f /var/log/grafana/grafana.log | \
grep -i --line-buffered "oauth\|github\|login\|logout\|auth"
EOF
```

Make it executable:

```bash
chmod +x ~/monitor_sso.sh
```

Run:

```bash
~/monitor_sso.sh
```

---

# 🧪 Subtask 5.3 - Test Security Scenarios

## Test 1 - Invalid OAuth Credentials

Temporarily use an invalid client secret.

Restart Grafana:

```bash
sudo systemctl restart grafana-server
```

Attempt GitHub authentication.

Expected result:

```text
Authentication fails
```

Restore the correct credentials after testing.

---

## Test 2 - Session Timeout

Configure a short session lifetime:

```ini
session_life_time = 60
```

Restart Grafana:

```bash
sudo systemctl restart grafana-server
```

Log in and wait for the session to expire.

Expected behavior:

```text
Session expires
      ↓
Protected resource accessed
      ↓
Authentication required
```

---

## Test 3 - Unauthorized Access

Log out of Grafana and test:

```bash
curl -i http://localhost:3000/api/dashboards/home
```

Verify that unauthenticated access does not expose protected dashboard information.

---

# 🛠️ Troubleshooting Common Issues

## ❌ Issue 1 - OAuth Callback URL Mismatch

### Symptoms

GitHub reports a callback or redirect URL mismatch.

### Solution

Verify the GitHub OAuth application callback:

```text
http://localhost:3000/login/github
```

Check Grafana:

```bash
grep -n "root_url" /etc/grafana/grafana.ini
```

Ensure that the Grafana `root_url` and OAuth callback configuration are consistent.

---

# ❌ Issue 2 - Grafana Service Fails to Start

Check service status:

```bash
sudo systemctl status grafana-server
```

Review logs:

```bash
sudo journalctl -u grafana-server --no-pager -n 50
```

Restore the backup if necessary:

```bash
sudo cp /etc/grafana/grafana.ini.backup \
/etc/grafana/grafana.ini
```

Restart:

```bash
sudo systemctl restart grafana-server
```

> 💡 Configuration validation options can vary by Grafana version. If a command such as `-test-config` is unavailable in your installed release, rely on the service logs and Grafana's documented configuration validation methods.

---

# ❌ Issue 3 - GitHub Login Button Is Missing

Check the configuration:

```bash
grep -A 15 "\[auth.github\]" \
/etc/grafana/grafana.ini
```

Confirm:

```ini
enabled = true
```

Restart:

```bash
sudo systemctl restart grafana-server
```

Check logs:

```bash
sudo journalctl -u grafana-server --no-pager -n 50
```

---

# ❌ Issue 4 - Permission Problems

Search for role information:

```bash
sudo grep -i "role\|permission" \
/var/log/grafana/grafana.log
```

Check automatic role assignment:

```bash
grep -A 5 "auto_assign_org_role" \
/etc/grafana/grafana.ini
```

Verify that the user's Grafana role matches the intended access level.

---

# ❌ Issue 5 - OAuth Secret Problems

Check that the configured values are present without exposing them publicly:

```bash
sudo grep -n "client_id" /etc/grafana/grafana.ini
```

Avoid commands that print the complete client secret to shared terminals, screenshots, or Git repositories.

---

# ✅ Lab Verification

Perform the following checks before completing the lab.

### 1️⃣ Verify Grafana

```bash
sudo systemctl is-active grafana-server
```

Expected:

```text
active
```

### 2️⃣ Verify Port 3000

```bash
sudo ss -tlnp | grep :3000
```

### 3️⃣ Verify GitHub OAuth Configuration

```bash
sudo grep -A 15 "\[auth.github\]" \
/etc/grafana/grafana.ini
```

Confirm:

```ini
enabled = true
```

### 4️⃣ Verify Login Page

```bash
curl -s http://localhost:3000/login | grep -i github
```

### 5️⃣ Review Authentication Logs

```bash
sudo tail -20 /var/log/grafana/grafana.log | \
grep -i oauth
```

### 6️⃣ Verify OAuth Authorization Endpoint

```bash
curl -s "https://github.com/login/oauth/authorize?client_id=YOUR_CLIENT_ID&response_type=code"
```

> ⚠️ Do not expose your real client secret in commands, screenshots, documentation, or Git repositories.

---

# 📋 Final Verification Checklist

* [ ] Grafana is installed
* [ ] Grafana service is running
* [ ] Grafana is accessible on port `3000`
* [ ] GitHub OAuth application is created
* [ ] GitHub Client ID is configured
* [ ] GitHub Client Secret is configured securely
* [ ] OAuth callback URL is correct
* [ ] GitHub login button appears
* [ ] GitHub authentication succeeds
* [ ] User information is correctly displayed
* [ ] Grafana roles are correctly assigned
* [ ] Logout works correctly
* [ ] Re-login works correctly
* [ ] Authentication logs are available
* [ ] Unauthorized access is rejected
* [ ] Security configuration has been reviewed

---

# 🔐 Security Best Practices

For a production deployment, consider the following improvements:

### HTTPS

Do not expose OAuth authentication over plain HTTP in production.

Use:

```text
HTTPS
   ↓
Reverse Proxy
   ↓
Grafana
```

### Secret Management

Avoid hardcoding OAuth secrets in Git repositories.

Use:

* Environment variables
* Secret managers
* Vault
* Cloud secret-management services
* Protected configuration management

### Least Privilege

Use the lowest Grafana role required:

```text
Viewer
   ↓
Editor
   ↓
Admin
```

Only grant administrative privileges when necessary.

### Restricted Organizations

Where appropriate, restrict authentication to approved GitHub organizations.

### Session Security

Use secure cookies and HTTPS for production deployments.

### Logging

Monitor:

* Successful logins
* Failed authentication attempts
* OAuth failures
* Logout events
* Role changes
* Administrative actions

---

# 🧠 Key Concepts Learned

| Concept              | Description                                                     |
| -------------------- | --------------------------------------------------------------- |
| 🔐 SSO               | One authentication mechanism for accessing applications         |
| 🔑 OAuth 2.0         | Authorization framework used for delegated authentication flows |
| 🐙 Identity Provider | External service responsible for user identity                  |
| 🔄 Callback URL      | Endpoint receiving the OAuth authorization response             |
| 👤 Role Mapping      | Assigning application permissions based on identity information |
| 🍪 Session           | Server-side state representing an authenticated user            |
| 📝 Audit Logging     | Recording security and authentication events                    |
| 🛡️ Least Privilege  | Granting only the permissions required                          |

---

# 🏁 Conclusion

Congratulations! 🎉

You have completed a hands-on Grafana SSO security lab and learned how to integrate Grafana with GitHub using OAuth 2.0.

During this lab, you:

* ✅ Installed Grafana on Linux
* ✅ Configured Grafana authentication
* ✅ Created a GitHub OAuth application
* ✅ Integrated GitHub with Grafana
* ✅ Implemented Single Sign-On
* ✅ Tested OAuth login and logout
* ✅ Verified user identity information
* ✅ Configured role assignment concepts
* ✅ Configured session management
* ✅ Enabled authentication logging
* ✅ Tested authentication failure scenarios
* ✅ Troubleshot common OAuth problems

---

# 🌐 Why SSO Matters

Single Sign-On is particularly valuable in enterprise monitoring environments because it provides:

### 🔒 Better Security

Centralized authentication reduces the number of independent credentials users must manage.

### 👨‍💻 Better User Experience

Users can authenticate using an existing organizational identity.

### ⚙️ Simplified Administration

User access can be managed through a centralized identity provider.

### 📊 Improved Auditing

Authentication activity can be correlated with centralized identity information.

### 📈 Scalability

Organizations can integrate Grafana into an existing identity and access-management architecture.

---

# 🚀 Next Steps

After completing this lab, explore:

* 🌐 Google OAuth
* ☁️ Microsoft Entra ID / Azure AD
* 🔐 Okta
* 🆔 Generic OAuth
* 🔑 OpenID Connect
* 👥 Automated team synchronization
* 🛡️ Grafana RBAC
* 🔒 HTTPS with NGINX
* 📜 Centralized audit logging
* 🔑 Secrets management with Vault
* ☁️ Enterprise identity federation

---

# 🧰 Technology Stack

![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat-square\&logo=grafana\&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub_OAuth-181717?style=flat-square\&logo=github\&logoColor=white)
![OAuth](https://img.shields.io/badge/OAuth_2.0-2F80ED?style=flat-square)
![Linux](https://img.shields.io/badge/Linux-E95420?style=flat-square\&logo=linux\&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=flat-square\&logo=gnu-bash\&logoColor=white)
![Security](https://img.shields.io/badge/Security-SSO-success?style=flat-square)
![Authentication](https://img.shields.io/badge/Authentication-OAuth-blue?style=flat-square)

---

## ⭐ Lab Outcome

```text
Linux
  │
  ├── Grafana
  │      │
  │      ├── OAuth 2.0
  │      │       │
  │      │       └── GitHub
  │      │
  │      ├── User Authentication
  │      ├── Role Management
  │      ├── Session Management
  │      └── Authentication Logging
  │
  └── Security Verification

             ↓

       🔐 Secure Grafana SSO
```

> 🚀 **Outcome:** You now have practical experience integrating Grafana with an external OAuth identity provider and securing monitoring infrastructure with centralized authentication.
