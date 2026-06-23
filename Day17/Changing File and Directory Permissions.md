# Changing File and Directory Permissions

## Objectives

- Change file and directory permissions using the `chmod` command.
- Understand symbolic and octal permission methods.
- Change file and directory ownership using `chown` and `chgrp`.

---

# Linux File Permissions

In Linux, you can configure permissions on files and directories to define specific access for different sets of users.

The primary utility used to modify permissions is:

```bash
chmod
```

`chmod` stands for **Change Mode**, where *mode* refers to file permissions.

### Key Features of chmod

- Changes file and directory permissions from the command line.
- Accepts a permission instruction followed by a file or directory name.
- Supports both:
  - Symbolic notation
  - Octal (numeric) notation

---

# Changing Permissions with the Symbolic Method

The symbolic method uses letters to represent user categories and permissions.

## Syntax

```bash
chmod WhoWhatWhich file|directory
```

Example:

```bash
chmod g+w file.txt
```

---

## Who (User Classes)

| Symbol | User Class | Description |
|----------|------------|-------------|
| u | User | File owner |
| g | Group | Members of the file's group |
| o | Other | All other users |
| a | All | User, Group, and Other |

> If no user class is specified, `a` (all) is assumed.

---

## What (Operation)

| Symbol | Operation | Description |
|----------|-----------|-------------|
| + | Add | Adds permissions |
| - | Remove | Removes permissions |
| = | Set Exactly | Replaces existing permissions |

---

## Which (Permissions)

| Symbol | Permission | Description |
|----------|------------|-------------|
| r | Read | Read file or list directory |
| w | Write | Modify file or directory |
| x | Execute | Run file or access directory |
| X | Special Execute | Execute only if directory or execute already exists |

---

## Examples

### Remove Read and Write Permission for Group and Others

```bash
chmod go-rw document.pdf
```

### Add Execute Permission for Everyone

```bash
chmod a+x myscript.sh
```

### Recursively Grant Group RWX Permissions

```bash
chmod -R g+rwx /home/user/myfolder
```

### Recursively Add Read and Write Access

Add execute only where appropriate:

```bash
chmod -R g+rwX demodir
```

> **Note:** The `X` option applies execute permissions only to directories or files that already have execute permissions set.

---

# Changing Permissions with the Octal Method

The octal method represents permissions using numbers.

## Syntax

```bash
chmod ### file|directory
```

Example:

```bash
chmod 644 sample.txt
```

---

## Permission Values

| Permission | Value |
|------------|--------|
| Read (r) | 4 |
| Write (w) | 2 |
| Execute (x) | 1 |

### Calculate Permissions

Start with `0`:

| Permission Set | Value |
|----------------|--------|
| Read | 4 |
| Write | 2 |
| Execute | 1 |

Examples:

| Octal | Permissions |
|--------|-------------|
| 7 | rwx |
| 6 | rw- |
| 5 | r-x |
| 4 | r-- |
| 0 | --- |

---

## Understanding 644

```text
User   Group  Other
 6       4      4
rw-     r--    r--
```

### Command

```bash
chmod 644 sample.txt
```

Result:

- User = Read + Write
- Group = Read
- Other = Read

---

## Understanding 750

```text
User   Group  Other
 7       5      0
rwx     r-x    ---
```

### Command

```bash
chmod 750 sampledir
```

Result:

- User = Read, Write, Execute
- Group = Read, Execute
- Other = No Access

---

# Changing File and Directory Ownership

By default:

- The creator becomes the file owner.
- The creator's primary group becomes the group owner.

To grant access based on group membership, ownership may need to be modified.

---

# chown Command

The `chown` command changes file ownership.

## Change File Owner

```bash
chown student app.conf
```

Changes ownership of `app.conf` to user `student`.

---

## Recursively Change Ownership

```bash
chown -R student Pictures
```

Changes ownership of:

- Pictures directory
- All files
- All subdirectories

---

## Change Group Ownership

```bash
chown :admins Pictures
```

Changes the group owner to `admins`.

---

## Change Owner and Group Together

```bash
chown visitor:guests Pictures
```

Changes:

- Owner = visitor
- Group = guests

---

# chgrp Command

The `chgrp` command changes only group ownership.

## Syntax

```bash
chgrp groupname filename
```

Example:

```bash
chgrp admins Pictures
```

Equivalent to:

```bash
chown :admins Pictures
```

---

# Ownership Rules

| Action | Regular User | Root User |
|----------|-------------|-----------|
| Change File Owner | ❌ No | ✅ Yes |
| Change File Group (member of group) | ✅ Yes | ✅ Yes |
| Change File Group (not a member) | ❌ No | ✅ Yes |

---

# Important Note

You may encounter the following syntax:

```bash
chown owner.group filename
```

Example:

```bash
chown student.admins file.txt
```

⚠️ **Avoid using this syntax.**

Because a period (`.`) is a valid character in usernames, Linux may misinterpret the command.

### Recommended Syntax

Always use a colon (`:`):

```bash
chown owner:group filename
```

Example:

```bash
chown student:admins file.txt
```

---

# Common Commands Summary

```bash
# Remove permissions
chmod go-rw file

# Add execute permission
chmod a+x script.sh

# Recursive permissions
chmod -R g+rwx directory

# Numeric permissions
chmod 644 file
chmod 750 directory

# Change owner
chown student file

# Change owner recursively
chown -R student directory

# Change group
chown :admins file

# Change owner and group
chown student:admins file

# Change group only
chgrp admins file
```

---

# Quick Permission Reference

| Octal | Symbolic | Meaning |
|--------|----------|----------|
| 777 | rwxrwxrwx | Full access to everyone |
| 755 | rwxr-xr-x | Common directory permission |
| 750 | rwxr-x--- | Owner full, group read/execute |
| 644 | rw-r--r-- | Common file permission |
| 600 | rw------- | Private file |
| 400 | r-------- | Read-only file |

---
