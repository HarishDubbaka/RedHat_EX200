### Guided Exercise: Manage Local User Accounts

#### 1. Log in to `servera` and switch to root

```bash
student@workstation:~$ ssh student@servera
...output omitted...

[student@servera ~]$ sudo -i
[sudo] password for student: student

[root@servera ~]#
```

---

#### 2. Create the `operator1` user

```bash
[root@servera ~]# useradd operator1
```

Verify the user exists:

```bash
[root@servera ~]# tail /etc/passwd

...output omitted...
operator1:x:1003:1003::/home/operator1:/bin/bash
```

Set the password:

```bash
[root@servera ~]# passwd operator1
New password: redhat
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: redhat
passwd: password updated successfully
```

---

#### 3. Create `operator2` and `operator3`

Create `operator2`:

```bash
[root@servera ~]# useradd operator2

[root@servera ~]# passwd operator2
New password: redhat
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: redhat
passwd: password updated successfully
```

Create `operator3`:

```bash
[root@servera ~]# useradd operator3

[root@servera ~]# passwd operator3
New password: redhat
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: redhat
passwd: password updated successfully
```

---

#### 4. Add comments (GECOS field)

Update `operator1`:

```bash
[root@servera ~]# usermod -c "Operator One" operator1
```

Update `operator2`:

```bash
[root@servera ~]# usermod -c "Operator Two" operator2
```

Verify:

```bash
[root@servera ~]# tail /etc/passwd

...output omitted...
operator1:x:1003:1003:Operator One:/home/operator1:/bin/bash
operator2:x:1004:1004:Operator Two:/home/operator2:/bin/bash
operator3:x:1005:1005::/home/operator3:/bin/bash
```

---

#### 5. Delete `operator3` and its home directory

```bash
[root@servera ~]# userdel -r operator3
```

Verify removal:

```bash
[root@servera ~]# tail /etc/passwd

...output omitted...
operator1:x:1003:1003:Operator One:/home/operator1:/bin/bash
operator2:x:1004:1004:Operator Two:/home/operator2:/bin/bash
```

Verify home directory removal:

```bash
[root@servera ~]# ls -l /home

total 0
drwx------. 3 cloud-user cloud-user 74 May  9 17:17 cloud-user
drwx------. 4 devops     devops     90 Mar  6 16:24 devops
drwx------. 2 operator1  operator1  62 May  9 22:35 operator1
drwx------. 2 operator2  operator2  62 May  9 22:35 operator2
drwx------. 3 student    student    74 Mar  6 16:23 student
```

---

#### 6. Exit root and return to workstation

```bash
[root@servera ~]# exit
logout

[student@servera ~]$ exit
logout
Connection to servera closed.

student@workstation:~$
```

---

### RHCSA Commands Summary

| Task                           | Command                         |
| ------------------------------ | ------------------------------- |
| Create user                    | `useradd username`              |
| Set password                   | `passwd username`               |
| Modify comment field           | `usermod -c "Comment" username` |
| Delete user only               | `userdel username`              |
| Delete user and home directory | `userdel -r username`           |
| View user information          | `tail /etc/passwd`              |
| View home directories          | `ls -l /home`                   |

### Exam Tips

* `useradd` creates a user and home directory by default on RHEL.
* The comment field (GECOS) is modified using `usermod -c`.
* `userdel` removes only the account; `userdel -r` removes the account and home directory.
* User account information is stored in:

  * `/etc/passwd`
  * `/etc/shadow`
  * `/etc/group`
* Always verify user creation and deletion using `/etc/passwd` and `/home`.

**Objective achieved:** Created, modified, verified, and deleted local user accounts using command-line tools.
