# 🔗 Linux Links Reference Guide (Hard Links vs Symbolic Links)

## 📖 What is a Link?

A **link** is another way to access a file or directory without duplicating its data.

Linux provides two types of links:

1. **Hard Links**
2. **Symbolic (Soft) Links**

---

# 1️⃣ Hard Links

A **hard link** is an additional filename that points directly to the same file data (inode) on disk.

Think of it like:

📄 One document stored on disk
🏷️ Multiple labels attached to the same document

Both names access the exact same file.

---

## Create a Hard Link

```bash
ln existing_file hardlink_name
```

### Example

```bash
touch newfile.txt

ln newfile.txt /tmp/newfile-hlink2.txt
```

Verify:

```bash
ls -l newfile.txt /tmp/newfile-hlink2.txt
```

Output:

```bash
-rw-rw-r-- 2 user user 0 Jun 20 10:00 newfile.txt
-rw-rw-r-- 2 user user 0 Jun 20 10:00 /tmp/newfile-hlink2.txt
```

### Understanding Link Count

The number:

```bash
2
```

indicates two hard links exist for the same file.

---

## Verify Using Inode Numbers

```bash
ls -il newfile.txt /tmp/newfile-hlink2.txt
```

Example:

```bash
8924107 newfile.txt
8924107 /tmp/newfile-hlink2.txt
```

✅ Same inode number = Hard linked files

---

## Hard Link Behavior

Create file:

```bash
echo "Hello World" > newfile.txt
```

Delete original:

```bash
rm newfile.txt
```

Read through hard link:

```bash
cat /tmp/newfile-hlink2.txt
```

Output:

```bash
Hello World
```

✅ Data still exists because another hard link points to it.

---

## Hard Link Characteristics

| Feature                         | Supported |
| ------------------------------- | --------- |
| Links to files                  | ✅ Yes     |
| Links to directories            | ❌ No      |
| Across file systems             | ❌ No      |
| Same inode                      | ✅ Yes     |
| Survives original file deletion | ✅ Yes     |

---

# 2️⃣ Symbolic Links (Soft Links)

A **symbolic link** is a special file that points to another file or directory by name.

Think of it like:

📍 A shortcut in Windows

The shortcut does not contain the actual data.

---

## Create a Symbolic Link

```bash
ln -s target_file symlink_name
```

### Example

```bash
ln -s /home/user/newfile.txt /tmp/newfile-symlink.txt
```

Verify:

```bash
ls -l /tmp/newfile-symlink.txt
```

Output:

```bash
lrwxrwxrwx 1 user user 22 Jun 20 10:00 /tmp/newfile-symlink.txt -> /home/user/newfile.txt
```

Notice:

```bash
l
```

at the beginning.

It indicates a symbolic link.

---

## Access Through Symbolic Link

```bash
cat /tmp/newfile-symlink.txt
```

Output:

```bash
Hello World
```

---

## Dangling Symbolic Link

Delete original file:

```bash
rm /home/user/newfile.txt
```

Try reading the symbolic link:

```bash
cat /tmp/newfile-symlink.txt
```

Output:

```bash
No such file or directory
```

The symbolic link still exists, but the target file is gone.

This is called a:

⚠️ **Dangling Symbolic Link**

---

## Symbolic Link Characteristics

| Feature                         | Supported |
| ------------------------------- | --------- |
| Links to files                  | ✅ Yes     |
| Links to directories            | ✅ Yes     |
| Across file systems             | ✅ Yes     |
| Same inode                      | ❌ No      |
| Survives original file deletion | ❌ No      |

---

# Hard Link vs Symbolic Link

| Feature                      | Hard Link  | Symbolic Link |
| ---------------------------- | ---------- | ------------- |
| Points To                    | Inode/Data | Filename      |
| Same Inode                   | ✅ Yes      | ❌ No          |
| Across File Systems          | ❌ No       | ✅ Yes         |
| Link Directories             | ❌ No       | ✅ Yes         |
| Works After Original Deleted | ✅ Yes      | ❌ No          |
| Creates Shortcut             | ❌ No       | ✅ Yes         |

---

# Checking File Systems

Hard links can only exist within the same filesystem.

View mounted filesystems:

```bash
df -h
```

Example:

```bash
Filesystem      Size Used Avail Mounted on
/dev/sda3        20G   6G   14G /
/dev/sda2       200M   8M  192M /boot/efi
```

### Valid Hard Link

```bash
ln /var/tmp/file1 /home/user/file1-link
```

Both are under:

```bash
/
```

Same filesystem ✅

---

### Invalid Hard Link

```bash
ln /boot/efi/grub.cfg /home/user/grub-link
```

Error:

```bash
Invalid cross-device link
```

Different filesystems ❌

---

# Symbolic Link to Directory

Create:

```bash
ln -s /etc ~/configfiles
```

Move into link:

```bash
cd ~/configfiles
pwd
```

Output:

```bash
/home/user/configfiles
```

Use physical path:

```bash
cd -P ~/configfiles
pwd
```

Output:

```bash
/ etc
```

(Without the space: `/etc`)

---

# Easy Memory Trick 🧠

### Hard Link

```text
Name ─────► Data
```

Multiple names point directly to the same data.

```text
file1 ─┐
        ├──► Data
file2 ─┘
```

---

### Symbolic Link

```text
symlink ─► filename ─► Data
```

A symbolic link points to another name, not directly to data.

```text
shortcut
    │
    ▼
file1 ───► Data
```

---

# Most Common Commands

```bash
# Create hard link
ln file1 hardlink

# Create symbolic link
ln -s file1 symlink

# Show inode numbers
ls -i

# Show detailed link info
ls -l

# Check filesystems
df -h

# Remove link
rm symlink
rm hardlink
```


