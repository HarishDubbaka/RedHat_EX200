# Managing User Passwords 

## 1. Shadow Password File (`/etc/shadow`)

Linux stores encrypted passwords in **`/etc/shadow`**, which is readable only by root.

Example:

```bash
user03:$y$j9T$qybndwEWzHhr0uTGAwO4Q0$OuNgGC5Mx2RrCO4JOXtR2VJfTA8dLPxa7NV1tvhziHC:20222:0:99999:7:2:20282::
```

### Fields in `/etc/shadow`

| Field      | Description                                  |
| ---------- | -------------------------------------------- |
| user03     | Username                                     |
| Hash       | Encrypted password                           |
| 20222      | Last password change (days since 1970-01-01) |
| 0          | Minimum days before password can be changed  |
| 99999      | Maximum password age                         |
| 7          | Warning days before expiration               |
| 2          | Inactive days after password expiration      |
| 20282      | Account expiration date                      |
| Last field | Reserved                                     |

---

## 2. Password Hash Format

Example:

```bash
$y$j9T$qybndwEWzHhr0uTGAwO4Q0$OuNgGC5Mx2RrCO4JOXtR2VJfTA8dLPxa7NV1tvhziHC
```

### Components

| Part                   | Meaning                    |
| ---------------------- | -------------------------- |
| y                      | yescrypt algorithm         |
| j9T                    | yescrypt tuning parameters |
| qybndwEWzHhr0uTGAwO4Q0 | Salt                       |
| OuNgGC5...             | Password hash              |

### Common Hash Types

| Identifier | Algorithm                  |
| ---------- | -------------------------- |
| `$y$`      | yescrypt (RHEL 10 default) |
| `$6$`      | SHA-512                    |
| `$5$`      | SHA-256                    |

### Why Salt?

Salt prevents attackers from using precomputed password tables (rainbow tables) and makes password cracking more difficult.

---

## 3. Password Verification Process

1. User enters password.
2. System retrieves salt and hash from `/etc/shadow`.
3. Password + salt are hashed.
4. New hash is compared with stored hash.
5. If hashes match → login succeeds.

---

# 4. Configuring Password Aging

Use the **`chage`** command.

### Syntax

```bash
chage [options] username
```

### Example

Set:

* Minimum days = 0
* Maximum days = 90
* Warning = 7
* Inactive = 14

```bash
chage -m 0 -M 90 -W 7 -I 14 sysadmin05
```

---

## View Password Aging Information

```bash
chage -l sysadmin05
```

Example output:

```bash
Last password change                                    : May 14, 2025
Password expires                                        : Aug 12, 2025
Password inactive                                       : Aug 26, 2025
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 90
Number of days of warning before password expires       : 7
```

---

# 5. Set Account Expiration Date

### Find Current Date

```bash
date +%F
```

Output:

```bash
2025-05-14
```

### Calculate Future Date

```bash
date -d "+30 days" +%F
```

Output:

```bash
2025-06-13
```

### Set Expiration Date

```bash
chage -E $(date -d "+30 days" +%F) cloudadmin10
```

### Verify

```bash
chage -l cloudadmin10
```

---

# 6. Force Password Change at Next Login

```bash
chage -d 0 cloudadmin10
```

When the user logs in:

```bash
You are required to change your password immediately.
```

---

# 7. Default Password Aging for New Users

## `/etc/login.defs`

Important parameters:

```bash
PASS_MAX_DAYS   90
PASS_MIN_DAYS   0
PASS_WARN_AGE   7
```

### Purpose

| Parameter     | Description                  |
| ------------- | ---------------------------- |
| PASS_MAX_DAYS | Password validity            |
| PASS_MIN_DAYS | Minimum days before changing |
| PASS_WARN_AGE | Warning days                 |

---

## `/etc/default/useradd`

Example:

```bash
INACTIVE=21
EXPIRE=2025-12-31
```

### Meaning

| Setting  | Description                        |
| -------- | ---------------------------------- |
| INACTIVE | Lock account after password expiry |
| EXPIRE   | Default account expiration date    |

**Note:** Changes affect only newly created users.

---

# 8. Lock User Accounts

### Lock Password

```bash
usermod -L sysadmin03
```

Verify:

```bash
su - sysadmin03
```

Output:

```bash
Authentication failure
```

### Important

`usermod -L` blocks password logins only.

SSH key authentication may still work.

---

# 9. Completely Disable a User Account

Recommended method:

```bash
usermod -L -e 2025-06-01 cloudadmin10
```

### What it does

* Locks password
* Expires account
* Blocks SSH key login
* Blocks all logins

Ideal for:

* Former employees
* Temporary account suspension

---

# 10. Unlock User Account

Unlock password:

```bash
usermod -U cloudadmin10
```

Remove expiration:

```bash
usermod -e '' cloudadmin10
```

Or:

```bash
usermod -U -e '' cloudadmin10
```

---

# 11. Disable Interactive Login with `nologin`

Set shell to:

```bash
usermod -s /sbin/nologin newapp
```

Test:

```bash
su - newapp
```

Output:

```bash
This account is currently not available.
```

---

## Important Note

`/sbin/nologin`:

✅ Prevents interactive login

❌ Does NOT prevent:

* FTP access
* Mail access
* Web application authentication
* File uploads/downloads through services

---

# RHCSA Exam Commands Summary

| Task                   | Command                         |
| ---------------------- | ------------------------------- |
| View password policy   | `chage -l user`                 |
| Set min age            | `chage -m DAYS user`            |
| Set max age            | `chage -M DAYS user`            |
| Set warning days       | `chage -W DAYS user`            |
| Set inactivity days    | `chage -I DAYS user`            |
| Set account expiration | `chage -E YYYY-MM-DD user`      |
| Force password change  | `chage -d 0 user`               |
| Lock account           | `usermod -L user`               |
| Unlock account         | `usermod -U user`               |
| Expire account         | `usermod -e YYYY-MM-DD user`    |
| Remove expiration      | `usermod -e '' user`            |
| Set nologin shell      | `usermod -s /sbin/nologin user` |
| View shadow entry      | `sudo cat /etc/shadow`          |

---

## RHCSA Exam Tips

1. Password aging → **`chage`**
2. Lock/unlock account → **`usermod -L/-U`**
3. Force password reset → **`chage -d 0`**
4. Disable interactive login → **`/sbin/nologin`**
5. RHEL 10 default password hash → **yescrypt (`$y$`)**
6. Account expiration date → **`chage -E`** or **`usermod -e`**
7. Practice viewing and modifying password policies using **`chage -l`** frequently.
