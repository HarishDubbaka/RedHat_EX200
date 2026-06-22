# Managing Local User Accounts 

## Objective

Create, modify, and delete local user accounts in Red Hat Enterprise Linux (RHEL).

---

# 1. Creating Users

## Syntax

```bash
useradd username
```

### Example

```bash
[root@servera ~]# useradd user01
```

### What Happens When `useradd` Runs?

1. Reads configuration from:

```bash
/etc/login.defs
```

2. Creates entries in:

```bash
/etc/passwd
/etc/shadow
/etc/group
/etc/gshadow
```

3. Creates home directory:

```bash
/home/username
```

4. Copies default files from:

```bash
/etc/skel
```

5. Creates mailbox:

```bash
/var/spool/mail/username
```

6. Updates subordinate ID files:

```bash
/etc/subuid
/etc/subgid
```

---

## Verify User Creation

```bash
id user01
```

Example:

```bash
uid=1000(user01) gid=1000(user01) groups=1000(user01)
```

---

## Set Password

A newly created user cannot log in until a password is assigned.

```bash
passwd user01
```

Example:

```bash
[root@servera ~]# passwd user01
New password:
Retype new password:
passwd: password updated successfully
```

---

# 2. User Account Files

| File                   | Purpose                  |
| ---------------------- | ------------------------ |
| `/etc/passwd`          | User account information |
| `/etc/shadow`          | Encrypted passwords      |
| `/etc/group`           | Group information        |
| `/etc/gshadow`         | Secure group passwords   |
| `/etc/login.defs`      | User creation defaults   |
| `/etc/default/useradd` | Default user settings    |

---

# 3. Modifying Existing Users

## Syntax

```bash
usermod [options] username
```

---

## Common Options

### Change Comment Field

```bash
usermod -c "SAP Basis Admin" user01
```

---

### Change Home Directory

```bash
usermod -d /data/user01 user01
```

Move existing files:

```bash
usermod -d /data/user01 -m user01
```

---

### Change Primary Group

```bash
usermod -g developers user01
```

---

### Add Supplementary Groups

```bash
usermod -aG wheel,dba user01
```

**Important:**

Use `-a` with `-G`.

Wrong:

```bash
usermod -G wheel user01
```

This replaces all existing supplementary groups.

Correct:

```bash
usermod -aG wheel user01
```

---

### Change Login Shell

```bash
usermod -s /bin/bash user01
```

View available shells:

```bash
cat /etc/shells
```

---

### Lock Account

```bash
usermod -L user01
```

---

### Unlock Account

```bash
usermod -U user01
```

---

# 4. Deleting Users

## Delete User Only

```bash
userdel user01
```

Removes:

* User entry from `/etc/passwd`

Keeps:

* Home directory
* Mail files

---

## Delete User and Home Directory

```bash
userdel -r user01
```

Removes:

* User account
* Home directory
* Mail spool

---

# 5. Security Risk of Deleting Users

Example:

Create user:

```bash
useradd user01
```

Home directory ownership:

```bash
drwx------ user01 user01 /home/user01
```

Delete user:

```bash
userdel user01
```

Ownership becomes:

```bash
drwx------ 1000 1000 /home/user01
```

Create another user with same UID:

```bash
useradd -u 1000 user02
```

Now:

```bash
user02 owns all files previously owned by user01
```

### Security Issue

New user gains access to old user's files.

---

## Find Unowned Files

```bash
find / -nouser -o -nogroup
```

---

# 6. Locking vs Deleting Accounts

### Recommended

```bash
usermod -L username
```

or

```bash
passwd -l username
```

Benefits:

* Preserves ownership
* Prevents login
* Avoids UID reuse issues

---

# 7. Managing Passwords

## Set Password

```bash
passwd username
```

Example:

```bash
passwd user01
```

---

## Lock Password

```bash
passwd -l user01
```

---

## Unlock Password

```bash
passwd -u user01
```

---

## Force Password Change at Next Login

```bash
passwd -e user01
```

---

# 8. Understanding UID Ranges

## UID 0

```text
root
```

Superuser account.

---

## UID 1–200

Static system accounts.

Examples:

```text
daemon
bin
mail
```

---

## UID 201–999

Dynamic service accounts.

Examples:

```text
apache
nginx
sshd
```

---

## UID 1000+

Regular users.

Examples:

```text
student
harish
user01
```

---

## Check UID

```bash
id user01
```

Output:

```bash
uid=1001(user01) gid=1001(user01)
```

---

# 9. Useful RHCSA Commands

```bash
useradd user01
passwd user01

usermod -aG wheel user01
usermod -L user01
usermod -U user01

userdel user01
userdel -r user01

id user01
getent passwd user01
getent group wheel

find / -nouser -o -nogroup
```

---

# RHCSA EX200 Exam Tips

✅ Create a user:

```bash
useradd user01
passwd user01
```

✅ Add user to wheel group:

```bash
usermod -aG wheel user01
```

✅ Lock account:

```bash
usermod -L user01
```

✅ Delete account and home directory:

```bash
userdel -r user01
```

✅ Verify user information:

```bash
id user01
getent passwd user01
```

### Exam Focus Areas

* Create users with `useradd`
* Modify users with `usermod`
* Set passwords using `passwd`
* Add supplementary groups using `-aG`
* Lock and unlock accounts
* Delete users safely with `userdel -r`
* Understand UID ranges and ownership issues after user deletion
