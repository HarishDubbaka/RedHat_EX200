# Controlling and Troubleshooting the Boot Process

## 1. First understand the big picture

When a RHEL server is powered on, Linux does **not** start immediately.

The boot process is roughly:

```text
          Power ON
             │
             ▼
        BIOS / UEFI
             │
             ▼
           GRUB2
             │
             ▼
      Linux Kernel
             │
             ▼
          initramfs
             │
             ▼
          systemd
             │
             ▼
       systemd target
             │
             ▼
        Services
             │
             ▼
           Login
```

The entire chapter is about controlling and troubleshooting these stages.

---

# 2. BIOS and UEFI

When you press the power button, the first thing that runs is **firmware**.

There are two major types:

### BIOS

Older systems commonly use BIOS.

```text
BIOS
 ↓
Find bootable disk
 ↓
Load boot loader
```

### UEFI

Modern systems normally use UEFI.

```text
UEFI
 ↓
Read boot configuration from NVRAM
 ↓
Find EFI application
 ↓
Load GRUB2
```

### Easy way to remember

> **BIOS/UEFI = finds and starts the boot loader.**

It does **not** start Linux directly.

---

# 3. What is GRUB2?

**GRUB2** is the default boot loader for RHEL.

GRUB stands for:

**GRand Unified Bootloader**

Its job is to load:

1. Linux kernel
2. initramfs
3. Kernel command-line parameters

So:

```text
BIOS/UEFI
     ↓
   GRUB2
     ↓
Kernel + initramfs
     ↓
Linux
```

---

# 4. GRUB2 with UEFI

In a UEFI system, there is an **EFI System Partition (ESP)**.

It is commonly mounted at:

```bash
/boot/efi
```

You can check it with:

```bash
ls /boot/efi
```

The simplified process is:

```text
UEFI
 ↓
NVRAM
 ↓
EFI System Partition
 ↓
GRUB2 EFI application
 ↓
Kernel
```

### What is NVRAM?

NVRAM stores firmware settings, including information about available boot entries.

Think of it as:

> **UEFI's saved boot configuration.**

---

# 5. GRUB2 with BIOS

BIOS uses a different mechanism.

Simplified:

```text
BIOS
 ↓
Boot sector
 ↓
boot.img
 ↓
core.img
 ↓
GRUB2 modules
 ↓
Kernel
```

You don't need to get too deep into `boot.img` and `core.img` for normal RHCSA administration.

The important concept is:

> **BIOS finds GRUB2, and GRUB2 loads Linux.**

---

# 6. What happens after GRUB2 starts?

GRUB2 displays a menu.

For example:

```text
GRUB2

Red Hat Enterprise Linux
Red Hat Enterprise Linux (previous kernel)
```

You can choose which kernel you want to boot.

This becomes very useful when troubleshooting.

For example:

```text
New kernel       ❌ Problem
Old kernel       ✅ Working
```

You can boot the old kernel from GRUB.

---

# 7. Why can there be multiple kernels?

When you install a new kernel, RHEL can keep older kernels.

For example:

```text
/boot/

vmlinuz-6.12.0-55
vmlinuz-6.12.0-54
vmlinuz-6.12.0-53
```

GRUB creates boot entries for these kernels.

Therefore:

```text
GRUB
 ├── Kernel 55
 ├── Kernel 54
 └── Kernel 53
```

If Kernel 55 doesn't work, you can select Kernel 54.

---

# 8. Pressing `E` in GRUB

This is **very important for troubleshooting**.

At the GRUB menu, select a boot entry and press:

```text
E
```

This opens the GRUB editor.

You'll see something similar to:

```text
linux /vmlinuz-6.12... root=UUID=xxxx ro quiet
```

You can modify the kernel command line.

For example, you might temporarily add:

```text
rd.break
```

or remove a problematic parameter.

Then boot with the modified entry.

### Important:

Changes made using `E` are:

> **TEMPORARY**

They affect **only that boot**.

After reboot, the change disappears.

---

# 9. Temporary vs Persistent

This is one of the most important concepts.

### Temporary

```text
GRUB menu
   ↓
Press E
   ↓
Modify kernel parameters
   ↓
Boot
```

Change applies to **one boot only**.

### Persistent

Use:

```bash
grubby
```

For example:

```bash
grubby --update-kernel ...
```

The change remains for future boots.

### Remember:

```text
E       → Temporary
grubby  → Persistent
```

---

# 10. What is `grubby`?

`grubby` is a RHEL command used to manage boot entries and kernel parameters.

You can use it to:

* View boot entries
* Change the default kernel
* Add kernel parameters
* Remove kernel parameters

---

# 11. Viewing a GRUB entry

Your example:

```bash
grubby --info 1
```

might produce:

```text
index=1
kernel="/boot/vmlinuz-6.12.0-55..."
args="console=tty0 crashkernel=..."
root="UUID=..."
initrd="/boot/initramfs-6.12..."
title="Red Hat Enterprise Linux..."
id="..."
```

Let's understand each one.

---

## `index`

```text
index=1
```

This is the boot entry number.

Indexes normally start at:

```text
0
1
2
3
```

So:

```text
0 → First entry
1 → Second entry
2 → Third entry
```

---

# 12. `kernel`

Example:

```text
kernel="/boot/vmlinuz-6.12..."
```

This tells GRUB:

> **Which Linux kernel should I load?**

`vmlinuz` is the Linux kernel image.

---

# 13. `args`

Example:

```text
args="console=tty0 crashkernel=2G-64G:256M ..."
```

`args` means:

> **Kernel command-line arguments.**

These parameters control kernel/early-boot behavior.

For example:

```text
quiet
```

reduces boot messages.

Another example:

```text
crashkernel=...
```

reserves memory for crash-dump handling.

---

# 14. `root`

Example:

```text
root="UUID=15507695..."
```

This identifies the filesystem/device from which the kernel's early boot environment is loaded.

The UUID uniquely identifies the filesystem.

You can see filesystem UUIDs with:

```bash
lsblk -f
```

or:

```bash
blkid
```

---

# 15. `initrd`

Example:

```text
initrd="/boot/initramfs-6.12....img"
```

This is the **initramfs image**.

`initrd` is an older/common name for the initial RAM filesystem image.

---

# 16. What is initramfs?

This is extremely important.

During early boot, the kernel may not yet have everything it needs to access the actual root filesystem.

For example, the root filesystem could depend on:

```text
LVM
RAID
Storage drivers
Multipath
Network storage
Filesystem modules
```

The **initramfs** provides the early userspace environment and required components to get the real root filesystem available.

Simplified:

```text
GRUB
 ↓
Kernel
 ↓
initramfs
 ↓
Find/mount real root filesystem
 ↓
systemd
```

Think of initramfs as:

> **A temporary mini Linux environment used during early boot.**

---

# 17. `title`

Example:

```text
title="Red Hat Enterprise Linux..."
```

This is simply the name displayed in the GRUB menu.

For example:

```text
Red Hat Enterprise Linux (6.12...)
```

---

# 18. `id`

Example:

```text
id="b70d796..."
```

This is the unique identifier for that boot entry.

For basic administration, you generally don't need to manipulate it manually.

---

# 19. Changing the default kernel

Suppose your GRUB entries are:

```text
index 0 → Kernel 6.12.0-55
index 1 → Kernel 6.12.0-54
index 2 → Kernel 6.12.0-53
```

You want index `0` as the default.

Run:

```bash
grubby --set-default-index 0
```

Now GRUB automatically chooses:

```text
index 0
```

during the next boot.

### Remember:

```text
--set-default-index N
```

means:

> **Make boot entry N the default.**

---

# 20. What are kernel command-line arguments?

These are parameters passed to the Linux kernel during boot.

Example:

```text
root=UUID=xxxx
ro
quiet
rhgb
crashkernel=...
```

You can think of them as:

> **Instructions given to the kernel when it starts.**

---

# 21. How do I see the current kernel parameters?

Very useful command:

```bash
cat /proc/cmdline
```

Example:

```text
BOOT_IMAGE=/vmlinuz-6.12... root=UUID=xxxx ro quiet
```

This tells you:

> **Which kernel parameters were actually used to boot the currently running system.**

This is an important troubleshooting command.

---

# 22. Adding a kernel parameter

Suppose you want to add:

```text
rhgb quiet
```

You can use:

```bash
grubby --update-kernel /boot/vmlinuz-6.12... \
--args="rhgb quiet"
```

Understand the command:

```text
grubby
   ↓
--update-kernel
   ↓
Which kernel?
   ↓
--args
   ↓
Add these parameters
```

---

# 23. Removing kernel parameters

To remove:

```text
rhgb quiet
```

use:

```bash
grubby --update-kernel /boot/vmlinuz-6.12... \
--remove-args="rhgb quiet"
```

So:

```text
--args
     ↓
ADD

--remove-args
     ↓
REMOVE
```

Easy to remember.

---

# 24. What does `rhgb quiet` mean?

You will often see:

```text
rhgb quiet
```

### `rhgb`

Red Hat Graphical Boot.

### `quiet`

Reduces the amount of boot messages displayed.

So:

```text
rhgb quiet
```

generally makes the boot process look cleaner/less verbose.

For troubleshooting, you may want to remove `quiet` so you can see more boot messages.

---

# 25. Why kernel parameters matter during troubleshooting

Suppose a server gets stuck during boot.

You may need to temporarily modify the kernel command line.

For example:

```text
GRUB
 ↓
Press E
 ↓
Modify kernel parameter
 ↓
Boot
 ↓
Troubleshoot
```

This is particularly useful when:

* The system cannot boot normally
* A service causes boot problems
* Root filesystem cannot be accessed
* You need emergency/recovery access
* A kernel parameter is causing a problem

---

# 26. Example troubleshooting scenario

Imagine you updated the kernel:

```text
Old Kernel
   ↓
Working

New Kernel
   ↓
Server doesn't boot
```

At the GRUB menu:

```text
GRUB
 ├── RHEL - New Kernel ❌
 └── RHEL - Old Kernel ✅
```

Select:

```text
Old Kernel
```

Boot the server.

Then investigate:

```bash
uname -r
```

This tells you which kernel is currently running.

You can also check:

```bash
rpm -q kernel
```

to see installed kernel packages.

---

# 27. Important commands for this lesson

| Command                                          | Purpose                                |
| ------------------------------------------------ | -------------------------------------- |
| `grubby --info 0`                                | View boot entry 0                      |
| `grubby --info 1`                                | View boot entry 1                      |
| `grubby --info=ALL`                              | View all boot entries                  |
| `grubby --set-default-index 0`                   | Set entry 0 as default                 |
| `grubby --update-kernel ... --args="..."`        | Add kernel arguments                   |
| `grubby --update-kernel ... --remove-args="..."` | Remove kernel arguments                |
| `cat /proc/cmdline`                              | Show parameters used by current kernel |
| `uname -r`                                       | Show currently running kernel          |
| `rpm -q kernel`                                  | List installed kernel packages         |
| `lsblk -f`                                       | Show filesystems and UUIDs             |
| `blkid`                                          | Show block-device UUIDs                |

---

# 28. One very important distinction

Don't confuse these three things:

### GRUB2

```text
Boot loader
```

Responsible for loading the kernel.

### Linux Kernel

```text
Core of the operating system
```

Manages CPU, memory, devices, processes, etc.

### systemd

```text
System/service manager
```

Takes over after the kernel/early boot stages and starts services and targets.

So:

```text
GRUB2
"What should I load?"

      ↓

Kernel
"Start Linux"

      ↓

systemd
"Start the system and services"
```

---

# 29. Full picture for RHCSA

Memorize this flow:

```text
                    POWER ON
                       │
                       ▼
                 BIOS / UEFI
                       │
              Finds boot loader
                       │
                       ▼
                    GRUB2
                       │
             Selects boot entry
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          Kernel              initramfs
             └─────────┬─────────┘
                       ▼
                 Root filesystem
                       │
                       ▼
                    systemd
                       │
                       ▼
               System target
                       │
                       ▼
                  Services
                       │
                       ▼
                    LOGIN
```

And the configuration side:

```text
GRUB menu
   │
   └── Press E
          ↓
     Temporary change


Running system
   │
   └── grubby
          ↓
     Persistent change
```

---

# ⭐ RHCSA Exam Cheat Sheet

### Boot loader

```text
RHEL → GRUB2
```

### Modern firmware

```text
UEFI
```

### UEFI boot partition

```text
/boot/efi
```

### View current kernel command line

```bash
cat /proc/cmdline
```

### Current kernel version

```bash
uname -r
```

### View GRUB entry

```bash
grubby --info 0
```

### View all entries

```bash
grubby --info=ALL
```

### Set default entry

```bash
grubby --set-default-index 0
```

### Add parameter

```bash
grubby --update-kernel <kernel> --args="parameter"
```

### Remove parameter

```bash
grubby --update-kernel <kernel> --remove-args="parameter"
```

### Temporary GRUB modification

```text
GRUB → E
```

### Persistent modification

```text
grubby
```

### Kernel

```text
/boot/vmlinuz-...
```

### initramfs

```text
/boot/initramfs-...img
```

---

## 🧠 The easiest way to remember

Use this sentence:

> **UEFI finds GRUB, GRUB loads the Kernel, initramfs helps find the root filesystem, and systemd starts the system.**

And for troubleshooting:

> **Press `E` for a temporary boot change; use `grubby` for a persistent change.**

That is the core idea of this entire **“Managing the Boot Loader and Kernel Command Line”** section.
