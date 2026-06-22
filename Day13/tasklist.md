# Guided Exercise: Gain Superuser Access

## Objective

Switch to the superuser account to manage a Linux system and understand the differences between login and non-login shells when using `sudo` and `su`.

---

## 1. Connect to Server

Log in to **servera** as the **student** user.

```bash
ssh student@servera
```

Example:

```bash
student@workstation:~$ ssh student@servera
[student@servera ~]$
```

---

## 2. Explore the Student User Environment

### View User and Group Information

```bash
id
```

Output:

```bash
uid=1000(student) gid=1000(student) groups=1000(student),10(wheel)
```

### Display Current Working Directory

```bash
pwd
```

Output:

```bash
/home/student
```

### Check Environment Variables

```bash
echo $HOME
echo $PATH
```

Output:

```bash
/ home/student

/home/student/.local/bin:/home/student/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin
```

---

## 3. Become Root Using a Non-Login Shell

Use `sudo su`:

```bash
sudo su
```

Output:

```bash
[root@servera student]#
```

### Verify Current User

```bash
id
```

Output:

```bash
uid=0(root) gid=0(root) groups=0(root)
```

### Check Current Directory

```bash
pwd
```

Output:

```bash
/home/student
```

### Check Environment

```bash
echo $HOME
echo $PATH
```

Output:

```bash
/root

/root/.local/bin:/root/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin
```

### Exit Root Shell

```bash
exit
```

---

## 4. Become Root Using a Login Shell

Use `sudo su -`:

```bash
sudo su -
```

Output:

```bash
[root@servera ~]#
```

### Verify Current User

```bash
id
```

Output:

```bash
uid=0(root) gid=0(root) groups=0(root)
```

### Check Current Directory

```bash
pwd
```

Output:

```bash
/root
```

### Check Environment

```bash
echo $HOME
echo $PATH
```

Output:

```bash
/root

/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
```

### Exit Root Shell

```bash
exit
```

---

## 5. Difference Between `sudo su` and `sudo su -`

| Command     | Shell Type      | Working Directory            | Environment                            |
| ----------- | --------------- | ---------------------------- | -------------------------------------- |
| `sudo su`   | Non-login shell | Remains in current directory | Partially inherits current environment |
| `sudo su -` | Login shell     | Changes to `/root`           | Loads root user's login environment    |

---

## 6. Review Sudo Privileges

Display the sudo configuration for `operator1`:

```bash
sudo cat /etc/sudoers.d/operator1
```

Output:

```bash
operator1 ALL=(ALL) ALL
```

### Meaning

* `operator1` can execute commands as any user.
* Authentication is required via the `sudo` command.

---

## 7. Access Protected Files Using Sudo

### Check Permissions on `/var/log/messages`

```bash
ls -l /var/log/messages
```

Output:

```bash
-rw-------. 1 root root ... /var/log/messages
```

Only the root user can read or modify this file.

### Attempt Access Without Sudo

```bash
tail -5 /var/log/messages
```

Output:

```bash
tail: cannot open '/var/log/messages' for reading: Permission denied
```

### Access With Sudo

```bash
sudo tail -5 /var/log/messages
```

Output:

```bash
May 14 13:03:11 servera ...
May 14 13:03:11 servera ...
May 14 13:03:11 servera ...
May 14 13:03:11 servera ...
May 14 13:03:36 servera ...
```

---

## Key Commands Summary

```bash
id                    # Display user and group information
pwd                   # Show current directory
echo $HOME            # Show home directory
echo $PATH            # Show executable search path

sudo su               # Switch to root (non-login shell)
sudo su -             # Switch to root (login shell)
exit                  # Leave root shell

sudo cat /etc/sudoers.d/operator1
sudo tail -5 /var/log/messages
```

## Key Takeaways

* The **root user** has unrestricted access to the system.
* `sudo` allows authorized users to execute commands with elevated privileges.
* `sudo su` creates a **non-login root shell**.
* `sudo su -` creates a **login root shell** and loads root's environment.
* Protected files such as `/var/log/messages` require root privileges to access.
