If someone **disabled the BaseOS repository**, there are several places you can investigate. Unfortunately, **RHEL does not maintain a dedicated "who disabled this repository" log by default**. You'll need to use system logs and auditing (if enabled).

## 1. Check DNF History (First place to look)

DNF records package transactions and some repository-related operations.

```bash
dnf history
```

Or for more details:

```bash
dnf history list
```

> **Note:** Repository enable/disable operations are not always recorded as DNF transactions.

---

## 2. Check Shell History (Most Common)

If an administrator manually ran the command:

```bash
dnf config-manager --set-disabled baseos
```

or edited the repo file, check:

```bash
history
```

Or check another user's history:

```bash
cat /home/<username>/.bash_history
```

For root:

```bash
cat /root/.bash_history
```

If history hasn't been cleared, you'll find commands like:

```text
dnf config-manager --set-disabled baseos
vi /etc/yum.repos.d/redhat.repo
```

---

## 3. Check the Repository File Timestamp

Find which file was modified:

```bash
ls -l /etc/yum.repos.d/
```

Or:

```bash
stat /etc/yum.repos.d/redhat.repo
```

Example:

```text
Modify: 2026-07-06 14:30:12
```

This tells you **when** the repository file changed.

---

## 4. Compare Repository Configuration

Check whether BaseOS is disabled:

```bash
grep -A5 "\[baseos\]" /etc/yum.repos.d/*.repo
```

or

```bash
grep enabled /etc/yum.repos.d/*.repo
```

Look for:

```text
enabled=0
```

instead of

```text
enabled=1
```

---

## 5. Check Audit Logs (Best Option)

If **auditd** was enabled before the change:

```bash
ausearch -f /etc/yum.repos.d/redhat.repo
```

or

```bash
ausearch -f /etc/yum.repos.d
```

This may show:

* Who modified the file
* User ID (UID)
* Time
* Process

Example:

```text
uid=1001
auid=1001
comm="vi"
```

This is the most reliable way to identify **who** made the change.

---

## 6. Check `journalctl`

Search for DNF-related activity:

```bash
journalctl | grep dnf
```

or

```bash
journalctl -u dnf*
```

You might find entries indicating repository configuration changes or package operations.

---

## 7. If You're Using Configuration Management

If your environment uses tools like:

* Ansible
* Puppet
* Chef
* SaltStack
* Red Hat Satellite

The repository may have been disabled automatically.

Check:

```bash
systemctl status ansible-pull
```

or look for cron jobs:

```bash
crontab -l
ls /etc/cron*
```

---

## 8. Check if Someone Edited the Repo File Directly

If the repository file is under version control (rare unless configured):

```bash
git log
```

Otherwise, there is no built-in version history.

---

# Can Linux Tell You Exactly Who Disabled It?

| Method                        | Can identify **who**?         |
| ----------------------------- | ----------------------------- |
| `.bash_history`               | ✅ Sometimes                   |
| `auditd` (`ausearch`)         | ✅ Yes (if enabled beforehand) |
| `journalctl`                  | ⚠️ Sometimes                  |
| `dnf history`                 | ❌ Usually no                  |
| File timestamp (`stat`)       | ❌ Only when it changed        |
| Configuration management logs | ✅ If managed by automation    |

### I have a few questions to narrow it down:

1. **Which RHEL version are you using?** (RHEL 8, 9, or 10)
2. Is this system **registered with Red Hat Subscription Manager (`subscription-manager`)** or is it using custom `.repo` files?
3. When you run:

```bash
dnf repolist all
```

does it show:

```text
baseos   disabled
```

or is the repository missing entirely?

With that information, I can guide you to the exact logs and commands for your setup.
