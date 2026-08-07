# **Day 49 – Extending LVM (Increase LV, VG Space in Linux)**

## Yesterday's Recap

Yesterday we discussed **renaming Physical Volumes (PV), Volume Groups (VG), and Logical Volumes (LV).**

One important point we learned was:

* **PV cannot be renamed.**
* **VG can be renamed** using `vgrename`.
* **LV can be renamed** using `lvrename`.

---

# Today's Topic

## **How to Check Available Space and Extend a Logical Volume (LVM)**

In real-time production environments, applications and databases continuously grow.

For example:

* SAP HANA data volume becomes full.
* `/usr/sap` reaches 100%.
* `/backup` requires additional storage.
* `/hana/log` needs more disk space.

Instead of creating a new filesystem or rebooting the server, **LVM allows us to increase storage online.**

This is one of the biggest advantages of Logical Volume Manager (LVM).

---

# LVM Architecture

```
Disk
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
Filesystem (XFS / EXT4)
 │
 ▼
Mount Point
```

To increase storage, the free space must already exist inside the Volume Group.

---

# Step 1 – Check Available Space in the Volume Group

Before extending an LV, verify that the VG has free space.

```bash
vgdisplay
```

Example:

```
VG Name               vgdata
VG Size               100 GB
Allocated PE / Size   80 GB
Free PE / Size        20 GB
```

Here,

* Total VG Size = **100 GB**
* Used = **80 GB**
* Free = **20 GB**

Since 20 GB is free, we can extend any LV within this VG.

You can also use:

```bash
vgs
```

Output:

```
VG     #PV #LV VSize   VFree
vgdata   1   3 100G    20G
```

---

# Step 2 – Check Current Logical Volume Size

```bash
lvdisplay
```

or

```bash
lvs
```

Example:

```
LV Name      lvdata
LV Size      50 GB
```

---

# Step 3 – Check the Mounted Filesystem

```bash
df -h
```

Example:

```
Filesystem              Size Used Avail Mounted on
/dev/vgdata/lvdata      50G  30G   20G  /data
```

Notice that the filesystem is still **50 GB**.

---

# Step 4 – Extend the Logical Volume

Suppose we want to add **10 GB**.

```bash
lvextend -L +10G /dev/vgdata/lvdata
```

Output:

```
Logical volume successfully resized.
```

### What does `+10G` mean?

The **+** means **add** 10 GB to the existing size.

Example:

Current Size:

```
50 GB
```

Command:

```bash
lvextend -L +10G
```

New Size:

```
60 GB
```

---

Without the plus sign:

```bash
lvextend -L 60G
```

This means:

> Make the final size **60 GB**.

---

# Step 5 – Resize the Filesystem

Extending the LV **does not automatically extend the filesystem**.

The filesystem must also be resized.

This depends on the filesystem type.

---

## If Filesystem is XFS

Most SAP servers use XFS.

Command:

```bash
xfs_growfs /data
```

Notice:

The command takes the **mount point**, not the device name.

Example:

```
xfs_growfs /hana/data
```

Output:

```
data blocks changed...
```

The filesystem immediately starts using the new space.

No reboot required.

---

## If Filesystem is EXT4

For EXT4, use:

```bash
resize2fs /dev/vgdata/lvdata
```

Notice:

Here we use the **device name**, not the mount point.

Output:

```
Filesystem resized successfully.
```

---

# One-Step Command

Instead of running:

```bash
lvextend
```

followed by

```bash
xfs_growfs
```

or

```bash
resize2fs
```

we can simply execute:

```bash
lvextend -r -L +10G /dev/vgdata/lvdata
```

The **`-r`** option automatically resizes the filesystem after extending the LV.

---

# Check the New Size

Verify using:

```bash
df -h
```

Example:

Before:

```
50G
```

After:

```
60G
```

Mission accomplished!

---

# Extending Swap Logical Volume

Swap is different because it is **not a filesystem**.

We cannot resize it while it is active.

### Step 1

Disable swap:

```bash
swapoff /dev/vgdata/swap
```

---

### Step 2

Extend the LV:

```bash
lvextend -L +2G /dev/vgdata/swap
```

---

### Step 3

Create a new swap signature:

```bash
mkswap /dev/vgdata/swap
```

---

### Step 4

Enable swap:

```bash
swapon /dev/vgdata/swap
```

---

### Verify

```bash
swapon --show
```

or

```bash
free -h
```

---

# Important Interview Question

**Q: Why do we run `xfs_growfs` after `lvextend`?**

**Answer:**
`lvextend` increases only the **Logical Volume** size. The filesystem still sees the old size. `xfs_growfs` (for XFS) or `resize2fs` (for EXT4) expands the filesystem to use the newly available space.

---

# XFS vs EXT4

| Feature           | XFS          | EXT4        |
| ----------------- | ------------ | ----------- |
| Grow Filesystem   | ✅ Yes        | ✅ Yes       |
| Shrink Filesystem | ❌ No         | ✅ Yes       |
| Resize Online     | ✅ Yes        | ✅ Yes       |
| Command           | `xfs_growfs` | `resize2fs` |
| Argument          | Mount Point  | Device Name |

---

# SAP BASIS Real-Time Scenario

Suppose your SAP HANA `/hana/data` filesystem reaches **95% usage**.

1. Check usage:

   ```bash
   df -h
   ```
2. Verify free space in the VG:

   ```bash
   vgs
   ```
3. Extend the LV:

   ```bash
   lvextend -L +50G /dev/vghana/lv_hana_data
   ```
4. Grow the XFS filesystem:

   ```bash
   xfs_growfs /hana/data
   ```
5. Confirm:

   ```bash
   df -h
   ```

This can usually be done **online**, without rebooting or stopping SAP HANA, making LVM an essential tool for SAP BASIS administrators.

---

## Summary

* Check free VG space using `vgdisplay` or `vgs`.
* Check LV size using `lvdisplay` or `lvs`.
* Check filesystem usage with `df -h`.
* Extend the LV using `lvextend`.
* Resize the filesystem:

  * **XFS:** `xfs_growfs <mount_point>`
  * **EXT4:** `resize2fs <device>`
* Use `lvextend -r` to perform both steps together.
* For swap LVs: `swapoff` → `lvextend` → `mkswap` → `swapon`.
* XFS can **only grow**, while EXT4 can **grow and shrink**.
