## Processes and the Process Lifecycle

### Process Lifecycle Overview

![Image](https://images.openai.com/static-rsc-4/e90JVbh-YltnhsYoPQNYcBqLwVCbzv6qRBQlwysAPsdWPYf8joSQsXTwMpxcG3r8M5yvxUCVaNxbMzvQQxUjW6lO2V7bMLzdkUc3_Ax0LaZtebYg4quvQ3l7mYu_KrMAeK-pJc0sd7QiQWQE4ugbuwyIgJKHYwfjEThHb5CA-i1q4F3gSP15QOEYWZG75TF7?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/vJJsEREd_C5Q31DXErfv62fazjG4RKsg3EbjzNqHqrpK8PSSn2epWeNKFsfbeHOo-d3fMwqIM5lJz4FCkJs3iyB-jP_mTtDyHU3B_AmDwLW0anroyQvsmD5iad4HZES3Ws48zqy7tATqGzQa4d_4PRmuI8_Iw_tFHEfG4TPMR_y1P5Vetm6SQAH3pdNEwvHd?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/CjjQ4wUWxLgA4Ja5wTEiE0KwlbhnnBC0fvIVbIh1MvnMuo3S5hRzDcAp9Xq1luP8z-CdeMzHIyd15DUGmXUhs6JDKsRqWEm0dMIFEo-C7smOzi6tIx3ZSjB0MEq-BwvpJdN9lHHdf6nOkmLhgFF91tO657-7EaYfpQIxJl1H_RuQPKNwkSDlOElHpIibXarq?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/baRbiKOhZBFdY55EZlmEnhfGhAmtxa38MO7jLKGrJW_J-WnjvdqSZMOAPx5CfRIq79FFSAb0h6b91qRVE2GyesFR_mWA4E2L_cNQ7Hgte6WNb8oGC6n-ytI5-0pZz4MFTKiJmx7EF1cX3GwybZe8BgDrgoBXR_NQfpJPXyXyiZjFjHO4JOunDsRfkaKQsVIJ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/yoG6DOsezxizhxrouob_NvI_Ymomf92v1JPy7O0nzrgqXKYJF-ZayGO7Zay0mCjwczVkS4YAjkBvnIqDHDbJZWliMjYlIY06y7UEFlEn7V1cooOGdbPqLySEpe_twWX3BLeo8yVIuyZbkFAi0qi0DTxnhg69dXdmq62Ngr48LwBV-m6u7ZL8rUrQJh-clJVl?purpose=fullsize)

## Objective

Identify the processes currently running on a Linux system, understand their lifecycle, determine their process states, and monitor their resource usage.

---

# What is a Process?

A **process** is a **running instance of an executable program**.

For example:

* Running `firefox` creates a Firefox process.
* Running `vim` creates a Vim process.
* Running `ls` creates an `ls` process.

Each process contains:

* 🧠 Allocated memory (Address Space)
* 🔐 Security credentials (Owner, Permissions)
* 🧵 One or more execution threads
* 📊 Current process state

The process environment includes:

* Environment variables
* Local and global variables
* Open file descriptors
* Network ports
* Scheduling information
* Parent Process ID (PPID)

---

# Process Creation (fork() and exec())

Every new process is created from an existing process using the **fork()** system call.

```text
Parent Process
      │
   fork()
      │
 ┌────────────┐
 │ Child Copy │
 └────────────┘
      │
    exec()
      │
 New Program Runs
```

### Child Process Inherits

* Memory
* User credentials
* Open files
* Environment variables
* Permissions
* Current working directory

Usually, after `fork()`, the child executes a new program using **exec()**.

Example:

```bash
bash
   │
 fork()
   │
 exec(ls)
   │
 ls
```

---

# PID and PPID

Every process has:

| Term | Description       |
| ---- | ----------------- |
| PID  | Unique Process ID |
| PPID | Parent Process ID |

Example

```text
systemd (PID 1)
      │
    bash (PID 1200)
      │
     vim (PID 2400)
```

Here:

* **vim PID = 2400**
* **vim PPID = 1200**

---

# systemd (PID 1)

After Linux boots,

```text
systemd
```

becomes the very first userspace process.

```
PID = 1
```

Every other process is ultimately a child (directly or indirectly) of **systemd**.

Example:

```text
systemd
 ├── sshd
 ├── NetworkManager
 ├── cron
 └── bash
      └── vim
```

---

# Process Lifecycle

```text
Program
   │
fork()
   │
Child Process Created
   │
exec()
   │
Running
   │
Sleeping / Waiting
   │
Running
   │
Exit
   │
Zombie
   │
Parent calls wait()
   │
Process Removed
```

---

## Lifecycle Explanation

### Step 1 – Program Starts

A user executes a command.

Example:

```bash
ls
```

---

### Step 2 – fork()

The parent creates a child process.

```text
bash
   │
fork()
   │
Child Process
```

---

### Step 3 – exec()

The child replaces its code with a new executable.

```text
bash
   │
fork()
   │
Child
   │
exec()
   │
ls
```

---

### Step 4 – Parent Waits

The parent process usually sleeps while the child executes.

---

### Step 5 – Child Exits

Resources released:

* Memory
* File descriptors
* Network ports

Only the PID entry remains.

---

### Step 6 – Zombie Process

The process becomes a **Zombie (Z)**.

It has finished execution but is waiting for its parent to collect its exit status.

---

### Step 7 – Parent Reaps Child

The parent calls:

```c
wait()
```

The zombie entry is removed from the process table.

---

# Linux Process States

### Linux Process States

![Image](https://images.openai.com/static-rsc-4/4RTCDyyyByDcMBCgRNyqxbL8o5lkG80ypMWPw7mIyZ02fP4KOmbgJaOw-Yf_yBWVq2IiAOwRlEUjQXUns0rqlvMDwkXlpl-Q-nZKRwnYlj8sdsauwSzWdgaXnP4DjjZ9FiwTloiUclZDCm1S7X55JN5gh4Q0J2dNy-lJLYR23JstwWv_ebVEjUYtY9j1XH4a?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/_MsmOsC_rnTNFCJzX7-dcYFA-fdTQs1D4RK8Egomff7RW9AMpC6q-ZDPXjfWKOfLdfJQeXnktleUSjGtNpypMWVblYtlgE7Tx9xSGc-4ACIGjuHcEKLJlwkVat3v0q4K5XiaBhOGGyYvlUPfTpUisJVHZKpq1bntPxL3dvelE65Jz6nS5GDmafXu75aby3dw?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/l7e9626KAqq5rFnhsSjuv4cGTJ9cbRueEZsom6MjBWp8IxYAxdzD0QA3E7zzLS9z8Qhl_mNZxAR5_xQ_ps1cNA05MkAaD-0RhCmKYQAb3SMpOXECKg28G15TJqPWo8l7W4ehSggeNGwIATDt2idUijQvUrYGsF1qtKzlrp0AGcFLF4xtIa2euxotcYa59Ock?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/7hk_pAn9HrL11tCISK4T-ejNh5YHt3Cn7uhXeRydFTmD0ovKKZO5xWsZxhwyRhNOJRpX2jZMgMWul9AZ8ZXLlX8B9L2Ez9pk1MFDSNI8Zp1dpE_bk8hYCp_ziurnDBxfLuGAy1GggC6XftUFtpRQRmxvMboOY-vQSsHr1athLBh4jU_UsiK94Av-lAGmOWGO?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/J5Z7HK8uXpdjBZiXRHvNrKAbTba0DFOeQYO3jINExkN5GnBzSTUd67AIcsMYzP-OQaG1xwF0ae_2APY9l3QjwtADz1zfYcQS8jMAYM-avsRoU3k_Emzp1DWlpTzkMTcR7dDkslDlyEFdps5A-kE5eE9rdow2Jrxld4ob_a9vJ7FgKg0wRe7wT0kOErZDSznA?purpose=fullsize)

| State                 | Flag  | Description                          |
| --------------------- | ----- | ------------------------------------ |
| Running               | **R** | Executing or ready to run            |
| Sleeping              | **S** | Waiting for an event or signal       |
| Uninterruptible Sleep | **D** | Waiting for I/O, ignores signals     |
| Killable Sleep        | **K** | Similar to D but can be killed       |
| Idle                  | **I** | Idle kernel thread                   |
| Stopped               | **T** | Suspended by signal or user          |
| Traced                | **T** | Suspended for debugging              |
| Zombie                | **Z** | Finished, waiting for parent cleanup |
| Dead                  | **X** | Completely removed (not visible)     |

---

## 1. Running (R)

* Currently executing on CPU
* Ready to execute

Example:

```text
top
gcc
python
```

---

## 2. Sleeping (S)

Waiting for:

* Keyboard input
* Network packets
* Timer
* User input

Interruptible by signals.

Example:

```bash
cat
```

(waiting for keyboard input)

---

## 3. Uninterruptible Sleep (D)

Waiting for hardware resources such as:

* Disk I/O
* Storage devices
* NFS

Cannot respond to signals until the operation completes.

---

## 4. Killable Sleep (K)

Same as D state but allows termination using kill signals.

---

## 5. Idle (I)

Kernel background threads.

Examples:

```text
kworker
ksoftirqd
```

These do **not** affect the system load average.

---

## 6. Stopped (T)

Suspended using:

```bash
Ctrl+Z
```

or

```bash
kill -STOP PID
```

Resume using:

```bash
fg
```

or

```bash
bg
```

---

## 7. Traced (T)

Process temporarily stopped while being debugged.

Example:

```bash
gdb
```

---

## 8. Zombie (Z)

Characteristics:

* Execution completed
* Memory released
* PID entry still exists
* Waiting for parent process

Example:

```text
Parent
   │
Zombie Child
```

Too many zombie processes usually indicate that the parent process is not properly collecting child exit statuses.

---

## 9. Dead (X)

The parent has already cleaned up the zombie.

The process no longer exists and cannot be seen with `ps`.

---

# Useful Process Commands

| Command                    | Purpose                               |
| -------------------------- | ------------------------------------- |
| `ps -ef`                   | List all running processes            |
| `ps aux`                   | Detailed process information          |
| `ps -eo pid,ppid,stat,cmd` | Display PID, PPID, state, and command |
| `top`                      | Interactive process monitor           |
| `htop`                     | Enhanced process viewer               |
| `pstree`                   | Display parent-child process tree     |
| `pidof process_name`       | Find the PID of a process             |
| `pgrep process_name`       | Search processes by name              |

Example:

```bash
ps -eo pid,ppid,stat,cmd
```

Output:

```text
PID   PPID  STAT  CMD
1       0    Ss    systemd
845     1    S     sshd
2145   845   R     top
3500  2145   Z     zombie
```

---

# RHCSA Exam Tips

✅ A **process** is a running instance of a program.

✅ Every process has a unique **PID** and a **PPID**.

✅ All user-space processes ultimately originate from **systemd (PID 1)**.

✅ Processes are created using **fork()** and typically execute a new program using **exec()**.

✅ A **Zombie (Z)** process has finished execution but is waiting for its parent to collect its exit status.

✅ **Sleeping (S)** processes can be interrupted by signals, whereas **Uninterruptible (D)** processes generally cannot until the I/O operation completes.

✅ Use `ps`, `top`, `htop`, and `pstree` to monitor and troubleshoot processes in Linux.

