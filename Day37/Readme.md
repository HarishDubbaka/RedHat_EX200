# Identifying System Processes that Start Automatically

## What is a Daemon?

A **daemon** is a background process that runs without user interaction.

Think of it as a **worker that is always waiting** to perform a task.

Examples:

* `sshd` → SSH remote login service
* `httpd` → Apache web server
* `chronyd` → Time synchronization
* `crond` → Cron scheduler

Most daemon names end with **d**.

Example:

```
sshd
httpd
crond
chronyd
```

---

# What is systemd?

`systemd` is the **service manager** of Linux.

It is the **first process** that starts when Linux boots.

Its Process ID (PID) is always:

```
PID 1
```

Everything else in Linux starts through **systemd**.

```
Power ON
    │
    ▼
Kernel Starts
    │
    ▼
systemd (PID 1)
    │
    ├── sshd
    ├── httpd
    ├── chronyd
    ├── crond
    └── many other services
```

---

# What does systemd do?

Systemd manages almost everything related to services.

Its responsibilities include:

* Starts services during boot
* Stops services
* Restarts services
* Checks service status
* Handles service dependencies
* Starts services only when needed
* Speeds up system boot

---

# Why is systemd faster?

Older Linux systems started services **one after another**.

Example:

```
Service A
↓

Service B
↓

Service C
↓

Service D
```

This is slow.

---

Systemd starts independent services simultaneously.

```
          Service A
               │
               │
Service B ─────┼───── Service C
               │
               │
          Service D
```

Multiple services start together, making boot much faster.

---

# Service Dependencies

Some services depend on others.

Example:

```
Apache Web Server
        │
        ▼
Needs Network
```

If the network is unavailable:

```
Apache
   X
```

Systemd waits until the network is ready before starting Apache.

```
Network
   │
   ▼
Apache
```

This avoids startup failures.

---

# On-Demand Service Start

Some services are rarely used.

Instead of starting them at boot:

```
Boot
↓

Do NOT start service
↓

User requests service
↓

Start service immediately
```

This saves RAM and CPU resources.

---

# What is a Unit?

Systemd manages everything through **Units**.

A unit is simply a configuration file that tells systemd **what to manage and how**.

```
systemd
    │
    ├── sshd.service
    ├── httpd.service
    ├── home.mount
    ├── network.target
    └── cron.timer
```

Each unit has a specific extension indicating its type.

---

# Types of Units

## 1. Service Unit (.service)

Represents a service or daemon.

Examples:

```
sshd.service
httpd.service
crond.service
```

Used to:

* Start
* Stop
* Restart
* Enable services

---

## 2. Socket Unit (.socket)

Starts a service **only when a connection arrives**.

Example:

```
Client
   │
   ▼
Socket
   │
   ▼
systemd
   │
   ▼
Starts sshd
```

The service isn't running until it's needed.

---

## 3. Path Unit (.path)

Monitors files or directories.

Example:

```
Watch Folder

New File Added

↓

Start Backup Service
```

Useful for printing systems, backups, and automation.

---

# systemctl Command

`systemctl` is the main command used to manage services.

General syntax:

```
systemctl <command> <service>
```

---

# List Running Services

```
systemctl list-units --type=service
```

Shows only active services.

Example:

```
UNIT                ACTIVE
-----------------------------
sshd.service        running
chronyd.service     running
crond.service       running
```

---

# List All Services

```
systemctl list-units --type=service --all
```

Shows:

* Running services
* Stopped services
* Failed services

---

# List Installed Service Files

```
systemctl list-unit-files --type=service
```

Example:

```
sshd.service      enabled
httpd.service     disabled
cups.service      static
```

This displays how services are configured at boot.

---

# Understanding the Columns

When you run:

```
systemctl list-units --type=service
```

You see:

```
UNIT
LOAD
ACTIVE
SUB
DESCRIPTION
```

### UNIT

Service name.

Example:

```
sshd.service
```

---

### LOAD

Indicates whether systemd successfully loaded the service file.

Example:

```
loaded
```

---

### ACTIVE

High-level status.

Examples:

```
active
inactive
failed
```

---

### SUB

Detailed status.

Examples:

```
running
dead
exited
waiting
```

---

### DESCRIPTION

Human-readable description.

Example:

```
OpenSSH Server
```

---

# Viewing Detailed Service Information

```
systemctl status sshd
```

Example output:

```
Loaded:
Active:
Main PID:
Memory:
CPU:
Logs:
```

---

## Loaded

Shows whether the service is loaded and whether it starts at boot.

Example:

```
Loaded: loaded
```

---

## Active

Current service state.

Example:

```
Active: active (running)
```

Meaning:

```
Service is running normally.
```

---

## Main PID

The process ID of the main service.

Example:

```
Main PID: 2212
```

You can inspect it with:

```
ps -fp 2212
```

---

## Memory

Memory consumed by the service.

Example:

```
Memory: 3.2 MB
```

---

## CPU

CPU time used by the service.

Example:

```
CPU: 8.9 seconds
```

---

## Logs

Recent log entries for the service.

Example:

```
Connection accepted

Connection closed

Authentication successful
```

---

# Common Service States

| State            | Meaning                                             |
| ---------------- | --------------------------------------------------- |
| loaded           | Service configuration loaded successfully           |
| active (running) | Service is currently running                        |
| active (exited)  | One-time task completed successfully                |
| active (waiting) | Waiting for an event                                |
| inactive         | Service is stopped                                  |
| enabled          | Starts automatically at boot                        |
| disabled         | Does not start automatically                        |
| static           | Cannot be enabled directly; started by another unit |

---

# Checking Service Status

## Is the service running?

```
systemctl is-active sshd
```

Output:

```
active
```

or

```
inactive
```

---

## Does it start automatically at boot?

```
systemctl is-enabled sshd
```

Output:

```
enabled
```

or

```
disabled
```

---

## Did the service fail?

```
systemctl is-failed sshd
```

Possible outputs:

```
active
```

or

```
failed
```

---

# List Failed Services

```
systemctl --failed --type=service
```

Example:

```
UNIT
-----------------------
httpd.service
mysql.service
```

These services encountered errors during startup.

---

# Quick Command Summary

| Command                                     | Purpose                                      |
| ------------------------------------------- | -------------------------------------------- |
| `systemctl`                                 | Show active loaded units                     |
| `systemctl list-units --type=service`       | List running services                        |
| `systemctl list-units --type=service --all` | List all services                            |
| `systemctl list-unit-files --type=service`  | Show installed service files and boot status |
| `systemctl status <service>`                | Detailed information about a service         |
| `systemctl is-active <service>`             | Check if a service is running                |
| `systemctl is-enabled <service>`            | Check if a service starts at boot            |
| `systemctl is-failed <service>`             | Check whether a service has failed           |
| `systemctl --failed --type=service`         | List all failed services                     |

### Interview Tip

A common interview question is: **"What is the difference between `systemctl list-units` and `systemctl list-unit-files`?"**

* **`systemctl list-units`** shows **currently loaded units** (usually active or those loaded into memory).
* **`systemctl list-unit-files`** shows **all installed unit files** and whether they are **enabled, disabled, static, or masked**, regardless of whether they are currently running.

This distinction is important because a service can be **installed but not currently loaded**, so it appears in `list-unit-files` but not necessarily in `list-units`. 
