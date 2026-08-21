# Maintaining Synchronized Time in RHEL

Have you ever wondered **how a Linux server knows the correct time and keeps it synchronized?** ⏰

The answer is **NTP + chronyd**.

On RHEL, `chronyd` communicates with configured NTP time sources and continuously helps keep the system clock accurate. Correct time is especially important for log analysis, authentication, distributed systems, and services that depend on accurate timestamps. 

---

## 1. The Big Picture

Think of Linux time synchronization like this:

```text
             NTP Time Sources
                    │
                    │ NTP
                    ▼
              ┌───────────┐
              │  chronyd  │
              └─────┬─────┘
                    │
                    ▼
             System Clock
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       Logs 📋            Applications
```

### Main components

| Component          | Purpose                                                |
| ------------------ | ------------------------------------------------------ |
| `timedatectl`      | View/change time and timezone settings                 |
| `chronyd`          | Background service that synchronizes the clock         |
| `chronyc`          | Command-line client used to communicate with `chronyd` |
| NTP                | Protocol used to synchronize time                      |
| `/etc/chrony.conf` | Chrony configuration file                              |

The source specifically identifies `timedatectl`, `chronyd`, and `chronyc` as the main tools for managing time synchronization. 

---

# 2. Why Is Correct Time Important?

Imagine you have three servers:

```text
server01 → 10:00:05
server02 → 10:03:21
server03 → 09:57:48
```

Now an application fails.

You check the logs:

```text
server01 → 10:00:05 ERROR
server02 → 10:03:21 connection failed
server03 → 09:57:48 service stopped
```

Which event happened first?

It becomes difficult to determine.

That's why synchronized time is important when analyzing logs from multiple systems. 

---

# 3. What Is NTP?

**NTP = Network Time Protocol**

It allows computers to obtain accurate time from other time sources over a network.

For example:

```text
Internet NTP Server
        │
        │ NTP
        ▼
     Linux Server
```

A Linux server can synchronize with public NTP services or with an organization's internal NTP server. 

---

# 4. What Is Chrony?

On RHEL, `chronyd` is responsible for time synchronization.

```text
chronyd
   │
   ├── contacts NTP servers
   ├── measures clock differences
   ├── calculates corrections
   └── adjusts the system clock
```

The source explains that `chronyd` synchronizes the local clock with configured NTP servers and can also calculate clock drift when network connectivity is unavailable. 

Check the service:

```bash
systemctl status chronyd
```

---

# 5. Check Current Time Configuration

The easiest command is:

```bash
timedatectl
```

Example:

```text
Local time:           Wed 2025-04-02 03:05:55 MST
Universal time:       Wed 2025-04-02 10:05:55 UTC
RTC time:             Wed 2025-04-02 10:05:55
Time zone:            America/Phoenix (MST, -0700)
System clock synchronized: yes
NTP service:          active
RTC in local TZ:      no
```

`timedatectl` provides an overview of:

* Local time
* UTC time
* RTC time
* Time zone
* Whether the system clock is synchronized
* Whether NTP is active 

### The two most important lines

```text
System clock synchronized: yes
NTP service: active
```

If you see:

```text
System clock synchronized: yes
```

your system is synchronized.

---

# 6. Time Zone vs Time Synchronization

These are **two different things**.

### Time synchronization

Answers:

> "Is my server's clock accurate?"

Handled by:

```text
NTP / chronyd
```

### Time zone

Answers:

> "How should I display that time for this location?"

Handled by:

```text
timedatectl
```

For example, the same moment can be:

```text
UTC:
10:00

India:
15:30

New York:
06:00
```

The actual moment is the same; only the displayed local time changes.

---

# 7. List Available Time Zones

Run:

```bash
timedatectl list-timezones
```

You will see entries such as:

```text
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
America/New_York
America/Denver
Asia/Kolkata
Asia/Tokyo
Europe/London
```

RHEL uses time zone names based on the **IANA Time Zone Database**. 

---

# 8. Find the Correct Time Zone

You can use:

```bash
tzselect
```

It interactively asks questions about your location and tells you the appropriate time zone name.

Important:

```text
tzselect
```

**does not change the system's timezone.**

It only helps you identify the correct name. 

---

# 9. Change the Time Zone

Use:

```bash
timedatectl set-timezone <timezone>
```

Example:

```bash
timedatectl set-timezone America/Phoenix
```

For India:

```bash
timedatectl set-timezone Asia/Kolkata
```

Then verify:

```bash
timedatectl
```

The source also notes that UTC can be selected directly with:

```bash
timedatectl set-timezone UTC
```

because `tzselect` does not provide UTC as a selectable location. 

---

# 10. Manually Change System Time

You can manually set the time with:

```bash
timedatectl set-time 9:00:00
```

Or specify a complete date and time:

```bash
timedatectl set-time "2026-08-21 10:30:00"
```

However, there is an important condition.

If automatic NTP synchronization is enabled, manual time changes can fail with:

```text
Failed to set time: Automatic time synchronization is enabled
```



---

# 11. Disable NTP Before Manually Setting Time

Disable automatic synchronization:

```bash
timedatectl set-ntp false
```

Then you can manually set the time:

```bash
timedatectl set-time 09:00:00
```

The source states that on RHEL, `timedatectl set-ntp` controls whether the `chronyd` NTP service is enabled. 

After manually setting the time, you would normally re-enable synchronization:

```bash
timedatectl set-ntp true
```

---

# 12. Where Does Chrony Get the Time?

Chrony gets time from configured NTP sources.

Configuration file:

```text
/etc/chrony.conf
```

You might see:

```ini
pool 2.rhel.pool.ntp.org iburst
```

or:

```ini
server ntp.example.com iburst
```

The source explains that `server` specifies an individual NTP server, while `pool` specifies a DNS name that can resolve to multiple NTP servers. 

---

# 13. `server` vs `pool`

### `server`

Points to one NTP server:

```ini
server ntp.example.com iburst
```

### `pool`

Points to an NTP pool:

```ini
pool 2.rhel.pool.ntp.org iburst
```

A pool can provide multiple NTP servers.

```text
             NTP Pool
          /     |      \
         /      |       \
      NTP1     NTP2     NTP3
        \        |       /
         \       |      /
             chronyd
```

The source notes that the RHEL pool name can resolve to multiple public NTP servers. 

---

# 14. What Does `iburst` Mean?

You may frequently see:

```ini
server ntp.example.com iburst
```

The `iburst` option tells `chronyd` to make several quick measurements when starting communication with the server.

This helps synchronize the clock faster. Red Hat recommends using `iburst` with the `server` directive. 

Easy memory trick:

```text
iburst → initial burst → faster initial synchronization
```

---

# 15. Why Use Multiple NTP Servers?

You could configure one NTP server:

```ini
server ntp1.example.com iburst
```

But if that server goes down, synchronization becomes less reliable.

Better:

```ini
server ntp1.example.com iburst
server ntp2.example.com iburst
server ntp3.example.com iburst
```

Multiple sources also help compensate for variable network delays. The source recommends three or more sources when possible. 

---

# 16. Restart Chronyd

After changing `/etc/chrony.conf`:

```bash
systemctl restart chronyd
```

Then check:

```bash
systemctl status chronyd
```

The source explicitly instructs restarting `chronyd` after changing its time source configuration. 

---

# 17. Check NTP Sources

Use:

```bash
chronyc sources
```

For detailed information:

```bash
chronyc sources -v
```

Example:

```text
MS Name/IP address     Stratum Poll Reach LastRx Last sample
================================================================
^* ntp.example.com          2   6   377     20   -28us
^- 192.0.2.10               2  10   377    936  -2329us
^+ clock3.example.net       2  10   377    149  -1220us
```

---

# 18. Understanding `*`, `+`, and `-`

This is very important for troubleshooting.

### `*` — Current Best Source

```text
^* ntp.example.com
```

Means:

> Chrony is currently using this server as the best time source.



---

### `+` — Also Contributing

```text
^+ clock3.example.net
```

Means:

> This source is also contributing to synchronization.



---

### `-` — Available but Not Selected

```text
^- 192.0.2.10
```

Means:

> The source is usable, but better sources are currently available.



---

### `?` — Currently Unusable

```text
^? ntp.example.com
```

Means:

> Chrony cannot currently use this server.

Possible reasons include:

* Server unreachable
* Network problem
* Server not responding
* Not enough successful measurements yet



---

# 19. What Is Stratum?

**Stratum tells you how far a time source is from a high-quality reference clock.**

Think of it as levels:

```text
Stratum 0
Atomic clock / GPS reference
        │
        ▼
Stratum 1
NTP server directly connected
        │
        ▼
Stratum 2
Server synchronized from Stratum 1
        │
        ▼
Stratum 3
Server synchronized from Stratum 2
```

The source defines the reference clock as **Stratum 0**, a directly connected NTP server as **Stratum 1**, and a machine synchronized from that server as **Stratum 2**. 

---

# 20. The Complete RHEL Time-Synchronization Flow

Put everything together:

```text
             Accurate Time Source
                     │
                     │ NTP
                     ▼
              ┌─────────────┐
              │   chronyd   │
              └──────┬──────┘
                     │
                     ▼
              Linux System Clock
                     │
                     ├──────────────┐
                     ▼              ▼
                   Logs          Services
                     │
                     ▼
              Accurate timestamps
```

Configuration:

```text
/etc/chrony.conf
       │
       ▼
NTP servers / pools
       │
       ▼
chronyd
       │
       ▼
System clock
```

Management:

```text
timedatectl
    │
    ├── View time
    ├── View timezone
    ├── Enable/disable NTP
    └── Change timezone

chronyc
    │
    ├── sources
    └── Monitor chronyd
```

---

# 21. RHCSA Commands to Remember 🧠

### Check time

```bash
timedatectl
```

### List time zones

```bash
timedatectl list-timezones
```

### Set time zone

```bash
timedatectl set-timezone Asia/Kolkata
```

### Enable NTP

```bash
timedatectl set-ntp true
```

### Disable NTP

```bash
timedatectl set-ntp false
```

### Set time manually

```bash
timedatectl set-time 09:00:00
```

### Check chronyd

```bash
systemctl status chronyd
```

### Restart chronyd

```bash
systemctl restart chronyd
```

### Check NTP sources

```bash
chronyc sources
```

### Detailed NTP source information

```bash
chronyc sources -v
```

---

# 22. Quick Troubleshooting

If the server time is wrong, follow this sequence:

### Step 1 — Check overall status

```bash
timedatectl
```

Look for:

```text
System clock synchronized: yes
NTP service: active
```

### Step 2 — Check chronyd

```bash
systemctl status chronyd
```

### Step 3 — Check NTP sources

```bash
chronyc sources -v
```

Look for:

```text
^*
```

A `*` indicates the current best source.

### Step 4 — Check configuration

```bash
cat /etc/chrony.conf
```

Look for:

```ini
server ...
```

or:

```ini
pool ...
```

### Step 5 — Restart if configuration was changed

```bash
systemctl restart chronyd
```

---

# 23. Final Memory Trick 🚀

Remember the three commands:

```text
timedatectl
     ↓
"What is my time configuration?"

chronyd
     ↓
"Keep my clock synchronized."

chronyc
     ↓
"Show me how synchronization is working."
```

And remember:

```text
NTP
 ↓
Provides time

chronyd
 ↓
Synchronizes the Linux clock

chronyc
 ↓
Monitors chronyd

timedatectl
 ↓
Manages time + timezone
```

### One-line interview answer

> **In RHEL, `chronyd` synchronizes the system clock with configured NTP time sources, while `timedatectl` is used to view and manage the system time, timezone, and NTP settings.** 
