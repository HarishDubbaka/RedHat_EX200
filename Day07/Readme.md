# 📂 Linux Absolute Paths and Relative Paths

## 📖 Overview

Every file and directory in Linux has a unique location in the file system. This location is identified using a **path**.

A path is simply the route Linux follows to locate a file or directory.

Think of a path like a postal address:

* Country → State → City → Street → House Number
* Root Directory (`/`) → Subdirectories → File

---

# 🛣️ Understanding File Paths

A file path specifies the exact location of a file or directory in the Linux file-system hierarchy.

Example:

```bash
/var/log/messages
```

Linux follows the directories one by one until it reaches the destination file.

---

# 🌍 Absolute Paths

An **Absolute Path** is the complete path from the root directory (`/`) to a file or directory.

### Rule

If a path starts with a forward slash (`/`), it is an absolute path.

### Example

```bash
/var/log/messages
```

Path Breakdown:

```text
/
└── var
    └── log
        └── messages
```

### Characteristics

✅ Starts from root (`/`)

✅ Always points to the same location

✅ Unique throughout the filesystem

### Examples

```bash
/home/user/Documents/report.txt

/etc/passwd

/var/log/messages

/usr/bin/bash
```

---

# 📍 Relative Paths

A **Relative Path** describes a location relative to the current working directory.

### Rule

If a path does NOT start with `/`, it is a relative path.

### Example

Current Directory:

```bash
/var
```

Absolute Path:

```bash
/var/log/messages
```

Relative Path:

```bash
log/messages
```

Linux starts from the current directory and follows the specified path.

### Characteristics

✅ Shorter to type

✅ Depends on current location

✅ Convenient for daily work

### Examples

```bash
Documents/report.txt

Downloads/file.zip

log/messages
```

---

# 🏠 Current Working Directory (PWD)

Every shell session operates from a current location called the **Current Working Directory (CWD)**.

To display it:

```bash
pwd
```

Example:

```bash
user@host:~$ pwd
/home/user
```

Output:

```text
/home/user
```

---

# 📋 Listing Directory Contents

Use the `ls` command to view files and directories.

### Basic Listing

```bash
ls
```

Example:

```bash
user@host:~$ ls

Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
```

---

# 🔄 Changing Directories

Use the `cd` command to move between directories.

## Change Using Relative Path

```bash
cd Videos
```

Example:

```bash
user@host:~$ cd Videos
user@host:~/Videos$
```

---

## Change Using Absolute Path

```bash
cd /home/user/Documents
```

Example:

```bash
user@host:~/Videos$ cd /home/user/Documents
user@host:~/Documents$
```

---

## Return to Home Directory

```bash
cd
```

Example:

```bash
user@host:~/Documents$ cd
user@host:~$
```

---

# 🏡 The Tilde (~) Shortcut

The tilde (`~`) represents the current user's home directory.

### Example

```bash
~
```

Means:

```bash
/home/user
```

### Examples

```bash
cd ~

ls ~

cd ~/Documents

cd ~/Downloads
```

---

# 📝 Creating Files with touch

The `touch` command creates empty files or updates timestamps.

### Create Files

```bash
touch Videos/blockbuster1.ogg

touch Videos/blockbuster2.ogg

touch Documents/thesis_chapter1.txt

touch Documents/thesis_chapter2.txt
```

Result:

```text
Videos
├── blockbuster1.ogg
└── blockbuster2.ogg

Documents
├── thesis_chapter1.txt
└── thesis_chapter2.txt
```

---

# 📄 Advanced ls Options

## Long Listing Format

```bash
ls -l
```

Shows:

* Permissions
* Owner
* Group
* Size
* Date
* File Name

Example:

```bash
ls -l
```

---

## Show Hidden Files

```bash
ls -la
```

Displays:

```text
.bashrc
.bash_profile
.bash_history
.cache
.config
```

Hidden files begin with a dot (`.`).

---

## Recursive Listing

```bash
ls -R
```

Displays all files and subdirectories recursively.

---

# 🔹 Special Directory Symbols

Linux provides two special directory entries in every directory.

| Symbol | Meaning           |
| ------ | ----------------- |
| `.`    | Current Directory |
| `..`   | Parent Directory  |

---

## Current Directory (`.`)

Represents the directory you are currently in.

Example:

```bash
cd .
```

No change occurs.

---

## Parent Directory (`..`)

Moves one level upward.

Example:

```bash
cd ..
```

---

# ⬆️ Moving Up Directory Levels

Current Directory:

```bash
/home/user/Videos
```

Move up:

```bash
cd ..
```

Result:

```bash
/home/user
```

Continue:

```bash
cd ..
```

Result:

```bash
/home
```

Again:

```bash
cd ..
```

Result:

```bash
/
```

---

# 🔙 Return to Previous Directory

The `cd -` command switches back to the previous working directory.

Example:

```bash
cd Videos

cd Documents

cd -
```

Output:

```bash
/home/user/Videos
```

Run again:

```bash
cd -
```

Output:

```bash
/home/user/Documents
```

This is extremely useful when switching between two directories repeatedly.

---

# 🔤 Linux is Case-Sensitive

Linux treats uppercase and lowercase letters as different characters.

Example:

```bash
touch MyExample.txt

touch myexample.txt
```

Result:

```text
MyExample.txt
myexample.txt
```

These are two different files.

---

# 🧠 Quick Comparison: Absolute vs Relative Paths

| Feature                      | Absolute Path | Relative Path         |
| ---------------------------- | ------------- | --------------------- |
| Starts With                  | `/`           | Anything except `/`   |
| Depends on Current Directory | ❌ No          | ✅ Yes                 |
| Full Location                | ✅ Yes         | ❌ No                  |
| Easier to Type               | ❌ No          | ✅ Yes                 |
| Always Unique                | ✅ Yes         | ❌ Depends on Location |

### Examples

Absolute:

```bash
/home/user/Documents/report.txt
```

Relative:

```bash
Documents/report.txt
```

---

# 🚀 Common Navigation Commands

| Command        | Description                  |
| -------------- | ---------------------------- |
| `pwd`          | Show current directory       |
| `ls`           | List files and directories   |
| `ls -l`        | Long listing                 |
| `ls -la`       | Show hidden files            |
| `ls -R`        | Recursive listing            |
| `cd directory` | Change directory             |
| `cd`           | Go to home directory         |
| `cd ..`        | Go to parent directory       |
| `cd -`         | Return to previous directory |
| `cd ~`         | Go to home directory         |
| `touch file`   | Create empty file            |

---

# 🎯 Key Takeaways

* Absolute paths start from the root directory (`/`).
* Relative paths start from the current working directory.
* Use `pwd` to verify your location.
* Use `cd` to navigate directories.
* `~` represents your home directory.
* `.` refers to the current directory.
* `..` refers to the parent directory.
* `cd -` returns to the previous directory.
* Linux filenames are case-sensitive.
* `touch` creates empty files quickly.

Mastering Linux paths is one of the most important skills for Linux System Administrators, DevOps Engineers, Cloud Engineers, and SAP BASIS Administrators.

