# 📋 System Log Architecture in RHEL 10 

System logging is one of the most important Linux administration topics because **logs tell us what happened on the system**.

When something goes wrong—such as an SSH failure, service failure, boot problem, or authentication issue—we check the logs to understand the reason.

RHEL 10 primarily uses two important components:

```text
systemd-journald
        +
      rsyslog
        ↓
System Logs
```

## 1️⃣ What is System Logging?

The Linux **kernel and applications continuously generate events** while the system is running.

Examples:

```text
SSH login attempt
Service started/stopped
Authentication failure
System boot
Kernel error
Scheduled job executed
Application error
```

These events are recorded as **log messages**.

Administrators use these logs for:

* 🔍 Troubleshooting
* 🛡️ Security auditing
* 📊 Monitoring
* 🚨 Investigating failures
* 🔧 Diagnosing services

You can inspect traditional text logs using commands such as:

```bash
less /var/log/messages
```

or:

```bash
tail /var/log/messages
```

---

# 2️⃣ The Two Main Logging Components

The easiest way to understand RHEL logging is:

```text
              APPLICATIONS
                   │
                   ↓
          systemd-journald
                   │
          ┌────────┴────────┐
          │                 │
          ↓                 ↓
     System Journal       rsyslog
                            │
                            ↓
                       /var/log/*
```

The two major components are:

### 🔵 systemd-journald

Responsible for **collecting and storing system events in the system journal**.

### 🟢 rsyslog

Responsible for **processing syslog messages and writing them to traditional log files under `/var/log`**, or forwarding them elsewhere according to its configuration.

---

# 3️⃣ What is `systemd-journald`?

Think of `systemd-journald` as the **central collection point** for system events.

It receives messages from multiple sources:

```text
Kernel
   ↓
Boot process
   ↓
Daemons
   ↓
Syslog events
   ↓
systemd-journald
```

According to the material, it collects:

* System kernel messages
* Early boot messages
* Standard output/error from daemons
* Syslog events

It then converts them into a **standard structured format** and stores them in an indexed system journal.

---

# 4️⃣ What is the System Journal?

The **system journal** is the structured log database managed by `systemd-journald`.

Unlike traditional text files, journal data is **structured and indexed**.

This allows you to query logs efficiently using:

```bash
journalctl
```

For example:

```bash
journalctl
```

shows journal messages.

To see recent messages:

```bash
journalctl -n
```

To follow new messages in real time:

```bash
journalctl -f
```

### ⚠️ Important point from your material

By default, the journal is stored on a filesystem that **does not persist across reboots**.

So depending on the system configuration, journal messages may not be available after a reboot.

---

# 5️⃣ What is `rsyslog`?

`rsyslog` is another major part of the logging architecture.

Its job is to:

1. Read syslog messages from the system journal.
2. Process/filter them.
3. Write them to traditional log files.
4. Forward them to other services when configured.

For example:

```text
systemd-journald
       ↓
     rsyslog
       ↓
   /var/log/
       ↓
messages
secure
cron
maillog
boot.log
```

The key point is:

> **journald collects the messages; rsyslog can process and write them into persistent traditional log files.**

---

# 6️⃣ What happens with `/dev/log` in RHEL 10?

This is an important RHEL 10 detail.

Applications that send local syslog messages send them to:

```text
/dev/log
```

In RHEL 10, `/dev/log` is a **symbolic link to a special socket** that is read by `systemd-journald`.

So the flow is:

```text
Application
     │
     ↓
 /dev/log
     │
     ↓
systemd-journald
     │
     ↓
System Journal
     │
     ↓
   rsyslog
     │
     ↓
 /var/log/*
```

### Important distinction

`rsyslog` **does not directly read `/dev/log`**.

According to your material, `rsyslog` uses its **`imjournal` module** to read syslog messages from the system journal as they arrive.

---

# 7️⃣ Why is `/var/log` important?

`/var/log` contains traditional persistent log files.

You will frequently work with:

```bash
ls -l /var/log/
```

Some important files are:

| File                | Purpose                            |
| ------------------- | ---------------------------------- |
| `/var/log/messages` | Most general syslog messages       |
| `/var/log/secure`   | Authentication and security events |
| `/var/log/maillog`  | Mail-server messages               |
| `/var/log/cron`     | Scheduled job messages             |
| `/var/log/boot.log` | Boot/startup-related messages      |

---

# 🔐 8️⃣ `/var/log/secure`

This is especially important for a Linux administrator.

It contains **security and authentication-related events**.

For example, when troubleshooting SSH authentication:

```bash
tail -f /var/log/secure
```

you can investigate authentication-related messages.

Think:

```text
SSH
sudo
Authentication
Security
       ↓
/var/log/secure
```

---

# ⚙️ 9️⃣ `/var/log/cron`

This file contains messages related to **scheduled jobs**.

For example, if a cron job is not running as expected, you can inspect:

```bash
tail /var/log/cron
```

Think:

```text
Scheduled Jobs
      ↓
    cron
      ↓
/var/log/cron
```

---

# 📧 🔟 `/var/log/maillog`

This contains messages related to the **mail server**.

Think:

```text
Mail Server
     ↓
/var/log/maillog
```

---

# 🚀 1️⃣1️⃣ `/var/log/boot.log`

`boot.log` contains console messages related to system startup.

Your material explains that this file can receive information from **two sources**:

### Source 1 — Plymouth

Plymouth displays the graphical startup screen and can save diagnostic messages to:

```text
/var/log/boot.log
```

### Source 2 — rsyslog

`rsyslog` also writes messages sent to the `local7` facility, typically related to the boot process, to:

```text
/var/log/boot.log
```

---

# 🆚 journald vs rsyslog

This is the most important comparison.

| Feature          | `systemd-journald`                              | `rsyslog`                         |
| ---------------- | ----------------------------------------------- | --------------------------------- |
| Main role        | Collects system events                          | Processes syslog messages         |
| Storage          | System journal                                  | Traditional log files             |
| Format           | Structured/indexed                              | Traditional text logs             |
| Main command     | `journalctl`                                    | `less`, `tail`, etc.              |
| Persistent logs  | Depends on journal configuration                | `/var/log` logs persist           |
| Can forward logs | Yes, through journal capabilities/configuration | Yes                               |
| RHEL role        | Central event collector                         | Syslog processing/log-file writer |

---

# 🧠 The complete architecture

Remember this diagram:

```text
                   SYSTEM EVENTS
                        │
       ┌────────────────┼─────────────────┐
       │                │                 │
     Kernel          Daemons         Applications
       │                │                 │
       └────────────────┼─────────────────┘
                        ↓
                systemd-journald
                        │
                        ↓
                SYSTEM JOURNAL
                        │
                        ↓
                    rsyslog
                        │
           ┌────────────┼────────────┐
           ↓            ↓            ↓
       messages       secure       cron
           │            │            │
           └────────────┴────────────┘
                        ↓
                    /var/log/
```

### ⭐ Easy memory trick

**`journald` = Collect**

**`rsyslog` = Process + Write/Forward**

**`journalctl` = Read the journal**

**`/var/log` = Traditional persistent log files**

---

# 🔧 Practical Commands

### View system journal

```bash
journalctl
```

### Show latest journal entries

```bash
journalctl -n
```

### Follow journal messages live

```bash
journalctl -f
```

### View traditional logs

```bash
less /var/log/messages
```

### Follow a log in real time

```bash
tail -f /var/log/messages
```

### Check authentication logs

```bash
tail -f /var/log/secure
```

### Check cron logs

```bash
tail -f /var/log/cron
```

### List log files

```bash
ls -lh /var/log/
```

---

## 🎯 Interview Question

**Q: What is the difference between `systemd-journald` and `rsyslog`?**

**Answer:**

> `systemd-journald` is the central system logging service that collects events from the kernel, boot process, daemons, and syslog sources and stores them in a structured system journal. `rsyslog` reads syslog messages from the journal, processes them according to its configuration, and writes them to traditional persistent log files under `/var/log` or forwards them to other services.

### One-line summary:

```text
systemd-journald → Collects system events
rsyslog          → Processes and writes/forwards logs
/var/log         → Traditional log files
journalctl       → Reads the system journal
```

This is the core architecture you should remember for **RHCSA/RHEL system administration**.
