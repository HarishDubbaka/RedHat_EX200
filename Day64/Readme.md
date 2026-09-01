## SELinux Booleans 

**SELinux Booleans** are switches that let you **turn optional SELinux policy behaviors ON or OFF** without changing the SELinux policy itself.

Think of them like **configuration switches**:

> **Boolean OFF → SELinux blocks the optional behavior**
> **Boolean ON → SELinux allows the optional behavior**

### 1. List all SELinux Booleans

```bash
getsebool -a
```

Example:

```text
abrt_anon_write --> off
abrt_handle_event --> off
abrt_upload_watch_anon_write --> on
httpd_enable_homedirs --> off
```

You can check a specific Boolean:

```bash
getsebool httpd_enable_homedirs
```

Output:

```text
httpd_enable_homedirs --> off
```

---

## 2. Temporary vs Persistent

This is the **most important interview point**.

### Temporary change

```bash
setsebool httpd_enable_homedirs on
```

This changes the Boolean **immediately**, but the change is **not persistent across reboot**.

Verify:

```bash
getsebool httpd_enable_homedirs
```

Output:

```text
httpd_enable_homedirs --> on
```

After reboot, it returns to its previous persistent/default setting.

### Persistent change

```bash
setsebool -P httpd_enable_homedirs on
```

The `-P` means:

> **Persist the Boolean setting across reboot.**

Verify:

```bash
getsebool httpd_enable_homedirs
```

And:

```bash
semanage boolean -l | grep httpd_enable_homedirs
```

---

## 3. Understanding `semanage boolean -l`

Run:

```bash
semanage boolean -l
```

For a specific Boolean:

```bash
semanage boolean -l | grep httpd_enable_homedirs
```

Example:

```text
httpd_enable_homedirs    (on, off)    Allow httpd to enable homedirs
```

The two values represent:

```text
(on, off)
 ↑    ↑
 |    |
Current  Default
```

So:

```text
(on, off)
```

means:

* **Current = ON**
* **Default = OFF**

Therefore, it is currently enabled but **not persistent**.

After:

```bash
setsebool -P httpd_enable_homedirs on
```

you would see:

```text
(on, on)
```

Meaning:

* Current = ON
* Default/persistent = ON

---

## 4. Example: `httpd_enable_homedirs`

Suppose Apache/httpd needs to serve files from users' home directories.

By default:

```bash
getsebool httpd_enable_homedirs
```

```text
httpd_enable_homedirs --> off
```

SELinux does not allow this optional behavior.

Enable it temporarily:

```bash
setsebool httpd_enable_homedirs on
```

For permanent configuration:

```bash
setsebool -P httpd_enable_homedirs on
```

Now httpd can access appropriately labeled home-directory content.

---

## 5. Turn a Boolean OFF

Temporary:

```bash
setsebool httpd_enable_homedirs off
```

Persistent:

```bash
setsebool -P httpd_enable_homedirs off
```

---

## 6. Find only customized Booleans

This is useful for checking what an administrator has changed from the original policy:

```bash
semanage boolean -l -C
```

Example:

```text
SELinux boolean                State  Default Description

httpd_enable_homedirs          (on   , off)  Allow httpd to enable homedirs
```

This tells you that the Boolean has been customized from its default.

---

# 🔥 Interview-Friendly Summary

| Command                      | Purpose                              |
| ---------------------------- | ------------------------------------ |
| `getsebool -a`               | List all Booleans and current status |
| `getsebool <boolean>`        | Check one Boolean                    |
| `setsebool <boolean> on`     | Enable temporarily                   |
| `setsebool <boolean> off`    | Disable temporarily                  |
| `setsebool -P <boolean> on`  | Enable permanently                   |
| `setsebool -P <boolean> off` | Disable permanently                  |
| `semanage boolean -l`        | Show Boolean details                 |
| `semanage boolean -l -C`     | Show customized Booleans             |

### ⭐ Remember this

```text
getsebool
    ↓
Check current status

setsebool
    ↓
Change current status

setsebool -P
    ↓
Change + make persistent

semanage boolean -l
    ↓
Current + Default + Description
```

**Interview answer:**

> “SELinux Booleans are policy switches that allow administrators to enable or disable optional SELinux policy behavior without modifying the policy itself. I use `getsebool` to check the current state, `setsebool` for a runtime change, and `setsebool -P` when the change needs to persist across reboot.”
