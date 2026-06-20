# 📂 Navigating the Linux File-System Hierarchy

## 📖 Overview

Linux stores all files on file systems that are organized into a single inverted tree structure known as the **File-System Hierarchy**.

The hierarchy starts from the **root directory (`/`)**, which sits at the top of the tree. All other directories and files branch out beneath it.

Understanding the Linux file-system hierarchy is essential for Linux administrators because it helps locate system files, configuration files, logs, applications, and user data efficiently.

---

## 🌳 The Root Directory (`/`)

The **root directory** is represented by a forward slash:

```bash
/
```

It is the highest-level directory in Linux.

The forward slash (`/`) is also used as a directory separator.

### Examples

```bash
/etc
```

* `etc` is a subdirectory under the root directory.

```bash
/etc/issue
```

* `issue` is a file located inside the `/etc` directory.

---

## 📝 File-System Content Types

Linux directories can contain different types of data:

| Type                   | Description                                                          |
| ---------------------- | -------------------------------------------------------------------- |
| **Static Content**     | Remains unchanged until manually modified or reconfigured.           |
| **Dynamic Content**    | Frequently changes due to system activity or running applications.   |
| **Persistent Content** | Remains available after a system reboot.                             |
| **Runtime Content**    | Exists only while the system is running and is removed after reboot. |

---

# 📁 Important Linux Directories

## `/boot`

Contains files required to start the Linux operating system.

Examples:

```bash
/boot/vmlinuz
/boot/grub2/
```

### Purpose

* Kernel files
* Boot loader files
* Initial RAM disk images

---

## `/dev`

Contains special device files that represent hardware devices.

Examples:

```bash
/dev/sda
/dev/null
/dev/tty
```

### Purpose

* Access storage devices
* Access terminals
* Access hardware resources

---

## `/etc`

Stores system-wide configuration files.

Examples:

```bash
/etc/passwd
/etc/hosts
/etc/ssh/
```

### Purpose

* System configuration
* Service configuration
* Network settings

---

## `/home`

Default location for regular user home directories.

Example:

```bash
/home/harish
```

### Purpose

* User documents
* Personal files
* User-specific settings

---

## `/root`

Home directory for the **root user** (system administrator).

Example:

```bash
/root
```

### Purpose

* Root user's files
* Administrative scripts
* System maintenance tasks

---

## `/run`

Contains runtime information generated since the last system boot.

Examples:

```bash
/var/run
/run/systemd
```

### Purpose

* Process ID (PID) files
* Lock files
* Runtime service information

### Important Notes

* Recreated every reboot
* Non-persistent storage
* Replaces older `/var/run` and `/var/lock`

---

## `/tmp`

Temporary storage for applications and users.

Example:

```bash
/tmp
```

### Purpose

* Temporary files
* Installation files
* Session data

### Important Notes

* World-writable directory
* Files not accessed for 10 days are automatically removed

---

## `/var/tmp`

Another temporary directory for longer-lived temporary files.

### Important Notes

* Files persist longer than `/tmp`
* Files older than 30 days may be automatically removed

---

## `/usr`

Contains installed software, shared libraries, documentation, and program data.

### Important Subdirectories

#### `/usr/bin`

User commands and executable programs.

Examples:

```bash
/usr/bin/ls
/usr/bin/cat
/usr/bin/cp
```

---

#### `/usr/sbin`

System administration commands.

Examples:

```bash
/usr/sbin/useradd
/usr/sbin/reboot
/usr/sbin/fsck
```

---

#### `/usr/local`

Locally installed or customized software.

Examples:

```bash
/usr/local/bin
/usr/local/lib
```

### Purpose

* Third-party applications
* Custom scripts
* Locally managed software

---

## `/var`

Stores variable data that changes during normal system operation.

Examples:

```bash
/var/log
/var/cache
/var/spool
```

### Purpose

* Log files
* Application cache
* Databases
* Mail queues
* Print jobs
* Web content

### Characteristics

* Dynamic content
* Persistent across reboots

---

# 🗂️ Quick Directory Reference

| Directory | Purpose                               |
| --------- | ------------------------------------- |
| `/`       | Root of the file-system hierarchy     |
| `/boot`   | Boot files and kernel                 |
| `/dev`    | Device files                          |
| `/etc`    | System configuration                  |
| `/home`   | User home directories                 |
| `/root`   | Root user's home directory            |
| `/run`    | Runtime process data                  |
| `/tmp`    | Temporary files                       |
| `/usr`    | Installed software and libraries      |
| `/var`    | Logs, cache, databases, variable data |

---

# 🎯 Key Takeaways

✅ Linux uses a single hierarchical directory structure.

✅ The root directory (`/`) is the starting point of the entire file system.

✅ Configuration files are typically stored in `/etc`.

✅ User data is stored under `/home`.

✅ Runtime information is stored in `/run`.

✅ Temporary files are stored in `/tmp` and `/var/tmp`.

✅ Logs, databases, and dynamic application data are stored in `/var`.

✅ Installed software and system commands are primarily located under `/usr`.

---

## 📚 Summary

The Linux File-System Hierarchy Standard (FHS) organizes files and directories into a predictable structure. Understanding the purpose of each major directory is a fundamental skill for Linux System Administrators, DevOps Engineers, Cloud Engineers, and SAP BASIS Administrators.

You can save this directly as **`README.md`** in your Linux learning repository.
