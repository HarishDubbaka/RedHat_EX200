# 🐧 Linux Practice Lab: Creating Hard Links and Symbolic Links

## 🎯 Objective

Learn how to create and verify:

- Hard Links
- Symbolic (Soft) Links
- Inode Numbers
- Link Counts

---

## 1️⃣ Connect to `servera`

```bash
student@workstation:~$ ssh student@servera
```

---

## 2️⃣ Verify the Original File

Check the inode number and link count of `target.file`.

```bash
[student@servera ~]$ ls -li files/target.file
```

### Example Output

```bash
8838187 -rw-r--r--. 1 student student 11 May 27 13:15 files/target.file
```

### Explanation

| Field | Description |
|---------|-------------|
| `8838187` | Inode Number |
| `1` | Link Count (one filename points to the file) |

---

## 3️⃣ Create a Hard Link

Create `file.hardlink` pointing to `target.file`.

```bash
[student@servera ~]$ ln files/target.file links/file.hardlink
```

---

## 4️⃣ Verify the Hard Link

```bash
[student@servera ~]$ ls -li files/target.file links/file.hardlink
```

### Example Output

```bash
8838187 -rw-r--r--. 2 student student 11 May 27 13:15 files/target.file
8838187 -rw-r--r--. 2 student student 11 May 27 13:15 links/file.hardlink
```

### ✅ Observation

| Item | Value |
|--------|--------|
| Inode Number | Same for both files |
| Link Count | 2 |
| Data Location | Same |

### Why?

A hard link creates another filename that points to the **same inode (same data on disk)**.

---

## 5️⃣ Create a Symbolic Link

Create a symbolic link named `tempdir` that points to `/tmp`.

```bash
[student@servera ~]$ ln -s /tmp /home/student/tempdir
```

---

## 6️⃣ Verify the Symbolic Link

```bash
[student@servera ~]$ ls -l /home/student/tempdir
```

### Example Output

```bash
lrwxrwxrwx. 1 student student 4 May 27 13:53 /home/student/tempdir -> /tmp
```

### ✅ Observation

- `l` at the beginning indicates a symbolic link.
- `-> /tmp` shows the target location.

---

## 7️⃣ Return to Workstation

```bash
[student@servera ~]$ exit
logout
Connection to servera closed.

student@workstation:~$
```

---

# 📌 Hard Link vs Symbolic Link

| Feature | Hard Link | Symbolic Link |
|----------|-----------|--------------|
| Shares same inode | ✅ Yes | ❌ No |
| Can link directories | ❌ No | ✅ Yes |
| Works across filesystems | ❌ No | ✅ Yes |
| Survives original file deletion | ✅ Yes | ❌ No |
| Shows different inode | ❌ No | ✅ Yes |

---

# 🔍 Useful Commands

```bash
# View inode and link count
ls -li file

# Create hard link
ln source_file hardlink_name

# Create symbolic link
ln -s target symlink_name

# Identify symbolic links
ls -l

# Display detailed inode information
stat filename
```

---

# 🎯 RHCSA Exam Tips

### Hard Link

✔ Same inode number

✔ Same data blocks

✔ Survives deletion of original filename

```bash
ls -li
```

---

### Symbolic Link

✔ Different inode number

✔ Stores path to target

✔ Breaks if target is removed

```bash
ls -l
```

---

# 🧪 Quick Practice Challenge

Create and verify the following:

```bash
mkdir practice
touch practice/file1

# Create hard link
ln practice/file1 practice/file1.hard

# Create symbolic link
ln -s practice/file1 practice/file1.soft

# Verify
ls -li practice/*
ls -l practice/*
```

### Questions

1. Which files have the same inode?
2. What is the link count after creating the hard link?
3. What happens to the symbolic link if `file1` is deleted?
4. Which command displays symbolic link targets?

---

## 🏆 Key Memory Trick

> **Hard Link = Same Inode**
>
> **Symbolic Link = Same Path**

Remember:

```text
Hard Link  → File ↔ Inode
Soft Link  → File → Path → Inode
```

🚀 RHCSA candidates should always verify:

- Hard Links with `ls -li`
- Symbolic Links with `ls -l`
````
