# Guided Exercise: Manage Local Group Accounts

## Objective

Create, modify, and verify local groups and group memberships. Configure administrative privileges for a group using Sudo.

---

## 1. Create Supplementary Groups

### Create the `operators` group with GID 30000

```bash
[root@servera ~]# groupadd -g 30000 operators
```

### Create the `admin` group with the next available GID

```bash
[root@servera ~]# groupadd admin
```

### Verify the groups

```bash
[root@servera ~]# tail /etc/group
```

Example output:

```text
operators:x:30000:
admin:x:30001:
```

---

## 2. Add Users to the `operators` Group

### Add operator users

```bash
[root@servera ~]# usermod -aG operators operator1
[root@servera ~]# usermod -aG operators operator2
[root@servera ~]# usermod -aG operators operator3
```

### Verify memberships

```bash
[root@servera ~]# id operator1
uid=1003(operator1) gid=1003(operator1) groups=1003(operator1),30000(operators)

[root@servera ~]# id operator2
uid=1004(operator2) gid=1004(operator2) groups=1004(operator2),30000(operators)

[root@servera ~]# id operator3
uid=1005(operator3) gid=1005(operator3) groups=1005(operator3),30000(operators)
```

---

## 3. Add Users to the `admin` Group

### Add sysadmin users

```bash
[root@servera ~]# usermod -aG admin sysadmin1
[root@servera ~]# usermod -aG admin sysadmin2
[root@servera ~]# usermod -aG admin sysadmin3
```

### Verify memberships

```bash
[root@servera ~]# id sysadmin1
uid=1006(sysadmin1) gid=1006(sysadmin1) groups=1006(sysadmin1),30001(admin)

[root@servera ~]# id sysadmin2
uid=1007(sysadmin2) gid=1007(sysadmin2) groups=1007(sysadmin2),30001(admin)

[root@servera ~]# id sysadmin3
uid=1008(sysadmin3) gid=1008(sysadmin3) groups=1008(sysadmin3),30001(admin)
```

---

## 4. Verify Group Memberships in `/etc/group`

```bash
[root@servera ~]# tail /etc/group
```

Example output:

```text
operators:x:30000:operator1,operator2,operator3
admin:x:30001:sysadmin1,sysadmin2,sysadmin3
```

---

## 5. Grant Administrative Rights to the `admin` Group

### Create a Sudoers Configuration File

```bash
[root@servera ~]# echo "%admin ALL=(ALL) ALL" > /etc/sudoers.d/admin
```

### Set Correct Permissions

```bash
[root@servera ~]# chmod 440 /etc/sudoers.d/admin
```

### Verify the File

```bash
[root@servera ~]# cat /etc/sudoers.d/admin
%admin ALL=(ALL) ALL
```

**Explanation:**

```text
%admin ALL=(ALL) ALL
```

* `%admin` → Applies to all members of the admin group.
* `ALL` → Any host.
* `(ALL)` → Run commands as any user.
* `ALL` → Execute any command.

---

## 6. Test Administrative Access

### Switch to an Admin User

```bash
[root@servera ~]# su - sysadmin1
```

### Attempt to Read a Protected Log File

```bash
[sysadmin1@servera ~]$ tail /var/log/messages
```

Expected output:

```text
tail: cannot open '/var/log/messages' for reading: Permission denied
```

### Use Sudo to Read the File

```bash
[sysadmin1@servera ~]$ sudo tail /var/log/messages
[sudo] password for sysadmin1: redhat
```

Example output:

```text
May 18 04:59:40 servera su[8080]: (to root) sysadmin1 on pts/2
May 18 04:59:40 servera systemd[1]: Starting systemd-hostnamed.service - Hostname Service...
May 18 04:59:40 servera systemd[1]: Started systemd-hostnamed.service - Hostname Service.
```

The command succeeds because the `sysadmin1` user is a member of the `admin` group and has Sudo privileges.

---

## 7. Return to the Workstation

```bash
[sysadmin1@servera ~]$ exit
logout

[root@servera ~]# exit
logout

[student@servera ~]$ exit
logout

Connection to servera closed.
student@workstation:~$
```

---

## Key Commands Summary

| Command                                              | Purpose                                  |
| ---------------------------------------------------- | ---------------------------------------- |
| `groupadd -g 30000 operators`                        | Create group with specific GID           |
| `groupadd admin`                                     | Create group with default GID            |
| `usermod -aG group user`                             | Add user to supplementary group          |
| `id username`                                        | Verify user group membership             |
| `tail /etc/group`                                    | View group information                   |
| `echo "%admin ALL=(ALL) ALL" > /etc/sudoers.d/admin` | Grant sudo access                        |
| `chmod 440 /etc/sudoers.d/admin`                     | Secure sudoers file                      |
| `sudo command`                                       | Execute command with elevated privileges |

### RHCSA Exam Tips

* Always use `-aG` with `usermod` to avoid removing existing group memberships.
* Verify group memberships with `id` or `groups`.
* Store custom Sudo rules in `/etc/sudoers.d/` instead of editing `/etc/sudoers`.
* Ensure Sudo configuration files have `440` permissions.
* Use `visudo -c` to validate Sudo configuration files before testing.
