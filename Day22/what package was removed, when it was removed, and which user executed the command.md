Yes. In **RHEL**, you can determine **what package was removed, when it was removed, and which user executed the command** (if auditing or shell history is available). Different logs provide different pieces of information.

---

# 1. DNF Transaction History (First place to check)

This is the primary source for package installation and removal history.

```bash
dnf history
```

Example:

```text
ID | Command Line       | Date and Time     | Action
------------------------------------------------------
20 | remove httpd       | 2026-07-07 10:30  | Remove
19 | install httpd      | 2026-07-06 15:10  | Install
```

To view details of a specific transaction:

```bash
dnf history info 20
```

Example:

```text
Transaction ID : 20
Begin time     : Tue Jul 7 10:30:15 2026
Command Line   : remove httpd
Packages Altered:
  Removed httpd-2.4.63-1.el10.x86_64
```

**What you get:**

* Package removed
* Date and time
* Command executed
* Packages affected

**What you don't get:**

* The username of the person who ran the command.

---

# 2. DNF Log Files

Check the DNF log:

```bash
cat /var/log/dnf.rpm.log
```

Or search for removals:

```bash
grep Removed /var/log/dnf.rpm.log
```

Example:

```text
2026-07-07T10:30:20 Removed: httpd-2.4.63-1.el10.x86_64
```

This shows:

* Package name
* Timestamp

It does **not** show the user.

---

# 3. Shell History (Shows who typed the command)

If each administrator has a separate account:

```bash
cat /home/username/.bash_history
```

For the root user:

```bash
cat /root/.bash_history
```

Search for the command:

```bash
grep "dnf remove" ~/.bash_history
```

Example:

```text
sudo dnf remove httpd
```

This can help identify the user **if** they used their own account and the history hasn't been cleared.

---

# 4. Sudo Log (Best way to identify the user)

If the package was removed using `sudo`, check the authentication log.

On RHEL, use:

```bash
journalctl _COMM=sudo
```

Or:

```bash
grep sudo /var/log/secure
```

Example:

```text
Jul 7 10:30:12 server sudo: harish : TTY=pts/0 ; \
PWD=/home/harish ; USER=root ; \
COMMAND=/usr/bin/dnf remove httpd
```

This tells you:

* **User:** `harish`
* **Time:** `Jul 7 10:30`
* **Command:** `dnf remove httpd`

This is one of the best places to identify who executed the command.

---

# 5. Audit Logs (Most Reliable)

If the Linux Audit daemon (`auditd`) is enabled:

Search for DNF executions:

```bash
ausearch -x dnf
```

Or search by command:

```bash
ausearch -k software
```

Example:

```text
type=EXECVE
uid=1001
auid=1001
exe=/usr/bin/dnf
```

This can identify:

* User ID (UID)
* Login user (AUID)
* Command
* Time
* Terminal

---

# 6. System Journal

You can also search the system journal:

```bash
journalctl | grep dnf
```

Or filter by time:

```bash
journalctl --since "2026-07-07 10:00"
```

---

# Where to Look

| Information Needed    | Command / Location                                  |
| --------------------- | --------------------------------------------------- |
| Package removed       | `dnf history`                                       |
| Detailed transaction  | `dnf history info <ID>`                             |
| DNF package log       | `/var/log/dnf.rpm.log`                              |
| Shell command history | `/home/<user>/.bash_history`, `/root/.bash_history` |
| Who ran `sudo`        | `journalctl _COMM=sudo` or `/var/log/secure`        |
| Audit information     | `ausearch -x dnf`                                   |
| System journal        | `journalctl`                                        |

---

# SAP BASIS Interview Scenario

**Question:** *A package was accidentally removed from a RHEL server. How would you find out what happened and who removed it?*

**Answer:**

1. Check the package transaction history:

   ```bash
   dnf history
   ```
2. View the transaction details:

   ```bash
   dnf history info <transaction_id>
   ```
3. Review the DNF log:

   ```bash
   grep Removed /var/log/dnf.rpm.log
   ```
4. Identify the user by checking:

   ```bash
   journalctl _COMM=sudo
   ```

   or

   ```bash
   grep sudo /var/log/secure
   ```
5. If `auditd` is enabled, use:

   ```bash
   ausearch -x dnf
   ```

This approach lets you determine **what was removed, when it happened, and—if logging is configured appropriately—who executed the removal command**.
