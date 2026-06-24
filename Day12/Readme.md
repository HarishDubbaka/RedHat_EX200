#  Managing Local Users and Groups 

## 🎯 Objective

* Understand Linux users and groups.
* Gain superuser access.
* Create, modify, and delete local users and groups.
* Manage passwords and password policies.

---

# 1. Linux Users and Groups

## What is a User?

A user account provides:

* Authentication (login access)
* Security boundaries
* Ownership of files and processes

Every process and file belongs to a user.

### User Types

| User Type            | Description                        |
| -------------------- | ---------------------------------- |
| **Root (Superuser)** | Full administrative access (UID 0) |
| **System Users**     | Used by services and daemons       |
| **Regular Users**    | Used by human users                |

---

## Display User Information

### Show Current User Information

```bash
id
```

Example:

```bash
uid=1000(user01) gid=1000(user01) groups=1000(user01)
```

### Show Another User's Information

```bash
id user02
```

---

## View File Ownership

### File

```bash
ls -l file.txt
```

Output:

```bash
-rw-rw-r-- 1 user01 user01 0 Feb 5 file.txt
```

### Directory

```bash
ls -ld Documents
```

Output:

```bash
drwxrwxr-x 2 user01 user01 6 Feb 5 Documents
```

---

## View Process Ownership

```bash
ps awwu
```

Example:

```bash
USER     PID COMMAND
root    1690 agetty
user01  3072 bash
```

The **USER** column shows process ownership.

---

# 2. Local User Accounts

User information is stored in:

```bash
/etc/passwd
```

View file:

```bash
cat /etc/passwd
```

Example:

```bash
user01:x:1000:1000:User One:/home/user01:/bin/bash
```

### Fields in `/etc/passwd`

| Field        | Description          |
| ------------ | -------------------- |
| user01       | Username             |
| x            | Password placeholder |
| 1000         | UID                  |
| 1000         | Primary GID          |
| User One     | Comment/Full Name    |
| /home/user01 | Home Directory       |
| /bin/bash    | Login Shell          |

---

# 3. Linux Groups

Groups help multiple users share access to resources.

Group information is stored in:

```bash
/etc/group
```

View file:

```bash
cat /etc/group
```

Example:

```bash
group01:x:10000:user01,user02,user03
```

### Fields in `/etc/group`

| Field                | Description          |
| -------------------- | -------------------- |
| group01              | Group Name           |
| x                    | Password Placeholder |
| 10000                | GID                  |
| user01,user02,user03 | Members              |

---

# 4. Primary and Supplementary Groups

## Primary Group

Every user has exactly one primary group.

Example:

```bash
user01:x:1000:1000
```

Primary GID = 1000

---

## Supplementary Groups

Additional groups that grant extra permissions.

Example:

```bash
id user01
```

Output:

```bash
uid=1001(user01)
gid=1003(user01)
groups=1003(user01),10(wheel),10000(group01)
```

Here:

* Primary Group → user01
* Supplementary Groups → wheel, group01

---

# RHCSA Exam Commands

| Command           | Purpose                    |
| ----------------- | -------------------------- |
| `id`              | Display user and group IDs |
| `whoami`          | Show current username      |
| `groups`          | Show group memberships     |
| `ls -l`           | Show file ownership        |
| `ls -ld`          | Show directory ownership   |
| `ps awwu`         | Show process owners        |
| `cat /etc/passwd` | View local users           |
| `cat /etc/group`  | View local groups          |

---

# Quick RHCSA Exam Facts

✅ Root user has **UID 0**

✅ User information stored in:

```bash
/etc/passwd
```

✅ Group information stored in:

```bash
/etc/group
```

✅ Every user has:

* One Primary Group
* Zero or More Supplementary Groups

✅ Files and processes are owned by users.

✅ Linux uses:

* **UID** = User ID
* **GID** = Group ID

---

## Memory Tip

```text
UID → User Identity
GID → Group Identity

/etc/passwd → Users
/etc/group  → Groups

root = UID 0
```

