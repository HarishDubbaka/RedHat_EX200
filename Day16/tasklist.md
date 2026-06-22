# Managing User Passwords and Password Policy (RHCSA Lab)

## Objective

* Lock and unlock user accounts.
* Expire and unexpire user accounts.
* Configure password aging policies.
* Force password changes at next login.
* Set default password expiration policies for new users.

---

## 1. Log in to `servera` as `student`

```bash
student@workstation:~$ ssh student@servera
[student@servera ~]$
```

---

## 2. Lock and Expire the `operator1` Account

Lock the account and immediately expire it:

```bash
[student@servera ~]$ sudo usermod -L -e 1 operator1
[sudo] password for student: student
```

### Verify Login Failure

```bash
[student@servera ~]$ su - operator1
Password: redhat
su: Authentication failure
```

---

## 3. Unlock the Account and Remove Expiration

```bash
[student@servera ~]$ sudo usermod -U -e '' operator1
```

### Verify Login Success

```bash
[student@servera ~]$ su - operator1
Password: redhat
[operator1@servera ~]$
```

Return to the student user:

```bash
[operator1@servera ~]$ exit
logout
[student@servera ~]$
```

---

## 4. Configure Password Expiration (90 Days)

Become root:

```bash
[student@servera ~]$ sudo -i
[sudo] password for student: student
[root@servera ~]#
```

Set maximum password age to 90 days:

```bash
[root@servera ~]# chage -M 90 operator1
```

Verify:

```bash
[root@servera ~]# chage -l operator1
```

Example output:

```text
Last password change                    : May 14, 2025
Password expires                        : Aug 12, 2025
Password inactive                       : never
Account expires                         : never
Minimum number of days between password change : 0
Maximum number of days between password change : 90
Number of days of warning before password expires : 7
```

---

## 5. Force Password Change at Next Login

```bash
[root@servera ~]# chage -d 0 operator1
```

Exit root:

```bash
[root@servera ~]# exit
logout
[student@servera ~]$
```

---

## 6. Change Password on First Login

Switch to `operator1`:

```bash
[student@servera ~]$ su - operator1
Password: redhat
```

Expected prompt:

```text
You are required to change your password immediately (administrator enforced).
Current password: redhat
New password: forsooth123
Retype new password: forsooth123
```

After successful change:

```bash
[operator1@servera ~]$
```

Return to student:

```bash
[operator1@servera ~]$ exit
logout
[student@servera ~]$
```

---

## 7. Set Account Expiration to 180 Days in the Future

Become root:

```bash
[student@servera ~]$ sudo -i
[sudo] password for student: student
[root@servera ~]#
```

Determine the future date:

```bash
[root@servera ~]# date -d "+180 days" +%F
```

Example:

```text
2025-11-10
```

Set account expiration:

```bash
[root@servera ~]# chage -E 2025-11-10 operator1
```

Verify:

```bash
[root@servera ~]# chage -l operator1
```

Example:

```text
Last password change                    : May 14, 2025
Password expires                        : Aug 12, 2025
Password inactive                       : never
Account expires                         : Nov 10, 2025
Minimum number of days between password change : 0
Maximum number of days between password change : 90
Number of days of warning before password expires : 7
```

---

## 8. Configure Default Password Expiration for New Users

Edit `/etc/login.defs`:

```bash
[root@servera ~]# vim /etc/login.defs
```

Locate the password aging section and modify:

```text
PASS_MAX_DAYS   180
PASS_MIN_DAYS   0
PASS_MIN_LEN    8
PASS_WARN_AGE   7
```

> Note: These settings affect only newly created users. Existing users retain their current password aging configuration unless changed with `chage`.

---

## 9. Return to Workstation

Exit root:

```bash
[root@servera ~]# exit
logout
```

Exit servera:

```bash
[student@servera ~]$ exit
logout
Connection to servera closed.
```

Back on workstation:

```bash
student@workstation:~$
```

---

## RHCSA Commands Summary

| Task                             | Command                        |
| -------------------------------- | ------------------------------ |
| Lock user                        | `usermod -L username`          |
| Unlock user                      | `usermod -U username`          |
| Expire account immediately       | `usermod -e 1 username`        |
| Remove expiration date           | `usermod -e '' username`       |
| Set password max age             | `chage -M DAYS username`       |
| Force password change            | `chage -d 0 username`          |
| Set account expiration date      | `chage -E YYYY-MM-DD username` |
| View password policy             | `chage -l username`            |
| Calculate future date            | `date -d "+180 days" +%F`      |
| Configure defaults for new users | Edit `/etc/login.defs`         |
