# Chapter — Archiving Files

## 1. What is an Archive?

An **archive** is a single file that contains multiple files and directories.

Think of it like a **box**:

```text
Before:

file1.txt
file2.txt
file3.txt
directory/
```

You can put everything into one archive:

```text
backup.tar
```

Inside:

```text
backup.tar
├── file1.txt
├── file2.txt
├── file3.txt
└── directory/
```

The important point is:

> **Archiving combines multiple files/directories into one file.**

It is useful for **backups** and **transferring files between systems**. 

---

# 2. What is `tar`?

`tar` is the most commonly used Linux utility for creating, managing, and extracting archives.

```bash
tar
```

The name historically comes from:

> **Tape ARchive**

Today, `tar` is commonly used even when no tape drive is involved.

For example:

```bash
tar -cf backup.tar file1.txt file2.txt
```

This creates:

```text
backup.tar
```

containing:

```text
file1.txt
file2.txt
```



---

# 3. Archive vs Compression

This is **very important**.

### Archiving

Combines multiple files into one file.

```text
file1
file2
file3
   ↓
backup.tar
```

### Compression

Reduces the size of data.

```text
backup.tar
   ↓
backup.tar.gz
```

### Archive + Compression

`tar` can do both:

```text
Multiple files
      ↓
    tar
      ↓
backup.tar
      ↓
  compression
      ↓
backup.tar.gz
```

So:

```text
.tar
```

usually means **archive only**.

```text
.tar.gz
```

means **archive + gzip compression**.

---

# 4. Important `tar` Options

There are three major actions:

| Option | Meaning               |
| ------ | --------------------- |
| `-c`   | Create archive        |
| `-t`   | List archive contents |
| `-x`   | Extract archive       |

The file option:

| Option | Meaning                  |
| ------ | ------------------------ |
| `-f`   | Specify archive filename |

And commonly:

| Option | Meaning              |
| ------ | -------------------- |
| `-v`   | Verbose output       |
| `-p`   | Preserve permissions |

These options are documented in the chapter. 

---

# 5. Creating an Archive

Suppose you have:

```text
myapp1.log
myapp2.log
myapp3.log
```

Create an archive:

```bash
tar -cf mybackup.tar myapp1.log myapp2.log myapp3.log
```

Break it down:

```text
tar
│
├── -c       Create
├── -f       Filename follows
├── mybackup.tar
│
├── myapp1.log
├── myapp2.log
└── myapp3.log
```

Result:

```text
mybackup.tar
```

The three files are now stored inside it. 

---

# 6. What does `-v` mean?

`-v` means **verbose**.

Without `-v`:

```bash
tar -cf backup.tar file1 file2 file3
```

You may not see the files being processed.

With `-v`:

```bash
tar -cvf backup.tar file1 file2 file3
```

You can see:

```text
file1
file2
file3
```

So remember:

```text
-v = show me what tar is doing
```

---

# 7. What does `-f` mean?

`-f` tells `tar`:

> "The next argument is the archive filename."

Example:

```bash
tar -cf backup.tar file1.txt
```

Here:

```text
-c  → create
-f  → archive filename follows
backup.tar → archive filename
file1.txt → file to put inside
```

A common mistake is forgetting `-f`.

---

# 8. Listing Archive Contents

You don't need to extract an archive just to see what's inside.

Use:

```bash
tar -tf backup.tar
```

Here:

```text
-t → list
-f → archive file
```

Example:

```bash
tar -tf /root/etc.tar
```

Output might be:

```text
etc/
etc/fstab
etc/crypttab
etc/mtab
```



### Remember

```bash
tar -tf archive.tar
```

means:

> **Show me what's inside the archive.**

---

# 9. Extracting an Archive

To extract:

```bash
tar -xf backup.tar
```

Breakdown:

```text
-x → extract
-f → archive filename
```

Example:

```bash
tar -xf /root/etc.tar
```

This extracts the files from the archive into the current directory. 

---

# 10. Why Extract into an Empty Directory?

Suppose your archive contains:

```text
/etc/passwd
/etc/hosts
/etc/ssh/
```

If you extract it directly into an existing system location, files could be overwritten.

Therefore, it is safer to create a separate directory:

```bash
mkdir /root/etcbackup
```

Then extract there:

```bash
cd /root/etcbackup
tar -xf /root/etc.tar
```

The chapter specifically recommends extracting into an empty directory to avoid overwriting existing files. 

---

# 11. Preserving Permissions

Normally, extracted files can have their permissions affected by the current `umask`.

Use:

```bash
tar -xpf backup.tar
```

Here:

```text
-x → extract
-p → preserve permissions
-f → archive file
```

Example:

```bash
tar -xpf /home/user/myscripts.tar
```



For `root`, `-p` is enabled by default when extracting, according to the chapter.

---

# 12. Compression with `tar`

`tar` supports several compression algorithms.

### gzip

```bash
tar -czf backup.tar.gz /etc
```

`-z` means:

```text
gzip
```

Result:

```text
backup.tar.gz
```

### bzip2

```bash
tar -cjf backup.tar.bz2 /var/log
```

`-j` means:

```text
bzip2
```

### xz

```bash
tar -cJf backup.tar.xz /etc/ssh
```

`-J` means:

```text
xz
```

The chapter identifies gzip as fast and widely available, bzip2 as providing smaller archives in many cases, and xz as offering the best compression ratio among these methods. 

---

# 13. Easy Compression Table

| Command    | Compression | Extension  |
| ---------- | ----------- | ---------- |
| `tar -cf`  | None        | `.tar`     |
| `tar -czf` | gzip        | `.tar.gz`  |
| `tar -cjf` | bzip2       | `.tar.bz2` |
| `tar -cJf` | xz          | `.tar.xz`  |

### Memory trick

```text
-z → gzip
-j → bzip2
-J → xz
```

---

# 14. Create a gzip Archive

Example:

```bash
tar -czf /root/etcbackup.tar.gz /etc
```

Break it down:

```text
tar
│
├── -c  → Create
├── -z  → gzip
├── -f  → Filename
│
├── /root/etcbackup.tar.gz
│
└── /etc → source directory
```

Result:

```text
/root/etcbackup.tar.gz
```



---

# 15. List a Compressed Archive

You can use:

```bash
tar -tf backup.tar.gz
```

You **do not need to specify `-z`** just to list the contents.

For example:

```bash
tar -tf /root/etcbackup.tar.gz
```

`tar` can determine the compression type from the archive. 

---

# 16. Extract a Compressed Archive

You can also simply use:

```bash
tar -xf backup.tar.gz
```

Modern `tar` automatically detects the compression type.

So you don't necessarily need:

```bash
tar -xzf backup.tar.gz
```

Although `-xzf` is still commonly seen.

---

# 17. Important Error to Understand

Suppose you have:

```text
backup.tar.xz
```

But you run:

```bash
tar -xzf backup.tar.xz
```

You are telling `tar`:

> "This is gzip."

But `.xz` means XZ compression.

Therefore you can get an error such as:

```text
gzip: stdin: not in gzip format
tar: Child returned status 1
```

The problem is that you selected the **wrong compression option**. 

---

# 18. `tar` vs `zip`

Linux commonly uses:

```bash
tar
```

for archiving.

The traditional ZIP utilities are:

```bash
zip
unzip
```

The chapter notes that RHEL supports both. 

For RHEL administration, you should be very comfortable with:

```bash
tar -cf
tar -tf
tar -xf
tar -czf
tar -cjf
tar -cJf
```

---

# 19. Very Important: `tar` Does Not Always Preserve Everything

By default, extended attributes are not necessarily preserved.

Examples include:

* ACLs
* SELinux contexts
* Other extended attributes

The chapter recommends options such as:

```bash
--acls
--selinux
--xattrs
```

when those attributes need to be included. 

For example:

```bash
tar --selinux -czf backup.tar.gz /etc
```

---

# 20. Absolute Path Warning

Suppose you run:

```bash
tar -cf backup.tar /etc
```

You may see:

```text
tar: Removing leading `/' from member names
```

This is **not an error**.

`tar` removes the leading `/` from stored path names.

Instead of storing:

```text
/etc/passwd
```

it stores:

```text
etc/passwd
```

This is safer because extracting absolute paths could otherwise overwrite existing files. 

---

# 21. The Most Important Commands to Remember

### Create

```bash
tar -cf backup.tar files/
```

### List

```bash
tar -tf backup.tar
```

### Extract

```bash
tar -xf backup.tar
```

### Create gzip archive

```bash
tar -czf backup.tar.gz files/
```

### Create bzip2 archive

```bash
tar -cjf backup.tar.bz2 files/
```

### Create xz archive

```bash
tar -cJf backup.tar.xz files/
```

### Extract while preserving permissions

```bash
tar -xpf backup.tar
```

---

# 🧠 RHCSA Memory Trick

Think of `tar` like this:

```text
        TAR
         │
 ┌───────┼────────┐
 │       │        │
 -c      -t       -x
 │       │        │
Create   List    Extract
```

Then compression:

```text
-z  = gzip
-j  = bzip2
-J  = xz
```

And:

```text
-f = filename
-v = verbose
-p = preserve permissions
```

So if the RHCSA asks:

> **Create a gzip-compressed archive of `/var/log` called `logs.tar.gz`:**

Answer:

```bash
tar -czf logs.tar.gz /var/log
```

If they ask:

> **List its contents:**

```bash
tar -tf logs.tar.gz
```

If they ask:

> **Extract it:**

```bash
tar -xf logs.tar.gz
```

That's the core of this chapter.
