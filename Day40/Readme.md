# Enabling Services to Start or Stop at Boot 

When managing services in Linux with **systemd**, there are **two separate concepts** that beginners often confuse:

1. **Is the service running right now?** (Current Session)
2. **Should the service automatically start after the system reboots?** (Boot Time)

These are **independent** of each other.

---

# Understanding the Difference

Imagine a **ceiling fan** in your room.

* Pressing the **ON** button starts the fan immediately.
* Pressing the **OFF** button stops it immediately.

But imagine there is another switch that decides:

> "Whenever electricity comes back after a power outage, should the fan automatically turn ON?"

That second switch is like **Enable/Disable**.

| Fan Example                               | Linux Equivalent    |
| ----------------------------------------- | ------------------- |
| Turn fan ON                               | `systemctl start`   |
| Turn fan OFF                              | `systemctl stop`    |
| Automatically turn ON after power returns | `systemctl enable`  |
| Do not automatically turn ON              | `systemctl disable` |

So Linux separates:

* **Running now**
* **Start automatically later**

---

# 1. Starting a Service

Command:

```bash
systemctl start sshd.service
```

What happens?

* SSH service starts immediately.
* Users can connect through SSH.
* If you reboot the server, **it may not start again**.

### Before

```
Server Booted

SSHD
❌ Not Running
```

After running:

```bash
systemctl start sshd.service
```

```
SSHD
✅ Running
```

Current Status:

```
Running : YES
Boot     : NO
```

---

# 2. Stopping a Service

Command:

```bash
systemctl stop sshd.service
```

What happens?

* SSH service stops immediately.
* Nobody can connect through SSH.
* If the service is enabled, it will start again after reboot.

Current Status:

```
Running : NO
Boot     : Depends on enable/disable
```

---

# 3. Enabling a Service

Command:

```bash
systemctl enable sshd.service
```

This **does NOT start the service immediately.**

Instead, it tells Linux:

> "Every time the computer starts, automatically start SSH."

### Before

```
Boot Process

↓

SSHD

❌ Not configured to start
```

After enabling:

```
Boot Process

↓

Systemd

↓

Automatically starts SSHD
```

Current Status:

```
Running : NO (if not already started)
Boot     : YES
```

---

# Why Doesn't Enable Start the Service?

Because **Enable only changes the boot configuration**.

Think of it like setting an alarm for tomorrow.

Setting the alarm doesn't wake you up now.

It only affects tomorrow.

---

# What Does `enable` Actually Do?

It creates a **symbolic link**.

Original service file:

```
/usr/lib/systemd/system/sshd.service
```

Systemd creates:

```
/etc/systemd/system/multi-user.target.wants/sshd.service
```

This symbolic link tells systemd:

```
Whenever the system reaches
multi-user.target

↓

Start sshd.service
```

Output:

```bash
systemctl enable sshd.service
```

```
Created symlink
/etc/systemd/system/multi-user.target.wants/sshd.service
→
/usr/lib/systemd/system/sshd.service
```

---

# What is a Symbolic Link?

A symbolic link (symlink) is simply a **shortcut**.

Example in Windows:

```
Desktop Shortcut

↓

Chrome.exe
```

Linux:

```
Shortcut

↓

sshd.service
```

The actual file stays here:

```
/usr/lib/systemd/system/
```

The shortcut is here:

```
/etc/systemd/system/multi-user.target.wants/
```

During boot, systemd checks the shortcuts.

If a shortcut exists:

```
Start this service.
```

---

# 4. Starting AND Enabling Together

Instead of typing:

```bash
systemctl start sshd.service
```

then

```bash
systemctl enable sshd.service
```

Use:

```bash
systemctl enable --now sshd.service
```

This performs **two actions**:

```
Action 1

Start SSH Now

        +

Action 2

Enable SSH for Next Boot
```

Result:

```
Running : YES

Boot     : YES
```

---

# 5. Disabling a Service

Command:

```bash
systemctl disable sshd.service
```

This means:

> "Do not automatically start this service after reboot."

Linux removes the symbolic link.

Output:

```
Removed
/etc/systemd/system/multi-user.target.wants/sshd.service
```

Notice:

The service may still be running.

```
Running : YES

Boot     : NO
```

---

# Why Doesn't Disable Stop the Service?

Because disable only changes future behavior.

Example:

You cancel tomorrow's alarm.

That doesn't make you fall asleep immediately.

Same concept.

---

# 6. Stop AND Disable Together

Instead of:

```bash
systemctl stop sshd.service
```

and

```bash
systemctl disable sshd.service
```

Use:

```bash
systemctl disable --now sshd.service
```

This means:

```
Stop Now

+

Never Start Automatically
```

Result:

```
Running : NO

Boot     : NO
```

---

# Checking Whether a Service is Enabled

Command:

```bash
systemctl is-enabled sshd.service
```

Possible outputs:

```
enabled
```

Meaning:

```
Will automatically start after reboot.
```

or

```
disabled
```

Meaning:

```
Will NOT automatically start after reboot.
```

---

# Complete Workflow Example

### Step 1

Start SSH.

```bash
systemctl start sshd.service
```

Status:

```
Running : YES

Boot     : NO
```

---

### Step 2

Enable SSH.

```bash
systemctl enable sshd.service
```

Status:

```
Running : YES

Boot     : YES
```

---

### Step 3

Reboot server.

```
Server Reboots

↓

Systemd checks enabled services

↓

Starts SSH automatically
```

SSH starts without manual intervention.

---

### Step 4

Disable SSH.

```bash
systemctl disable sshd.service
```

Status:

```
Running : YES

Boot     : NO
```

The service is still running until you stop it.

---

### Step 5

Stop SSH.

```bash
systemctl stop sshd.service
```

Status:

```
Running : NO

Boot     : NO
```

---

# Understanding `mask`

`mask` is stronger than `disable`.

Normally:

```
Disabled

↓

Administrator can still run

systemctl start sshd
```

But if masked:

```bash
systemctl mask sshd.service
```

Even this command fails:

```bash
systemctl start sshd.service
```

Reason:

Systemd blocks the service completely.

Think of it like this:

```
Disable

↓

Door is closed

But you can still open it.

-------------------------

Mask

↓

Door is locked with chains.

Nobody can open it.
```

To allow it again:

```bash
systemctl unmask sshd.service
```

---

# Summary Table

| Command                   | Starts Now          | Stops Now | Starts After Boot        | Notes                                       |
| ------------------------- | ------------------- | --------- | ------------------------ | ------------------------------------------- |
| `systemctl start`         | ✅ Yes               | ❌ No      | ❌ No                     | Start immediately                           |
| `systemctl stop`          | ❌ No                | ✅ Yes     | ❌ No                     | Stop immediately                            |
| `systemctl enable`        | ❌ No                | ❌ No      | ✅ Yes                    | Auto-start after reboot                     |
| `systemctl disable`       | ❌ No                | ❌ No      | ❌ No                     | Prevent auto-start                          |
| `systemctl enable --now`  | ✅ Yes               | ❌ No      | ✅ Yes                    | Start now + enable                          |
| `systemctl disable --now` | ❌ No                | ✅ Yes     | ❌ No                     | Stop now + disable                          |
| `systemctl restart`       | ✅ Yes               | ✅ Yes     | No change                | Restart service                             |
| `systemctl reload`        | Reload config       | ❌ No      | No change                | No restart if supported                     |
| `systemctl mask`          | ❌ No                | ❌ No      | ❌ No                     | Completely blocks starting                  |
| `systemctl unmask`        | ❌ No                | ❌ No      | Restores normal behavior | Removes the block                           |
| `systemctl status`        | Shows current state | —         | —                        | Displays detailed service information       |
| `systemctl is-enabled`    | —                   | —         | Shows boot configuration | Displays `enabled`, `disabled`, or `masked` |

## Quick Memory Trick

* **start** → Run the service **now**.
* **stop** → Stop the service **now**.
* **enable** → Start the service **every time the system boots**.
* **disable** → Do not start the service **after reboot**.
* **enable --now** → **Start now + Enable for boot**.
* **disable --now** → **Stop now + Disable for boot**.
* **mask** → **Completely prevent** the service from being started, even manually.
* **unmask** → Remove the block so the service can be started again.

The key idea is to always remember that **`start`/`stop` affect the current running system**, while **`enable`/`disable` affect what happens during the next and future boots**.
