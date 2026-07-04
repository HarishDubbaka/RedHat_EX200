# Linux File Permissions Lab (umask, setgid, Sticky Bit)

## Step 1: Login as `operator1`

```bash
ssh operator1@servera
```

---

# Step 2: Create the directory

Create a directory named **operators** inside `/tmp`.

```bash
mkdir /tmp/operators
```

---

## Step 3: Change the group owner

Make the group owner **operators**.

```bash
chown :operators /tmp/operators
```

Verify:

```bash
ls -ld /tmp/operators
```

Output

```text
drwxrwx--- 2 operator1 operators ...
```

Explanation

* Owner = operator1
* Group = operators
* Others = No permissions

---

# Step 4: Check the current umask

```bash
umask
```

Output

```text
0007
```

### Meaning of umask 0007

Default permissions

Directories

```
777
```

Subtract umask

```
777
007
---
770
```

Files

```
666
```

Subtract umask

```
666
007
---
660
```

So,

Directories become

```
rwxrwx---
```

Files become

```
rw-rw----
```

---

# Step 5: Verify directory permissions

```bash
ls -ld /tmp/operators
```

Output

```text
drwxrwx---
```

Matches the **770** permission.

---

# Step 6: Set the setgid bit

```bash
chmod g+s /tmp/operators
```

Verify

```bash
ls -ld /tmp/operators
```

Output

```text
drwxrws---
```

Notice

```
s
```

instead of

```
x
```

### What does setgid do?

Normally

```
operator1 creates file
↓

Group owner = operator1's primary group
```

With **setgid**

```
Any file created inside

/tmp/operators

↓

Automatically belongs to

operators group
```

Very useful for shared project folders.

---

# Step 7: Add Sticky Bit

Current permission

```
2770
```

Add sticky bit

```bash
chmod 3770 /tmp/operators
```

Verify

```bash
ls -ld /tmp/operators
```

Output

```text
drwxrws--T
```

Explanation

```
3

=

2 (setgid)

+

1 (sticky bit)
```

So

```
3770

=

Sticky Bit

+

Setgid

+

770 permissions
```

---

## What does Sticky Bit do?

Normally

Anyone with write permission can delete files.

With Sticky Bit

Only

* File owner
* Directory owner
* root

can delete the file.

Example

```
/tmp

drwxrwxrwt
```

This is why users cannot delete each other's files in `/tmp`.

---

# Step 8: Create first file

```bash
touch /tmp/operators/ops_file1.txt
```

Go inside

```bash
cd /tmp/operators
```

Check

```bash
ls -l ops_file1.txt
```

Output

```text
-rw-rw---- 1 operator1 operators
```

Notice

Group owner

```
operators
```

because of **setgid**.

Permissions

```
660
```

because umask is

```
0007
```

---

# Step 9: Change umask

```bash
umask 0022
```

Meaning

Directories

```
777-022

=

755
```

Files

```
666-022

=

644
```

---

# Step 10: Create second file

```bash
touch ops_file2.txt
```

Verify

```bash
ls -l ops_file{1,2}.txt
```

Output

```text
-rw-rw---- 1 operator1 operators ops_file1.txt
-rw-r--r-- 1 operator1 operators ops_file2.txt
```

### Why are permissions different?

First file

Created with

```
umask 0007
```

Result

```
660
```

Second file

Created with

```
umask 0022
```

Result

```
644
```

Changing the umask affects only **newly created** files. Existing files keep their original permissions.

---

# Step 11: Test Sticky Bit

Become another user

```bash
su - operator2
```

Password

```
redhat
```

Try deleting

```bash
rm /tmp/operators/ops_file1.txt
```

Output

```text
rm: cannot remove ... : Operation not permitted
```

Reason

Sticky Bit prevents `operator2` from deleting a file owned by `operator1`, even though both users may have write access to the directory.

---

# Step 12: Exit

```bash
exit
exit
```

---

# Summary Table

| Command                           | Purpose                                            |
| --------------------------------- | -------------------------------------------------- |
| `mkdir /tmp/operators`            | Create directory                                   |
| `chown :operators /tmp/operators` | Change group ownership                             |
| `umask`                           | Display current umask                              |
| `chmod g+s directory`             | Enable setgid                                      |
| `chmod 3770 directory`            | Add sticky bit while preserving permissions        |
| `touch file`                      | Create file                                        |
| `umask 0022`                      | Change default permissions for new files           |
| `ls -ld directory`                | View directory permissions                         |
| `ls -l file`                      | View file permissions                              |
| `rm file`                         | Delete file (blocked by sticky bit for non-owners) |

## Key Concepts to Remember

* **umask** defines the default permissions for newly created files and directories.
* **setgid (`g+s`)** on a directory causes all new files and subdirectories to inherit the directory's group ownership.
* **Sticky Bit (`+t`)** on a directory allows only the file owner, the directory owner, or `root` to delete or rename files within it.
* Existing files are **not** affected when the umask changes; only files created afterward use the new umask.
* A permission like **3770** combines the **sticky bit (1)**, **setgid (2)**, and standard permissions **770**.
