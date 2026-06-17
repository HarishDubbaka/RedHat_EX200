# 📖 Getting Help from Local Documentation (Man Pages)

Linux systems provide extensive built-in documentation through **manual pages (man pages)**. These pages are installed locally with software packages and can be accessed directly from the command line using the `man` command.

Manual pages are stored under:

```bash
/usr/share/man
```

They serve as the primary source of documentation for Linux commands, configuration files, system calls, administrative tools, and more.

---

# 📚 Linux Manual Sections

The Linux manual is divided into multiple sections based on content type.

| Section | Content Type                          | Description                                    |
| ------- | ------------------------------------- | ---------------------------------------------- |
| 1       | User Commands                         | Executable commands and shell programs         |
| 2       | System Calls                          | Kernel routines invoked from user space        |
| 3       | Library Functions                     | Functions provided by program libraries        |
| 4       | Special Files                         | Device files and hardware interfaces           |
| 5       | File Formats                          | Configuration files and file structures        |
| 6       | Games and Screensavers                | Historical section for games and entertainment |
| 7       | Conventions, Standards, Miscellaneous | Protocols, file systems, and standards         |
| 8       | System Administration Commands        | Maintenance and privileged commands            |
| 9       | Linux Kernel API                      | Internal kernel functions                      |

> **System administrators** most frequently use:
>
> * Section 1 → User Commands
> * Section 5 → File Formats
> * Section 8 → Administrative Commands
>
> Troubleshooting often involves Section 2 (System Calls).

---

# 🔍 Accessing Man Pages

To view documentation for a specific command, use:

```bash
man <command>
```

### Example

```bash
man ssh
```

Sample output:

```text
SSH(1)                        General Commands Manual                       SSH(1)

NAME
       ssh — OpenSSH remote login client

SYNOPSIS
       ssh [options] [user@]hostname

DESCRIPTION
       ssh is a program for logging into a remote machine and
       executing commands securely over a network.
```

---

# 📖 Navigating Inside a Man Page

Once a man page opens:

| Key      | Action                 |
| -------- | ---------------------- |
| ↑ / ↓    | Scroll line by line    |
| Space    | Move forward one page  |
| b        | Move backward one page |
| /keyword | Search for text        |
| n        | Next search match      |
| N        | Previous search match  |
| q        | Quit the man page      |

---

# 🔎 Searching for Commands by Keyword

If you do not know the exact command name, search the manual database using:

```bash
man -k <keyword>
```

This searches both titles and descriptions of all manual pages.

### Example

```bash
man -k ssh
```

Output:

```text
scp (1)          - OpenSSH secure file copy
sftp (1)         - OpenSSH secure file transfer
ssh (1)          - OpenSSH remote login client
ssh-add (1)      - adds private key identities
ssh-agent (1)    - OpenSSH authentication agent
```

> `man -k` is equivalent to the `apropos` command.

---

# 🎯 Finding a Specific Manual Page

To search manual pages by title only:

```bash
man -f <command>
```

Equivalent command:

```bash
whatis <command>
```

### Example

```bash
man -f passwd
```

Output:

```text
passwd (1) - update user's authentication tokens
passwd (5) - password file
```

This helps identify which section contains the documentation you need.

---

# 🔍 Advanced Keyword Search

For broader searches, use:

```bash
man -k passwd
```

Example output:

```text
chgpasswd (8) - update group passwords in batch mode
chpasswd (8)  - update passwords in batch mode
gpasswd (1)   - administer /etc/group and /etc/gshadow
```

Unlike `man -f`, this command searches both page titles and descriptions.

---

# 🔎 Full-Text Search in Man Pages

To search for a keyword within the actual content of all man pages:

```bash
man -K <keyword>
```

### Example

```bash
man -K password
```

### How It Works

* Searches the full text of every man page.
* More comprehensive than `man -k`.
* Uses more CPU and takes longer to complete.
* Displays the first matching page.

When viewing results:

```text
Press q
```

to skip the current page and move to the next match.

---

# 📝 Common Examples

### View documentation for a command

```bash
man ls
```

### View documentation for a file format

```bash
man 5 passwd
```

### View documentation for an administrative command

```bash
man 8 useradd
```

### Search for SSH-related commands

```bash
man -k ssh
```

### Find what "passwd" refers to

```bash
man -f passwd
```

### Search all manuals for a keyword

```bash
man -K authentication
```

---

# 💡 Pro Tips

* Use `man <command>` as your first troubleshooting resource.
* If multiple sections exist, specify the section number:

```bash
man 5 passwd
man 8 passwd
```

* Use `/keyword` inside a man page to quickly locate information.
* Combine `man -k` with keywords when you only know the function but not the command name.

---

# ✅ Summary

Linux manual pages provide comprehensive local documentation for commands, configuration files, system calls, and administrative tools.

Essential commands:

```bash
man <command>     # Open a manual page
man -f <topic>    # Search by title (whatis)
man -k <keyword>  # Search titles and descriptions (apropos)
man -K <keyword>  # Full-text search
```

Learning to use man pages effectively is one of the most important skills for Linux administrators and engineers because help is always available directly from the command line.
