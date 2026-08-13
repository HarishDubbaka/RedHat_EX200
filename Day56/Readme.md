# 🔍 Finding & Interpreting System Journal Logs in RHEL 10

The previous topic covered **syslog and rsyslog**. Now we go one step deeper into the **system journal** managed by `systemd-journald`.

The key command is:

```bash
journalctl
```

Think of it as:

> **`journalctl` = Search, filter, and analyze system journal events.**

---

## 1️⃣ What is the System Journal?

`systemd-journald` stores system logging information in a **structured and indexed journal**.

Unlike a simple text log, journal entries can contain additional fields such as:

```text
Priority
Facility
Process ID
User ID
Systemd unit
Hostname
Executable
Message
Boot ID
```

By default, RHEL stores the journal in:

```text
/run/log
```

⚠️ The `/run` filesystem is memory-based, so journal contents stored there are **lost when the system shuts down**.

---

# 2️⃣ View the Journal

To display journal entries:

```bash
journalctl
```

As `root`, you have full access to the journal.

Regular users may have restricted access to some entries.

---

# 3️⃣ Show the Last Few Entries

By default:

```bash
journalctl -n
```

shows the **last 10 entries**.

To see the last 5:

```bash
journalctl -n 5
```

This is useful when you want a quick look at what recently happened.

---

# 4️⃣ Monitor Logs in Real Time

Similar to:

```bash
tail -f
```

you can use:

```bash
journalctl -f
```

This displays recent journal entries and continues showing **new entries as they arrive**.

For example, while troubleshooting SSH:

```bash
journalctl -f
```

Then perform an SSH login attempt from another terminal.

You can watch the events appear in real time.

Press:

```text
Ctrl + C
```

to stop monitoring.

---

# 5️⃣ Filter by Priority

One of the most useful options is:

```bash
journalctl -p err
```

This displays entries with **`err` priority or higher severity**.

The priority levels are:

```text
debug
info
notice
warning
err
crit
alert
emerg
```

For troubleshooting, you don't always want thousands of messages.

Instead of:

```bash
journalctl
```

you can narrow it down:

```bash
journalctl -p err
```

This makes troubleshooting much easier.

---

# 6️⃣ Check Logs for a Specific Service

Suppose SSH is having problems.

Instead of searching the entire journal, filter for the SSH service:

```bash
journalctl -u sshd.service
```

This shows journal entries associated with:

```text
sshd.service
```

This is extremely useful for troubleshooting individual services.

For example:

```bash
journalctl -u sshd.service
```

can help identify:

```text
Service started
Service stopped
Connection events
Service failures
Configuration-related events
```

---

# 7️⃣ Search by Time

Sometimes you already know **when the problem occurred**.

You can use:

```bash
journalctl --since today
```

This displays journal entries from today.

### Specific start and end time

```bash
journalctl --since "2025-06-01 20:30" --until "2025-06-04 10:00"
```

This is very useful when someone tells you:

> "The issue happened between 8:30 PM and 10:00 AM."

You don't need to search the entire journal.

---

# ⏱️ 8️⃣ Search Relative Time

You can also search using relative time.

For example:

```bash
journalctl --since "-1 hour"
```

This shows events from the **last hour**.

This is particularly useful when troubleshooting a problem that happened recently.

---

# 🔎 9️⃣ Verbose Journal Output

Normally:

```bash
journalctl
```

shows a readable summary.

But sometimes you need more information.

Use:

```bash
journalctl -o verbose
```

This displays the **full fields associated with journal entries**.

You may see fields such as:

```text
PRIORITY=5
SYSLOG_FACILITY=0
SYSLOG_IDENTIFIER=kernel
MESSAGE=...
_HOSTNAME=localhost
_BOOT_ID=...
```

This becomes very powerful when performing detailed troubleshooting.

---

# 🧩 🔟 Search Using Journal Fields

The journal contains many searchable fields.

Important ones include:

| Field           | Meaning         |
| --------------- | --------------- |
| `_COMM`         | Command name    |
| `_EXE`          | Executable path |
| `_PID`          | Process ID      |
| `_UID`          | User ID         |
| `_SYSTEMD_UNIT` | systemd unit    |

You can combine fields to perform a **very specific search**.

For example:

```bash
journalctl _SYSTEMD_UNIT=sshd.service _PID=2188
```

This means:

> Show journal entries for `sshd.service` associated with PID `2188`.

This is much more precise than searching the entire journal.

---

# 🔐 Connecting This to Your Login Investigation

Remember your previous scenario:

> Multiple login attempts → account locked → team investigated the logs.

With the system journal, we can investigate authentication-related events using:

```bash
journalctl
```

or narrow the search:

```bash
journalctl -u sshd.service
```

You can also monitor the events live:

```bash
journalctl -f
```

And if you know approximately when the problem occurred:

```bash
journalctl --since "-1 hour"
```

Then you can analyze:

```text
Timestamp
   ↓
User
   ↓
Process
   ↓
Authentication event
   ↓
Source information
   ↓
Repeated failures
   ↓
Account lockout
```

---

# 🧠 The Most Important `journalctl` Commands

Keep this cheat sheet:

```bash
# View all journal entries
journalctl

# Last 10 entries
journalctl -n

# Last 5 entries
journalctl -n 5

# Follow new entries
journalctl -f

# Errors and higher
journalctl -p err

# Specific service
journalctl -u sshd.service

# Today's logs
journalctl --since today

# Last hour
journalctl --since "-1 hour"

# Specific time range
journalctl --since "2025-06-01 20:30" \
           --until "2025-06-04 10:00"

# Full journal fields
journalctl -o verbose

# Specific service + PID
journalctl _SYSTEMD_UNIT=sshd.service _PID=2188
```

---

## 🎯 Easy Troubleshooting Approach

Don't immediately run:

```bash
journalctl
```

and try to read thousands of lines.

Instead:

```text
          PROBLEM
             ↓
      What component?
             ↓
       Which service?
             ↓
       What time?
             ↓
     What severity?
             ↓
   Filter the journal
             ↓
     Analyze events
             ↓
     Find root cause 🔍
```

For example:

**SSH problem**

```bash
journalctl -u sshd.service
```

**Only errors**

```bash
journalctl -p err
```

**Problem happened recently**

```bash
journalctl --since "-1 hour"
```

**Need detailed information**

```bash
journalctl -o verbose
```

### ⭐ Key takeaway

> **Don't just read the system journal — filter it.**

The power of `journalctl` comes from combining:

**Service + Priority + Time + Journal Fields**

```text
journalctl
    ↓
Filter
    ↓
Relevant Events
    ↓
Analyze
    ↓
Troubleshoot 🔍
```

That is the real skill in **system journal analysis**.
