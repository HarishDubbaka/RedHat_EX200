## Configuring a Persistent System Journal

### Objectives

- Configure a system journal to persistently store the log events after a reboot and to rotate the log events automatically.

### System Journal Storage

By default, Red Hat Enterprise Linux stores the system journal files in the `/run/log/journal` directory, and the system clears the system journal after a reboot. You can configure `systemd-journald` service to keep the journal files persistently, so that you can review journal events across reboots of the system.

#### Configure a Persistent System Journal

Configure the `systemd-journald` service as follows to preserve system journals persistently across a reboot:

1. Create the `/var/log/journal` directory.
   ```
   root@host:~# mkdir /var/log/journal
   ```
2. Run the `journalctl --flush` command to flush the current journal to storage. If the `systemd-journald` service successfully flushes the current journal, then the service creates subdirectories in the `/var/log/journal` directory.
   ```
   root@host:~# journalctl --flush
   ```
3. The subdirectory in the `/var/log/journal` directory has hexadecimal characters in its long name and contains files with the `.journal` extension.
   ```
   root@host:~# ls /var/log/journal
   4ec03abd2f7b40118b1b357f479b3112
   ```
4. The `.journal` binary files store structured and indexed journal entries.
   ```
   root@host:~# ls /var/log/journal/4ec03abd2f7b40118b1b357f479b3112
   system.journal  user-1000.journal
   ```

### Important

This service works because the default setting of the `Storage` parameter in the journal configuration is set to `Storage=auto`. This parameter means that if the `/var/log/journal` directory exists, then persistent storage is automatically enabled, but if the storage does not get enabled, then volatile storage in the `/run/log/journal` directory is used instead.

The journal configuration files are discussed in more detail later in this section.

#### Review Journal Entries Relative to Boot Time

Although the system journals persist after a reboot, the `journalctl` command output includes entries from the current system boot as well as from the previous system boots. To limit the output to a specific system boot, use the `journalctl` command `-b` option.

The following `journalctl` command retrieves the entries from the first system boot only:

```
root@host:~# journalctl -b 1
...output omitted...
```

The following `journalctl` command retrieves the entries from the second system boot only. The argument is meaningful only if the system was rebooted at least twice:

```
root@host:~# journalctl -b 2
...output omitted...
```

You can list the system boot events that the `journalctl` command recognizes, by using the `--list-boots` option.

```
root@host:~# journalctl --list-boots
  -6 27de... Wed 2025-06-04 20:04:32 EDT—Wed 2025-06-04 21:09:36 EDT
  -5 6a18... Tue 2025-06-26 08:32:22 EDT—Thu 2025-06-26 16:02:33 EDT
  -4 e2d7... Thu 2025-07-04 16:02:46 EDT—Fri 2025-07-04 20:59:29 EDT
  -3 45c3... Sat 2025-07-05 11:19:47 EDT—Sat 2025-07-05 11:53:32 EDT
  -2 dfae... Sat 2025-07-05 13:11:13 EDT—Sat 2025-07-05 13:27:26 EDT
  -1 e754... Sat 2025-07-05 13:58:08 EDT—Sat 2025-07-05 14:10:53 EDT
   0 ee2c... Mon 2025-07-07 09:56:45 EDT—Mon 2025-07-07 12:57:21 EDT
```

The following `journalctl` command retrieves the entries from the current system boot only:

```
root@host:~# journalctl -b
...output omitted...
```

### Note

When debugging a system crash with a persistent journal, you must limit the journal query to the reboot before the crash happened. You can use the `journalctl` command `-b` option with a negative number to indicate how many earlier system boots to include in the output.

For example, the `journalctl` command `-b -1` option limits the output to only the previous boot.

### System Journal Rotation

The advantage of persistent system journals is that the historical data is available immediately at boot. However, even with a persistent journal, the system does not keep all data forever.

The journal has a built-in log rotation mechanism that triggers monthly. In addition, the system does not allow the journals to get larger than 10% of the file system that they are on, or to leave less than 15% of the file system free. You can modify these values for both the runtime and persistent journals in the `/etc/systemd/journald.conf` configuration file. The `systemd-journald` process logs the current limits on the size of the journal when it starts.

The following command output shows the journal entries that reflect the current size limits:

```
[user@host ~]$ journalctl | grep -E 'Runtime Journal|System Journal'
Jun 10 22:22:57 localhost systemd-journald[263]: Runtime Journal (/run/log/journal/93c43a50cada46e39819e0da68f790a2) is 4.2M, max 34.1M, 29.8M free.
Jun 10 22:23:01 host systemd-journald[740]: Runtime Journal (/run/log/journal/da54d37671ac4429bcdfacfe7bebb5a1) is 4.2M, max 34.1M, 29.8M free.
Jun 10 22:30:50 host systemd-journald[740]: System Journal (/var/log/journal/da54d37671ac4429bcdfacfe7bebb5a1) is 8M, max 997.3M, 989.3M free.
...output omitted...
```

### Note

In the previous `grep` command, the vertical bar (`|`) symbol in the expression acts as an *or* operator. That is, the `grep` command matches any line with either the `Runtime Journal` string or the `System Journal` string from the `journalctl` command output. This command fetches the current size limits on the volatile (`Runtime`) journal store and on the persistent (`System`) journal store.

#### Locating Journal Configuration Files

System administrators normally configure journal settings by editing the `/etc/systemd/journald.conf` configuration file, or by adding configuration files with a `.conf` suffix to the `/etc/systemd/journald.conf.d` directory.

However, a RHEL system might not have those files or that directory. In that case, lower-priority locations that are documented in the `journal.conf`(5) man page are checked for settings, and if none of those files or directories exist, then compiled-in default settings are used.

### Important

Starting from RHEL 10, the `/etc/systemd/journald.conf` file is not present at installation time. The `systemd-journald` service reads its default settings from the `/usr/lib/systemd/journald.conf` configuration file.

To change the journal's behavior, do not edit the `/usr/lib/systemd/journald.conf` configuration file directly. Instead, copy the `/usr/lib/systemd/journald.conf` file to the `/etc/systemd` directory, and then uncomment and change the appropriate settings in the resulting `/etc/systemd/journald.conf` file. Configuration settings in the `/etc/systemd/journald.conf` file take precedence over the default settings in the `/usr/lib/systemd/journald.conf` file.

#### Configure Automatic Journal Rotation

To configure automatic journal rotation for persistent or runtime-only journals, set the following parameters in the `/etc/systemd/journald.conf` file.

Prefix the settings with `System` for persistent journals in the `/var/log/journal` directory.

Prefix the settings with `Runtime` for volatile journals in the `/run/log/journal` directory.

**SystemMaxUse and RuntimeMaxUse**

The maximum amount of file system space that the journal can use. This value is 10% of the total file system space by default, and is capped at 4 GB.

**SystemMaxFileSize and RuntimeMaxFileSize**

The maximum file size of a journal before it rotates. This value is one eighth of the `SystemMaxUse/RuntimeMaxUse` value by default, and is capped at 128 M, and usually supports keeping seven rotated journal files as history. If journal compact mode is enabled (the default), this value is capped at 4 GB.

**SystemKeepFree and RuntimeKeepFree**

The minimum free file system space that must remain before archived journals are dropped. At least 15% of the total file system space is always kept free by default. This value is capped at 4 GB.

### Important

Only files with the `.journal` or `.journal~` suffixes are considered journal files.

After editing the `/etc/systemd/journald.conf` configuration file, restart the `systemd-journald` service to apply the configuration changes.

```
root@host:~# systemctl restart systemd-journald
```

### References

`journald.conf`(5) and `systemd-journald.service`(8) man pages

For more information, refer to the *Troubleshooting Problems by Using Log Files* chapter of the *Red Hat Enterprise Linux 10 Risk Reduction and Recovery Operations* guide at [https://docs.redhat.com/en/documentation/red\_hat\_enterprise\_linux/10/html-single/risk\_reduction\_and\_recovery\_operations/index#troubleshooting-problems-by-using-log-files](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html-single/risk_reduction_and_recovery_operations/index#troubleshooting-problems-by-using-log-files)

generate as linkedin cover image
