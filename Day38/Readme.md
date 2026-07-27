# Controlling System Services with `systemctl`

## Objective

Learn how to **control Linux system services (daemons)** using the **`systemctl`** command.

A **service (daemon)** is a program that runs in the background and provides functionality to the operating system.

Examples:

* `sshd` → SSH remote login service
* `httpd` → Apache Web Server
* `firewalld` → Firewall service
* `chronyd` → Time synchronization service

---

# What is `systemctl`?

`systemctl` is the command used to manage services in **systemd-based Linux systems** (RHEL 7/8/9, CentOS, Rocky Linux, Ubuntu, etc.).

It allows you to:

* Start services
* Stop services
* Restart services
* Reload configuration
* Check service status
* Enable or disable services during boot

---

# 1. Check Service Status

Before doing anything, check whether a service is running.

### Syntax

```bash
systemctl status <service-name>
```

Example

```bash
systemctl status sshd
```

Example Output

```text
● sshd.service - OpenSSH server daemon

Loaded: loaded
Active: active (running)
Main PID: 1052
```

### Understanding the Output

```
Loaded:
```

Shows whether the service file exists.

```
loaded
```

means the service is installed.

---

```
Active:
```

Shows the current state.

Possible states:

* active (running)
* inactive (stopped)
* failed
* activating
* deactivating

---

```
Main PID
```

Shows the Process ID of the running service.

Example:

```
Main PID: 1052
```

means the service is running with Process ID **1052**.

---

# 2. Start a Service

If a service is stopped, start it.

### Syntax

```bash
sudo systemctl start <service-name>
```

Example

```bash
sudo systemctl start sshd
```

What happens?

Before:

```
inactive
```

After:

```
active (running)
```

No output usually means the command succeeded.

---

# 3. Stop a Service

Stops a running service.

### Syntax

```bash
sudo systemctl stop <service-name>
```

Example

```bash
sudo systemctl stop sshd
```

After running:

```
Active: inactive (dead)
```

---

# 4. Restart a Service

A restart means:

```
Stop
↓

Start
```

It completely stops the service and starts it again.

### Syntax

```bash
sudo systemctl restart <service-name>
```

Example

```bash
sudo systemctl restart sshd
```

### What changes?

The service gets a **new Process ID (PID)** because the old process is terminated and a new one is created.

Example

Before restart

```
PID = 1500
```

After restart

```
PID = 1892
```

The PID changes because it's a new process.

---

# 5. Reload a Service

Some services allow configuration changes to be applied **without stopping the service**.

Instead of restarting:

```
Stop
↓

Start
```

they simply **reload the configuration**.

### Syntax

```bash
sudo systemctl reload <service-name>
```

Example

```bash
sudo systemctl reload sshd
```

---

### What changes?

Only the configuration is re-read.

The service keeps running.

The Process ID stays the same.

Example

Before reload

```
PID = 1500
```

After reload

```
PID = 1500
```

The PID does **not** change.

---

# Restart vs Reload

| Feature               | Restart     | Reload     |
| --------------------- | ----------- | ---------- |
| Stops Service         | ✅ Yes       | ❌ No       |
| Starts Again          | ✅ Yes       | ❌ No       |
| Configuration Updated | ✅ Yes       | ✅ Yes      |
| Process ID Changes    | ✅ Yes       | ❌ No       |
| Service Downtime      | Yes (brief) | Usually No |

---

# 6. Reload or Restart

Sometimes you don't know whether a service supports reload.

Linux provides a safe option:

```bash
sudo systemctl reload-or-restart <service-name>
```

Example

```bash
sudo systemctl reload-or-restart sshd
```

How it works:

```
Does service support reload?

       |
   Yes | No
       ↓
Reload  Restart
```

* If the service supports reload → reloads the configuration.
* If not → restarts the service automatically.

This is the safest option after changing configuration files.

---

# Real-Life Example (Editing SSH Configuration)

Suppose you edit:

```bash
sudo vi /etc/ssh/sshd_config
```

After saving the file, apply the changes.

If SSH supports reload:

```bash
sudo systemctl reload sshd
```

If you're unsure:

```bash
sudo systemctl reload-or-restart sshd
```

If reload is not supported:

```bash
sudo systemctl restart sshd
```

---

# Common `systemctl` Commands

| Command                            | Purpose                               |
| ---------------------------------- | ------------------------------------- |
| `systemctl status sshd`            | Check service status                  |
| `systemctl start sshd`             | Start the service                     |
| `systemctl stop sshd`              | Stop the service                      |
| `systemctl restart sshd`           | Stop and start the service            |
| `systemctl reload sshd`            | Reload configuration without stopping |
| `systemctl reload-or-restart sshd` | Reload if possible; otherwise restart |

---

# Visual Workflow

```text
                 systemctl

                      │
      ┌───────────────┼────────────────┐
      │               │                │
      ▼               ▼                ▼

 status           start            stop
(Check)      (Start Service)   (Stop Service)

                      │
                      ▼

                restart
             (Stop + Start)

                      │
          PID Changes (New Process)

                      ▼

                 reload
         (Read Config Again)

                      │
         PID Does NOT Change

                      ▼

        reload-or-restart
     Reload if supported,
     otherwise Restart
```

---

# Interview Questions

### 1. How do you check whether a service is running?

```bash
systemctl status sshd
```

---

### 2. How do you start a service?

```bash
systemctl start sshd
```

---

### 3. How do you stop a service?

```bash
systemctl stop sshd
```

---

### 4. What is the difference between restart and reload?

* **Restart:** Stops and starts the service again. The **PID changes**, and there may be brief downtime.
* **Reload:** Re-reads the configuration without stopping the service. The **PID remains the same**, and downtime is usually avoided.

---

### 5. When should you use `reload-or-restart`?

Use it when you're **not sure whether the service supports reloading**. It reloads the configuration if possible; otherwise, it performs a restart automatically.
