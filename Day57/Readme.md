Configuring a Persistent System Journal
Configuring a Persistent System Journal
Objectives
Configure the system journal to persistently store log events after a reboot.
Configure automatic journal rotation to control disk usage.
Learn how to view journal entries from specific system boots.
Understand where systemd-journald configuration files are located.
1. Configure Persistent Journal Storage

By default, Red Hat Enterprise Linux stores system journal files in:

/run/log/journal


This storage is volatile, so the journal is normally cleared after a reboot.

To enable persistent journal storage, create the /var/log/journal directory:

root@host:~# mkdir /var/log/journal


Then flush the current journal to persistent storage:

root@host:~# journalctl --flush


If the operation succeeds, systemd-journald creates a machine-specific directory under /var/log/journal.

Check the directory:

root@host:~# ls /var/log/journal
4ec03abd2f7b40118b1b357f479b3112


The directory contains binary journal files with the .journal extension:

root@host:~# ls /var/log/journal/4ec03abd2f7b40118b1b357f479b3112
system.journal  user-1000.journal


These binary files contain structured and indexed journal entries.

How Persistent Storage Works

The default Storage setting is:

Storage=auto


With Storage=auto:

If /var/log/journal exists, persistent journal storage is enabled.
If /var/log/journal does not exist, the journal uses volatile storage under /run/log/journal.
2. View Journal Entries by Boot

Persistent journals allow you to review logs from previous system boots.

List Available Boots

Use --list-boots to display the boots recognized by journalctl:

root@host:~# journalctl --list-boots


Example:

  -6 27de... Wed 2025-06-04 20:04:32 EDT—Wed 2025-06-04 21:09:36 EDT
  -5 6a18... Tue 2025-06-26 08:32:22 EDT—Thu 2025-06-26 16:02:33 EDT
  -4 e2d7... Thu 2025-07-04 16:02:46 EDT—Fri 2025-07-04 20:59:29 EDT
  -3 45c3... Sat 2025-07-05 11:19:47 EDT—Sat 2025-07-05 11:53:32 EDT
  -2 dfae... Sat 2025-07-05 13:11:13 EDT—Sat 2025-07-05 13:27:26 EDT
  -1 e754... Sat 2025-07-05 13:58:08 EDT—Sat 2025-07-05 14:10:53 EDT
   0 ee2c... Mon 2025-07-07 09:56:45 EDT—Mon 2025-07-07 12:57:21 EDT


The boot numbering works as follows:

Option	Meaning
-b	Current boot
-b 0	Current boot
-b -1	Previous boot
-b -2	Two boots before the current boot
-b 1	Boot numbered 1, when available
View the Current Boot
root@host:~# journalctl -b

View the Previous Boot

This is particularly useful when troubleshooting a crash:

root@host:~# journalctl -b -1

View a Specific Boot

For example:

root@host:~# journalctl -b 1


Or:

root@host:~# journalctl -b 2


Important: The numeric boot argument is meaningful only when the requested boot exists in the persistent journal.

3. Troubleshoot a System Crash

When troubleshooting a system crash, do not inspect only the current boot.

Instead, query the boot that occurred immediately before the crash:

root@host:~# journalctl -b -1


Negative boot numbers are useful because they identify boots relative to the current boot.

For example:

-b 0   -> current boot
-b -1  -> previous boot
-b -2  -> two boots ago

4. System Journal Rotation

Persistent journals are not stored forever.

systemd-journald automatically rotates journal files to control disk usage.

By default:

Journal rotation occurs monthly.
The journal is limited to a percentage of the file system.
The journal does not consume more than 10% of the file system by default.
At least 15% of the file system is kept free by default.
Journal size limits apply to both runtime and persistent journals.

You can customize these limits in:

/etc/systemd/journald.conf

5. Check Current Journal Size Limits

Use the following command:

root@host:~# journalctl | grep -E 'Runtime Journal|System Journal'


Example output:

Jun 10 22:22:57 localhost systemd-journald[263]: Runtime Journal (/run/log/journal/93c43a50cada46e39819e0da68f790a2) is 4.2M, max 34.1M, 29.8M free.
Jun 10 22:23:01 host systemd-journald[740]: Runtime Journal (/run/log/journal/da54d37671ac4429bcdfacfe7bebb5a1) is 4.2M, max 34.1M, 29.8M free.
Jun 10 22:30:50 host systemd-journald[740]: System Journal (/var/log/journal/da54d37671ac4429bcdfacfe7bebb5a1) is 8M, max 997.3M, 989.3M free.


The grep expression:

Runtime Journal|System Journal


uses | as an OR operator. Therefore, it matches lines containing either:

Runtime Journal
System Journal

This allows you to see the current limits for both volatile and persistent journal storage.

6. Journal Configuration Files

Journal configuration can normally be customized using:

/etc/systemd/journald.conf


You can also create drop-in configuration files under:

/etc/systemd/journald.conf.d/

RHEL 10 Configuration

Starting with RHEL 10, /etc/systemd/journald.conf is not necessarily present after installation.

The default configuration is provided by:

/usr/lib/systemd/journald.conf


Do not edit /usr/lib/systemd/journald.conf directly.

Instead, copy it to /etc/systemd:

root@host:~# cp /usr/lib/systemd/journald.conf /etc/systemd/journald.conf


Then edit the copy:

root@host:~# vi /etc/systemd/journald.conf


Configuration files under /etc take precedence over the default configuration under /usr/lib.

7. Configure Automatic Journal Rotation

The following parameters control journal storage and rotation.

SystemMaxUse and RuntimeMaxUse

These parameters specify the maximum amount of file system space that the journal can use.

SystemMaxUse=
RuntimeMaxUse=


By default, the maximum is approximately 10% of the file system, with a maximum cap of 4 GB.

SystemMaxUse applies to persistent journals in /var/log/journal.
RuntimeMaxUse applies to volatile journals in /run/log/journal.
SystemMaxFileSize and RuntimeMaxFileSize

These parameters specify the maximum size of an individual journal file before rotation occurs:

SystemMaxFileSize=
RuntimeMaxFileSize=


By default, the value is approximately one-eighth of the corresponding MaxUse value, subject to the applicable size limits.

SystemKeepFree and RuntimeKeepFree

These parameters specify how much file system space must remain free:

SystemKeepFree=
RuntimeKeepFree=


By default, at least 15% of the file system is kept free, subject to the applicable limits.

8. Configuration Example

A customized persistent journal configuration might look like this:

[Journal]

Storage=auto

SystemMaxUse=1G
SystemMaxFileSize=128M
SystemKeepFree=500M

RuntimeMaxUse=500M
RuntimeMaxFileSize=64M
RuntimeKeepFree=250M


Note: Only files ending in .journal or .journal~ are considered journal files.

9. Apply Configuration Changes

After modifying /etc/systemd/journald.conf, restart the systemd-journald service:

root@host:~# systemctl restart systemd-journald


Verify that the service is running:

root@host:~# systemctl status systemd-journald

10. Quick Reference
Enable Persistent Storage
mkdir /var/log/journal
journalctl --flush

Check Persistent Journal Files
ls /var/log/journal
ls /var/log/journal/*/

List System Boots
journalctl --list-boots

View Current Boot
journalctl -b

View Previous Boot
journalctl -b -1

View a Specific Boot
journalctl -b 1

Check Journal Size Limits
journalctl | grep -E 'Runtime Journal|System Journal'

Edit Journal Configuration
vi /etc/systemd/journald.conf

Restart journald
systemctl restart systemd-journald

11. Configuration Workflow

The complete process can be summarized as:

Create /var/log/journal
        |
        v
Run journalctl --flush
        |
        v
Persistent journal enabled
        |
        v
Configure journald.conf
        |
        v
Set SystemMaxUse / SystemMaxFileSize / SystemKeepFree
        |
        v
Restart systemd-journald
        |
        v
Journal persists and rotates automatically

12. Useful Man Pages

For additional information, consult:

man journald.conf
man systemd-journald.service
man journalctl

References
Red Hat Enterprise Linux 10 documentation: Risk Reduction and Recovery Operations
journald.conf(5)
systemd-journald.service(8)
journalctl(1)
