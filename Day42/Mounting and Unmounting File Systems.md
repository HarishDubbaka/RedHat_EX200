# Mounting and Unmounting File Systems

One of the biggest mistakes beginners make is thinking that **mounting means copying files** or **creating a new folder**.

**It doesn't.**

Mounting simply means:

> **"Connect a storage device to a directory so Linux can access its files."**

Let's understand this slowly.

---

# First, How Does Linux See Storage?

Imagine you buy a brand-new **64 GB USB drive**.

You plug it into your computer.

## 🪟 Windows

Windows immediately says:

```text
This PC

C:
D:
E:
```

You simply click **E:**.

Everything works.

You don't know what Windows is doing internally.

---

## 🐧 Linux

Linux works differently.

When you plug in the USB, Linux says:

> "I found a new storage device."

That's all.

Linux creates something like:

```bash
/dev/sdb
```

If the USB has one partition:

```bash
/dev/sdb1
```

Notice something.

Linux **did not show your files yet.**

---

# Why?

Because Linux separates two things.

## Step 1

Detect the storage device.

## Step 2

Decide where you want to access it.

Windows combines these steps.

Linux keeps them separate.

---

# Imagine Your House

Suppose you bought a new wardrobe.

Inside it are

* Clothes
* Shoes
* Books

The wardrobe is delivered outside your house.

```text
Outside

📦 Wardrobe
```

Question:

Can you use your clothes?

No.

Why?

Because the wardrobe is still outside.

---

Now someone brings it into your bedroom.

```text
Bedroom

📦 Wardrobe

👕 Clothes
👟 Shoes
📚 Books
```

Now you can use everything.

Linux mounting is exactly this process.

---

# What Is `/dev/sdb1`?

Many beginners think

```bash
/dev/sdb1
```

is a folder.

It is NOT.

It is **the partition itself**.

Think like this.

```text
USB

↓

Partition

↓

File System

↓

Files
```

Linux represents that partition as

```bash
/dev/sdb1
```

---

# How Does Linux Know About It?

The Linux kernel detects the USB.

It creates a device file.

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
sdb1
```

These are **device files**, not ordinary files.

---

# How Do We Find the Correct USB?

Suppose your computer has

* Internal SSD = 512 GB
* USB = 64 GB

Run

```bash
lsblk
```

Output

```text
NAME   SIZE TYPE

sda    512G disk
├─sda1 500M part
├─sda2 100G part
└─sda3 411G part

sdb     64G disk
└─sdb1  64G part
```

Immediately we know

```text
64G
```

is the USB.

This is why RHCSA teaches `lsblk`.

---

# Why Doesn't Linux Automatically Show It?

Think like a Linux administrator.

Suppose your production server has

* 20 SAN disks
* Backup disks
* Database disks
* Shared storage
* USB drive

Should Linux automatically mount everything?

Imagine mounting the wrong database disk!

That would be dangerous.

Linux says

> "Administrator, you decide."

---

# What Is a Mount Point?

This is the most important concept.

A mount point is simply

> **An empty directory where another filesystem becomes visible.**

Example

```bash
mkdir /mnt/usb
```

Now

```text
/mnt/usb
```

is just an empty folder.

Nothing special.

---

# What Happens During Mounting?

Before mounting

```text
USB

↓

/dev/sdb1

↓

Not Connected

↓

No Files
```

Now run

```bash
mount /dev/sdb1 /mnt/usb
```

Linux creates a connection.

```text
USB

↓

/dev/sdb1

↓

Connected

↓

/mnt/usb

↓

Photos

Music

Videos
```

Now

```bash
ls /mnt/usb
```

shows

```text
holiday.jpg
movie.mp4
song.mp3
```

---

# Did Linux Copy the Files?

No.

Nothing moved.

Nothing copied.

Linux simply says

> "Whenever someone opens `/mnt/usb`, show the contents of `/dev/sdb1`."

Think of a shortcut to another room.

The room didn't move.

You're just opening a different door.

---

# Why Use `/mnt`?

Linux reserves

```text
/mnt
```

for temporary mounts.

Common examples

```text
/mnt/data

/mnt/backup

/mnt/usb
```

---

# Can I Mount Anywhere?

Yes.

Example

```bash
mount /dev/sdb1 /project/data
```

Now

```text
/project/data
```

contains the USB contents.

The mount point can be almost any directory, provided it exists.

---

# What Happens If the Folder Already Contains Files?

Suppose

```text
/mnt/data
```

contains

```text
report.txt

notes.txt
```

Now you mount

```bash
mount /dev/sdb1 /mnt/data
```

Suddenly

```text
report.txt
```

disappears.

Did Linux delete it?

No.

It is hidden.

Instead you see

```text
Movies

Photos

Music
```

After unmounting

```bash
umount /mnt/data
```

The original files come back.

Think of putting a carpet over the floor.

The floor is still there.

It's just covered.

---

# Why Use UUID?

Today your USB is

```text
/dev/sdb1
```

Tomorrow you plug another USB first.

Now Linux calls it

```text
/dev/sdc1
```

Oops.

The name changed.

How do we identify the same partition?

Linux gives every filesystem a permanent identity.

Example

```text
15507695-22bb-4c65-94e6-a438e095983f
```

This is the UUID.

It never changes unless the filesystem is recreated.

View it

```bash
lsblk -fp
```

Mount

```bash
mount UUID="15507695-22bb-4c65-94e6-a438e095983f" /mnt/data
```

This is much safer than relying on `/dev/sdb1`.

---

# What Is Unmounting?

Suppose you're editing a file on your USB.

When you press **Save**, Linux may not write everything immediately to the USB.

Instead it uses RAM as a cache.

```text
Application

↓

RAM Cache

↓

USB
```

Writing to RAM is much faster than writing directly to a disk.

---

# What Happens If You Remove the USB Immediately?

Imagine:

```text
RAM

↓

Still Contains Unsaved Data
```

Now you pull out the USB.

```text
USB Removed
```

Those unwritten changes are lost.

The filesystem may become corrupted.

---

# What Does `umount` Do?

When you run:

```bash
umount /mnt/usb
```

Linux performs these steps:

1. Stops new writes.
2. Writes all cached data from RAM to the storage device.
3. Closes open files.
4. Disconnects the filesystem from the mount point.

Only after that is it safe to remove the USB.

---

# Why Does "Target is Busy" Appear?

Suppose you're inside the mounted directory:

```bash
cd /mnt/usb
```

Now you try:

```bash
umount /mnt/usb
```

Linux replies:

```text
umount: target is busy
```

Why?

Because your shell is standing inside that directory.

It's like trying to remove a ladder while you're still standing on it.

Move somewhere else:

```bash
cd ~
```

Now:

```bash
umount /mnt/usb
```

works.

---

# Another Reason for "Target is Busy"

Maybe another program is using the filesystem.

For example:

* A text editor (`vim`)
* A media player
* A file manager
* Another terminal

Find them with:

```bash
lsof /mnt/usb
```

or

```bash
fuser -vm /mnt/usb
```

Stop those processes, then unmount.

---

# Complete Workflow

```text
Plug in USB
      │
      ▼
Linux detects it
      │
      ▼
Creates /dev/sdb
      │
      ▼
Find the partition
lsblk
      │
      ▼
Create a mount point
mkdir /mnt/usb
      │
      ▼
Mount the filesystem
mount /dev/sdb1 /mnt/usb
      │
      ▼
Access files
ls /mnt/usb
      │
      ▼
Finished?
      │
      ▼
Leave the mount point
cd
      │
      ▼
Unmount
umount /mnt/usb
      │
      ▼
Safely remove the USB
```

# 🎯 RHCSA Exam Memory Trick

Remember this simple sequence:

```text
Storage Device
        │
        ▼
Block Device (/dev/sdb)
        │
        ▼
Partition (/dev/sdb1)
        │
        ▼
File System (XFS, ext4, exFAT)
        │
        ▼
Mount Point (/mnt/usb)
        │
        ▼
Files Become Accessible
```

If you can explain this flow in your own words, you'll have a solid understanding of one of the core Linux administration concepts used every day by system administrators.
