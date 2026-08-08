# LVM – Replacing PV and Removing LVM Components

First, remember the LVM structure:

```text
Physical Disk/Partition
        ↓
       PV
Physical Volume
        ↓
       VG
Volume Group
        ↓
       LV
Logical Volume
        ↓
   Filesystem
        ↓
      Data
```

For example:

```text
/dev/sdb1  ──→ PV1 ──┐
                     │
/dev/sdc1  ──→ PV2 ──┤──→ vg01 ──→ lv01 ──→ /data
                     │
/dev/sdd1  ──→ PV3 ──┘
```

Think of it like this:

* **PV = Storage boxes**
* **VG = One big storage pool**
* **LV = A portion of that pool given to an application/filesystem**

---

# 1. Why do we replace a PV?

Suppose we have:

```text
vg01
 ├── /dev/sdb1  ← 500 GB
 └── /dev/sdc1  ← 500 GB
```

Now `/dev/sdb1` has a problem.

Maybe:

* Disk is failing
* Disk needs to be replaced
* We need to remove an old disk
* We want to migrate data to a new disk

We have a new disk:

```text
/dev/sdd1
```

Our goal is:

```text
BEFORE:

vg01
 ├── /dev/sdb1  ← Old
 └── /dev/sdc1


AFTER:

vg01
 ├── /dev/sdc1
 └── /dev/sdd1  ← New
```

But there is an important question:

### What happens to the data currently stored on `/dev/sdb1`?

We need to **move that data somewhere else first**.

That's exactly what `pvmove` does.

---

# 2. Step 1 – Create the new PV

First, prepare the new disk:

```bash
pvcreate /dev/sdd1
```

Output:

```text
Physical volume "/dev/sdd1" successfully created.
```

Now:

```text
/dev/sdd1
    ↓
   PV
```

But this PV is **not yet part of `vg01`**.

---

# 3. Step 2 – Add the new PV to the VG

Use:

```bash
vgextend vg01 /dev/sdd1
```

Now:

```text
/dev/sdb1 ──→ PV ──┐
                   │
/dev/sdc1 ──→ PV ──┤──→ vg01
                   │
/dev/sdd1 ──→ PV ──┘
```

We now have additional storage available inside `vg01`.

---

# 4. Step 3 – Move the data

This is the **most important step**.

Suppose `/dev/sdb1` contains data:

```text
/dev/sdb1

[Data][Data][Free][Data][Free][Data]
```

We don't want to simply remove `/dev/sdb1`, because some of the LV's data is physically stored there.

So we use:

```bash
pvmove /dev/sdb1 /dev/sdd1
```

This means:

> "Move all allocated physical extents from `/dev/sdb1` to `/dev/sdd1`."

Conceptually:

```text
BEFORE

/dev/sdb1
[DATA][DATA][FREE][DATA][DATA]
     ↓
     ↓ pvmove
     ↓
/dev/sdd1
[DATA][DATA][DATA][DATA]
```

After `pvmove`, `/dev/sdb1` should have **no allocated extents**.

You can check using:

```bash
pvs
```

You might see:

```text
PV          VG     PSize   PFree
/dev/sdb1   vg01   500G    500G
/dev/sdc1   vg01   500G     50G
/dev/sdd1   vg01   500G     50G
```

The important thing is that `/dev/sdb1` is now completely free.

---

# 5. Step 4 – Remove the old PV from the VG

Now we can safely remove `/dev/sdb1` from `vg01`:

```bash
vgreduce vg01 /dev/sdb1
```

Now:

```text
BEFORE:

vg01
 ├── /dev/sdb1
 ├── /dev/sdc1
 └── /dev/sdd1


AFTER:

vg01
 ├── /dev/sdc1
 └── /dev/sdd1
```

The old PV is no longer part of `vg01`.

---

# 6. Step 5 – Remove LVM metadata

If you are going to physically remove or reuse `/dev/sdb1`, execute:

```bash
pvremove /dev/sdb1
```

This removes the LVM PV metadata/label.

Now the device can be reused.

---

# Complete PV Replacement

Remember this sequence:

```bash
pvcreate /dev/sdd1
vgextend vg01 /dev/sdd1
pvmove /dev/sdb1 /dev/sdd1
vgreduce vg01 /dev/sdb1
pvremove /dev/sdb1
```

### Easy memory trick:

> **Create → Extend → Move → Reduce → Remove**

```text
New Disk
   ↓
pvcreate
   ↓
vgextend
   ↓
pvmove
   ↓
vgreduce
   ↓
pvremove
   ↓
Old Disk Ready for Removal
```

---

# 7. What is `pvmove` actually moving?

This is an important interview question.

`pvmove` does **not move the filesystem itself**.

It moves **physical extents**.

For example:

```text
LV: /dev/vg01/lv01

Logical View:
┌─────────────────────────────┐
│          LV01               │
│        100 GB               │
└─────────────────────────────┘
```

Behind the scenes, the LV is made up of physical extents:

```text
LV01
 ↓
PE PE PE PE PE PE PE PE
 ↓
PV
```

`pvmove` relocates those physical extents from one PV to another.

So:

> **pvmove = move allocated physical extents from one PV to another PV.**

---

# 8. Can users continue using the filesystem during `pvmove`?

Generally, **yes**.

One of the major advantages of LVM is that you can perform `pvmove` while the LV remains available.

For example:

```text
Application
     ↓
   /data
     ↓
    LV
     ↓
    VG
     ↓
pvmove happening
```

The application can generally continue using the filesystem while LVM moves the extents.

However, for production systems, you should still follow your organization's change procedure and have backups.

---

# 9. What does `vgextend` do?

Suppose:

```text
vg01 = 500 GB
```

You add a 200 GB PV:

```bash
vgextend vg01 /dev/sdd1
```

Now approximately:

```text
vg01 = 700 GB
```

So:

> **`vgextend` increases the available capacity of a Volume Group by adding a PV.**

It does **not automatically increase an existing LV or filesystem**.

For example:

```text
Before:

VG = 500 GB
LV = 300 GB


vgextend


After:

VG = 700 GB
LV = 300 GB
```

The VG gets bigger, but the LV remains 300 GB.

---

# 10. What does `vgreduce` do?

The opposite of `vgextend` is:

```bash
vgreduce
```

`vgextend`:

> **Adds PV to VG**

`vgreduce`:

> **Removes PV from VG**

But before using `vgreduce`, make sure the PV doesn't contain allocated extents.

Check:

```bash
pvs
```

If the PV still contains data, first use:

```bash
pvmove
```

---

# 11. Removing an LV

Now let's say the application no longer needs:

```text
/dev/vg01/lv01
```

First identify the filesystem:

```bash
lsblk
```

Unmount it:

```bash
umount /mnt/data
```

Then remove the LV:

```bash
lvremove /dev/vg01/lv01
```

### What happens?

The LV disappears:

```text
Before:

vg01
 ├── lv01  ← 100 GB
 └── lv02  ← 200 GB


After lvremove:

vg01
 └── lv02  ← 200 GB

Free space = 100 GB
```

The 100 GB becomes available inside the VG.

---

# 12. Removing a VG

Suppose the entire VG is no longer required:

```bash
vgremove vg01
```

This removes the VG.

But remember:

**Don't think `vgremove` means "erase the disks."**

It removes the **Volume Group configuration**.

The underlying PVs can then be reused or removed.

---

# 13. Removing a PV

Finally:

```bash
pvremove /dev/sdb1
```

This removes the **LVM metadata identifying the device as a PV**.

Think:

```text
pvcreate
    ↓
"I am an LVM PV"


pvremove
    ↓
"I am no longer an LVM PV"
```

---

# 14. Very Important Difference

This is one of the most important things to remember:

| Command    | What it does                   |
| ---------- | ------------------------------ |
| `pvmove`   | Moves data/extents between PVs |
| `vgreduce` | Removes PV from VG             |
| `pvremove` | Removes LVM metadata from PV   |
| `lvremove` | Deletes an LV                  |
| `vgremove` | Deletes a VG                   |
| `vgextend` | Adds PV to VG                  |
| `pvcreate` | Creates a PV                   |

### In one sentence:

> **`pvmove` moves the data, `vgreduce` removes the PV from the VG, and `pvremove` removes the PV's LVM metadata.**

---

# 15. Complete Example

Suppose we have:

```text
/dev/sdb1 → PV → vg01 → lv01 → /data
```

We want to replace `/dev/sdb1` with `/dev/sdc1`.

### Commands:

```bash
# 1. Create new PV
pvcreate /dev/sdc1

# 2. Add new PV to VG
vgextend vg01 /dev/sdc1

# 3. Move data from old PV to new PV
pvmove /dev/sdb1 /dev/sdc1

# 4. Remove old PV from VG
vgreduce vg01 /dev/sdb1

# 5. Remove LVM metadata
pvremove /dev/sdb1
```

Final result:

```text
BEFORE

/dev/sdb1
    ↓
   PV
    ↓
  vg01
    ↓
   lv01
    ↓
  /data


AFTER

/dev/sdc1
    ↓
   PV
    ↓
  vg01
    ↓
   lv01
    ↓
  /data
```

**The important point:** the LV and filesystem don't need to be recreated just because the underlying PV was replaced. `pvmove` relocates the physical extents while preserving the LV structure.

---

## ⭐ Interview Questions You Should Know

**Q1. What is `pvmove`?**

`pvmove` moves allocated physical extents from one PV to another PV within the same VG.

**Q2. Can you run `pvmove` while the filesystem is mounted?**

Yes, LVM supports moving extents while the LV remains available, subject to the normal operational/change-management considerations.

**Q3. What is the difference between `vgreduce` and `pvremove`?**

`vgreduce` removes the PV from the Volume Group. `pvremove` removes the LVM PV metadata from the device.

**Q4. Can you run `vgreduce` directly if the PV contains data?**

Normally no. First move the allocated extents using `pvmove`.

**Q5. Does `vgextend` increase the size of an LV automatically?**

No. `vgextend` increases VG capacity. You must separately extend the LV, and then usually the filesystem.

**Q6. What happens to the space after `lvremove`?**

The LV's extents become free space in the VG and can be allocated to another LV.

**Q7. What is the safest sequence for replacing a PV?**

```text
pvcreate
   ↓
vgextend
   ↓
pvmove
   ↓
vgreduce
   ↓
pvremove
```

### 🔑 One-line memory formula

> **Add the new PV → add it to VG → move the data → remove the old PV from VG → wipe its LVM metadata.**
