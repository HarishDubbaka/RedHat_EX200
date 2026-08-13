# 🔐 Interpreting & Managing Syslog Events 

This topic goes one step deeper than simply **checking logs**.

Previously, we learned:

> **Logs tell us what happened.**

Now we learn:

> **How does RHEL decide which messages go to which log file?**

The main component responsible for this is **`rsyslog`**.

---

## 1️⃣ What is a Syslog Event?

Many Linux programs use the **syslog protocol** to record events.

Every syslog message has two important properties:

```text
                    SYSLOG MESSAGE
                          │
              ┌───────────┴───────────┐
              ↓                       ↓
          FACILITY                 PRIORITY
       "Who generated it?"       "How serious?"
```

### 🏷️ Facility

The **facility** identifies the subsystem that generated the message.

Examples:

| Facility          | Meaning                         |
| ----------------- | ------------------------------- |
| `kern`            | Kernel messages                 |
| `user`            | User-level messages             |
| `mail`            | Mail system                     |
| `daemon`          | System daemons                  |
| `auth`            | Authentication/security         |
| `syslog`          | Internal syslog messages        |
| `cron`            | Scheduled jobs                  |
| `authpriv`        | Authorization/security messages |
| `ftp`             | FTP messages                    |
| `local0`–`local7` | Custom/local messages           |

### 🚨 Priority

Priority tells us **how serious the message is**.

From highest to lowest:

```text
emerg      0   🔴 System unusable
alert      1   🔴 Immediate action required
crit       2   🔴 Critical condition
err        3   🟠 Error
warning    4   🟡 Warning
notice     5   🔵 Significant normal event
info       6   🔵 Information
debug      7   ⚪ Debugging
```

### 🧠 Easy way to remember

**Facility = WHO**

**Priority = HOW SERIOUS**

For example:

```text
authpriv.warning
```

means:

> Authentication/security messages with **warning or higher severity**.

---

# 2️⃣ What is `rsyslog`?

`rsyslog` uses the **facility + priority** of a message to decide what to do with it.

Its main configuration files are:

```text
/etc/rsyslog.conf
/etc/rsyslog.d/*.conf
```

You can check the configuration with:

```bash
less /etc/rsyslog.conf
```

And:

```bash
ls /etc/rsyslog.d/
```

---

# 3️⃣ How Does an Rsyslog Rule Work?

An `rsyslog` rule has two parts:

```text
FILTER                    ACTION
  │                          │
  ↓                          ↓
Which messages?        What should happen?
```

For example:

```text
authpriv.*    /var/log/secure
```

### Filter

```text
authpriv.*
```

Means:

> All priorities from the `authpriv` facility.

### Action

```text
/var/log/secure
```

Means:

> Save those messages in `/var/log/secure`.

So:

```text
authpriv.* /var/log/secure
     │             │
     │             └── Where to save
     │
     └── What messages to select
```

---

# 4️⃣ Understanding `*`

The `*` is a wildcard.

For example:

```text
authpriv.*
```

means:

> All priorities for `authpriv`.

Similarly:

```text
*.info
```

means:

> `info` and higher-priority messages from all facilities.

---

# 5️⃣ Priority Filtering — Very Important

Suppose we have:

```text
authpriv.warning    /var/log/secure
```

This does **not** mean only `warning`.

It means:

```text
warning
err
crit
alert
emerg
```

Because syslog priorities are ordered by severity.

Think:

```text
emerg
  ↓
alert
  ↓
crit
  ↓
err
  ↓
warning
  ↓
notice
  ↓
info
  ↓
debug
```

If you select `warning`, messages at **warning and more severe levels** are included.

---

# 6️⃣ Example from `/etc/rsyslog.conf`

A common rule is:

```text
*.info;mail.none;authpriv.none;cron.none    /var/log/messages
```

Let's break it down.

### `*.info`

Take:

> All facilities with `info` priority or higher.

But then:

```text
mail.none
```

means:

> Don't include mail messages.

And:

```text
authpriv.none
```

means:

> Don't include `authpriv` messages.

And:

```text
cron.none
```

means:

> Don't include cron messages.

Finally:

```text
/var/log/messages
```

means:

> Write the matching messages to `/var/log/messages`.

---

# 🔐 7️⃣ Why is `/var/log/secure` Important?

The configuration:

```text
authpriv.*    /var/log/secure
```

means authentication/authorization-related messages are stored in:

```text
/var/log/secure
```

This is particularly useful for investigating things like:

```text
SSH login
Failed authentication
Successful authentication
sudo activity
Security-related events
```

For example:

```bash
tail -f /var/log/secure
```

You can watch authentication events **as they happen**.

---

# 🔎 8️⃣ How to Read a Log Entry

Consider this example:

```text
Mar 20 20:11:48 localhost sshd[1433]: Failed password for student from 172.25.0.10 port 59344 ssh2
```

Break it into pieces:

```text
Mar 20 20:11:48
       ↓
Timestamp
```

```text
localhost
       ↓
Host
```

```text
sshd[1433]
       ↓
Program + PID
```

```text
Failed password for student
       ↓
Actual event
```

```text
from 172.25.0.10
       ↓
Source IP
```

This is exactly the kind of information you would analyze during an authentication investigation.

---

# 👀 9️⃣ Monitor Logs in Real Time

One of the most useful commands is:

```bash
tail -f /var/log/secure
```

`-f` means:

> Keep watching the file and display new lines as they are written.

For example:

### Terminal 1

```bash
tail -f /var/log/secure
```

### Terminal 2

Attempt SSH:

```bash
ssh root@hostA
```

Then Terminal 1 may immediately show:

```text
Accepted password for root from 172.25.254.254
```

or:

```text
Failed password for student from 172.25.0.10
```

This is a great way to **reproduce an issue and observe exactly what gets logged**.

---

# 🧪 🔟 The `logger` Command

You can also manually generate a syslog message.

Command:

```bash
logger "Test message"
```

By default, the material explains that `logger` sends a `user.notice` message unless another priority/facility is specified.

You can explicitly specify one:

```bash
logger -p local7.notice "Log entry created on host"
```

Because the RHEL configuration has:

```text
local7.*    /var/log/boot.log
```

the test message can appear in:

```text
/var/log/boot.log
```

This is extremely useful when **testing an rsyslog configuration**.

---

# 🔄 1️⃣1️⃣ What is Log Rotation?

Imagine this:

```text
/var/log/messages
```

keeps growing every day.

Eventually it could consume a huge amount of disk space.

That's why RHEL uses **`logrotate`**.

It rotates old log files and creates new ones.

For example:

```text
messages
   ↓
messages-20250620
   ↓
new messages file
```

The rotated file contains older logs.

The material explains that a scheduled job checks rotation requirements daily, with most log files rotated weekly, and older rotated files eventually removed according to the configured retention policy.

---

# 🧠 Complete Syslog Architecture

Now connect everything together:

```text
                 APPLICATION / SERVICE
                          │
                          ↓
                   SYSLOG MESSAGE
                          │
                ┌─────────┴─────────┐
                ↓                   ↓
            FACILITY             PRIORITY
              WHO?             HOW SERIOUS?
                │                   │
                └─────────┬─────────┘
                          ↓
                       rsyslog
                          │
                    Rsyslog Rules
                          │
                ┌─────────┴─────────┐
                ↓                   ↓
             FILTER               ACTION
                │                   │
                └─────────┬─────────┘
                          ↓
                     /var/log/*
                          │
                          ↓
                     logrotate
                          │
                          ↓
                 Manage old logs
```

---

# 🎯 Real-World Example: Your User Lockout

This connects directly with the scenario you mentioned earlier.

Suppose you repeatedly attempt to log in:

```text
Login Attempt 1 → Failed ❌
Login Attempt 2 → Failed ❌
Login Attempt 3 → Failed ❌
Login Attempt 4 → Failed ❌
              ↓
        Account Locked 🔒
```

You can investigate the authentication events:

```bash
tail -f /var/log/secure
```

or search historical entries:

```bash
grep "Failed password" /var/log/secure
```

Then analyze:

```text
Timestamp
   ↓
Username
   ↓
Source IP
   ↓
sshd process
   ↓
Authentication result
   ↓
Number/frequency of failures
```

That's how logs become **evidence for troubleshooting**, rather than just text files.

---

## ⭐ Interview Cheat Sheet

| Concept                 | Remember                        |
| ----------------------- | ------------------------------- |
| Facility                | Who generated the message?      |
| Priority                | How serious is it?              |
| `rsyslog`               | Processes syslog messages       |
| `/etc/rsyslog.conf`     | Main configuration              |
| `/etc/rsyslog.d/*.conf` | Additional configuration        |
| Filter                  | Selects messages                |
| Action                  | Determines what happens to them |
| `authpriv.*`            | All authpriv priorities         |
| `/var/log/secure`       | Authentication/security logs    |
| `tail -f`               | Monitor logs live               |
| `logger`                | Generate a test syslog message  |
| `logrotate`             | Rotates/retains log files       |

### 💡 One formula to remember:

```text
FACILITY + PRIORITY
        ↓
      FILTER
        ↓
      ACTION
        ↓
    LOG FILE / OTHER TARGET
```

**Facility tells you WHO.
Priority tells you HOW SERIOUS.
Rsyslog decides WHERE it goes.**
