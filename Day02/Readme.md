# 🐧 Linux CLI & Remote Access Basics (System Administrator Guide)

As a System Administrator, you can access Linux servers in different environments:

- 🏢 **On-premise server** (within your data center)
- ☁️ **Cloud VM** (Azure, AWS, GCP)

Understanding the **Command Line Interface (CLI)** is essential before logging into any Linux system.

---

# 🖥️ Basics of Command Line (Text Interface)

A **Command Line Interface (CLI)** is a text-based interface where you interact with the system by typing commands instead of using a graphical interface (mouse & windows).

---

# 1. 📟 What is a Terminal?

A **terminal** is an application that provides access to the CLI.

- It acts as a bridge between the user and the shell
- When opened, it starts a shell session

### ✅ Examples:
- **Linux**: Terminal (default)
- **Windows**:
  - Command Prompt
  - PowerShell
  - Git Bash

---

# 2. 🐚 What is a Shell?

The **shell** is a program that:

- Takes user commands
- Sends them to the Operating System
- Displays the output

### ✅ Common Shells:
- `bash` (most widely used ✅)
- `sh`
- `zsh`

---

## 🔹 Default Shell in RHEL

The default shell in **Red Hat Enterprise Linux (RHEL)** is:

> **GNU Bourne-Again Shell (Bash)**

Bash is an enhanced version of the original UNIX `sh` shell.

---

# 💡 Shell Prompt

The shell shows a **prompt** indicating it is ready to accept commands.

### 👤 Regular User:
```bash
user@host:~$
````

* Ends with `$`

### 👑 Root User (Superuser):

```bash
root@host:~#
```

* Ends with `#`
* Has full administrative privileges

***

# 🌐 Logging into a Remote System (SSH)

In modern environments, servers are often:

* Headless (no GUI)
* Cloud-based virtual machines

👉 The most common way to access them is:

## 🔐 SSH (Secure Shell)

### ✅ Example:

```bash
user@host:~$ ssh remoteuser@remotehost
remoteuser@remotehost's password:
remoteuser@remotehost:~$
```

* Encrypts communication ✅
* Protects passwords and data ✅

***

## 🔑 SSH Key-Based Authentication (Password-less Login)

For better security:

* Uses **public/private key pair**
* No password required

### 🔸 How it works:

* Private key → stored on your machine (keep it secret 🔐)
* Public key → stored on server

### ✅ Login using key:

```bash
ssh -i private_key.pem remoteuser@remotehost
```

***

# 🖥️ Performing Tasks in Terminal

In Linux, you can perform all system tasks using the shell:

* File management
* System monitoring
* User management
* Process control

Even without a GUI, you can fully manage the system.

***

# ⚙️ Running Commands

Commands typically follow this structure:

```bash
command [options] [arguments]
```

***

## 🧾 Example: `hostname`

### Default:

```bash
hostname
```

Output:

```
mycomputer.example.com
```

### With Options:

```bash
hostname -d   # domain name
hostname -s   # short hostname
```

***

# 📌 Commands with Options & Arguments

## ✅ Example: `id`

### Default (current user):

```bash
id
```

Output:

```
uid=1000(user) gid=1000(user) groups=1000(user),10(wheel)
```

### Option only:

```bash
id -u
```

Output:

```
1000
```

### Option + Argument:

```bash
id -u devops
```

Output:

```
1001
```

***

# 📖 Getting Help for Commands

Use:

```bash
command --help
```

### Example:

```bash
who --help
```

***

# 👥 Checking Logged-In Users

```bash
who
```

Shows users currently logged into the system.

***

# 🕒 Date & Time Commands

## ✅ `date`

```bash
date
```

## ✅ `timedatectl`

Provides:

* Time zone
* NTP sync status
* System clock details

***

# 🔐 Managing Passwords

## ✅ Change current user's password:

```bash
passwd
```

## ✅ Root changing another user's password:

```bash
passwd username
```

***

# 🚪 Logging Out of a Session

### Method 1:

```bash
exit
```

### Method 2:

Press:

```
Ctrl + D
```

### Example:

```bash
remoteuser@remotehost:~$ exit
logout
Connection to remotehost closed.
user@host:~$
```

***

💡 *Mastering CLI is the foundation for becoming a strong Linux/System Administrator.*

```
```
