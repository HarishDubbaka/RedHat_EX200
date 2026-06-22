# Gaining Superuser Access

## Objectives

- Switch to the superuser account to manage a Linux system.
- Grant other users superuser access using `sudo`.

---

# The Superuser (root)

In Red Hat Enterprise Linux (RHEL), the **root** user is the superuser with unrestricted access to the system.

### Root User Capabilities

- Install and remove software.
- Manage system files and directories.
- Create, modify, and delete user accounts.
- Configure system services and devices.
- Override file permissions.

### Important Notes

- In **RHEL 10**, the root account has **no valid password by default**.
- Red Hat recommends logging in as a regular user and using privilege escalation tools when administrative access is required.

### Security Risks

Running as root continuously increases security risks because:

- Any mistake can damage the system.
- Compromised applications gain full system control.
- Malware or vulnerabilities can affect the entire system.

---

# Switching User Accounts with `su`

The `su` command allows users to switch to another account.

## Switch to Another User

```bash
su - user02
```

Example:

```bash
user01@host:~$ su - user02
Password: user02_password
user02@host:~$
```

## Switch to Root

```bash
su -
```

Example:

```bash
user01@host:~$ su -
Password: root_password
root@host:~#
```

## Difference Between `su` and `su -`

| Command | Description |
|----------|-------------|
| `su` | Starts a non-login shell and partially preserves the current environment. |
| `su -` | Starts a login shell and loads the target user's environment. |

### Recommended

```bash
su -
```

because it provides a clean environment identical to a normal login session.

---

# Running Commands with `sudo`

In RHEL 10, administrative tasks are typically performed using `sudo`.

### Benefits of `sudo`

- Uses the user's own password.
- No need to know the root password.
- Logs all executed commands.
- Allows fine-grained access control.

### Example

Lock a user account:

```bash
sudo usermod -L user02
```

Output:

```bash
[sudo] password for user01:
```

---

# Sudo Authentication Cache

After successful authentication, `sudo` remembers your password for **5 minutes**.

Clear the cache:

```bash
sudo -k
```

---

# Failed Sudo Access

If a user is not authorized:

```bash
sudo tail /var/log/secure
```

Output:

```bash
user02 is not in the sudoers file.
This incident will be reported.
```

---

# Sudo Logging

All sudo activities are logged.

Example log entry:

```text
Mar 9 20:45:46 host sudo[2577]:
user01 : USER=root ; COMMAND=/sbin/usermod -L user02
```

Log file:

```bash
/var/log/secure
```

---

# Wheel Group

In RHEL 7 and later, members of the **wheel** group can use sudo.

Example configuration:

```bash
%wheel ALL=(ALL:ALL) ALL
```

---

# Getting a Root Shell

## Interactive Login Shell

```bash
sudo -i
```

Example:

```bash
ec2-user@host:~$ sudo -i
[sudo] password for ec2-user:
root@host:~#
```

## Non-Login Shell

```bash
sudo -s
```

---

# Configuring Sudo

The main sudo configuration file:

```bash
/etc/sudoers
```

Always edit using:

```bash
visudo
```

Benefits:

- Prevents simultaneous editing conflicts.
- Validates syntax before saving.

---

# Understanding Sudoers Entries

## Wheel Group Access

```bash
%wheel ALL=(ALL:ALL) ALL
```

### Breakdown

| Component | Meaning |
|------------|----------|
| `%wheel` | Applies to wheel group |
| `ALL` | Any host |
| `(ALL:ALL)` | Any user and group |
| `ALL` | Any command |

---

# Using `/etc/sudoers.d`

Additional sudo rules can be placed in:

```bash
/etc/sudoers.d/
```

### User-Specific Access

```bash
user01 ALL=(ALL) ALL
```

File:

```bash
/etc/sudoers.d/user01
```

---

### Group-Specific Access

```bash
%group01 ALL=(ALL) ALL
```

File:

```bash
/etc/sudoers.d/group01
```

---

### Allow a Specific Command

Allow members of the `games` group to run `id` as `operator`:

```bash
%games ALL=(operator) /bin/id
```

---

# Passwordless Sudo

Grant passwordless sudo:

```bash
ansible ALL=(ALL) NOPASSWD: ALL
```

### Use Cases

- Automation
- Ansible
- Cloud provisioning
- CI/CD pipelines

### Security Warning

Only use when:

- Account is highly protected.
- SSH key authentication is enforced.
- Access is tightly controlled.

---

# AWS Example

The default AWS RHEL AMI includes:

```bash
ec2-user ALL=(ALL) NOPASSWD: ALL
```

Characteristics:

- Root password locked.
- ec2-user password locked.
- SSH key authentication required.
- Passwordless sudo enabled.

---

# Best Practices

✅ Log in as a regular user.

✅ Use `sudo` instead of direct root login.

✅ Use `su -` instead of `su`.

✅ Grant minimum required privileges.

✅ Edit sudo rules with `visudo`.

✅ Monitor `/var/log/secure`.

❌ Avoid working continuously as root.

❌ Avoid unnecessary `NOPASSWD` access.

❌ Never share root credentials.

---

# Useful Commands Summary

```bash
su -
sudo command
sudo -i
sudo -s
sudo -k
visudo
```

---

# References

Manual Pages:

```bash
man su
man sudo
man visudo
man sudoers
man bash
```

Additional Documentation:

- GNU C Library Reference Manual
- Section 30.2: Persona of a Process
