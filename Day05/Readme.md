# 📦 Registering Systems for Red Hat Support

## 📖 Overview

Red Hat provides access to software repositories through the **Red Hat Content Delivery Network (CDN)** using a subscription-based model. These repositories deliver:

* Security patches
* Bug fixes
* Software enhancements
* New package releases

An active Red Hat subscription also provides access to:

* Red Hat Customer Portal
* Knowledgebase Solutions
* Product Documentation
* Red Hat Insights
* Technical Support

> **Note:** Since 2024, Red Hat uses the **Simple Content Access (SCA)** model, which manages subscriptions at the organization level rather than attaching subscriptions to individual systems. 

---

# 🎯 Benefits of Red Hat Subscription

✅ Access to software repositories

✅ Security updates and patches

✅ Red Hat Insights analytics

✅ Customer Portal resources

✅ Technical support

✅ Centralized subscription management

---

# 🔐 Registering a RHEL System

You can register a Red Hat Enterprise Linux system using:

1. **rhc command** (Recommended for RHEL 8.8+)
2. **subscription-manager command**
3. **RHEL Web Console**
4. **Anaconda Installer during installation**

Registration provides:

* Unique identity certificates
* Repository access
* Subscription tracking
* Optional Red Hat Insights integration

---

# 🚀 Registering with the `rhc` Tool

The **rhc** tool is the preferred registration method for RHEL 8.8 and later.

By default, it enables:

| Feature           | Purpose                  |
| ----------------- | ------------------------ |
| Content           | Repository access        |
| Analytics         | Red Hat Insights         |
| Remote Management | Remote system management |

### Register System

```bash
sudo rhc connect
```

Example:

```bash
sudo rhc connect

Username: red-hat-username
Password: red-hat-password
```

Successful output:

```text
✓ Connected to Red Hat Subscription Management
✓ Content
✓ Analytics
✓ Remote Management

Successfully connected to Red Hat!
```

---

## Disable Analytics and Remote Management

If you only need repository access:

```bash
sudo rhc connect \
--disable-feature=analytics,remote-management
```

This enables only:

```text
✓ Content
✗ Analytics
✗ Remote Management
```

---

# 🔑 Using Activation Keys with `rhc`

Activation Keys provide a more secure and automated registration process.

### Register with Activation Key

```bash
sudo rhc connect \
--activation-key=<activation_key_name> \
--organization=<organization_ID>
```

Benefits:

* No username/password required
* Easier automation
* Consistent system configuration

---

# ❌ Unregister Using `rhc`

```bash
sudo rhc disconnect
```

Output:

```text
✓ Disconnected from Subscription Management
✓ Disconnected from Red Hat Insights
✓ Deactivated Remote Management
```

---

# 🛠 Registering with `subscription-manager`

> Use this method for RHEL 8.7 and earlier systems.

### Register System

```bash
sudo subscription-manager register
```

Example:

```bash
sudo subscription-manager register

Username: red-hat-username
Password: red-hat-password
```

---

## Register Using Activation Key

```bash
sudo subscription-manager register \
--activationkey <activation_key_name> \
--org <organization_ID>
```

---

## Verify Registration Status

```bash
sudo subscription-manager status
```

Example:

```text
Overall Status: Registered
```

---

# 📊 Register with Red Hat Insights

The `subscription-manager` command does not automatically enable Insights.

Register separately:

```bash
sudo insights-client --register
```

Example output:

```text
Successfully registered host
Uploading Insights data
Successfully uploaded report
```

---

# 🏷 Setting System Purpose Attributes

System Purpose helps categorize systems in inventory.

## Available Roles

* Red Hat Enterprise Linux Server
* Red Hat Enterprise Linux Workstation
* Red Hat Enterprise Linux Compute Node

### Set Role

```bash
sudo subscription-manager syspurpose role \
--set "Red Hat Enterprise Linux Workstation"
```

---

## Available Service Levels

* Premium
* Standard
* Self-Support

### Set Service Level

```bash
sudo subscription-manager syspurpose service-level \
--set "Premium"
```

---

## Available Usage Types

* Production
* Development/Test
* Disaster Recovery

### Set Usage

```bash
sudo subscription-manager syspurpose usage \
--set "Development/Test"
```

### List Available Values

```bash
sudo subscription-manager syspurpose usage --list
```

---

# ❌ Unregister Using `subscription-manager`

```bash
sudo subscription-manager unregister
```

Example:

```text
System has been unregistered.
```

---

# 🌐 Registering Through the RHEL Web Console

The RHEL Web Console provides a graphical method for registration.

### Steps

1. Log in as a privileged user.
2. Open **Subscriptions**.
3. Click **Register**.
4. Enter:

   * Red Hat Username & Password
   * OR Activation Key & Organization ID
5. Click **Register**.

By default, Red Hat Insights registration is enabled.

---

# 💽 Registering During RHEL Installation

The Anaconda installer can register a system before package installation begins.

### Steps

1. Launch RHEL installation.
2. Click **Connect to Red Hat**.
3. Authenticate using:

   * Username & Password
   * OR Activation Key
4. Optionally configure:

   * System Purpose
   * Red Hat Insights

For automated deployments, use:

```bash
rhsm
```

within a Kickstart configuration.

---

# 🔍 Inspecting Identity Certificates

Registered systems store certificates under:

```text
/etc/pki/
```

## Product Certificates

```text
/etc/pki/product-default/
```

Used to identify installed Red Hat products.

---

## Consumer Certificates

```text
/etc/pki/consumer/
```

Used to identify the registered system.

---

## Entitlement Certificates

```text
/etc/pki/entitlement/
```

Used for repository authentication.

---

## View Certificate Details

Use the `rct` command:

```bash
sudo rct cat-cert /etc/pki/product-default/479.pem
```

Example output:

```text
Product Certificate

Name: Red Hat Enterprise Linux
Version: 10.0
Architecture: x86_64
```

---

# 📌 Key Takeaways

* **rhc** is the preferred registration method for RHEL 8.8+.
* **subscription-manager** remains available and supported.
* Activation Keys are recommended for automation.
* Registration enables repository access and subscription management.
* Red Hat Insights provides analytics and proactive issue detection.
* System Purpose helps organize systems in Hybrid Cloud Console.
* Certificates stored in `/etc/pki` identify and authenticate registered systems.

---

## 📚 Useful Commands Summary

```bash
# Register using rhc
rhc connect

# Register using activation key
rhc connect --activation-key=<key> --organization=<org_id>

# Disconnect rhc
rhc disconnect

# Register using subscription-manager
subscription-manager register

# Check registration status
subscription-manager status

# Register Insights
insights-client --register

# Set system purpose
subscription-manager syspurpose role --set "RHEL Server"

# Unregister system
subscription-manager unregister

# View certificate
rct cat-cert /etc/pki/product-default/479.pem
```

Source: Red Hat subscription registration documentation provided in the uploaded file. 
