# Chapter 14: Accessing Removable Media 

## 🎯 Learning Objective

After completing this chapter, you should be able to:

* Identify storage devices in Linux.
* Understand what a file system is.
* Recognize block devices and partitions.
* Check disk usage and free space.
* Prepare to mount and unmount removable media like USB drives and DVDs.

---

# Think of a Computer Like a Library

Imagine your computer is a **library**.

* **Hard Disk / SSD** → The building.
* **Partitions** → Different rooms in the building.
* **File System** → The way books are arranged in each room.
* **Mount Point** → The door you use to enter a room.

Without a door, you cannot enter the room.

Similarly, **Linux cannot access a file system until it is mounted.**

---

# 1. What is Storage?

Storage is where your data is permanently saved.

Examples

* Hard Disk (HDD)
* SSD
* USB Drive
* DVD
* SD Card
* NVMe SSD

Example

```text
Computer
│
├── Hard Disk
├── USB Drive
├── DVD
└── SD Card
```

Linux detects all of these as **block devices**.

---

# 2. What is a File System?

A storage device is just empty space.

A **file system** organizes that space so files and folders can be stored.

Think of it like this.

Without a file system

```text
Hard Disk

□□□□□□□□□□□□□□

No organization
```

With a file system

```text
Documents
Pictures
Videos
Music
Downloads
```

Now Linux knows

* where files are stored
* where free space exists
* where deleted files were located

---

## Common File Systems

| File System | Used For                    |
| ----------- | --------------------------- |
| XFS         | Default in RHEL             |
| ext4        | Older Linux systems         |
| exFAT       | USB drives                  |
| FAT32       | Windows/USB                 |
| NTFS        | Windows                     |
| tmpfs       | RAM-based temporary storage |

---

## Why Does Linux Need a File System?

Suppose you buy a brand-new hard disk.

Initially

```text
Hard Disk

Empty Space
```

Linux cannot store files because there is no organization.

After formatting

```text
mkfs.xfs /dev/sdb1
```

the disk becomes

```text
XFS File System

Documents
Downloads
Music
Pictures
```

Now Linux can save files.

---

# 3. What is a Block Device?

Linux represents every storage device as a file inside

```bash
/dev
```

Example

```bash
ls /dev
```

Output

```text
sda
sda1
sda2
sdb
nvme0n1
vda
```

These are **not normal files**.

They represent physical storage devices.

---

## Why Are They Called Block Devices?

Storage devices read and write data in **blocks**.

Example

Suppose one block is 4 KB.

When you save a 10 KB file

```text
File

10 KB
```

Linux stores it like

```text
Block 1 → 4 KB

Block 2 → 4 KB

Block 3 → 2 KB
```

Since storage works block by block,

they are called **block devices**.

---

# 4. Device Naming in Linux

Linux gives names to storage devices.

## SATA / USB

First disk

```text
/dev/sda
```

Second disk

```text
/dev/sdb
```

Third disk

```text
/dev/sdc
```

Example

```text
Computer

├── HDD → /dev/sda
├── USB → /dev/sdb
└── External HDD → /dev/sdc
```

---

## Virtual Machines

VMs often use

```text
/dev/vda
/dev/vdb
```

instead of

```text
/dev/sda
```

---

## NVMe SSD

Modern SSDs use names like

```text
/dev/nvme0n1
```

Meaning

```text
nvme0
```

First NVMe disk

```text
n1
```

Namespace 1

Complete device

```text
/dev/nvme0n1
```

---

## SD Cards

Example

```text
/dev/mmcblk0
```

---

# 5. What is a Partition?

Suppose your disk is 1 TB.

Instead of using one large space,

you divide it into multiple sections.

```text
1 TB Disk

┌───────────────┐
│               │
│               │
└───────────────┘
```

After partitioning

```text
1 TB Disk

┌─────┬─────┬─────┐
│100G │500G │400G │
└─────┴─────┴─────┘
```

Each section is called a **partition**.

---

## Linux Names Partitions

Disk

```text
/dev/sda
```

Partitions

```text
/dev/sda1

/dev/sda2

/dev/sda3
```

Meaning

```text
Disk

/dev/sda

│

├── Partition 1

├── Partition 2

└── Partition 3
```

---

## NVMe Partition Naming

Disk

```text
/dev/nvme0n1
```

Partitions

```text
/dev/nvme0n1p1

/dev/nvme0n1p2

/dev/nvme0n1p3
```

Notice the **p**

because

```text
nvme0n1
```

already ends with a number.

---

# 6. What is LVM?

Instead of fixed partitions,

Linux can create **Logical Volumes**.

Normal partition

```text
Disk

↓

Partition

↓

File System
```

LVM

```text
Disk

↓

Volume Group

↓

Logical Volume

↓

File System
```

Example

```text
/dev/myvg/mylv
```

Advantages

* Resize easily
* Add disks later
* Better storage management

---

# 7. What Does `ls -l /dev/sda1` Show?

Command

```bash
ls -l /dev/sda1
```

Output

```text
brw-rw---- 1 root disk ...
```

The important part is

```text
b
```

Meaning

```text
Block Device
```

Other file types

| Symbol | Meaning          |
| ------ | ---------------- |
| -      | Regular file     |
| d      | Directory        |
| l      | Symbolic link    |
| c      | Character device |
| b      | Block device     |

---

# 8. What is Mounting?

This is the **most important concept**.

Suppose your USB is connected.

Linux detects it.

```text
USB

↓

/dev/sdb1
```

Can you open it?

**No.**

Because Linux hasn't attached it to the file system.

After mounting

```text
/dev/sdb1

↓

Mounted

↓

/mnt/usb
```

Now

```bash
ls /mnt/usb
```

shows

```text
Movies
Photos
Documents
```

Think of it like opening a door to access a room.

---

# Windows vs Linux

Windows

```text
C:
D:
E:
```

Each drive gets a separate letter.

Linux

```text
/

├── home

├── boot

├── media

├── mnt

└── var
```

USB becomes another folder.

Example

```text
/media/harish/USB
```

or

```text
/mnt/usb
```

Everything exists under one root directory (`/`).

---

# 9. Checking Disk Space (`df`)

Command

```bash
df
```

Example

```text
Filesystem      Mounted on

/dev/sda3       /

/dev/sda2       /boot

tmpfs           /run
```

Shows

* Mounted file systems
* Available space
* Used space
* Mount point

---

## Human Readable

```bash
df -h
```

Example

```text
Filesystem      Size Used Avail

/dev/sda3       20G 6.5G 13G
```

Without `-h`

```text
20971520
```

Hard to read.

With `-h`

```text
20G
```

Easy to read.

---

# 10. Checking Folder Size (`du`)

Suppose

```text
/home
```

contains

```text
Movies

Music

Downloads
```

To know how much space they occupy

```bash
du -sh /home
```

Output

```text
8.5G
```

Meaning

The `/home` directory uses **8.5 GB**.

---

## Difference Between `df` and `du`

Imagine a **100 GB hard disk**.

You saved **40 GB** of files.

```text
Disk

100 GB

Used

40 GB

Free

60 GB
```

### `df -h`

Asks:

> **How much of the entire disk is used?**

Output

```text
Filesystem      Size Used Avail

100G            40G 60G
```

### `du -sh /home`

Asks:

> **How much space does this specific folder use?**

Output

```text
12G /home
```

So:

| Command  | Answers                      |
| -------- | ---------------------------- |
| `df -h`  | How full is the disk?        |
| `du -sh` | How large is this directory? |

---

# RHCSA Exam Commands to Remember

```bash
# List storage devices
ls /dev

# View a block device
ls -l /dev/sda1

# Display mounted file systems
df

# Human-readable disk usage
df -h

# Display directory usage
du -h /usr/share

# Display only summary
du -sh /etc
```

---

# Memory Map (Revision)

```text
Physical Disk
      │
      ▼
Block Device (/dev/sda)
      │
      ▼
Partition (/dev/sda1)
      │
      ▼
File System (XFS/ext4)
      │
      ▼
Mount Point (/mnt/usb)
      │
      ▼
Files Become Accessible
```

## 📝 RHCSA Exam Tips

* **Block device** = Represents a storage device under `/dev`.
* **Partition** = A subdivision of a disk (for example, `/dev/sda1`).
* **File system** = Organizes data on a partition (XFS, ext4, exFAT).
* **Mount point** = A directory where a file system is attached.
* Use `df -h` to check **free space on mounted file systems**.
* Use `du -sh <directory>` to check **the size of a specific directory**.
* Recognize common device names:

  * `/dev/sda` (SATA/SCSI/USB disk)
  * `/dev/vda` (VirtIO virtual disk)
  * `/dev/nvme0n1` (NVMe SSD)
  * `/dev/mmcblk0` (SD/MMC card)

This sequence—**Disk → Partition → File System → Mount Point → Files**—is one of the most important concepts for both RHCSA and day-to-day Linux administration.
