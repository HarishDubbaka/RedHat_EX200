## Repairing Damaged File Systems at Boot Time 

This RHEL topic is about **what happens when Linux cannot mount a file system during boot**, why it enters **Emergency Mode**, and how an administrator fixes the problem.

### 1. Why can file-system problems stop booting?

During boot, **systemd** reads:

```bash
/etc/fstab
```

This file tells Linux:

> “Which disks/file systems should be mounted, where, and with which options?”

For example:

```text
UUID=xxxx-xxxx   /data   xfs   defaults   0 0
```

During boot, systemd tries to mount `/data`.

If something is wrong, boot can stop.

Common problems:

| Problem               | Example                            |
| --------------------- | ---------------------------------- |
| Corrupted file system | XFS/ext4 has errors                |
| Wrong UUID            | UUID in `/etc/fstab` doesn't exist |
| Missing disk          | `/dev/sdb1` is unavailable         |
| Missing mount point   | `/data` directory doesn't exist    |
| Typo in `/etc/fstab`  | Incorrect device or mount options  |

---

# 2. What happens when mounting fails?

Suppose `/etc/fstab` contains:

```text
/dev/sda2   /mnt/mountfolder   xfs   defaults   0 0
```

But `/dev/sda2` doesn't exist.

systemd waits for the device:

```text
A start job is running for /dev/sda2
```

Eventually:

```text
Timed out waiting for device /dev/sda2.
```

Then:

```text
Dependency failed for /mnt/mountfolder
Dependency failed for Local File Systems
```

Finally, Linux enters:

```text
Emergency Mode
```

You may see:

```text
Give root password for maintenance
(or press Control-D to continue):
```

### Simple meaning

**Linux is basically saying:**

> “I cannot complete the boot because an important file system mentioned in `/etc/fstab` cannot be mounted. Please log in as root and fix it.”

---

# 3. What is `fsck`?

`fsck` means:

**File System Check**

It is a general command family used to check and repair file systems.

For example:

```bash
fsck /dev/sdb1
```

But the actual repair utility depends on the file-system type.

### ext4

For ext4:

```bash
fsck.ext4 /dev/sdb1
```

or:

```bash
e2fsck /dev/sdb1
```

For automatic repair of minor problems:

```bash
fsck.ext4 -p /dev/sdb1
```

`-p` means:

> Automatically repair problems that can safely be repaired without asking the administrator.

---

# 4. Important: XFS is different

For **XFS**, don't use `fsck` in the same way as ext4.

Use:

```bash
xfs_repair /dev/sdb1
```

Example:

```bash
xfs_repair /dev/sdb1
```

The `fsck.xfs` program exists mainly for compatibility with the boot process. It does **not perform the actual XFS repair**.

### Remember this for interviews

```text
ext4 → e2fsck / fsck.ext4
XFS  → xfs_repair
```

---

# 5. VERY IMPORTANT: Unmount before repairing

You should **not normally run `xfs_repair` or `e2fsck` against a mounted file system**.

First make sure the file system is unmounted.

For example:

```bash
umount /dev/sdb1
```

Then:

```bash
xfs_repair /dev/sdb1
```

or:

```bash
fsck.ext4 /dev/sdb1
```

### Why?

Because repairing a mounted file system can cause:

* Data corruption
* Loss of data
* Inconsistent file-system state

### Easy rule

> **Check/repair → Unmounted file system**

---

# 6. First step in Emergency Mode

Suppose you are dropped into:

```text
Emergency Mode
```

Log in with the **root password**.

Then determine what is currently mounted:

```bash
mount
```

You may see:

```text
/dev/sda1 on / type xfs (ro,relatime,seclabel,...)
```

Notice:

```text
(ro)
```

That means:

**Read Only**

---

# 7. Why is `/` sometimes read-only?

Your root file system may be mounted as:

```text
ro
```

which means:

```text
read-only
```

If `/` is read-only, you cannot modify:

```bash
/etc/fstab
```

because `/etc/fstab` is located on `/`.

So temporarily remount `/` as read/write:

```bash
mount -o remount,rw /
```

Now `/` becomes:

```text
rw
```

meaning:

**read/write**

You can then edit:

```bash
vi /etc/fstab
```

or:

```bash
vim /etc/fstab
```

---

# 8. Test `/etc/fstab`

After fixing the configuration, test all entries with:

```bash
mount --all
```

or:

```bash
mount -a
```

These commands attempt to mount the file systems defined in `/etc/fstab`, while skipping file systems that are already mounted.

---

# 9. Example: Mount point doesn't exist

Suppose:

```bash
mount --all
```

returns:

```text
mount: /mnt/mountfolder: mount point does not exist.
```

The problem is simply that:

```bash
/mnt/mountfolder
```

doesn't exist.

Create it:

```bash
mkdir -p /mnt/mountfolder
```

Then try again:

```bash
mount --all
```

---

# 10. Example: Wrong UUID

Suppose `/etc/fstab` contains:

```text
UUID=1234-ABCD /data xfs defaults 0 0
```

But that UUID doesn't exist.

Check available disks and UUIDs:

```bash
lsblk -f
```

You might find:

```text
sdb1   xfs   5678-EFGH
```

Then correct `/etc/fstab`:

```text
UUID=5678-EFGH /data xfs defaults 0 0
```

Test:

```bash
mount --all
```

---

# 11. Reload systemd

After modifying `/etc/fstab`, run:

```bash
systemctl daemon-reload
```

### Why?

systemd converts `/etc/fstab` entries into mount units.

When you change `/etc/fstab`, tell systemd:

> “The configuration has changed. Reload your unit configuration.”

So:

```bash
systemctl daemon-reload
```

Then:

```bash
mount --all
```

---

# 12. Final test — reboot

If:

```bash
mount --all
```

works without errors, reboot:

```bash
systemctl reboot
```

If the system boots normally, your repair was successful.

---

# Complete troubleshooting flow

Think of the whole process like this:

```text
             BOOT
               ↓
        systemd reads
          /etc/fstab
               ↓
       Attempts to mount
         file systems
               ↓
        ┌──────┴──────┐
        ↓             ↓
     SUCCESS        FAILURE
        ↓             ↓
     Continue     Emergency Mode
                      ↓
                Root password
                      ↓
                Check mounts
                      ↓
                   mount
                      ↓
          Is / mounted read-only?
                ↓          ↓
               YES         NO
                ↓           ↓
     mount -o remount,rw /  |
                ↓           |
                └─────┬─────┘
                      ↓
              Check /etc/fstab
                      ↓
           Fix UUID/device/path/
             mount-point errors
                      ↓
               If FS corrupted
                      ↓
        Unmount the file system
                      ↓
        ┌─────────────┴─────────────┐
        ↓                           ↓
       XFS                         ext4
        ↓                           ↓
 xfs_repair /dev/...       fsck.ext4 /dev/...
        ↓                           ↓
        └─────────────┬─────────────┘
                      ↓
             systemctl daemon-reload
                      ↓
                mount --all
                      ↓
                   SUCCESS?
                      ↓
                  Reboot
```

---

# 13. What is `nofail`?

You can add:

```text
nofail
```

to an `/etc/fstab` entry.

For example:

```text
/dev/sdb1 /backup xfs defaults,nofail 0 0
```

This tells Linux:

> “If this file system cannot be mounted, don't stop the boot process because of it.”

### But be careful!

Don't blindly use `nofail` for important production file systems.

For example, if an SAP application requires:

```text
/sapmnt
```

and you configure it with `nofail`, the OS might boot even though `/sapmnt` isn't mounted.

The SAP application could then start without the expected data/file system, potentially causing serious problems.

---

# 14. Key commands to remember

| Purpose                      | Command                    |
| ---------------------------- | -------------------------- |
| See mounted file systems     | `mount`                    |
| See disks and file systems   | `lsblk -f`                 |
| Remount `/` read/write       | `mount -o remount,rw /`    |
| Edit fstab                   | `vi /etc/fstab`            |
| Test all fstab entries       | `mount --all`              |
| Reload systemd configuration | `systemctl daemon-reload`  |
| Repair XFS                   | `xfs_repair /dev/device`   |
| Check/repair ext4            | `fsck.ext4 /dev/device`    |
| Automatic minor ext4 repair  | `fsck.ext4 -p /dev/device` |
| Reboot                       | `systemctl reboot`         |

---

## Interview-ready answer

**Q: What would you do if RHEL enters Emergency Mode because of an `/etc/fstab` issue?**

> First, I log in using the root password and use `mount` to identify the currently mounted file systems. If the root file system is mounted read-only, I remount it using `mount -o remount,rw /`. Then I check `/etc/fstab` for incorrect UUIDs, device names, mount points, or options. If the problem is file-system corruption, I make sure the affected file system is unmounted and use the appropriate repair utility, such as `xfs_repair` for XFS or `fsck.ext4` for ext4. After correcting the issue, I run `systemctl daemon-reload` and `mount --all` to test the configuration. If there are no errors, I reboot the system and verify that it boots normally.

### One-line memory trick

**Emergency Mode → Check `mount` → Fix `/etc/fstab` → Repair FS if needed → `daemon-reload` → `mount --all` → Reboot.**
