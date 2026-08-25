# Transferring Files Between Systems

Till now, we have seen how to check logs and identify errors, and we have also discussed how to archive logs. Today, we are going to discuss another important topic: how to transfer files from one machine to another machine.

You can then introduce the topic with:

### File Transfer Between Linux Machines

In Linux, there are several ways to transfer files from one machine to another. The most commonly used methods are:

* **SCP (Secure Copy Protocol)** – Used to securely copy files between machines over SSH.
* **SFTP (SSH File Transfer Protocol)** – Used for secure file transfer and file management.
* **rsync** – Used to efficiently synchronize files and directories between machines.
* **FTP** – Used for file transfer, but it is not encrypted by default.
* **NFS** – Used when we want to share a directory between multiple Linux machines.



## 1. Big Picture

Suppose you have two Linux servers:

```text
Local Server                         Remote Server
-----------                          -------------
server1                              server2
user@server1                         remoteuser@server2

        ─────── File Transfer ───────>
```

You want to:

* Upload a file → Local → Remote
* Download a file → Remote → Local
* Copy directories
* Transfer files securely

SSH provides the secure communication channel.

### Main tools

| Command | Purpose                   | Interactive? |
| ------- | ------------------------- | ------------ |
| `sftp`  | Secure file transfer      | Yes          |
| `scp`   | Secure file copy          | No           |
| `ssh`   | Execute commands remotely | Yes/No       |

---

# 2. SFTP — Secure File Transfer Protocol

`sftp` is part of the **OpenSSH** suite.

It provides:

* Secure authentication
* Encrypted file transfer
* Interactive file management
* Upload and download capability

Basic syntax:

```bash
sftp user@remotehost
```

Example:

```bash
sftp remoteuser@remotehost
```

You will get:

```text
remoteuser@remotehost's password:
Connected to remotehost.
sftp>
```

Now you are inside the **SFTP interactive shell**.

---

# 3. Local vs Remote — Very Important

This is one of the most important concepts for the exam.

When you are inside:

```text
sftp>
```

there are actually **two systems** involved.

```text
              SFTP Session
                  |
        +---------+---------+
        |                   |
     LOCAL                REMOTE
   Your server          SSH server
```

For example:

```bash
sftp> pwd
```

shows the **remote** directory.

```text
Remote working directory: /home/remoteuser
```

But:

```bash
sftp> lpwd
```

shows the **local** directory.

```text
Local working directory: /home/user
```

### Easy way to remember

**`pwd` = remote**

**`lpwd` = local**

The `l` means **local**.

---

# 4. Important SFTP Commands

## Remote directory commands

These commands operate on the remote server:

```bash
pwd
ls
cd
mkdir
rmdir
```

Example:

```bash
sftp> pwd
Remote working directory: /home/remoteuser

sftp> ls
file1
file2
backup

sftp> cd backup
```

---

# 5. Uploading Files — `put`

If you want to transfer:

```text
LOCAL → REMOTE
```

use:

```bash
put
```

For example:

```bash
sftp> put /etc/hosts
```

This uploads:

```text
Local:
/etc/hosts

        ↓

Remote:
/home/remoteuser/hosts
```

You might see:

```text
Uploading /etc/hosts to /home/remoteuser/hosts
/etc/hosts 100% 227 0.2KB/s
```

### Remember

```text
put = Upload
```

---

# 6. Uploading a Directory — `put -r`

Suppose your local machine has:

```text
/home/user/directory/
├── file1
├── file2
└── file3
```

To upload the entire directory:

```bash
sftp> put -r directory
```

The `-r` means:

> **recursive**

It copies the directory and everything inside it.

```text
Local                         Remote

directory/        ───────>    directory/
 ├── file1                     ├── file1
 ├── file2                     ├── file2
 └── file3                     └── file3
```

---

# 7. Downloading Files — `get`

If you want:

```text
REMOTE → LOCAL
```

use:

```bash
get
```

Example:

```bash
sftp> get /etc/dnf/dnf.conf
```

This means:

```text
Remote:
/etc/dnf/dnf.conf

        ↓

Local:
./dnf.conf
```

### Remember

```text
get = Download
```

So:

```text
put → Local → Remote
get → Remote → Local
```

This is extremely important for RHCSA.

---

# 8. SFTP Command Cheat Sheet

| Command      | Meaning                 |
| ------------ | ----------------------- |
| `pwd`        | Show remote directory   |
| `lpwd`       | Show local directory    |
| `ls`         | List remote files       |
| `lls`        | List local files        |
| `cd`         | Change remote directory |
| `lcd`        | Change local directory  |
| `mkdir`      | Create remote directory |
| `rmdir`      | Remove remote directory |
| `put file`   | Upload file             |
| `put -r dir` | Upload directory        |
| `get file`   | Download file           |
| `exit`       | Exit SFTP               |
| `help`       | Show SFTP commands      |

### Important pattern

Commands beginning with **`l`** generally operate on the **local system**.

```text
pwd     → remote
lpwd    → local

ls      → remote
lls     → local

cd      → remote
lcd     → local
```

---

# 9. SFTP Example — Complete Scenario

Suppose:

```text
Local server:
server1

Remote server:
server2

Remote user:
student
```

Connect:

```bash
sftp student@server2
```

Then:

```bash
sftp> pwd
```

Check remote directory.

```bash
sftp> lpwd
```

Check local directory.

Create a directory remotely:

```bash
sftp> mkdir backup
```

Move into it:

```bash
sftp> cd backup
```

Upload a file:

```bash
sftp> put /etc/hosts
```

Download a file:

```bash
sftp> get /etc/hostname
```

Exit:

```bash
sftp> exit
```

---

# 10. One-Line SFTP Download

You can download a remote file without entering the interactive SFTP shell.

Example:

```bash
sftp remoteuser@remotehost:/home/remoteuser/remotefile
```

This directly fetches the file.

Conceptually:

```text
Remote file
     ↓
sftp
     ↓
Local directory
```

### Important limitation from your material

The single-command form is useful for **getting/downloading** a file.

For uploading, use an interactive SFTP session with `put`.

---

# 11. SCP — Secure Copy

The second important command is:

```bash
scp
```

`scp` is used to copy files between systems.

Unlike SFTP, you normally don't enter an interactive shell.

For example:

```bash
scp file.txt remoteuser@remotehost:/home/remoteuser
```

This means:

```text
Local file
file.txt
   |
   | scp
   ↓
Remote server
/home/remoteuser/file.txt
```

---

# 12. SCP Upload

Syntax:

```bash
scp localfile user@remotehost:/remote/path
```

Example:

```bash
scp /etc/hosts remoteuser@remotehost:/home/remoteuser
```

Meaning:

```text
Local:
/etc/hosts

       ↓ scp

Remote:
/home/remoteuser/hosts
```

---

# 13. SCP Download

You can also copy from remote → local.

Syntax:

```bash
scp user@remotehost:/remote/file /local/path
```

Example:

```bash
scp remoteuser@remotehost:/etc/hostname /home/user
```

Meaning:

```text
Remote:
/etc/hostname

       ↓ scp

Local:
/home/user/hostname
```

---

# 14. SCP Multiple Files

You can copy multiple files at once.

Example:

```bash
scp /etc/dnf/dnf.conf /etc/hosts remoteuser@remotehost:/home/remoteuser
```

This copies:

```text
/etc/dnf/dnf.conf
/etc/hosts
```

to:

```text
/home/remoteuser/
```

on the remote system.

---

# 15. SCP and Directories

To copy directories recursively with `scp`, use:

```bash
scp -r directory remoteuser@remotehost:/home/remoteuser
```

Here:

```text
-r = recursive
```

Example:

```bash
scp -r /home/user/project remoteuser@server2:/home/remoteuser
```

---

# 16. SFTP vs SCP

This is a very important comparison.

| Feature                | SFTP                  | SCP           |
| ---------------------- | --------------------- | ------------- |
| Command                | `sftp`                | `scp`         |
| Interactive            | Yes                   | Usually no    |
| Upload                 | `put`                 | `scp`         |
| Download               | `get`                 | `scp`         |
| Directory transfer     | `put -r`              | `scp -r`      |
| Uses SSH security      | Yes                   | Yes           |
| Remote file management | Excellent             | Limited       |
| Best for               | Interactive transfers | Quick copying |

### Simple way to remember

**SFTP = File Transfer Session**

```bash
sftp user@server
```

Then:

```bash
put file
get file
```

**SCP = Secure Copy**

```bash
scp file user@server:/path
```

---

# 17. RHEL 10 — Important SCP Change

This part of your chapter is particularly important.

Starting with **RHEL 10**, the `scp` command uses **SFTP for transferring files**.

So conceptually:

```text
scp command
     ↓
SFTP transfer mechanism
     ↓
SSH
     ↓
Remote server
```

This is different from the old/legacy SCP protocol.

---

# 18. Legacy SCP — `-O`

The old SCP protocol can still be requested with:

```bash
scp -O
```

But Red Hat recommends **not using the legacy SCP protocol** because it has a known code-injection vulnerability.

Therefore:

```bash
scp file user@server:/path
```

is preferred.

Avoid:

```bash
scp -O file user@server:/path
```

unless you specifically need the legacy protocol for compatibility.

---

# 19. `/etc/ssh/disable_scp`

RHEL 10 can disable the SCP protocol.

If this file exists:

```bash
/etc/ssh/disable_scp
```

attempts to use the SCP protocol can fail.

However:

```text
SFTP still works
```

So if SCP has an issue, you can use:

```bash
sftp user@server
```

instead.

---

# 20. Remote Location Syntax

Both SFTP and SCP commonly use:

```text
user@host:path
```

For example:

```bash
remoteuser@server2:/home/remoteuser/file.txt
```

Break it down:

```text
remoteuser
     ↓
Username

@
     ↓

server2
     ↓
Hostname

:
     ↓

/home/remoteuser/file.txt
     ↓
Remote path
```

So:

```text
user@host:path
```

is a very important format.

---

# 21. What If Username Is Not Provided?

Suppose you run:

```bash
sftp server2
```

You did not specify:

```text
user@
```

The command uses the **current local username** to connect to the remote server.

For example, if you are logged in as:

```text
harish
```

then:

```bash
sftp server2
```

attempts to connect as:

```text
harish@server2
```

Similarly:

```bash
scp file.txt server2:/tmp/
```

uses your current local username as the remote username.

---

# 22. SFTP vs SCP — Direction

This is probably the easiest way to remember everything.

### SFTP

```text
             SFTP
              |
       +------+------+
       |             |
     PUT            GET
       |             |
       ↓             ↓
   LOCAL → REMOTE  REMOTE → LOCAL
```

### SCP

The direction is determined by where you put the source and destination.

```bash
scp localfile user@server:/path
```

means:

```text
LOCAL → REMOTE
```

While:

```bash
scp user@server:/path/file /local/path
```

means:

```text
REMOTE → LOCAL
```

---

# 23. Common RHCSA Exam Commands

### Connect using SFTP

```bash
sftp user@server
```

### Upload a file

```bash
sftp> put file.txt
```

### Download a file

```bash
sftp> get file.txt
```

### Upload directory

```bash
sftp> put -r directory
```

### Check remote directory

```bash
sftp> pwd
```

### Check local directory

```bash
sftp> lpwd
```

### Upload using SCP

```bash
scp file.txt user@server:/tmp/
```

### Download using SCP

```bash
scp user@server:/tmp/file.txt .
```

### Copy directory using SCP

```bash
scp -r directory user@server:/tmp/
```

---

# 24. Easy Memory Trick

Remember these four commands:

```text
PUT  → Push file → Local → Remote
GET  → Get file  → Remote → Local

SFTP → Interactive
SCP  → Direct copy
```

Or simply:

```text
              FILE TRANSFER
                    |
          +---------+---------+
          |                   |
        SFTP                 SCP
          |                   |
   Interactive             Direct
          |
      +---+---+
      |       |
     put     get
      |       |
      ↓       ↓
    Upload  Download
```

---

# 25. Real-World Example

Suppose you are administering SAP servers:

```text
SAP Server 1
10.10.10.10

SAP Server 2
10.10.10.20
```

You need to transfer:

```text
backup.tar.gz
```

from Server 1 to Server 2.

Using SCP:

```bash
scp backup.tar.gz root@10.10.10.20:/backup/
```

Or using SFTP:

```bash
sftp root@10.10.10.20
```

Then:

```bash
sftp> cd /backup
sftp> put backup.tar.gz
```

For a directory:

```bash
scp -r /sapbackup root@10.10.10.20:/backup/
```

---

# 26. Exam-Focused Summary

For **RHCSA**, remember these points:

### SFTP

```bash
sftp user@server
```

Interactive session.

```bash
put file
```

**Upload**

```bash
get file
```

**Download**

```bash
put -r directory
```

**Upload directory**

```bash
pwd
```

**Remote working directory**

```bash
lpwd
```

**Local working directory**

---

### SCP

```bash
scp file user@server:/path
```

**Local → Remote**

```bash
scp user@server:/path/file .
```

**Remote → Local**

```bash
scp -r directory user@server:/path
```

**Directory → Remote**

---

### Most important concepts

```text
sftp = interactive secure file transfer

scp = secure copy

put = upload

get = download

-r = recursive

pwd = remote directory

lpwd = local directory

user@host:/path = remote location
```

And for **RHEL 10**:

> `scp` uses SFTP as its transfer mechanism by default; the legacy SCP protocol is discouraged, and `-O` requests the legacy protocol.
