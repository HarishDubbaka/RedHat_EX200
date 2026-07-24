# Real-Time Process Monitoring with `top` (RHEL/Linux)

If `ps` is like taking a **photograph** of your system, then `top` is like watching a **live CCTV feed**. It updates continuously, allowing you to monitor processes, CPU usage, memory consumption, and system load in real time.

---

# What is `top`?

The `top` command displays:

* Live process information
* CPU usage
* Memory usage
* Load average
* Running, sleeping, and zombie processes
* Process priorities
* Uptime

Unlike `ps`, which shows a snapshot, `top` refreshes every few seconds by default.

```bash
top
```

Example:

```text
top - 10:30:42 up 12 days, 4:21, 2 users,
Tasks: 248 total, 1 running, 247 sleeping, 0 stopped, 0 zombie
%Cpu(s): 7.2 us, 2.1 sy, 0.0 ni, 90.1 id, 0.2 wa
MiB Mem : 15900 total, 4200 free, 5800 used, 5900 buff/cache
MiB Swap: 4096 total, 4096 free, 0 used

PID USER      PR NI   VIRT   RES   SHR S %CPU %MEM TIME+ COMMAND
1234 root     20  0 220000 32000  9000 S 15.4  0.2 01:22.12 httpd
2356 oracle   20  0 2.5g   1.2g 65000 S 30.2  8.0 22:18.45 ora_pmon
```

---

# Anatomy of the `top` Screen

```text
+------------------------------------------------+
| Current Time                                   |
| System Uptime                                  |
| Logged-in Users                                |
| Load Average                                   |
+------------------------------------------------+

+------------------------------------------------+
| Task Summary                                   |
+------------------------------------------------+

+------------------------------------------------+
| CPU Usage                                      |
+------------------------------------------------+

+------------------------------------------------+
| Memory & Swap                                  |
+------------------------------------------------+

+------------------------------------------------+
| Live Process List                              |
+------------------------------------------------+
```

---

# Header Section

## Current Time

```text
10:30:42
```

Current system time.

---

## Uptime

```text
up 12 days
```

Time since last reboot.

---

## Users

```text
2 users
```

Currently logged-in sessions.

---

## Load Average

```text
load average: 0.85, 1.12, 1.30
```

Average system load over:

* 1 minute
* 5 minutes
* 15 minutes

---

# Tasks

Example:

```text
Tasks: 248 total,
1 running,
247 sleeping,
0 stopped,
0 zombie
```

Meaning:

```text
248 Processes

│
├──1 Running
├──247 Sleeping
├──0 Stopped
└──0 Zombie
```

---

# CPU Usage

Example:

```text
%Cpu(s): 7.2 us, 2.1 sy, 0.0 ni,
90.1 id, 0.2 wa
```

Meaning:

| Field | Meaning                       |
| ----- | ----------------------------- |
| us    | User processes                |
| sy    | Kernel processes              |
| ni    | Nice processes                |
| id    | Idle CPU                      |
| wa    | Waiting for Disk I/O          |
| hi    | Hardware Interrupt            |
| si    | Software Interrupt            |
| st    | Steal time (Virtual Machines) |

Example:

```text
Idle = 90%
```

CPU is mostly free.

---

# Memory Information

Example:

```text
MiB Mem

15900 total

4200 free

5800 used

5900 cache
```

Linux uses free memory as cache to improve performance.

More cache is usually a good thing.

---

# Swap

```text
Swap

4096 total

0 used
```

If swap usage continuously increases, investigate memory pressure.

---

# Process Table

```text
PID USER PR NI VIRT RES SHR S %CPU %MEM TIME+ COMMAND
```

Each row represents one process.

---

# Understanding Each Column

## PID

```text
1234
```

Unique Process ID.

Useful for:

```bash
kill 1234
```

---

## USER

Owner of the process.

Example:

```text
root

oracle

mysql

student
```

---

## PR (Priority)

Example:

```text
20
```

Lower values mean higher scheduling priority.

---

## NI (Nice Value)

Example:

```text
0
```

Range:

```text
-20 Highest Priority

0 Default

19 Lowest Priority
```

---

## VIRT

Virtual memory used.

Includes:

* RAM
* Shared libraries
* Swapped pages
* Mapped files

Equivalent to **VSZ** in `ps`.

---

## RES

Resident memory.

Actual RAM currently used.

Equivalent to **RSS**.

---

## SHR

Shared memory.

Shared with other processes.

Examples:

* libc
* Shared libraries

---

## Process State (S)

Possible values:

| State | Meaning                                             |
| ----- | --------------------------------------------------- |
| R     | Running                                             |
| S     | Sleeping                                            |
| D     | Uninterruptible Sleep (I/O wait)                    |
| T     | Stopped                                             |
| Z     | Zombie                                              |
| I     | Idle kernel thread (common on modern Linux kernels) |

Example:

```text
R
```

Running on CPU.

---

Example:

```text
S
```

Sleeping and waiting for an event.

---

Example:

```text
D
```

Waiting for disk or network I/O.

---

Example:

```text
Z
```

Zombie process.

Process has finished, but its parent hasn't collected its exit status.

---

# %CPU

Current CPU usage.

Example:

```text
245%
```

Possible on multicore systems.

Example:

```text
4 CPUs

One process uses

CPU0 =100%

CPU1 =100%

CPU2 =45%

Total =245%
```

---

# %MEM

Percentage of physical RAM used.

Example:

```text
8.4%
```

---

# TIME+

```text
02:18.45
```

Total CPU time consumed since the process started.

Not wall-clock time.

---

# COMMAND

Process name.

Example:

```text
httpd

mysqld

java

sapstartsrv
```

---

# Most Useful `top` Keyboard Shortcuts

| Key         | Action                                      |
| ----------- | ------------------------------------------- |
| `h` or `H`  | Help screen                                 |
| `q`         | Quit `top`                                  |
| `1`         | Show CPU usage per core or summary          |
| `Shift + P` | Sort by CPU usage                           |
| `Shift + M` | Sort by memory usage                        |
| `Shift + H` | Show/hide individual threads                |
| `u`         | Filter by user                              |
| `k`         | Kill a process (prompts for PID and signal) |
| `r`         | Renice a process (change priority)          |
| `s`         | Change refresh interval                     |
| `f`         | Select or hide columns                      |
| `Shift + W` | Save your current configuration             |

---

# Example: Find the CPU-Hungry Process

Press:

```text
Shift + P
```

Processes are sorted by CPU usage:

```text
java        120%

mysqld       85%

chrome       45%
```

The process consuming the most CPU appears at the top.

---

# Example: Find the Memory-Hungry Process

Press:

```text
Shift + M
```

Output:

```text
oracle      8 GB

java        5 GB

python      2 GB
```

---

# Show CPU Usage Per Core

Press:

```text
1
```

Instead of:

```text
CPU 30%
```

You will see:

```text
CPU0 25%

CPU1 10%

CPU2 95%

CPU3 20%
```

Useful for identifying uneven CPU utilization.

---

# Kill a Process

Press:

```text
k
```

`top` prompts:

```text
PID to kill:
```

Enter:

```text
2456
```

Then choose a signal (default is `15` for SIGTERM).

---

# Renice a Process

Press:

```text
r
```

Enter the PID and a new nice value to increase or decrease its scheduling priority.

---

# Save Your Preferred Layout

After customizing `top` (columns, sorting, refresh rate, etc.), press:

```text
Shift + W
```

Your settings will be saved and restored the next time you run `top`.

---

# `top` vs `ps`

| Feature              | `ps`                  | `top`   |
| -------------------- | --------------------- | ------- |
| Real-time updates    | ❌                     | ✅       |
| Static snapshot      | ✅                     | ❌       |
| Interactive controls | ❌                     | ✅       |
| Sort while running   | ❌                     | ✅       |
| Kill processes       | ❌ (requires `kill`)   | ✅ (`k`) |
| Change priority      | ❌ (requires `renice`) | ✅ (`r`) |

---

# Key Takeaways

* **`top` provides a live, continuously updating view of system activity**, making it indispensable for troubleshooting performance issues.
* The **header** gives a quick overview of uptime, load average, tasks, CPU, memory, and swap usage.
* The **process table** lets you identify CPU- and memory-intensive processes in real time.
* Interactive shortcuts like **`Shift+P`**, **`Shift+M`**, **`1`**, **`k`**, and **`r`** make `top` a powerful tool for monitoring and managing processes without leaving the terminal.
