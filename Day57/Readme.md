# 🔐 Configuring a Persistent System Journal in RHEL 10

Continuing my Linux learning journey, today I explored **persistent system journaling** — making sure important journal logs are still available **after a system reboot**.

By default, RHEL stores the system journal under:

```bash
/run/log/journal
```

Because `/run` is temporary, these logs can be lost after a reboot.

### 💾 Enable Persistent Journal Storage

Create the persistent journal directory:

```bash
mkdir /var/log/journal
```

Then flush the current journal:

```bash
journalctl --flush
```

Now journal data can be stored under:

```text
/var/log/journal/
        ↓
<machine-id>/
        ↓
system.journal
user-1000.journal
```

This allows us to investigate events **across system reboots**.

---

### 🔄 Investigate Previous Boots

Once persistent journaling is enabled, `journalctl` can show logs from different boots.

List available boots:

```bash
journalctl --list-boots
```

Current boot:

```bash
journalctl -b
```

Previous boot:

```bash
journalctl -b -1
```

Specific boot:

```bash
journalctl -b 1
```

This becomes especially useful when troubleshooting:

💥 System crashes
⚠️ Service failures
🔐 Authentication issues
🚀 Boot problems

For example, if the system crashed before the latest reboot:

```bash
journalctl -b -1
```

can help investigate what happened **before the reboot**.

---

### ♻️ Journal Rotation

Persistent logs should not grow forever.

`systemd-journald` has built-in journal rotation and storage limits.

Important settings include:

```text
SystemMaxUse
RuntimeMaxUse

SystemMaxFileSize
RuntimeMaxFileSize

SystemKeepFree
RuntimeKeepFree
```

These control:

➡️ Maximum journal storage
➡️ Maximum individual journal file size
➡️ Minimum free disk space
➡️ When old journal files are removed

Configuration is managed through:

```bash
/etc/systemd/journald.conf
```

and drop-in configuration files can be placed under:

```bash
/etc/systemd/journald.conf.d/
```

⚠️ In **RHEL 10**, `/etc/systemd/journald.conf` may not exist immediately after installation. The default configuration comes from:

```text
/usr/lib/systemd/journald.conf
```

The `/usr/lib` file should **not be edited directly**. Administrator configuration belongs under `/etc`.

---

### 🔧 Apply Configuration Changes

After changing the journald configuration:

```bash
systemctl restart systemd-journald
```

You can also inspect journal size information with:

```bash
journalctl | grep -E 'Runtime Journal|System Journal'
```

---

### 🧠 Real-World Troubleshooting Example

Imagine a server suddenly crashes and reboots.

Without persistent journaling:

```text
Server Crash 💥
     ↓
Reboot 🔄
     ↓
Previous runtime logs may be unavailable
     ↓
Difficult investigation ❌
```

With persistent journaling:

```text
Server Crash 💥
     ↓
Reboot 🔄
     ↓
Previous boot logs available
     ↓
journalctl -b -1
     ↓
Analyze events 🔍
     ↓
Find possible root cause
```

### ⭐ Key Takeaway

**Persistent journaling = historical evidence after reboot.**

The commands I want to remember:

```bash
mkdir /var/log/journal
journalctl --flush

journalctl --list-boots
journalctl -b
journalctl -b -1

systemctl restart systemd-journald
```

And the troubleshooting mindset:

**Event → Persistent Log → Previous Boot → Analyze → Root Cause 🔍**

📚 **Learn → Practice → Analyze → Troubleshoot → Understand 🚀**

#Linux #RHEL #RHCSA #SystemLogs #Journalctl #Systemd #LinuxAdministration #LinuxSecurity #Troubleshooting #RedHat #DevOps #SAPBasis #CloudComputing #Day57
