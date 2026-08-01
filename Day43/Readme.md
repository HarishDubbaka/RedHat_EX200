# 📘 Locating Files on a File System 

This lesson teaches you **how to search for files in Linux**. As a Linux administrator, you often need to answer questions like:

* Where is the `httpd.conf` file?
* Which files belong to a user?
* Which files are larger than 1 GB?
* Which files were modified today?
* Which files have 777 permissions?

Linux provides **two commands** for this:

1. **`locate`** → Very fast (uses a database)
2. **`find`** → Accurate (searches the filesystem in real time)

These are the main topics covered in the uploaded lesson. 

---

# 1. `locate` Command

Think of **locate** like **Google Search**.

Google doesn't search every website when you type something.
Instead, it searches its **already-built index**.

`locate` works the same way.

It searches a **database (mlocate database)** instead of checking every directory.

```
Filesystem
     │
     ▼
Updated Database
     │
     ▼
 locate
     │
     ▼
Instant Results
```

### Example

Search for passwd:

```bash
locate passwd
```

Output

```bash
/etc/passwd
/etc/passwd-
/etc/pam.d/passwd
```

Very fast because Linux is **not searching the disk**—it's searching the database. 

---

# Why is `locate` Fast?

Because it searches an index instead of scanning the filesystem.

Imagine a library.

Without an index:

```
Book 1
Book 2
Book 3
...
Book 5000
```

You check every book.

With an index:

```
Linux → Shelf 12
Networking → Shelf 8
Python → Shelf 20
```

You go directly to the correct shelf.

That is exactly how `locate` works.

---

# Problem with `locate`

The database **is not updated in real time**.

Example

Create a file:

```bash
touch myfile.txt
```

Now search:

```bash
locate myfile.txt
```

Maybe nothing appears.

Why?

Because the database hasn't been updated yet.

---

# Updating the Database

Root can update it immediately.

```bash
updatedb
```

Now search again.

```bash
locate myfile.txt
```

Now the file appears.

Normally, the database is updated automatically once a day. 

---

# Useful `locate` Options

## Case-insensitive search

```bash
locate -i messages
```

Finds

```
messages
Messages
MESSAGES
```

---

## Limit results

```bash
locate -n 5 passwd
```

Shows only the first five matches.

---

# When to Use `locate`

Use it when:

* You know the file name.
* You want results quickly.
* The database is up to date.

---

# 2. `find` Command

Now let's learn the most important command.

Unlike `locate`, **`find` searches the filesystem in real time**.

```
find
   │
   ▼
Disk
   │
   ▼
Every Directory
   │
   ▼
Matching Files
```

This makes it:

* Slower
* Much more accurate

---

# Basic Syntax

```bash
find <directory> <options>
```

Example

```bash
find / -name sshd_config
```

Search from root (`/`) and return files named exactly `sshd_config`. 

---

# If You Omit the Directory

```bash
find -name file.txt
```

Linux searches from the **current directory**.

If you're in

```
/home/harish
```

it searches only inside `/home/harish` and its subdirectories. 

---

# Searching with Wildcards

Suppose you want every text file.

```bash
find / -name '*.txt'
```

Notice the quotes.

Without quotes:

```bash
find / -name *.txt
```

the shell expands `*.txt` before `find` receives it, which can lead to incorrect results. The lesson recommends using **single quotes** around wildcard patterns. 

---

## Find files containing "pass"

```bash
find /etc -name '*pass*'
```

Possible results

```
passwd
passwd-
opasswd
```

---

# Case-insensitive Search

```bash
find / -iname '*messages*'
```

Matches

```
Messages
messages
MESSAGES
```

---

# Search by Owner

Find files owned by user `developer`:

```bash
find -user developer
```

Search by group:

```bash
find -group developer
```

Search by UID:

```bash
find -uid 1000
```

Search by GID:

```bash
find -gid 1000
```

These searches operate on files visible to the user running the command. 

---

# Search by Permissions

Example

```bash
find /home -perm 764
```

Find files whose permissions are exactly:

```
Owner  : rwx
Group  : rw-
Others : r--
```

---

## View Permissions with Results

```bash
find /home -perm 764 -ls
```

Shows permission details together with each matching file. 

---

## Search for Minimum Permissions

```bash
find /home -perm -324
```

The **`-`** means the file must have **at least** those permissions.

---

## Search if Any Permission Matches

```bash
find /home -perm /442
```

The **`/`** means a match occurs if **any** of the specified permission bits are set.

This distinction between `/` and `-` is highlighted in the lesson. 

---

# Search by Size

Exact 10 MB

```bash
find -size 10M
```

Greater than 10 GB

```bash
find -size +10G
```

Smaller than 10 KB

```bash
find -size -10k
```

Units used in the lesson:

* `k` → Kilobytes
* `M` → Megabytes
* `G` → Gigabytes

The lesson also notes that `find -size` rounds to whole units. 

---

# Search by Modification Time

Changed exactly 120 minutes ago

```bash
find / -mmin 120
```

More than 200 minutes ago

```bash
find / -mmin +200
```

Less than 150 minutes ago

```bash
find / -mmin -150
```

---

# Search by File Type

Directories

```bash
find /etc -type d
```

Regular files

```bash
find / -type f
```

Symbolic links

```bash
find / -type l
```

Block devices

```bash
find /dev -type b
```

These are the file-type flags presented in the lesson. 

---

# Search by Hard Link Count

Find regular files with more than one hard link:

```bash
find / -type f -links +1
```

This is useful for identifying files that have multiple hard links. 

---

# `locate` vs `find`

| Feature                     | `locate`                   | `find`          |
| --------------------------- | -------------------------- | --------------- |
| Speed                       | Very fast                  | Slower          |
| Searches                    | Database                   | Live filesystem |
| Finds newly created files   | No (until database update) | Yes             |
| Search by name              | ✅                          | ✅               |
| Search by size              | ❌                          | ✅               |
| Search by permissions       | ❌                          | ✅               |
| Search by owner             | ❌                          | ✅               |
| Search by modification time | ❌                          | ✅               |

---

# Memory Trick

**`locate` = Library Index 📚**

* Uses a database
* Very fast
* Can miss recently created files until the database is updated

**`find` = Walking Every Shelf 🚶**

* Searches the real filesystem
* Slower
* Always current
* Supports many search criteria

---

# Quick Command Cheat Sheet

```bash
# Search using database
locate passwd

# Update locate database
updatedb

# Find exact file
find / -name sshd_config

# Find using wildcard
find / -name '*.txt'

# Case-insensitive search
find / -iname '*messages*'

# Find by owner
find -user developer

# Find by permissions
find /home -perm 764

# Find by size
find -size +10G

# Find by modification time
find / -mmin -120

# Find directories
find /etc -type d

# Find symbolic links
find / -type l
```

## Key Takeaway

* Use **`locate`** when you need a **fast filename search** and the database is current.
* Use **`find`** when you need **accurate, real-time searches** or want to filter by attributes such as owner, permissions, size, modification time, file type, or link count. These are the core distinctions emphasized throughout the uploaded lesson. 
