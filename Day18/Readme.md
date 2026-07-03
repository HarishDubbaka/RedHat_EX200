# Managing Default Permissions and Special Permissions in Linux 

## 🎯 Objectives

After completing this topic, you will be able to:

* Understand **Special Permissions** in Linux
* Configure **SetUID (SUID)**, **SetGID (SGID)**, and **Sticky Bit**
* Control **default permissions** using **umask**
* Calculate permissions for newly created files and directories
* Apply these concepts in real-world Linux administration

---

# Linux Permission Recap

Every Linux file and directory has **three standard permission sets**.

| Permission Type | Symbol | Meaning                             |
| --------------- | ------ | ----------------------------------- |
| Read            | r      | View file contents / list directory |
| Write           | w      | Modify file / create-delete files   |
| Execute         | x      | Run file / enter directory          |

These permissions are assigned to:

* **Owner (User)**
* **Group**
* **Others**

Example:

```bash
-rwxr-xr--
```

means

```
Owner : rwx
Group : r-x
Others: r--
```

---

# What are Special Permissions?

Linux provides three additional permissions called **Special Permissions**.

These permissions provide functionality beyond normal read, write, and execute permissions.

They are:

| Permission | Symbol | Octal |
| ---------- | ------ | ----- |
| SetUID     | s      | 4     |
| SetGID     | s      | 2     |
| Sticky Bit | t      | 1     |

---

# Overview of Special Permissions

| Permission | Files                  | Directories                         |
| ---------- | ---------------------- | ----------------------------------- |
| SetUID     | Executes as file owner | No effect                           |
| SetGID     | Executes as file group | New files inherit directory's group |
| Sticky Bit | No effect              | Only owner can delete files         |

---

# 1. SetUID (SUID)

## Definition

Normally,

```
User executes file
↓

Program runs with user's permissions
```

With **SetUID**,

```
User executes file
↓

Program runs with OWNER'S permissions
```

---

## Why is it needed?

Some programs require temporary root privileges.

Example:

```
passwd
```

Changing a password modifies

```
/etc/shadow
```

Only root can modify this file.

Instead of giving users root access,

Linux makes

```
passwd
```

run with **root privileges**.

---

## Example

```bash
ls -l /usr/bin/passwd
```

Output

```bash
-rwsr-xr-x
```

Notice

```
rws
```

instead of

```
rwx
```

The **s** replaces owner's execute permission.

---

## Meaning

```
Owner : root

User : harish

harish runs passwd

↓

passwd runs as root

↓

Password changes successfully
```

---

## Visual Representation

```
Without SUID

User
 │
 ▼
Program
 │
 ▼
Runs as User


With SUID

User
 │
 ▼
Program
 │
 ▼
Runs as Owner
```

---

## Setting SUID

### Symbolic

```bash
chmod u+s file
```

Example

```bash
chmod u+s myprogram
```

---

### Octal

```
4 = SUID
```

Example

```bash
chmod 4755 myprogram
```

```
4 = SUID

755 = normal permissions
```

---

# Identifying SUID

Example

```bash
-rwsr-xr-x
```

Lowercase **s**

means

```
Execute permission exists
```

Uppercase

```bash
-rwSr-xr-x
```

means

```
Execute permission missing
```

---

# 2. SetGID (SGID)

SetGID behaves differently for

* Files
* Directories

---

# SGID on Files

Programs execute with the **group ownership** of the file.

Instead of

```
User's primary group
```

the program runs with

```
File owner's group
```

---

Example

```bash
ls -l /usr/bin/plocate
```

Output

```bash
-rwxr-sr-x
```

Program executes as group

```
plocate
```

instead of user's group.

---

# SGID on Directories

This is the **most common usage**.

Normally

```
User creates file

↓

File gets user's primary group
```

Example

```
User : Alice

Primary Group : developers

Directory Group : project
```

Without SGID

```
newfile

Owner : Alice

Group : developers
```

---

With SGID enabled

```
newfile

Owner : Alice

Group : project
```

Every newly created file inherits

```
Directory Group
```

instead of user's group.

---

## Why use SGID?

Perfect for

* Development teams
* Shared folders
* Project directories
* Department folders

Everyone automatically shares the same group.

---

Example

```bash
ls -ld shared
```

Output

```bash
drwxr-sr-x
```

Notice

```
r-s
```

instead of

```
r-x
```

---

## Setting SGID

### Symbolic

```bash
chmod g+s shared
```

---

### Octal

```
2 = SGID
```

Example

```bash
chmod 2775 shared
```

---

# Visual Representation

Without SGID

```
Shared Folder

↓

Alice creates file

↓

Group = developers
```

---

With SGID

```
Shared Folder (Group = project)

↓

Alice creates file

↓

Group = project
```

---

# 3. Sticky Bit

Sticky Bit applies **only to directories**.

---

## Why is it needed?

Suppose everyone has write access.

Without Sticky Bit

```
Alice creates file

Bob deletes it

Charlie deletes another file
```

Anyone can delete anyone else's files.

---

With Sticky Bit

```
Alice creates file

↓

Only

Alice

or

root

can delete it.
```

---

## Best Example

```
/tmp
```

Every user writes files here.

But users cannot delete others' files.

---

Check

```bash
ls -ld /tmp
```

Output

```bash
drwxrwxrwt
```

Notice

```
t
```

instead of

```
x
```

---

## Setting Sticky Bit

### Symbolic

```bash
chmod o+t directory
```

---

### Octal

```
1 = Sticky
```

Example

```bash
chmod 1777 temp
```

---

# Removing Special Permissions

## Remove SUID

```bash
chmod u-s file
```

---

## Remove SGID

```bash
chmod g-s directory
```

---

## Remove Sticky Bit

```bash
chmod o-t directory
```

---

# Octal Values of Special Permissions

| Permission | Value |
| ---------- | ----- |
| SUID       | 4     |
| SGID       | 2     |
| Sticky     | 1     |

They are added as the **first digit**.

Examples

| Command           | Meaning     |
| ----------------- | ----------- |
| chmod 4755 file   | SUID        |
| chmod 2775 folder | SGID        |
| chmod 1777 folder | Sticky      |
| chmod 6755 file   | SUID + SGID |

---

# Default Permissions in Linux

When you create a new file or directory, Linux first assigns **base permissions**, then applies the **umask**.

## Initial Permissions

### Files

```
666

rw-rw-rw-
```

No execute permission by default.

---

### Directories

```
777

rwxrwxrwx
```

---

# Why aren't files executable?

Security.

Imagine downloading

```
virus.sh
```

If files were executable by default,

double-clicking could immediately run malicious code.

Linux requires you to explicitly enable execution:

```bash
chmod +x script.sh
```

---

# What is umask?

**umask** stands for

```
User File Creation Mask
```

It removes permissions from the default permissions.

Formula:

```
Final Permission

=

Default Permission

-

umask
```

It is not arithmetic subtraction but **bitwise masking**.

---

# Default Permission Calculation

## Files

```
666

minus

022

=

644
```

Result

```
rw-r--r--
```

---

## Directories

```
777

minus

022

=

755
```

Result

```
rwxr-xr-x
```

---

# Common umask Values

| umask | Files | Directories |
| ----- | ----- | ----------- |
| 022   | 644   | 755         |
| 027   | 640   | 750         |
| 002   | 664   | 775         |
| 077   | 600   | 700         |
| 000   | 666   | 777         |

---

# Checking Current umask

```bash
umask
```

Output

```bash
0022
```

---

# Changing umask Temporarily

```bash
umask 027
```

Verify

```bash
umask
```

---

# Example 1 (Default 022)

```bash
umask
```

```
0022
```

Create file

```bash
touch file1
```

Check

```bash
ls -l file1
```

Output

```bash
-rw-r--r--
```

---

Create directory

```bash
mkdir dir1
```

Output

```bash
drwxr-xr-x
```

---

# Example 2 (umask 000)

```bash
umask 000
```

Create

```bash
touch test
```

Permissions

```
rw-rw-rw-
```

Create directory

```
rwxrwxrwx
```

Everyone has full permissions.

---

# Example 3 (umask 027)

```bash
umask 027
```

Create

```bash
touch file
```

Permissions

```
rw-r-----
```

Create directory

```
rwxr-x---
```

Others receive no permissions.

---

# Permanent umask Configuration

System-wide defaults are defined in:

```bash
/etc/login.defs
```

and

```bash
/etc/bashrc
```

A user can override them in:

```bash
~/.bash_profile
```

or

```bash
~/.bashrc
```

After editing:

```bash
source ~/.bashrc
```

---

# RHEL Version Differences

| Version            | Default umask                                                                           |
| ------------------ | --------------------------------------------------------------------------------------- |
| RHEL 8 and earlier | `0002` for many regular users (UID ≥ 200 with matching primary group), otherwise `0022` |
| RHEL 9 and later   | `0022` for all users                                                                    |

This change in **RHEL 9+** provides more consistent and predictable default permissions across login and non-login shells.

---

# Summary

| Feature           | Purpose                                                                   | Typical Use Case                            |
| ----------------- | ------------------------------------------------------------------------- | ------------------------------------------- |
| **SetUID (SUID)** | Program runs with the file owner's privileges                             | `passwd` needs temporary root access        |
| **SetGID (SGID)** | Files inherit the directory's group or programs run with the file's group | Shared project directories                  |
| **Sticky Bit**    | Prevents users from deleting others' files in a shared directory          | `/tmp`                                      |
| **umask**         | Removes permissions from default file/directory creation permissions      | Enforce organization-wide security defaults |

---

# Quick Command Cheat Sheet

```bash
# View permissions
ls -l
ls -ld directory

# Set SUID
chmod u+s file
chmod 4755 file

# Set SGID
chmod g+s directory
chmod 2775 directory

# Set Sticky Bit
chmod o+t directory
chmod 1777 directory

# Remove special permissions
chmod u-s file
chmod g-s directory
chmod o-t directory

# Show current umask
umask

# Set temporary umask
umask 022
umask 027
umask 077

# Create test files/directories
touch file
mkdir directory
ls -l file
ls -ld directory
```

This combination of **special permissions** and **umask** is fundamental for Linux system administration, helping administrators balance security with collaboration by controlling both runtime privileges and default access to newly created files and directories.
