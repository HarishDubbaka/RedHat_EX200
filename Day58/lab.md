
1. Setting the correct **time zone**.
2. Configuring **NTP time synchronization** using `chronyd`.

### 1. Start the lab

On `workstation`:

```bash
student@workstation:~$ lab start logs-maintain
```

### 2. Log in to `servera`

```bash
student@workstation:~$ ssh student@servera
student@servera:~$ sudo -i
[sudo] password for student: student
```

You are now `root`.

---

## 3. Set the Haiti time zone

First, you can identify Haiti's time zone:

```bash
[root@servera ~]# tzselect
```

For Haiti, the required time zone is:

```text
America/Port-au-Prince
```

Set it permanently:

```bash
[root@servera ~]# timedatectl set-timezone America/Port-au-Prince
```

Verify:

```bash
[root@servera ~]# timedatectl
```

You should see:

```text
Time zone: America/Port-au-Prince
```

### Important

`tzselect` **only helps you identify the time zone**. It does not permanently change the system time zone.

The command that actually changes it is:

```bash
timedatectl set-timezone America/Port-au-Prince
```

---

# 4. Configure the NTP server

RHEL uses **chronyd** for network time synchronization.

Edit:

```bash
[root@servera ~]# vi /etc/chrony.conf
```

Add:

```text
server classroom.example.com iburst
```

### What does `iburst` do?

`iburst` tells `chronyd` to send several requests quickly when starting synchronization.

This helps the system synchronize its clock **faster initially**.

---

# 5. Enable NTP synchronization

Run:

```bash
[root@servera ~]# timedatectl set-ntp true
```

Check:

```bash
[root@servera ~]# timedatectl
```

Look for:

```text
System clock synchronized: yes
NTP service: active
```

If it says:

```text
System clock synchronized: no
```

wait a few seconds and run:

```bash
timedatectl
```

again.

---

# 6. Verify the actual NTP source

Run:

```bash
[root@servera ~]# chronyc sources -v
```

Look for:

```text
^* classroom.example.com
```

### The most important symbol: `*`

```text
^* classroom.example.com
```

The `*` means:

> **This is the current NTP source being used to synchronize the system clock.**

For example:

```text
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* classroom.example.com         3   9   377   469   +107us[ +130us] +/- 13ms
```

---

## Quick exam-style workflow

Remember this sequence:

```bash
# 1. Identify timezone
tzselect

# 2. Set timezone
timedatectl set-timezone America/Port-au-Prince

# 3. Configure NTP
vi /etc/chrony.conf

# Add:
server classroom.example.com iburst

# 4. Enable NTP
timedatectl set-ntp true

# 5. Verify
timedatectl

# 6. Verify NTP source
chronyc sources -v
```

### Easy way to remember

**Timezone → `timedatectl`**

**NTP configuration → `/etc/chrony.conf`**

**Enable synchronization → `timedatectl set-ntp true`**

**Check synchronization → `chronyc sources -v`**

**`*` = currently synchronized source** ⭐

Finally, return to the workstation:

```bash
[root@servera ~]# exit
[student@servera ~]$ exit
student@workstation:~$
```

Then complete the lab:

```bash
student@workstation:~$ lab finish logs-maintain
```
