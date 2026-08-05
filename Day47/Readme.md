# Creating LVM (Logical Volume Manager) 

![Image](https://images.openai.com/static-rsc-4/uA0JQxI6e5Al4BwmSj_3NXB4PJJEGm4DJLFv0t3NykafXjdTNk0aMAoL8az1zRkjyeZQDrGhZE_LYeRH3h93uxqRVWA8XynZ_hyKMjn7cA60SM1ar8A1aVeh-c2ydmLBKdTuHmZQ5TTLhPvVA2CDl8owPV8p01rmvDD-rxjd4oZ2hatJgDU6ckQH3yGDq9-C?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/F3uRJ_kWpTg4qqFuAatbPow0XokzxplR1PqkgI3hX9d7aS8KWpxcUHaPGeHc0Cyh6sCb39RQCoMiQza6POJLZZyuH13PHs7Bq2OUuvekWcOEQBTbozXBZBado7j_3gEwqCTzYI_B9_mUUqOe1pLvRJvuYZjkEUeUSTsLGTepNv2Jn9jNbKlIGqU4NJJN1ePB?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/FCkyc7_5x2ErIqxJvow6ccXUEgjt0vwsZrcjdCv4rKwqtsUJj-uU_EOr62cNDZrbR528HWZPO3zjPwm7auWaHgKpOkp_vOkenCYx5_XE74NTG286650bYZAfFC2E0M43gnvrGRcEo9qwrTzC6evqd5z671Rm4kJ4hbK90ziXFJkDaOtQKNHCBz09bYiYXuab?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/gmZifJrwGFlbtU4zcMfmDE3AHI4T2JyNDZhsjNp_MZ7CiRPrxmp7VtLBDfNoi7fTSAC7VbSC9zW1zd1yTIonXO20DyJhXiCsB2W68flRUfc64ihUnXpNYLt5di_wfEM6WiPWp7SRHuMKEPDKqvChLc5D-uIX7BcCoTm47bldL1jH7Hh76Qvcu2PHX3g_W649?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/XDiDvZ7HZgPRforsIAH_VOAZBIFCmtkKP84gCnmD_4H8HBbEqmGsQ51_Few1ynM5h60ZZEbsGVpyAOoSNq4eaWBxX2j-hlEbKHKWE-pyfJbrOhrucvvQh8z3c9RZwa4Fr0d8wxH-M0MJiP_c2Dm6uSYbfF1K2410k8GqQANXWcmvywb5tqJSJdL-tW3puCF9?purpose=fullsize)

## What is LVM?

**LVM (Logical Volume Manager)** is a storage management layer in Linux that sits between **physical disks** and the **file system**. It allows you to create, resize, and manage storage flexibly without depending directly on physical disk partitions.

### LVM Architecture

```text
Physical Disk (/dev/sdb)
        │
        ▼
Partition (/dev/sdb1)
        │
        ▼
Physical Volume (PV)
        │
        ▼
Volume Group (VG)
        │
        ▼
Logical Volume (LV)
        │
        ▼
File System (XFS / EXT4)
        │
        ▼
Mount Point (/data)
```

---

# Step 1: Create a Physical Volume (PV)

### What is a Physical Volume?

A **Physical Volume (PV)** is a disk or partition initialized for use with LVM.

It is the **first building block** of LVM.

### Command

```bash
pvcreate /dev/sdb1 /dev/sdb2
```

### Output

```text
Physical volume "/dev/sdb1" successfully created.
Physical volume "/dev/sdb2" successfully created.
```

### What happens internally?

* LVM metadata is written to the partition.
* The partition becomes available for LVM.
* Linux now recognizes it as an LVM Physical Volume.

### Verify

```bash
pvs
```

or

```bash
pvdisplay
```

Example:

```text
PV         VG    Size   Free
/dev/sdb1        10.00G 10.00G
/dev/sdb2        10.00G 10.00G
```

---

# Step 2: Create a Volume Group (VG)

### What is a Volume Group?

A **Volume Group (VG)** is a pool of storage created by combining one or more Physical Volumes.

Think of it as a **storage container**.

### Command

```bash
vgcreate vg01 /dev/sdb1 /dev/sdb2
```

### Output

```text
Volume group "vg01" successfully created
```

### What happens internally?

* Combines multiple PVs into one storage pool.
* Storage becomes available for creating Logical Volumes.

Example:

```
PV1 = 10 GB
PV2 = 10 GB

VG = 20 GB
```

### Verify

```bash
vgs
```

or

```bash
vgdisplay
```

Example:

```text
VG    #PV  #LV  Size    Free
vg01   2    0   20.00G  20.00G
```

---

# Step 3: Create a Logical Volume (LV)

### What is a Logical Volume?

A **Logical Volume (LV)** is the virtual partition created from the free space in a Volume Group.

It behaves like a normal disk partition.

### Command

```bash
lvcreate -n lv01 -L 300M vg01
```

### Meaning of Options

| Option | Description         |
| ------ | ------------------- |
| `-n`   | Logical Volume name |
| `-L`   | Size (MB, GB, etc.) |
| `vg01` | Volume Group name   |

### Output

```text
Logical volume "lv01" created.
```

### Verify

```bash
lvs
```

or

```bash
lvdisplay
```

Example:

```text
LV    VG    Size
lv01  vg01  300M
```

---

# PE (Physical Extent)

LVM divides storage into equal-sized blocks called **Physical Extents (PEs)**.

Default PE size:

```
4 MB
```

Example:

```
20 GB VG

↓

4 MB blocks

↓

5120 Physical Extents
```

LVM allocates storage using these blocks.

---

# Difference Between `-L` and `-l`

## Option 1: Use Size

```bash
lvcreate -n lv01 -L 128M vg01
```

Creates an LV of **128 MB**.

---

## Option 2: Use Physical Extents

```bash
lvcreate -n lv01 -l 32 vg01
```

Since:

```
32 PE × 4 MB = 128 MB
```

It creates the same size LV.

---

## Comparison

| Option | Meaning                            |
| ------ | ---------------------------------- |
| `-L`   | Specify size (MB, GB, TB)          |
| `-l`   | Specify number of Physical Extents |

---

# Complete LVM Workflow

```text
New Disk
   │
   ▼
Partition (fdisk/parted)
   │
   ▼
pvcreate
   │
   ▼
Physical Volume (PV)
   │
   ▼
vgcreate
   │
   ▼
Volume Group (VG)
   │
   ▼
lvcreate
   │
   ▼
Logical Volume (LV)
   │
   ▼
mkfs.xfs / mkfs.ext4
   │
   ▼
Mount
   │
   ▼
Application Uses Storage
```

---

# Common Verification Commands

| Command     | Purpose                              |
| ----------- | ------------------------------------ |
| `pvs`       | List Physical Volumes                |
| `pvdisplay` | Detailed Physical Volume information |
| `vgs`       | List Volume Groups                   |
| `vgdisplay` | Detailed Volume Group information    |
| `lvs`       | List Logical Volumes                 |
| `lvdisplay` | Detailed Logical Volume information  |
| `lsblk`     | View disk and LVM hierarchy          |
| `blkid`     | Display filesystem UUIDs and types   |

---

# Real-Time Example

Suppose you have:

```
Disk 1 = 50 GB
Disk 2 = 50 GB
```

### Create PVs

```bash
pvcreate /dev/sdb1 /dev/sdc1
```

↓

```
PV1 = 50 GB
PV2 = 50 GB
```

### Create VG

```bash
vgcreate vg_data /dev/sdb1 /dev/sdc1
```

↓

```
VG = 100 GB
```

### Create LV

```bash
lvcreate -L 60G -n lv_app vg_data
```

↓

```
LV = 60 GB
Remaining VG Free = 40 GB
```

Create a filesystem:

```bash
mkfs.xfs /dev/vg_data/lv_app
```

Mount it:

```bash
mount /dev/vg_data/lv_app /data
```

---

# Interview Questions

### 1. What is a Physical Volume (PV)?

A **Physical Volume** is a disk or partition initialized with `pvcreate` so it can be managed by LVM.

---

### 2. What is a Volume Group (VG)?

A **Volume Group** combines one or more Physical Volumes into a single storage pool from which Logical Volumes are created.

---

### 3. What is a Logical Volume (LV)?

A **Logical Volume** is virtual storage carved out of a Volume Group. It is formatted with a filesystem and mounted for use.

---

### 4. What is the difference between `-L` and `-l` in `lvcreate`?

* `-L` specifies the size directly (for example, `300M`, `10G`).
* `-l` specifies the number of Physical Extents (PEs).

---

### 5. What happens if the Volume Group doesn't have enough free space?

The `lvcreate` command fails because there are not enough free Physical Extents available to allocate the requested Logical Volume.
