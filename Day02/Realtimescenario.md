# 📘 Linux (RHEL) Quick Commands Guide

This README provides commonly used Linux commands for checking system information, user details, and performing basic administrative tasks.

***

## ✅ 1. Check Which Shell You Are Using

### Default shell:

```bash
echo $SHELL
```

### Current running shell:

```bash
ps -p $$
```

***

## ✅ 2. Check RHEL Version

```bash
cat /etc/redhat-release
```

Alternative methods:

```bash
cat /etc/os-release
```

```bash
hostnamectl
```

***

## ✅ 3. Check Users in the System

### List all users:

```bash
cat /etc/passwd
```

### Count total users:

```bash
wc -l /etc/passwd
```

### Check currently logged-in users:

```bash
who
```

```bash
w
```

***

## ✅ 4. Change Date, Time, and Timezone

### Change system date:

```bash
date -s "2026-06-16"
```

### ✅ Change system time:

```bash
date -s "20:30:00"
```

### ✅ Change both date & time together:

```bash
date -s "2026-06-16 20:30:00"
```

***

### ✅ Recommended (RHEL 7/8/9 using timedatectl)

### Set time:

```bash
timedatectl set-time 20:30:00
```

### Set date:

```bash
timedatectl set-time 2026-06-16
```

### List available timezones:

```bash
timedatectl list-timezones
```

### Set timezone:

```bash
timedatectl set-timezone Asia/Kolkata
```

### Enable automatic time sync (NTP):

```bash
timedatectl set-ntp true
```

### Verify date, time, timezone:

```bash
timedatectl
```

***

## ✅ 5. Check Last OS Upgrade & System Provision Info

### Last system boot:

```bash
who -b
```

### Package update / upgrade history (RHEL 7/8/9):

```bash
yum history
```

```bash
dnf history
```

### Recently installed packages:

```bash
rpm -qa --last | head
```

### Check system installation / provision time:

```bash
ls -l /
```

Better method:

```bash
tune2fs -l /dev/sda1 | grep 'Filesystem created'
```

> ⚠️ Replace `/dev/sda1` with your actual root partition.

***

## 📌 Quick Summary

* **Shell** → `echo $SHELL`
* **RHEL Version** → `cat /etc/redhat-release`
* **User Count** → `wc -l /etc/passwd`
* **Logged Users** → `who`
* **Set Date/Time** → `date -s` or `timedatectl set-time`
* **Set Timezone** → `timedatectl set-timezone`
* **Enable NTP** → `timedatectl set-ntp true`
* **Upgrade History** → `yum history`
* **Install Time** → `tune2fs -l`

***

## 🚀 Notes

* Some commands require **root/sudo privileges**
* Use `timedatectl` instead of `date -s` in production systems
* Enabling **NTP** is recommended to keep system time accurate
* Always verify disk/partition before running `tune2fs`

***
