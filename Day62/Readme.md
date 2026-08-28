## 1. What is SELinux?

**SELinux is an additional security layer in Linux** that controls what processes can access and what actions they are allowed to perform.

Normal Linux permissions answer:

> **“Who can access this file?”**

SELinux additionally answers:

> **“Is this process allowed to access this resource in this particular way?”**

The file explains that SELinux provides granular control over files, ports, processes, and other resources. 

### Simple example

Suppose Apache is running as the `apache` user.

Linux permissions might allow:

```text
apache → read /data/file.txt
```

But SELinux can still say:

```text
Apache process → NOT allowed to access /data/file.txt
```

So even if normal Linux permissions allow access, **SELinux can deny it**.

---

# 2. DAC vs MAC

This is very important for interviews.

### DAC — Discretionary Access Control

This is the normal Linux permission system:

```text
User → Owner / Group / Others → r / w / x
```

Example:

```bash
-rw-r--r--  harish  users  file.txt
```

The owner can decide permissions.

### MAC — Mandatory Access Control

SELinux provides **MAC**.

SELinux policies determine what processes can access, and these rules apply regardless of normal discretionary permissions. 

Think of it like:

```text
Linux permissions
       +
SELinux policy
       ↓
Final access decision
```

So:

**DAC = Who can access?**

**MAC/SELinux = What is the process allowed to do?**

---

# 3. How SELinux works

SELinux uses **security contexts/labels**.

Every important resource can have an SELinux context:

* Files
* Directories
* Processes
* Ports

SELinux checks the label against its policy. If there is no rule allowing the access, access is denied. 

For example:

```text
Apache process
      ↓
   httpd_t
      ↓
SELinux Policy
      ↓
Can it access the file?
      ↓
YES / NO
```

---

# 4. SELinux Context

An SELinux context generally looks like:

```text
user:role:type:security_level
```

Example:

```text
system_u:object_r:httpd_sys_content_t:s0
```

There are four fields:

```text
system_u
   ↓
user

object_r
   ↓
role

httpd_sys_content_t
   ↓
type

s0
   ↓
security level
```

For the **targeted policy**, the most important field for administrators is generally the **type**.

The file notes that targeted policy is the default in RHEL and uses the type context for its rules. 

---

# 5. What is Targeted Policy?

RHEL uses **targeted SELinux policy by default**.

It mainly protects specific services/applications.

For example:

```text
Apache
   ↓
httpd_t
```

Apache's files may have:

```text
httpd_sys_content_t
```

Apache ports may have:

```text
http_port_t
```

The SELinux policy defines what `httpd_t` can access. 

### Important types to remember

| Resource                  | SELinux type              |
| ------------------------- | ------------------------- |
| Apache process            | `httpd_t`                 |
| Apache web content        | `httpd_sys_content_t`     |
| Apache executable scripts | `httpd_sys_script_exec_t` |
| Temporary files           | `tmp_t`                   |
| MariaDB process           | `mysqld_t`                |
| MariaDB database files    | `mysqld_db_t`             |
| HTTP port                 | `http_port_t`             |

---

# 6. Real-world example

Imagine you create:

```bash
mkdir /webdata
echo "Hello" > /webdata/index.html
```

You configure Apache to use:

```text
/webdata
```

Linux permissions might be perfectly correct.

But SELinux may see:

```text
/webdata/index.html
        ↓
wrong SELinux context
        ↓
Apache = DENIED
```

Why?

Because Apache expects its content to have an appropriate SELinux type such as:

```text
httpd_sys_content_t
```

This is one of the most common SELinux troubleshooting scenarios.

---

# 7. SELinux Modes

There are three modes:

### 1. Enforcing

```text
SELinux = ON
Policy = enforced
Violations = blocked + logged
```

This is the **default and recommended mode** in RHEL. 

---

### 2. Permissive

```text
SELinux = ON
Policy = NOT enforced
Violations = logged
```

The system allows the operation but records the SELinux violation.

Useful for:

* Testing
* Troubleshooting
* Policy development



---

### 3. Disabled

```text
SELinux = OFF
```

SELinux policies aren't enforced or logged.

Also, files/directories aren't labeled, making future re-enablement difficult. The source strongly discourages disabling SELinux. 

### Easy way to remember

```text
Enforcing  → Deny + Log
Permissive → Allow + Log
Disabled   → No SELinux
```

---

# 8. Check current SELinux mode

Use:

```bash
getenforce
```

Example:

```bash
[root@server ~]# getenforce
Enforcing
```

So the server is currently running in **Enforcing** mode. 

---

# 9. Temporarily change SELinux mode

Use:

```bash
setenforce
```

Syntax:

```bash
setenforce Enforcing
```

or:

```bash
setenforce 1
```

For permissive:

```bash
setenforce Permissive
```

or:

```bash
setenforce 0
```

Example:

```bash
setenforce 0
```

Then:

```bash
getenforce
```

Output:

```text
Permissive
```



### Important

`setenforce` changes the **current runtime mode**. It does not permanently configure the mode for future boots.

---

# 10. Permanent SELinux configuration

The configuration file is:

```bash
/etc/selinux/config
```

Example:

```bash
SELINUX=enforcing
SELINUXTYPE=targeted
```

The system reads this configuration during boot. 

### Important distinction

```text
getenforce
     ↓
Current mode

setenforce
     ↓
Temporary/runtime change

/etc/selinux/config
     ↓
Default mode at boot
```

---

# 11. `-Z` option

A very important RHCSA concept is the `-Z` option.

Many commands support `-Z` to display SELinux contexts.

For example:

```bash
ls -Z
```

shows file/directory contexts.

Example:

```bash
ls -Z /var/www
```

Output can look like:

```text
system_u:object_r:httpd_sys_content_t:s0 html
```



---

## 12. Check process SELinux context

Use:

```bash
ps -Z
```

For Apache:

```bash
ps -ZC httpd
```

You may see:

```text
system_u:system_r:httpd_t:s0
```

This tells you Apache is running under:

```text
httpd_t
```



---

# 13. Check file SELinux context

Use:

```bash
ls -Z
```

Example:

```bash
ls -Z /var/www
```

You might see:

```text
httpd_sys_content_t
```

This means the file has an SELinux type that Apache's policy can use for web content. 

---

# 14. SELinux decision-making

The basic logic is:

```text
Process
   ↓
Process SELinux context
   ↓
SELinux Policy
   ↓
Resource SELinux context
   ↓
Is access explicitly allowed?
   ↓
YES              NO
 ↓                 ↓
ALLOW            DENY
```

The key rule is:

> **If an SELinux policy does not explicitly allow the access, SELinux denies it.** 

---

# 15. Why SELinux is important

Imagine Apache gets compromised.

Without SELinux:

```text
Attacker
   ↓
Compromised Apache
   ↓
Potentially access files
   ↓
More damage
```

With SELinux:

```text
Attacker
   ↓
Compromised Apache
   ↓
httpd_t
   ↓
SELinux policy
   ↓
Only permitted resources
```

Even if the Apache process is compromised, SELinux can restrict what that process is allowed to access. The chapter specifically describes this protection against unintended access by compromised services. 

---

# 16. Important commands for RHCSA

| Requirement                | Command               |
| -------------------------- | --------------------- |
| Check current mode         | `getenforce`          |
| Set enforcing temporarily  | `setenforce 1`        |
| Set permissive temporarily | `setenforce 0`        |
| View file context          | `ls -Z`               |
| View process context       | `ps -Z`               |
| View Apache context        | `ps -ZC httpd`        |
| Persistent configuration   | `/etc/selinux/config` |

---

# 17. Interview questions

### Q1. What is SELinux?

**Answer:**
SELinux is a Linux security mechanism that provides Mandatory Access Control (MAC) using security policies and contexts to control access between processes and resources.

### Q2. Difference between DAC and MAC?

**Answer:**

```text
DAC → Linux user/group/file permissions
MAC → SELinux security policies
```

DAC controls access based on ownership and permissions, while SELinux provides an additional mandatory policy-based security layer. 

### Q3. What are the three SELinux modes?

```text
Enforcing
Permissive
Disabled
```

### Q4. Which is the default recommended mode?

```text
Enforcing
```

### Q5. How do you check SELinux status?

```bash
getenforce
```

### Q6. How do you temporarily change to permissive?

```bash
setenforce 0
```

### Q7. How do you check an SELinux file context?

```bash
ls -Z filename
```

### Q8. How do you check an application's SELinux context?

```bash
ps -Z
```

### Q9. Where is the persistent SELinux configuration?

```bash
/etc/selinux/config
```

### Q10. What is `httpd_t`?

It is the SELinux **type context for the Apache/httpd process** under the targeted policy. 

---

## ⭐ Most important points to remember

```text
SELinux
  │
  ├── Additional Linux security layer
  │
  ├── MAC = Mandatory Access Control
  │
  ├── Uses security contexts/labels
  │
  ├── Targeted policy = default in RHEL
  │
  ├── Enforcing  = Deny + Log
  ├── Permissive = Allow + Log
  └── Disabled   = SELinux off
```

And memorize these commands:

```bash
getenforce
setenforce 0
setenforce 1
ls -Z
ps -Z
ps -ZC httpd
cat /etc/selinux/config
```

**One-line interview definition:**

> **SELinux is a Mandatory Access Control security mechanism that uses policies and security contexts to restrict what processes can access, even when normal Linux file permissions would otherwise allow the access.** 
