# 🐧 Linux CLI & Bash Shell Fundamentals

## 📖 Overview

The GNU Bourne-Again Shell (**Bash**) is the default command-line interpreter used in most Linux distributions. It allows users and system administrators to interact with the operating system by executing commands.

This guide covers:

- Basic command syntax
- Viewing file contents
- Command history
- Tab completion
- Multi-line commands
- Bash productivity shortcuts

---

# 🖥️ Basic Command Syntax

A Linux command generally consists of three parts:

```bash
command [options] [arguments]
```

### Components

| Component | Description |
|------------|------------|
| Command | Program to execute |
| Options | Modify command behavior |
| Arguments | Target objects such as files/directories |

### Example

```bash
whoami
```

Output:

```bash
student
```

---

# ➕ Running Multiple Commands

Use a semicolon (`;`) to run multiple commands on a single line.

```bash
command1 ; command2
```

Example:

```bash
date ; whoami
```

---

# 📅 Date Command

Display current date and time.

```bash
date
```

Display current time:

```bash
date +%R
```

Display date in local format:

```bash
date +%x
```

Example:

```bash
Tue Apr 29 17:58:22 UTC 2025
17:58
04/29/25
```

---

# 🔐 Passwd Command

Change your password.

```bash
passwd
```

Process:

```bash
Current password:
New password:
Retype new password:
passwd: password updated successfully
```

A privileged user can also change passwords for other users.

---

# 📂 File Command

Determine the type of a file.

```bash
file /etc/passwd
```

Output:

```bash
/etc/passwd: ASCII text
```

Example:

```bash
file /bin/passwd
```

Output:

```bash
ELF executable
```

Check a directory:

```bash
file /home
```

Output:

```bash
/home: directory
```

---

# 📄 Viewing File Contents

## cat Command

Display file contents.

```bash
cat /etc/passwd
```

Display multiple files:

```bash
cat file1 file2
```

---

## less Command

View large files page by page.

```bash
less /etc/passwd
```

Useful Keys:

| Key | Action |
|------|--------|
| Up Arrow | Scroll Up |
| Down Arrow | Scroll Down |
| Q | Quit |

---

# 🔝 Head Command

Display the first 10 lines of a file.

```bash
head /etc/passwd
```

Display a specific number of lines:

```bash
head -n 5 /etc/passwd
```

---

# 🔚 Tail Command

Display the last 10 lines.

```bash
tail /etc/passwd
```

Display last 3 lines:

```bash
tail -n 3 /etc/passwd
```

---

# 📊 Word Count Command (wc)

Count lines, words, and characters.

```bash
wc /etc/passwd
```

Output:

```bash
45 113 2738 /etc/passwd
```

### Count Lines

```bash
wc -l /etc/passwd
```

### Count Words

```bash
wc -w /etc/passwd
```

### Count Characters

```bash
wc -m /etc/passwd
```

---

# ⚡ Tab Completion

Bash can automatically complete commands and filenames.

### Complete Commands

Type:

```bash
pass
```

Press:

```text
Tab Tab
```

Possible output:

```bash
passwd
paste
pasta
```

Then type:

```bash
passw
```

Press:

```text
Tab
```

Result:

```bash
passwd
```

---

## Complete File Names

Example:

```bash
ls /etc/pas
```

Press:

```text
Tab
```

Result:

```bash
ls /etc/passwd
```

---

# 👤 User Management with Tab Completion

Instead of remembering all options:

```bash
useradd --
```

Press:

```text
Tab Tab
```

Output:

```bash
--home-dir
--shell
--groups
--create-home
--uid
```

---

# 📝 Writing Long Commands

Use a backslash (`\`) to continue a command on multiple lines.

Example:

```bash
head -n 3 \
/usr/share/dict/words \
/usr/share/dict/linux.words
```

Benefits:

- Better readability
- Easier maintenance
- Useful for scripts and long commands

---

# 📜 Command History

View previously executed commands.

```bash
history
```

Example:

```bash
23 clear
24 who
25 pwd
26 ls /etc
27 uptime
28 ls -l
29 date
```

---

## Re-run Commands

Run the most recent command beginning with "ls":

```bash
!ls
```

Run command number 26:

```bash
!26
```

---

# ⌨️ History Navigation

Use arrow keys:

| Key | Action |
|------|--------|
| ↑ | Previous command |
| ↓ | Next command |
| ← | Move cursor left |
| → | Move cursor right |

---

# 🚀 Useful Bash Shortcuts

| Shortcut | Description |
|-----------|------------|
| Ctrl + A | Move to beginning of line |
| Ctrl + E | Move to end of line |
| Ctrl + U | Delete from cursor to beginning |
| Ctrl + K | Delete from cursor to end |
| Ctrl + Left Arrow | Move to previous word |
| Ctrl + Right Arrow | Move to next word |
| Ctrl + R | Search command history |

---

# 💡 Productivity Tips

✅ Use **Tab Completion** to avoid typing long commands.

✅ Use **History Expansion** (`!`) to rerun commands quickly.

✅ Use **Ctrl + R** to search previously executed commands.

✅ Use **Backslash (\)** for long multi-line commands.

✅ Use **less** instead of **cat** for large files.

---

# 🎯 Summary

By mastering Bash shell productivity features, you can:

- Work faster on Linux systems
- Reduce typing mistakes
- Navigate command history efficiently
- Manage files and users effectively
- Improve overall system administration skills

These skills form the foundation for Linux Administration, DevOps, Cloud Engineering, and Red Hat Certification preparation.

---

## 📚 Next Topic

➡️ Linux File System Structure (FHS)
➡️ Navigating Directories (`pwd`, `cd`, `ls`)
➡️ Creating Files and Directories
➡️ File Permissions & Ownership
➡️ User and Group Management

Happy Learning! 🚀🐧
````

This README is ready to paste directly into a `README.md` file in your Linux learning GitHub repository.

