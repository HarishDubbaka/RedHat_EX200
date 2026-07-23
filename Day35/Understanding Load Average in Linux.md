# Monitoring Process Activity – Understanding Load Average in Linux (RHEL)

Load Average is one of the most misunderstood Linux performance metrics. Many administrators think it represents **CPU usage**, but it actually measures **how busy the system is**, including both CPU and I/O waits.

---

# What is Load Average?

**Load Average** represents the average number of processes that are either:

* 🟢 Running on the CPU (State **R**)
* 🟡 Waiting for CPU time (Runnable Queue)
* 🔴 Waiting for Disk or Network I/O (State **D**)

It tells you **how many processes are waiting for system resources**.

Think of it like a queue at a supermarket.

* One cashier = CPU
* Customers = Processes

If there are:

* 1 customer and 1 cashier → No waiting
* 5 customers and 1 cashier → 4 customers wait

That waiting line is similar to the Linux Load Average.

---

# How Linux Calculates Load Average

Every **5 seconds**, the Linux kernel checks:

* How many processes are runnable (R)
* How many are in uninterruptible sleep (D)

Then it calculates an **Exponential Moving Average (EMA)** over:

* Last 1 minute
* Last 5 minutes
* Last 15 minutes

This gives smoother values instead of sudden spikes.

---

# Visual Representation

```text
Every 5 Seconds

Kernel Checks

        +-------------------------+
        | Runnable Processes (R)  |
        +-------------------------+
                    +
        +-------------------------+
        | Waiting for I/O (D)     |
        +-------------------------+
                    |
                    v

        Exponential Moving Average

        Last 1 Minute
        Last 5 Minutes
        Last 15 Minutes
```

---

# Viewing Load Average

Use:

```bash
uptime
```

Example:

```bash
$ uptime

15:29:03 up 14 min, 2 users,
load average: 2.92, 4.48, 5.20
```

Output:

```text
Current Time
     |
15:29:03

System Uptime
      |
up 14 min

Logged-in Users
        |
2 users

Load Average
      |
2.92   4.48   5.20
```

Meaning:

| Value | Represents      |
| ----- | --------------- |
| 2.92  | Last 1 minute   |
| 4.48  | Last 5 minutes  |
| 5.20  | Last 15 minutes |

---

# Understanding the Trend

Example:

```
Load Average

15 Minutes → 5.20
5 Minutes  → 4.48
1 Minute   → 2.92
```

This means:

```
5.20
 ↓
4.48
 ↓
2.92
```

The load is **decreasing**.

---

Another example:

```
1.2 2.4 3.6
```

Means:

```
Past 15 mins → 3.6
Past 5 mins  → 2.4
Current      → 1.2
```

System is recovering.

---

If:

```
1.2 3.5 5.6
```

Then:

```
Current → 1.2

Past 5 mins → 3.5

Past 15 mins → 5.6
```

Again, improving.

---

If:

```
5.5 3.0 1.5
```

Then:

```
Past

1.5
 ↑
3.0
 ↑
5.5

Current load increasing
```

This indicates the system is becoming busier.

---

# Logical CPUs

Before interpreting Load Average, determine how many CPUs Linux sees:

```bash
lscpu
```

Example:

```bash
CPU(s): 4
```

Linux treats Hyper-Threads as logical CPUs.

Example hardware:

```
1 Socket

    |
2 Cores

    |
2 Threads/Core

Total Logical CPUs

2 × 2 = 4
```

---

# Why CPU Count Matters

A load average of **4** means different things on different systems.

### System A

```
1 CPU

Load = 4

CPU can execute only one process.

Remaining 3 wait.

System overloaded.
```

---

### System B

```
4 CPUs

Load = 4

Each CPU runs one process.

Nobody waits.

System healthy.
```

---

### System C

```
8 CPUs

Load = 4

Only half CPUs busy.

System lightly loaded.
```

---

# Per-CPU Load Calculation

Formula:

```
Per CPU Load

Load Average
-------------
Logical CPUs
```

Example:

```
Load = 2.92

CPU = 4

2.92 / 4

= 0.73
```

---

For all values:

```
Load Average

2.92
4.48
5.20

CPUs = 4

2.92/4 = 0.73

4.48/4 = 1.12

5.20/4 = 1.30
```

Result:

| Time   | Load | Per CPU |
| ------ | ---- | ------- |
| 1 min  | 2.92 | 0.73    |
| 5 min  | 4.48 | 1.12    |
| 15 min | 5.20 | 1.30    |

---

# How to Interpret Per-CPU Load

### Less than 1

```
Per CPU Load = 0.40
```

Meaning:

* CPU has spare capacity
* No queue
* Healthy

---

### Equal to 1

```
Per CPU Load = 1
```

Meaning:

* CPU fully utilized
* No waiting

Perfect utilization.

---

### Greater than 1

```
Per CPU Load = 2
```

Meaning:

```
CPU Busy

Process 1 → Running

Process 2 → Waiting
```

There are more tasks than CPUs available, so waiting occurs.

---

# Load Average Is NOT CPU Usage

This is the most common misconception.

Example:

```
CPU Usage

10%
```

But:

```
Load Average

15
```

How?

Processes may be waiting for:

* Slow disk
* NFS mount
* SAN storage
* Network filesystem
* Database I/O

The CPU is idle, but processes are blocked waiting for I/O, increasing the load average.

---

# Process States That Affect Load

### Running

```
State R

Running

CPU executes process.
```

Contributes to Load Average.

---

### Runnable

```
State R

Waiting for CPU.
```

Also contributes.

---

### Uninterruptible Sleep

```
State D

Waiting for Disk

Waiting for SAN

Waiting for Network Storage
```

Also contributes.

---

### Sleeping

```
State S

Sleeping

Waiting for keyboard input
```

Does **not** contribute.

---

# Real Example

Suppose:

```
8 CPUs
```

Processes:

```
4 Running

2 Waiting CPU

6 Waiting Disk
```

Total counted:

```
4 + 2 + 6 = 12
```

Load Average ≈ 12

Per CPU:

```
12 / 8 = 1.5
```

Even if CPU usage is only 50%, the load is high because many processes are blocked on I/O.

---

# When Is Load Average Healthy?

| Per-CPU Load | Meaning                     |
| ------------ | --------------------------- |
| 0.0–0.7      | Excellent                   |
| 0.7–1.0      | Normal                      |
| ≈1.0         | Fully utilized              |
| 1.0–2.0      | Moderately overloaded       |
| >2.0         | Heavy overload; investigate |

> These are general guidelines. Workloads differ, so sustained values above 1 warrant investigation rather than immediate concern.

---

# Commands to Monitor Load

```bash
uptime
```

Show current load average.

```bash
w
```

Shows users and load average.

```bash
top
```

Real-time CPU, memory, and load information.

```bash
htop
```

Interactive process viewer (if installed).

```bash
vmstat 1
```

Shows CPU queues, memory, and I/O statistics every second.

```bash
iostat
```

Checks disk performance and identifies I/O bottlenecks.

---

# Troubleshooting High Load

If you observe a high load average:

1. Check CPU usage:

   ```bash
   top
   ```

2. Check disk I/O:

   ```bash
   iostat -x 1
   ```

3. Check memory pressure:

   ```bash
   free -h
   vmstat 1
   ```

4. Check process states:

   ```bash
   ps -eo pid,stat,comm | grep '^.* D'
   ```

5. Check network activity if using network storage:

   ```bash
   sar -n DEV 1
   ```

---

# Key Takeaways

* **Load Average measures demand for system resources, not just CPU usage.**
* It includes processes in **Running (R)** and **Uninterruptible Sleep (D)** states.
* The three values represent the **1-, 5-, and 15-minute** averages.
* Always compare the load average with the **number of logical CPUs** to understand whether the system is overloaded.
* High load with low CPU usage often points to **disk or network I/O bottlenecks**, making tools like `iostat` and `vmstat` essential for diagnosis.
