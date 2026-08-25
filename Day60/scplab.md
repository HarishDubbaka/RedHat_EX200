Exactly. This is the **SCP part of the same exercise**. The main difference from the previous SFTP task is that **SCP performs the transfer noninteractively**, meaning you don't enter an `sftp>` session.

# SCP Exercise — Copy `/etc/ssh` from serverb → servera

The goal is:

```text
serverb                              servera
--------                             --------
/etc/ssh/                            /home/student/
    │                                serverbackup2/
    │                                    │
    │        scp -r                     │
    └──────────────────────────────────►│
```

You are currently on **servera** as the `student` user.

---

## 1. Copy the `/etc/ssh` directory using SCP

Run:

```bash
scp -r root@serverb:/etc/ssh ~/serverbackup2
```

You will be prompted for the `root` password:

```text
root@serverb's password:
```

Enter:

```text
redhat
```

---

## 2. Understand the SCP command

Let's break this command down:

```bash
scp -r root@serverb:/etc/ssh ~/serverbackup2
```

### `scp`

```text
scp
 ↓
Secure Copy
```

It copies files/directories between systems over SSH.

### `-r`

```text
-r
 ↓
recursive
```

This is required because `/etc/ssh` is a **directory** containing files and subdirectories.

Without `-r`, SCP cannot recursively copy the directory.

### `root@serverb:/etc/ssh`

This is the **source**.

Break it down:

```text
root
 ↓
Remote username

@
 ↓

serverb
 ↓
Remote hostname

:
 ↓

/etc/ssh
 ↓
Remote source directory
```

So:

```text
root@serverb:/etc/ssh
```

means:

> Copy `/etc/ssh` from `serverb`, connecting as `root`.

### `~/serverbackup2`

This is the **destination**.

Because you're logged in as:

```text
student@servera
```

`~` means:

```text
/home/student
```

Therefore:

```text
~/serverbackup2
```

means:

```text
/home/student/serverbackup2
```

---

# 3. Understand the Direction

This is extremely important.

```text
REMOTE                         LOCAL
serverb                        servera
--------                       --------
/etc/ssh
    │
    │
    │  scp -r
    └───────────────────────►
                               /home/student/serverbackup2/
```

So the command:

```bash
scp -r root@serverb:/etc/ssh ~/serverbackup2
```

means:

**REMOTE → LOCAL**

---

# 4. Why `root@serverb`?

The source directory is:

```text
/etc/ssh
```

The exercise specifies that `root` should be used because the complete contents need to be readable.

Therefore:

```bash
root@serverb
```

is used instead of:

```bash
student@serverb
```

---

# 5. Verify the Copy

After SCP finishes, run:

```bash
ls -lR ~/serverbackup2
```

The `-R` means:

```text
Recursive
```

So Linux will display:

```text
serverbackup2/
└── ssh/
    ├── moduli
    ├── ssh_config
    ├── sshd_config
    ├── ssh_host_rsa_key
    ├── ssh_host_rsa_key.pub
    ├── ssh_host_ecdsa_key
    ├── ssh_host_ecdsa_key.pub
    ├── ssh_host_ed25519_key
    ├── ssh_host_ed25519_key.pub
    ├── ssh_config.d/
    │   ├── 01-training.conf
    │   └── 50-redhat.conf
    └── sshd_config.d/
        ├── 40-redhat-crypto-policies.conf
        ├── 50-cloud-init.conf
        └── 50-redhat.conf
```

The important thing is that you have:

```text
/home/student/serverbackup2/ssh
```

with the contents of the original:

```text
serverb:/etc/ssh
```

---

# 6. Why is the directory called `ssh`?

You specified:

```bash
root@serverb:/etc/ssh
```

as the source and:

```bash
~/serverbackup2
```

as the destination.

SCP therefore places the source directory inside the destination:

```text
/home/student/serverbackup2/ssh
```

So:

```text
Source:
serverb:/etc/ssh

Destination:
servera:/home/student/serverbackup2/ssh
```

---

# 7. Compare SFTP and SCP

You have now performed the **same transfer using two different methods**.

### SFTP

You first opened a session:

```bash
sftp root@serverb
```

Then:

```bash
lcd /home/student/serverbackup1
get -r /etc/ssh
```

So SFTP is **interactive**.

```text
sftp>
```

---

### SCP

You perform everything in one command:

```bash
scp -r root@serverb:/etc/ssh ~/serverbackup2
```

So SCP is **noninteractive**.

```text
scp command
     ↓
Transfer
     ↓
Done
```

### Easy comparison

| SFTP                                     | SCP                             |
| ---------------------------------------- | ------------------------------- |
| Interactive                              | Noninteractive                  |
| `sftp root@serverb`                      | `scp ...`                       |
| `get -r`                                 | `scp -r`                        |
| Uses `lcd` for local destination         | Destination directly in command |
| Good for multiple interactive operations | Good for quick copying          |

---

# 8. Final Step — Return to Workstation

After verifying the backup, you're still on:

```text
[student@servera ~]$
```

Run:

```bash
exit
```

You should see:

```text
logout
Connection to servera closed.
student@workstation:~$
```

Now you're back on the **workstation**.

---

# 🔥 Complete Exercise Commands

If you want to remember the entire flow:

```bash
# From workstation
ssh student@servera

# Create backup directories
mkdir ~/serverbackup{1,2}

# SFTP transfer
sftp root@serverb

# Inside SFTP
lcd /home/student/serverbackup1
get -r /etc/ssh
exit

# Verify SFTP backup
ls -lR ~/serverbackup1

# SCP transfer
scp -r root@serverb:/etc/ssh ~/serverbackup2

# Verify SCP backup
ls -lR ~/serverbackup2

# Return to workstation
exit
```

## 🧠 RHCSA Memory Trick

```text
SFTP:
sftp user@server
        ↓
     get -r
        ↓
REMOTE → LOCAL


SCP:
scp -r user@server:/source /destination
                    ↓
              REMOTE → LOCAL
```

The most important command from this exercise is:

```bash
scp -r root@serverb:/etc/ssh ~/serverbackup2
```

Remember:

**`scp -r SOURCE DESTINATION`**

Here:

```text
SOURCE      = root@serverb:/etc/ssh
DESTINATION = ~/serverbackup2
```

So the direction is:

**serverb → servera**.
