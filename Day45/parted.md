# Linux Partition Management 

## Step 1: View Existing Partitions

Display the current partition table:

```bash
parted /dev/sda print
```

Example:

```text
Model: QEMU HARDDISK
Disk: 53.7GB
Partition Table: msdos

Number  Start   End     Size    Type
1       1049kB  10.7GB  10.7GB  primary
2       10.7GB  53.7GB  42.9GB  primary
```

You can also start interactive mode:

```bash
parted /dev/sda
```

Inside `parted`:

```bash
(parted) print
(parted) quit
```

---

# Step 2: Choose the Partition Table

A new disk must first be initialized with a partition table.

### Create an MBR (msdos) partition table

```bash
parted /dev/sdb mklabel msdos
```

### Create a GPT partition table

```bash
parted /dev/sdb mklabel gpt
```

> **Warning:** `mklabel` erases the existing partition table and makes previous partitions inaccessible. 

---

# Step 3: Create an MBR Partition

Start `parted`:

```bash
parted /dev/sdb
```

Create a partition:

```text
(parted) mkpart
Partition type? primary
File system type? xfs
Start? 2048s
End? 1000MB
```

Exit:

```text
(parted) quit
```

Wait for the device node:

```bash
udevadm settle
```

Or create it in one command:

```bash
parted /dev/sdb mkpart primary xfs 2048s 1000MB
```

---

# Step 4: Create a GPT Partition

For GPT, partitions are given a **name**.

```bash
parted /dev/sdb
```

```text
(parted) mkpart
Partition name? userdata
File system type? xfs
Start? 2048s
End? 1000MB
```

Exit:

```text
(parted) quit
```

Then:

```bash
udevadm settle
```

Non-interactive example:

```bash
parted /dev/sdb mkpart userdata xfs 2048s 1000MB
```

---

# Step 5: Format the Partition

Create an XFS file system:

```bash
mkfs.xfs /dev/sdb1
```

Or create an EXT4 file system:

```bash
mkfs.ext4 /dev/sdb1
```

---

# Step 6: Mount the File System

Create a mount point:

```bash
mkdir /mnt/data
```

Mount it:

```bash
mount /dev/sdb1 /mnt/data
```

Verify:

```bash
mount | grep sdb1
```

Example:

```text
/dev/sdb1 on /mnt/data type xfs
```

---

# Step 7: Configure Persistent Mounting

Check the UUID:

```bash
lsblk --fs
```

or

```bash
blkid
```

Edit `/etc/fstab`:

```bash
vi /etc/fstab
```

Example entry:

```text
UUID=a8063676-44dd-409a-b584-68be2c9f5570   /mnt/data   xfs   defaults   0   0
```

Reload systemd after updating `/etc/fstab`:

```bash
systemctl daemon-reload
```

> Red Hat recommends using **UUIDs** instead of device names (such as `/dev/sdb1`) because UUIDs remain constant even if device names change. 

---

# Step 8: Delete a Partition

Open `parted`:

```bash
parted /dev/sdb
```

View partitions:

```text
(parted) print
```

Delete partition 1:

```text
(parted) rm 1
```

Exit:

```text
(parted) quit
```

Or run:

```bash
parted /dev/sdb rm 1
```

---

# Complete Workflow

```text
New Disk (/dev/sdb)
        │
        ▼
Create Partition Table
(mklabel msdos/gpt)
        │
        ▼
Create Partition
(mkpart)
        │
        ▼
udevadm settle
        │
        ▼
Format Partition
(mkfs.xfs)
        │
        ▼
Create Mount Point
(mkdir)
        │
        ▼
Mount
(mount)
        │
        ▼
Verify
(lsblk --fs)
        │
        ▼
Update /etc/fstab
        │
        ▼
systemctl daemon-reload
        │
        ▼
Persistent Mount
```

---

# Important `parted` Commands

| Command         | Purpose                              |
| --------------- | ------------------------------------ |
| `print`         | Display the partition table          |
| `mklabel msdos` | Create an MBR partition table        |
| `mklabel gpt`   | Create a GPT partition table         |
| `mkpart`        | Create a new partition               |
| `rm`            | Remove a partition                   |
| `quit`          | Exit `parted`                        |
| `unit s`        | Display values in sectors            |
| `help mkpart`   | Show syntax and options for `mkpart` |

---

# RHCSA & SAP Basis Interview Questions

1. **What is the difference between MBR and GPT?**

   * **MBR (msdos):** Supports up to 2 TB disks and a maximum of four primary partitions.
   * **GPT:** Supports disks larger than 2 TB, many more partitions, and is the modern standard for UEFI systems.

2. **Why is `udevadm settle` used after creating a partition?**

   * It waits until the kernel detects the new partition and creates the corresponding device node in `/dev`.

3. **Why should UUIDs be used in `/etc/fstab`?**

   * UUIDs are stable and do not change if device names (such as `/dev/sdb`) are reassigned.

4. **Does `mkpart` create a file system?**

   * No. It only creates the partition and optionally labels its intended file-system type. You must still run `mkfs.xfs` or `mkfs.ext4` to format it.

5. **Why is XFS commonly used in Red Hat Enterprise Linux?**

   * XFS is Red Hat's recommended default file system due to its scalability and performance, making it well suited for enterprise workloads, including SAP systems. 
