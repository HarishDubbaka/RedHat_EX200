## 🔐 Recovering Superuser Access

### 🎯 Objective

The main goal is:

> **Gain administrative/root access when you don't know the root password.**

Normally, you can reset the password using `passwd` if you already have root or sudo access. If you have **no administrative access**, you need to boot the system into a **rescue environment**.

There are two approaches:

1. **Using rescue media — recommended**
2. **Without rescue media — advanced/risky**

---

# 1. Using Rescue Media — Recommended

![Image](https://images.openai.com/static-rsc-4/JxfbFWO3L2lBV5kFC4McDuJDjcPdPRZ_zDdaLWJsbfbOoE5QoyYe7xsq23fnHLjC0HemWfC7KsQYVotAZOt2bgkK9qtIXCICDhC1Z8wyHxuXTbbKNMNSIHYwMVh7W3i3v_M0HN3UaXf5l1H5oJ918amgFWXsqAUf7QbEnn5u4fZTQWlIlZ92fG7TmNH9IqCl?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/NlhMEWf6hhutNC5fp1p0T_jV6zHdyI9PKC_7gZ3RF8ronUXEfaNzpB2DPms394Ct-NMUsihdU_aO1V_et2g4cBGvSX8bn4KSpDXVHpZPrAZw_BlG0_DACziqZHu9ReK1HDQDwIQN8okGNgJiwDep6L1pbkPWaa6SEKcpX3TSoR3dQ4616bftnC2IdxcLKk3b?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/OviysP1cQrVNcrxYXQ8GlaJq_1JvZDlYPhXLYzWY33tA_GqRoLLr5YQe2zQJexPT48x8MjHbHUTczYLrTDHpCHUYXhUhZY4XpsBwh7a4AjF_JzMBBTZF6L9qrGACjLaP80xhRFPJU_nsF66lB1m4NMWL2jc9BctYu50tvZT2Ucr-A1w1dRkedzarGNU-X8XB?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/OC2wUOyeyYnJf9-u34mJiF-qQ0T77KOlOYgOLpziIRanpj-5uYHBZaFBpv_QTxP2HcH7s4KKO727jSgsYYBX1yjEIsvVLCPFJg2nqGllPN_2nCUGo3ubDxtitRxcAOXrAX_XWqxpsXZZ9PTqsfF6a1_DiSiqHAwdfXNWVcpY7mE8ULO3GwknaTeSvlhExa2f?purpose=fullsize)

This method uses a RHEL installation ISO/Boot ISO to start a temporary rescue environment.

### Step 1 — Boot from RHEL ISO

Reboot the machine and boot from the appropriate RHEL installation media.

For a VM, this is normally an **ISO file**.

**Important:** Use rescue media compatible with the installed RHEL version.

---

### Step 2 — Select Rescue Mode

From the GRUB/boot menu:

**Troubleshooting → Rescue a Red Hat Enterprise Linux system**

This starts the rescue environment.

---

### Step 3 — Mount the Existing System

When the rescue menu appears, select:

**Continue**

The existing RHEL installation is mounted under:

```bash
/mnt/sysroot
```

---

### Step 4 — Enter the Installed System

Use:

```bash
chroot /mnt/sysroot
```

### What does `chroot` do?

`chroot` means **change root directory**.

It makes `/mnt/sysroot` appear as `/` for the commands you execute.

So instead of modifying the temporary rescue environment, you're working with the **actual installed RHEL system**.

---

### Step 5 — Reset the Root Password

Run:

```bash
passwd root
```

Then enter the new password twice:

```text
New password:
Retype new password:
passwd: password updated successfully
```

---

### Step 6 — Handle SELinux Relabeling

This is a **very important step**.

Run:

```bash
touch /.autorelabel
```

### Why?

The rescue environment has not initialized SELinux in the normal way.

When `passwd` modifies `/etc/shadow`, the newly created/modified file can have an incorrect or missing SELinux context.

Creating:

```text
/.autorelabel
```

tells RHEL:

> "During the next boot, relabel the files with the correct SELinux contexts."

---

### Step 7 — Exit and Reboot

You are inside the `chroot`, so exit twice:

```bash
exit
exit
```

Think of it as:

```text
First exit  → leave chroot
Second exit → leave rescue environment/reboot
```

The system then:

1. Boots normally
2. Performs SELinux relabeling
3. Reboots again
4. Allows you to log in with the new root password

---

# 🧠 Complete Rescue-Media Flow

Memorize this sequence:

```text
Boot RHEL ISO
     ↓
Troubleshooting
     ↓
Rescue a Red Hat Enterprise Linux system
     ↓
Continue
     ↓
chroot /mnt/sysroot
     ↓
passwd root
     ↓
touch /.autorelabel
     ↓
exit
     ↓
exit
     ↓
SELinux relabel
     ↓
Reboot
     ↓
Login with new root password
```

---

# 2. Without Rescue Media

This is an **advanced method** and should be used carefully.

Instead of booting from an ISO, you modify the GRUB kernel parameters so the system starts directly with a Bash shell.

### Step 1 — Reboot

Restart the system.

During boot, press:

**Esc**

to interrupt the boot-loader countdown.

---

### Step 2 — Edit the GRUB Entry

Select the kernel entry and press:

```text
E
```

Find the line beginning with something like:

```text
linux ...
```

Remove any `console=` options as instructed in the exercise.

Go to the end of the line with:

```text
Ctrl + E
```

Add:

```text
init=/bin/bash
```

Then boot using:

```text
Ctrl + X
```

---

### Step 3 — Remount `/` as Read/Write

The root filesystem initially mounts as **read-only**.

Therefore, this won't work properly until you remount it.

Run:

```bash
mount -o remount,rw /
```

Now `/` is writable.

---

### Step 4 — Change Root Password

Run:

```bash
passwd
```

Enter the new password.

---

### Step 5 — Create SELinux Relabel Marker

Run:

```bash
touch /.autorelabel
```

Again, this ensures SELinux contexts are corrected during the next boot.

---

### Step 6 — Continue Normal Boot

Run:

```bash
exec /sbin/init
```

The normal initialization process starts.

The system performs the SELinux relabeling and reboots.

---

# ⚖️ Two Methods Compared

| Feature          | Rescue Media                       | Without Rescue Media  |
| ---------------- | ---------------------------------- | --------------------- |
| Recommended      | ✅ Yes                              | ❌ No                  |
| Requires ISO     | ✅ Yes                              | ❌ No                  |
| Difficulty       | Easy/Moderate                      | Advanced              |
| Risk             | Lower                              | Higher                |
| Main command     | `chroot /mnt/sysroot`              | `init=/bin/bash`      |
| Root filesystem  | Mounted through rescue environment | Remount manually      |
| Password command | `passwd root`                      | `passwd`              |
| SELinux step     | `touch /.autorelabel`              | `touch /.autorelabel` |

---

# ⭐ Important Commands to Remember

### Rescue method

```bash
chroot /mnt/sysroot
passwd root
touch /.autorelabel
exit
exit
```

### No-rescue-media method

```bash
mount -o remount,rw /
passwd
touch /.autorelabel
exec /sbin/init
```

---

## 🧩 Why Each Command Is Used

| Command                 | Purpose                            |
| ----------------------- | ---------------------------------- |
| `chroot /mnt/sysroot`   | Enter the installed RHEL system    |
| `passwd root`           | Set a new root password            |
| `mount -o remount,rw /` | Make root filesystem writable      |
| `touch /.autorelabel`   | Request SELinux relabeling         |
| `exit`                  | Leave the current shell/chroot     |
| `exec /sbin/init`       | Start normal system initialization |

### 💡 Interview/Exam Point

If asked:

**"How do you reset a forgotten root password in RHEL?"**

A good short answer is:

> Boot into RHEL rescue mode, mount the installed system, use `chroot /mnt/sysroot`, reset the password with `passwd root`, create `/.autorelabel` for SELinux relabeling, and reboot.

The **most important thing to remember is `/.autorelabel`**. Without it, SELinux file contexts can cause problems after changing the password from the rescue environment.
