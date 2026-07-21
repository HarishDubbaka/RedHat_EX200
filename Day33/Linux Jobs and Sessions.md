# 🐧 Linux Jobs and Sessions

## 🎯 Objective

Understand how Linux manages multiple commands in a terminal using **jobs**, **process groups**, and **sessions**.

---

# 1. What is a Job?

A **job** is a command or group of commands started from the shell.

Whenever you execute a command, Bash creates a **job**.

Example:

```bash
sleep 100
```

This creates:

* One Job
* One Process

Example:

```bash
ls | sort | wc -l
```

This creates:

* One Job
* Three Processes

```
Job
 │
 ├── ls
 ├── sort
 └── wc -l
```

Even though three commands are running, Bash treats them as **one job** because they belong to the same pipeline.

---

# 2. What is a Pipeline?

A pipeline is commands connected using the pipe (`|`) operator.

Example

```bash
cat file.txt | grep error | wc -l
```

Here:

```
cat
  │
  ▼
grep
  │
  ▼
wc
```

Each command is a separate process.

Together they form

* One Job
* One Process Group

---

# 3. What is a Process Group?

Linux groups related processes together.

Example

```bash
ls | sort | less
```

```
Process Group
│
├── ls
├── sort
└── less
```

All these processes have the **same PGID (Process Group ID)**.

Why?

Because if you press

```
Ctrl+C
```

Linux terminates the **entire process group**, not just one command.

---

# 4. What is a Session?

A **session** is a collection of one or more process groups attached to a terminal.

Every terminal window starts its own session.

Example

```
Terminal 1
    │
    ▼
 Bash (Session Leader)
    │
 ├───────────────┐
 │               │
 Job1          Job2
```

Open another terminal:

```
Terminal 2
    │
    ▼
 Bash (Another Session Leader)
```

Each terminal has its own session.

---

# 5. Session Leader

The first process started in a session is called the **Session Leader**.

Usually this is

```
bash
```

Example

```
Terminal

     bash
       │
       ├── sleep
       ├── vim
       └── top
```

Bash controls all jobs started from that terminal.

---

# 6. Foreground Process

Only **one job** can run in the foreground.

The foreground process

* Reads keyboard input
* Receives Ctrl+C
* Receives Ctrl+Z

Example

```bash
vim file.txt
```

Now Vim is the foreground process.

Keyboard input goes directly to Vim.

```
Keyboard
    │
    ▼
  Terminal
    │
    ▼
   Vim
```

---

# 7. Background Process

A background process runs without occupying the terminal.

Example

```bash
sleep 1000 &
```

Output

```
[1] 2345
```

Now

```
Terminal

bash
 │
 ├── sleep 1000 (Background)
```

The shell prompt returns immediately.

You can continue typing commands.

---

# Why can't Background Jobs Read Keyboard Input?

Suppose

```
vim &
```

and

```
nano &
```

Both try reading your keyboard.

Which editor should receive your keystrokes?

Linux avoids this confusion.

Therefore

Only the foreground process can receive keyboard input.

Background processes cannot.

---

# What Happens if a Background Job Tries to Read Input?

Example

```bash
cat &
```

The `cat` command waits for keyboard input.

But it's running in the background.

Linux immediately suspends it.

State becomes

```
Stopped
```

The kernel prevents background jobs from reading your keyboard.

---

# 8. Controlling Terminal

Every interactive shell has a **controlling terminal**.

Example

```
pts/0
```

View it using:

```bash
ps
```

Output

```
PID   TTY      CMD

2328  pts/0    bash
8910  pts/0    ps
```

TTY means

```
Terminal
```

All these processes belong to

```
pts/0
```

---

# 9. System Daemons

Some processes are not started from terminals.

Example

```
systemd
sshd
NetworkManager
cron
```

These have

```
TTY ?
```

Example

```
PID TTY CMD

1   ?   systemd
```

Why?

Because they were started during boot.

No terminal exists.

They cannot become foreground jobs.

---

# 10. Understanding `ps j`

Run

```bash
ps j
```

Example

```
PPID PID PGID SID TTY TPGID STAT COMMAND

8528 8559 8559 8528 pts/0 8633 T sleep
```

Let's understand every column.

---

## PID

Process ID

Every process gets a unique number.

Example

```
sleep

PID = 8559
```

---

## PPID

Parent Process ID

Who created this process?

```
bash
   │
fork()
   │
sleep
```

Example

```
bash PID = 8528

sleep PPID = 8528
```

Meaning

```
bash created sleep.
```

---

## PGID

Process Group ID

Processes in the same job share one PGID.

Example

```
ls | grep txt | sort
```

```
PGID = 9200

ls
grep
sort
```

All belong to one job.

---

## SID

Session ID

The session leader's PID.

Usually

```
bash
```

Example

```
bash PID = 8528

SID = 8528
```

Every process started from this shell belongs to the same session.

---

## TTY

Terminal

Example

```
pts/0
```

Means

```
Pseudo Terminal 0
```

---

## TPGID

Terminal Process Group ID

Which process group currently owns the terminal?

If

```
vim
```

is running,

its PGID becomes

```
TPGID
```

---

## STAT

Process state.

Examples

```
R
Running
```

```
S
Sleeping
```

```
T
Stopped
```

```
Z
Zombie
```

```
R+
```

Running in foreground.

```
Ss
```

Sleeping and session leader.

---

# 11. Running a Background Job

Command

```bash
sleep 1000 &
```

Output

```
[1] 5947
```

Meaning

```
[1]
Job Number

5947
Process ID
```

Shell immediately returns

```
$
```

because the job runs in the background.

---

# 12. View Jobs

```bash
jobs
```

Output

```
[1]+ Running sleep 1000 &
```

The shell tracks all background jobs.

---

# 13. Bring Job to Foreground

```bash
fg %1
```

Output

```
sleep 1000
```

Now

```
sleep
```

owns the terminal.

---

# 14. Suspend a Foreground Job

Run

```bash
sleep 1000
```

Press

```
Ctrl+Z
```

Output

```
Stopped
```

The process still exists.

It is simply paused.

State

```
T
```

---

# 15. Resume the Job

Run

```bash
bg %1
```

Output

```
Running
```

The job resumes in the background.

---

# 16. Job Number Symbols

Example

```bash
jobs
```

Output

```
[1]+ Running

[2]- Stopped
```

Meaning

| Symbol | Meaning               |
| ------ | --------------------- |
| **+**  | Current (default) job |
| **-**  | Previous job          |

If you type

```bash
fg
```

Linux automatically selects

```
+
```

---

# 17. Complete Workflow

```
                 Start Command
                       │
                       ▼
               Foreground Process
                       │
            Ctrl + Z (Suspend)
                       │
                       ▼
              Stopped Process
                       │
                  bg %1
                       │
                       ▼
             Background Process
                       │
                  fg %1
                       │
                       ▼
              Foreground Again
                       │
                Ctrl + C
                       │
                       ▼
               Process Terminated
```

---

# 📝 RHCSA EX200 Key Points

* **Job:** A command or pipeline started from the shell.
* **Pipeline:** Multiple commands connected with `|`, treated as one job.
* **Process Group (PGID):** Groups related processes in the same job.
* **Session (SID):** A collection of jobs managed by one shell; each terminal has its own session.
* **Foreground:** Only one job at a time; receives keyboard input and signals like `Ctrl+C` and `Ctrl+Z`.
* **Background:** Multiple jobs allowed; cannot read terminal input.
* **TTY:** Shows the controlling terminal (for example, `pts/0`).
* **System daemons:** Show `TTY ?` because they are not attached to a terminal.
* **Essential commands:** `jobs`, `fg`, `bg`, `ps`, and `ps j` are fundamental for managing jobs and understanding process relationships.
