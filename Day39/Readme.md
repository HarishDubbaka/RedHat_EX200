# Listing Unit Dependencies & Masking Services in Linux (systemd)

This topic explains **how systemd manages service dependencies** and **the difference between disabling and masking services**.

---

# 1. Listing Unit Dependencies

Many Linux services **depend on other services** to function properly.

### Example

* The **SSH service (`sshd.service`)** needs:

  * System initialization
  * SSH host keys
  * System slices
* If these services are not available, `sshd` cannot start.

When you start a service:

```bash
systemctl start sshd
```

systemd automatically starts any required dependencies.

---

## View Dependencies

Use:

```bash
systemctl list-dependencies <service>
```

Example:

```bash
systemctl list-dependencies sshd.service
```

Output:

```text
sshd.service
├── ssh-host-keys-migration.service
├── system.slice
├── sshd-keygen.target
│   ├── sshd-keygen@ecdsa.service
│   ├── sshd-keygen@ed25519.service
│   └── sshd-keygen@rsa.service
└── sysinit.target
```

### Meaning

```
sshd.service
│
├── system.slice
│
├── sshd-keygen.target
│     ├── RSA Key
│     ├── ECDSA Key
│     └── ED25519 Key
│
└── sysinit.target
```

Before SSH starts:

* System initialization must finish.
* SSH host keys must exist.
* Required targets must be active.

Only then does **sshd.service** start.

---

# 2. Reverse Dependencies

Sometimes you want to know:

> **Which services depend on this service?**

Use:

```bash
systemctl list-dependencies --reverse <service>
```

Example:

```bash
systemctl list-dependencies --reverse NetworkManager.service
```

Output

```text
NetworkManager.service
├── NetworkManager-wait-online.service
└── multi-user.target
      └── graphical.target
```

### Meaning

```
NetworkManager
       │
       ▼
NetworkManager-wait-online
       │
       ▼
multi-user.target
       │
       ▼
graphical.target
```

Without **NetworkManager**, these services may not function correctly.

---

# 3. Why Dependencies Matter

Imagine:

```
SAP Application
       │
       ▼
SAP Host Agent
       │
       ▼
NetworkManager
```

If **NetworkManager** fails:

* SAP Host Agent cannot communicate.
* SAP Application may fail.
* Monitoring stops.

systemd ensures dependencies start automatically.

---

# 4. Masking Services

Sometimes **two services cannot run together.**

Example:

* Apache (`httpd`)
* Nginx (`nginx`)

Both use:

```
TCP Port 80
```

Only one service can bind to the port.

To prevent accidental startup:

**Mask one service.**

---

## Mask a Service

Command:

```bash
systemctl mask httpd.service
```

Output:

```text
Created symlink '/etc/systemd/system/httpd.service' → '/dev/null'
```

---

## What Happens?

Normally:

```
httpd.service
      │
      ▼
/usr/lib/systemd/system/httpd.service
```

After masking:

```
httpd.service
      │
      ▼
/dev/null
```

The service file is redirected to **`/dev/null`**, making it impossible for systemd to load it.

---

# 5. Check Masked Status

```bash
systemctl status httpd.service
```

Output:

```text
Loaded: masked
Active: inactive (dead)
```

Or:

```bash
systemctl list-unit-files httpd.service
```

Output

```text
UNIT FILE       STATE
httpd.service   masked
```

---

# 6. Trying to Start a Masked Service

```bash
systemctl start httpd.service
```

Output

```text
Failed to start httpd.service:
Unit httpd.service is masked.
```

It cannot be started.

---

# 7. Unmask the Service

Remove the mask:

```bash
systemctl unmask httpd.service
```

Output:

```text
Removed '/etc/systemd/system/httpd.service'
```

Now start it normally:

```bash
systemctl start httpd.service
```

---

# 8. Disabled vs Masked (Most Important Exam Topic)

| Feature                      | Disabled           | Masked                       |
| ---------------------------- | ------------------ | ---------------------------- |
| Starts automatically at boot | ❌ No               | ❌ No                         |
| Can be started manually      | ✅ Yes              | ❌ No                         |
| Can dependency start it      | ✅ Yes              | ❌ No                         |
| Service file exists          | ✅ Yes              | Redirected to `/dev/null`    |
| Purpose                      | Prevent auto-start | Completely block the service |

---

# Example

### Disabled

```bash
systemctl disable httpd
```

Result:

* Won't start during boot.
* Administrator can still run:

```bash
systemctl start httpd
```

Works successfully.

---

### Masked

```bash
systemctl mask httpd
```

Now:

```bash
systemctl start httpd
```

Output:

```text
Failed to start httpd.service:
Unit is masked.
```

Even the **root** user cannot start it until it is unmasked.

---

# Real-Time SAP BASIS Examples

### Scenario 1: Web Server Conflict

Suppose your SAP system uses **SAP Web Dispatcher** on port **80**, and **Apache (`httpd`)** is installed but should never run.

Mask Apache:

```bash
sudo systemctl mask httpd.service
```

This prevents accidental startup and avoids port conflicts.

---

### Scenario 2: Maintenance

During server maintenance, you may want to ensure a service cannot be started by automation or another administrator.

```bash
sudo systemctl mask nginx.service
```

After maintenance:

```bash
sudo systemctl unmask nginx.service
sudo systemctl start nginx.service
```

---

# Common Commands Summary

| Task                      | Command                                                        |
| ------------------------- | -------------------------------------------------------------- |
| List dependencies         | `systemctl list-dependencies sshd.service`                     |
| List reverse dependencies | `systemctl list-dependencies --reverse NetworkManager.service` |
| Mask a service            | `systemctl mask httpd.service`                                 |
| Unmask a service          | `systemctl unmask httpd.service`                               |
| Check status              | `systemctl status httpd.service`                               |
| List unit files           | `systemctl list-unit-files httpd.service`                      |
| Start service             | `systemctl start httpd.service`                                |

---

# RHCSA Exam Tips

* Use `systemctl list-dependencies <service>` to view required services.
* Use `systemctl list-dependencies --reverse <service>` to identify dependent services.
* **Disabled** services do not start at boot but can still be started manually.
* **Masked** services cannot be started manually or automatically because the unit file is linked to `/dev/null`.
* Use `systemctl unmask <service>` before attempting to start a masked service.
* In SAP BASIS environments, masking is useful to prevent conflicting services (such as web servers) from interfering with SAP components.
