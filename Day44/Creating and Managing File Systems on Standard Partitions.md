# 📘 RHCSA EX200 – Creating and Managing File Systems on Standard Partitions 

This lesson is one of the most important topics in RHCSA because almost every Linux server requires additional storage. Before learning the commands (`fdisk`, `parted`, `mkfs`, `mount`), you should understand **how Linux views a storage device**.

---

# 📖 Real-World Scenario

Imagine your company purchases a **2 TB SSD** for a Linux server.

The server will be used for:

* Operating System
* SAP Applications
* Database
* User Files
* Backup
* Log Files

Would you store everything in one single storage area?

**No.**

Instead, you divide the disk into separate sections so that each type of data has its own space.

This process is called **Disk Partitioning**.

---

# What Happens When You Connect a New Disk?

Suppose you attach a new disk to your Linux server.

Initially, the disk is completely empty.

```text
New Disk

/dev/sdb

+----------------------------------------+
|                                        |
|              Empty Disk                |
|                                        |
+----------------------------------------+
```

Linux detects the hardware, but you **cannot store files yet**.

Why?

Because the disk has:

* ❌ No partitions
* ❌ No filesystem
* ❌ No mount point

It is just raw storage.

---

# What Must Be Done Before Using the Disk?

Linux requires several preparation steps before the disk becomes usable.

```text
New Disk
    │
    ▼
Create Partition
    │
    ▼
Create Filesystem
    │
    ▼
Create Mount Point
    │
    ▼
Mount Filesystem
    │
    ▼
Ready to Store Files
```

This entire process is called **Storage Provisioning**.

---

# Step 1 – Create Partitions

Suppose the new disk is **500 GB**.

Initially:

```text
500 GB Disk

+--------------------------------------+
|                                      |
|          Unallocated Space           |
|                                      |
+--------------------------------------+
```

Now divide it into three sections.

```text
500 GB Disk

+----------+------------+--------------+
|100 GB    |200 GB      |200 GB        |
|Partition1|Partition2  |Partition3    |
+----------+------------+--------------+
```

These sections are called **partitions**.

Linux identifies them as:

```text
/dev/sdb1
/dev/sdb2
/dev/sdb3
```

Notice:

* Disk → `/dev/sdb`
* First partition → `/dev/sdb1`
* Second partition → `/dev/sdb2`

---

# Why Not Keep One Large Partition?

Imagine everything is stored together.

```text
500 GB

Operating System
Applications
Logs
Database
Users
Backups
```

Now suppose log files suddenly grow.

```text
Logs

490 GB
```

The disk becomes full.

Result:

* Applications stop.
* Database may fail.
* Users cannot log in.
* Linux may become unstable.

---

Now consider separate partitions.

```text
100 GB /

200 GB /home

100 GB /var

100 GB Backup
```

If log files fill `/var`, only that partition becomes full.

The operating system and user data continue to work.

This is one of the biggest advantages of partitioning.

---

# What is a Filesystem?

Creating a partition only divides the disk into sections. It does **not** make it usable for storing files.

A **filesystem** organizes how data is stored and retrieved.

Without a filesystem, Linux has no way to understand where files begin or end.

### Analogy

Think of a partition as an empty notebook.

```text
Notebook

Blank Pages
```

You cannot quickly find information because there are no page numbers or structure.

Now add:

* Page numbers
* Chapters
* Index

```text
Notebook

Chapter 1
Chapter 2
Chapter 3
Index
```

Now information is organized.

A filesystem does the same thing for a partition.

---

## Popular Linux Filesystems

| Filesystem | Description                                                                         |
| ---------- | ----------------------------------------------------------------------------------- |
| XFS        | Default in Red Hat Enterprise Linux; optimized for large files and high performance |
| ext4       | Very common Linux filesystem with broad compatibility                               |
| vfat       | Used for USB drives and Windows-compatible media                                    |
| swap       | Special area used as virtual memory                                                 |

Example:

```bash
mkfs.xfs /dev/sdb1
```

This creates an XFS filesystem on the first partition.

---

# What is a Mount Point?

Even after creating a filesystem, Linux still cannot access it automatically.

You must **mount** it.

Linux does not assign drive letters like Windows (`C:`, `D:`).

Instead, every storage device is attached somewhere inside the single directory tree.

Example:

```text
/

├── home

├── var

├── opt

└── data
```

If `/dev/sdb1` is mounted on `/data`:

```text
/

├── home

├── var

├── opt

└── data
      │
      ▼
   /dev/sdb1
```

Anything written to `/data` is actually stored on `/dev/sdb1`.

---

# Complete Storage Flow

```text
Physical Disk
      │
      ▼
Partition
      │
      ▼
Filesystem
      │
      ▼
Mount Point
      │
      ▼
Mounted Filesystem
      │
      ▼
Users Can Store Files
```

---

# Understanding MBR (Master Boot Record)

MBR is the **older** partitioning method, introduced in **1983**.

It is designed for systems using **BIOS** firmware.

### How MBR Works

The first sector of the disk contains:

* Boot loader
* Partition table

```text
+-----------------------------+
| Master Boot Record          |
+-----------------------------+
| Partition Information       |
+-----------------------------+
| Actual Data                 |
+-----------------------------+
```

---

## Primary Partitions

MBR supports only **4 primary partitions**.

```text
Disk

+-----+-----+-----+-----+
| P1  | P2  | P3  | P4  |
+-----+-----+-----+-----+
```

Need a fifth partition?

You cannot create another primary partition.

Instead, one primary partition becomes an **Extended Partition**, inside which you create **Logical Partitions**.

```text
+-----+-----+-----+------------------+
| P1  | P2  | P3  | Extended         |
|     |     |     | ├─ Logical 5     |
|     |     |     | ├─ Logical 6     |
|     |     |     | └─ Logical 7     |
+-----+-----+-----+------------------+
```

---

## Why MBR Is Limited to 2 TiB

MBR uses **32-bit Logical Block Addressing (LBA)**.

A 32-bit number can address only **2³² blocks**.

With standard 512-byte sectors:

```text
2³² × 512 bytes ≈ 2 TiB
```

So disks larger than 2 TiB cannot be fully addressed by MBR.

---

# Understanding GPT (GUID Partition Table)

GPT is the modern replacement for MBR.

It is designed for **UEFI** systems and large-capacity disks.

### GPT Layout

```text
+-----------------------------+
| Primary GPT Header          |
+-----------------------------+
| Partition Entries           |
+-----------------------------+
| User Data                   |
+-----------------------------+
| Backup Partition Entries    |
+-----------------------------+
| Backup GPT Header           |
+-----------------------------+
```

Unlike MBR, GPT stores **two copies** of the partition information.

If the primary table becomes corrupted, the backup can be used for recovery.

---

## Why GPT Is Better

* Supports **128 partitions** by default.
* No extended or logical partitions are required.
* Supports disks up to **8 ZiB**.
* Uses **GUIDs** to uniquely identify disks and partitions.
* Protects partition data with **CRC checksums**.

---

# MBR vs GPT (Quick Comparison)

| Feature                   | MBR       | GPT           |
| ------------------------- | --------- | ------------- |
| Firmware                  | BIOS      | UEFI          |
| Year Introduced           | 1983      | Modern        |
| Max Partitions            | 4 Primary | 128           |
| Extended Partition Needed | Yes       | No            |
| Max Disk Size             | 2 TiB     | 8 ZiB         |
| Backup Partition Table    | No        | Yes           |
| Error Detection           | No        | CRC Checksums |
| Recommended Today         | No        | Yes           |

---

# RHCSA Practical Workflow

Whenever you add a new disk in RHCSA, follow this sequence:

```text
1. Detect the disk
   lsblk

        │
        ▼
2. Create a partition
   fdisk /dev/sdb
   or
   parted /dev/sdb

        │
        ▼
3. Create a filesystem
   mkfs.xfs /dev/sdb1

        │
        ▼
4. Create a mount point
   mkdir /data

        │
        ▼
5. Mount the filesystem
   mount /dev/sdb1 /data

        │
        ▼
6. Verify
   df -h
   lsblk

        │
        ▼
7. Make it persistent
   Edit /etc/fstab
```

---

# 💡 Memory Trick

Think of storage like building a new house:

| House Analogy                 | Linux Storage                                |
| ----------------------------- | -------------------------------------------- |
| Buy land                      | Add a new disk (`/dev/sdb`)                  |
| Divide the land into plots    | Create partitions (`/dev/sdb1`, `/dev/sdb2`) |
| Build roads and house numbers | Create a filesystem (`mkfs.xfs`)             |
| Assign an address             | Create a mount point (`/data`)               |
| Open the house for living     | Mount the filesystem (`mount`)               |

This analogy helps remember the order:

**Disk → Partition → Filesystem → Mount Point → Mount → `/etc/fstab`**

This is the exact sequence you'll follow in both the RHCSA exam and real-world Linux administration.
