# Linux Partition Creation 

Creating partitions in Linux is the process of dividing a physical disk into separate logical sections. Each partition can have its own file system and purpose.

## Why Create Partitions?

* Separate operating system and user data
* Improve data management
* Enhance security
* Support dual boot
* Simplify backup and recovery

---

## Example Disk Layout

Suppose you have a new disk:

```text
/dev/sdb   (100 GB)
```

After partitioning:

```text
/dev/sdb1   30 GB   /
/dev/sdb2   20 GB   /home
/dev/sdb3   10 GB   swap
/dev/sdb4   40 GB   /data
```

---

# Step 1: Check Available Disks

Use either of these commands:

```bash
lsblk
```

or

```bash
fdisk -l
```

Example:

```text
NAME   SIZE TYPE
sda    100G disk
├─sda1  50G part
└─sda2  50G part

sdb    200G disk
```

Here, `/dev/sdb` is an empty disk.

---

# Step 2: Create a Partition

Using `fdisk`:

```bash
fdisk /dev/sdb
```

Interactive menu:

```text
Command (m for help): n
```

Choose:

```text
Partition type:
p   Primary
e   Extended
```

Press:

```text
p
```

Partition number:

```text
1
```

First sector:

```text
Press Enter
```

Last sector:

```text
+20G
```

Example:

```text
Last sector:
+20G
```

Save changes:

```text
Command:
w
```

The partition is now created.

---

# Step 3: Verify the Partition

```bash
lsblk
```

Output:

```text
sdb
└── sdb1 20G
```

---

# Step 4: Inform the Kernel

Sometimes Linux doesn't immediately detect the new partition.

Run:

```bash
partprobe
```

or

```bash
partx -u /dev/sdb
```

---

# Step 5: Create a File System

For XFS:

```bash
mkfs.xfs /dev/sdb1
```

For EXT4:

```bash
mkfs.ext4 /dev/sdb1
```

---

# Step 6: Create a Mount Point

Example:

```bash
mkdir /data
```

---

# Step 7: Mount the Partition

```bash
mount /dev/sdb1 /data
```

Verify:

```bash
df -h
```

Example:

```text
Filesystem   Size Used Avail Mounted on
/dev/sdb1     20G  100M   20G /data
```

---

# Step 8: Configure Persistent Mount

Find the UUID:

```bash
blkid /dev/sdb1
```

Example:

```text
UUID="e4a1-22d5-98ff"
```

Edit:

```bash
vi /etc/fstab
```

Add:

```text
UUID=e4a1-22d5-98ff   /data   xfs   defaults   0 0
```

Test the configuration:

```bash
mount -a
```

If there are no errors, the configuration is correct.

---

# Complete Flow Diagram

```text
New Disk
   │
   ▼
lsblk
   │
   ▼
fdisk /dev/sdb
   │
   ▼
Create Partition (n)
   │
   ▼
Write Changes (w)
   │
   ▼
partprobe
   │
   ▼
mkfs.xfs /dev/sdb1
   │
   ▼
mkdir /data
   │
   ▼
mount /dev/sdb1 /data
   │
   ▼
blkid
   │
   ▼
Edit /etc/fstab
   │
   ▼
mount -a
   │
   ▼
Partition Ready
```

---

# Common `fdisk` Commands

| Command | Description            |
| ------- | ---------------------- |
| `m`     | Show help menu         |
| `p`     | Print partition table  |
| `n`     | Create a new partition |
| `d`     | Delete a partition     |
| `t`     | Change partition type  |
| `l`     | List partition types   |
| `w`     | Write changes and exit |
| `q`     | Quit without saving    |

---

# Interview Questions

### 1. How do you create a partition in Linux?

* Use `fdisk`, `gdisk`, or `parted` to create the partition, then format it with a file system (`mkfs`), mount it, and add it to `/etc/fstab` for persistence.

### 2. What is the difference between MBR and GPT?

| MBR                          | GPT                             |
| ---------------------------- | ------------------------------- |
| Supports up to 2 TB disks    | Supports disks larger than 2 TB |
| Maximum 4 primary partitions | Up to 128 partitions by default |
| Older partitioning scheme    | Modern UEFI-compatible scheme   |

### 3. Why is `partprobe` used?

* It notifies the Linux kernel about changes to the partition table without requiring a reboot.

### 4. What is the difference between `mkfs.xfs` and `mkfs.ext4`?

* `mkfs.xfs` creates an XFS file system, which is optimized for large files and scalability.
* `mkfs.ext4` creates an EXT4 file system, which is widely supported and suitable for general-purpose workloads.

### 5. How do you make a partition mount automatically after reboot?

* Add an entry for the partition's UUID in `/etc/fstab` and verify it with `mount -a`.

> **Tip for RHCSA and SAP Basis:** XFS is the default file system on Red Hat Enterprise Linux 8 and 9. In SAP environments, administrators commonly create XFS file systems on new partitions or LVM logical volumes for SAP application directories and data storage.
