# Synchronizing Content Between Systems — `rsync`

## 🎯 Objectives

After this topic, you should be able to:

* Synchronize files/directories between local and remote systems.
* Use `rsync` efficiently and securely.
* Perform a **dry run** before synchronization.
* Understand archive mode (`-a`).
* Understand the importance of a **trailing `/`**.
* Preserve permissions, ownership, timestamps, ACLs, and SELinux contexts.

---

## 🔄 What is `rsync`?

`rsync` is a Linux command used to **synchronize files and directories** between systems.

The main advantage is that it transfers **only the changed data**, rather than copying the entire file every time.

### Example

Suppose you have:

```text
Server A                         Server B
---------                        ---------
file.txt  ────────────────────> file.txt
          Initial synchronization
```

Later, only 10 KB of the file changes:

```text
Server A                         Server B
---------                        ---------
file.txt  ───── 10 KB ────────> file.txt
```

Instead of transferring the entire file, `rsync` can transfer only the differences.

### ⭐ Key advantage

> **First synchronization → similar to normal copying**
> **Later synchronizations → much faster because only changes are transferred**

---

# 🧪 Dry Run — `-n`

Before actually synchronizing, use:

```bash
rsync -avn source destination
```

`-n` means **dry run**.

It shows what `rsync` *would* do without actually making the changes.

### Example

```bash
rsync -avn /data/ hosta:/backup/
```

This is especially useful before a production synchronization because it helps you identify:

* Files that will be copied
* Files that will be updated
* Files that may be deleted when deletion options are used
* Unexpected changes

### Best practice

```text
Dry Run
   ↓
Review output
   ↓
Confirm changes
   ↓
Run actual rsync
```

---

# ⭐ Important `rsync` Options

## `-v` — Verbose

```bash
rsync -v source destination
```

Provides more detailed output.

Useful for:

* Monitoring the operation
* Troubleshooting
* Seeing which files are processed

---

## `-a` — Archive Mode

```bash
rsync -a source destination
```

Archive mode is commonly used for directory synchronization.

It enables recursive copying and preserves many important file attributes.

### `-a` includes:

| Option | Meaning                  |
| ------ | ------------------------ |
| `-r`   | Recursive                |
| `-l`   | Preserve symbolic links  |
| `-p`   | Preserve permissions     |
| `-t`   | Preserve timestamps      |
| `-g`   | Preserve group ownership |
| `-o`   | Preserve owner           |
| `-D`   | Preserve device files    |

Therefore:

```bash
rsync -a
```

is approximately equivalent to:

```bash
rsync -rlptgoD
```

---

# 🔗 Hard Links

Archive mode does **not** preserve hard links by default.

Use:

```bash
-H
```

Example:

```bash
rsync -aH source/ destination/
```

### Remember

```text
-a  → Archive
-H  → Preserve hard links
```

---

# 🔐 ACL and SELinux Attributes

For additional file attributes, use:

### `-A` — ACLs

```bash
rsync -aA source/ destination/
```

Preserves **Access Control Lists**.

### `-X` — Extended Attributes

```bash
rsync -aX source/ destination/
```

Preserves extended attributes, including **SELinux file contexts**.

### Common combination

```bash
rsync -aAX source/ destination/
```

This is useful when maintaining Linux system files where ACLs and SELinux contexts matter.

---

# 🌐 Remote Synchronization

`rsync` specifies remote systems using:

```text
user@host:path
```

For example:

```text
root@hosta:/backup
```

The remote system can be either:

* Source
* Destination

---

# 📤 Local → Remote

Suppose you want to synchronize `/var/log` from `host` to `/tmp` on `hosta`.

```bash
rsync -av /var/log hosta:/tmp
```

Result:

```text
host
/var/log
   │
   │ rsync
   ↓
hosta
/tmp/log
```

Because `/var/log` does **not** have a trailing `/`, the `log` directory itself is copied into `/tmp`.

---

# 📥 Remote → Local

You can also synchronize in the opposite direction:

```bash
rsync -av hosta:/var/log /tmp
```

Here:

```text
hosta:/var/log
      │
      │ rsync
      ↓
host:/tmp
```

---

# 💻 Local → Local

`rsync` can also synchronize two directories on the same machine.

```bash
sudo rsync -av /var/log /tmp
```

Here `sudo` is used to obtain root privileges.

Result:

```text
/tmp/log/
```

---

# ⚠️ Very Important: Trailing Slash `/`

This is one of the most important concepts in `rsync`.

There is a difference between:

```bash
rsync -av /var/log /tmp
```

and:

```bash
rsync -av /var/log/ /tmp
```

---

## ❌ Without trailing `/`

```bash
rsync -av /var/log /tmp
```

The **directory itself** is synchronized.

Result:

```text
/tmp/log/
```

Think:

```text
/var/log
    ↓
/tmp/log
```

---

## ✅ With trailing `/`

```bash
rsync -av /var/log/ /tmp
```

Only the **contents** of `/var/log` are synchronized.

Result:

```text
/tmp/
├── README
├── boot.log
├── dnf.log
├── audit/
└── ...
```

Think:

```text
/var/log/
     │
     ├── README ───────→ /tmp/README
     ├── boot.log ─────→ /tmp/boot.log
     └── audit/ ───────→ /tmp/audit/
```

### 🧠 Easy way to remember

> **No `/` → copy the directory itself**
> **With `/` → copy the contents of the directory**

---

# 📊 Quick Comparison

| Command                    | Result                              |
| -------------------------- | ----------------------------------- |
| `rsync -av /var/log /tmp`  | `/tmp/log/` is created              |
| `rsync -av /var/log/ /tmp` | Contents go directly into `/tmp/`   |
| `rsync -avn ...`           | Dry run                             |
| `rsync -av ...`            | Verbose + archive                   |
| `rsync -aH ...`            | Archive + hard links                |
| `rsync -aA ...`            | Archive + ACLs                      |
| `rsync -aX ...`            | Archive + extended attributes       |
| `rsync -aAX ...`           | Archive + ACL + extended attributes |

---

# 👑 Root Privileges and Ownership

If you need to preserve **file ownership** on the destination, you generally need root privileges.

For a remote destination, authenticate as root where appropriate:

```bash
rsync -av source/ root@hosta:/destination/
```

For a local destination:

```bash
sudo rsync -av source/ /destination/
```

Without sufficient privileges, `rsync` may not be able to preserve ownership or other protected attributes.

---

# 🔐 Is `rsync` Secure?

`rsync` itself is a synchronization tool. When used with a remote host over SSH, the transfer can be secured using SSH.

Typical usage:

```bash
rsync -av -e ssh /data/ user@hosta:/backup/
```

Conceptually:

```text
Local Server
     │
     │ rsync + SSH
     │
     ▼
Remote Server
```

---

# 🧠 Interview Questions

### 1. What is `rsync`?

`rsync` is a Linux utility used to synchronize files and directories between local and remote systems efficiently by transferring only the changed portions.

### 2. What is the advantage of `rsync` over normal copying?

After the initial synchronization, `rsync` transfers only changed data, which reduces network traffic and synchronization time.

### 3. What does `-n` mean?

`-n` performs a **dry run**. It shows what would happen without actually changing files.

### 4. What does `-a` mean?

`-a` means **archive mode**. It enables recursive copying and preserves attributes such as permissions, timestamps, ownership, groups, and symbolic links.

### 5. What does `-v` mean?

`-v` means **verbose** and provides detailed output.

### 6. How do you preserve hard links?

Use:

```bash
-H
```

Example:

```bash
rsync -aH source/ destination/
```

### 7. How do you preserve ACLs?

Use:

```bash
-A
```

### 8. How do you preserve SELinux contexts?

Use:

```bash
-X
```

### 9. What is the difference between these two?

```bash
rsync -av /data /backup
```

Copies the `data` directory itself.

```bash
rsync -av /data/ /backup
```

Copies the **contents** of `data` into `backup`.

### ⭐ Most important point

```text
rsync -av /source /destination
                 ↓
          Directory itself

rsync -av /source/ /destination
                  ↓
          Directory contents
```

This **trailing slash behavior** is one of the most common `rsync` points to remember for RHCSA/Linux administration.
