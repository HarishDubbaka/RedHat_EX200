# Interpreting Linux File Permissions

## Objectives

- List file system permissions on files and directories.
- Interpret how permissions control access by users and groups.

---

# Linux File-System Permissions

File permissions control access to files and directories in Linux. Every file and directory has:

- An **owner user**
- An **owner group**
- Permission settings for:
  - User (Owner)
  - Group
  - Others

The most specific permissions take precedence:

```text
User Permissions > Group Permissions > Other Permissions
```

---

# Permission Types

| Permission | Files | Directories |
|------------|--------|-------------|
| **r (read)** | View file contents | List directory contents |
| **w (write)** | Modify file contents | Create or delete files |
| **x (execute)** | Run file as a command | Enter directory (`cd`) |

---

# Directory Permission Behavior

### Read Only (`r`)

- Can list filenames.
- Cannot access file details or contents.

### Execute Only (`x`)

- Cannot list files.
- Can access a file if its name is known and permissions allow it.

### Read + Execute (`r-x`)

- Can list files.
- Can access readable files.

### Write + Execute (`wx`)

- Can create and delete files.
- Can remove files regardless of file ownership.

> **Note:** The sticky bit can prevent unauthorized file deletion.

---

# Important Notes

## Linux vs Windows Permissions

Linux permissions apply only to the file or directory where they are set.

Unlike NTFS:

- Subdirectories do not automatically inherit permissions.
- Directory permissions can restrict access to contents.

### Permission Comparison

| Linux | Windows Equivalent |
|---------|-------------------|
| Read on directory | List Folder Contents |
| Write on directory | Modify |
| Sticky Bit + Write | Similar to Windows Write |

---

# Root User Permissions

The Linux root user has full access to all files and directories.

However:

- SELinux policies can restrict root access based on security contexts.

---

# Group Collaboration Example

Suppose:

| User | Groups |
|--------|--------|
| alice | alice, web |
| bob | bob, wheel, web |

Files owned by the **web** group can be shared between Alice and Bob through group permissions.

---

# Viewing Permissions

## Files

```bash
ls -l test
```

Example:

```text
-rw-rw-r--. 1 student student 0 Mar 8 17:36 test
```

## Directories

```bash
ls -ld /home
```

Example:

```text
drwxr-xr-x. 5 root root 4096 Feb 31 22:00 /home
```

---

# Understanding the Output

Example:

```text
-rw-rw-r--. 1 student student 0 Mar 8 17:36 test
```

## File Type

First character indicates file type:

| Symbol | Type |
|---------|------|
| - | Regular file |
| d | Directory |
| l | Symbolic link |
| c | Character device |
| b | Block device |
| p | Named pipe |
| s | Socket |

---

## Permission Fields

```text
-rw-rw-r--
```

Split into three groups:

```text
rw- rw- r--
│   │   │
│   │   └── Others
│   └────── Group
└────────── User (Owner)
```

### Meaning

| Category | Permissions |
|-----------|------------|
| User | Read, Write |
| Group | Read, Write |
| Others | Read Only |

---

# Ownership Fields

Example:

```text
-rw-rw-r--. 1 student student 0 Mar 8 17:36 test
```

The two names after the link count indicate:

```text
student student
│       │
│       └── Group Owner
└────────── User Owner
```

---

# Permission Precedence

The most specific permissions apply.

Example:

If a user:

- Owns a file
- Is also a member of the owning group

Then:

```text
User permissions override group permissions.
```

Even if group permissions are less restrictive.

---

# Permission Scenario

## User and Group Memberships

| User | Group Memberships |
|--------|------------------|
| operator1 | operator1, consultant1 |
| database1 | database1, consultant1 |
| database2 | database2, operator2 |
| contractor1 | contractor1, operator2 |

---

## Directory Listing

```bash
ls -la
```

```text
drwxrwxr-x.  2 database1 consultant1   4096 Mar 4 10:23 .
drwxr-xr-x. 10 root      root          4096 Mar 1 17:34 ..
-rw-rw-r--.  1 operator1 operator1     1024 Mar 4 11:02 app1.log
-rw-r--rw-.  1 operator1 consultant1   3144 Mar 4 11:02 app2.log
-rw-rw-r--.  1 database1 consultant1  10234 Mar 4 10:14 db1.conf
-rw-r-----.  1 database1 consultant1   2048 Mar 4 10:18 db2.conf
```

---

# Permission Analysis

## db1.conf

```text
-rw-rw-r--
```

| Category | Access |
|-----------|---------|
| database1 | Read, Write |
| consultant1 | Read, Write |
| Others | Read Only |

---

# Access Scenarios

| Scenario | Result | Reason |
|-----------|---------|---------|
| operator1 modifies db1.conf | ✅ Allowed | Member of consultant1 group with rw permissions |
| database1 modifies db2.conf | ✅ Allowed | File owner |
| operator1 reads db2.conf | ✅ Allowed | consultant1 group has read access |
| operator1 writes db2.conf | ❌ Denied | consultant1 group lacks write permission |
| database2 reads db2.conf | ❌ Denied | Others have no permissions |
| contractor1 reads db2.conf | ❌ Denied | Others have no permissions |
| operator1 modifies app1.log | ✅ Allowed | Owner and only member of operator1 group |
| database2 modifies app2.log | ✅ Allowed | Others have write permission |
| database1 reads app2.log | ✅ Allowed | consultant1 group has read permission |
| database1 writes app2.log | ❌ Denied | Group permissions override others permissions |
| database1 deletes app1.log | ✅ Allowed | Has write permission on directory |
| database1 deletes app2.log | ✅ Allowed | Directory write permission allows deletion |

---

# Key Takeaways

- Permissions are assigned to:
  - User
  - Group
  - Others

- Permission Types:
  - `r` = Read
  - `w` = Write
  - `x` = Execute

- Permission precedence:

```text
User > Group > Others
```

- File permissions control access to files.
- Directory permissions control access to directory contents.
- Write permission on a directory allows file deletion.
- Root can access everything unless restricted by SELinux.
- Group permissions enable collaborative access to shared files.
````



