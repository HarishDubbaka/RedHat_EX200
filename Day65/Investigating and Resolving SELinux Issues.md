# Investigating and Resolving SELinux Issues

## 1. What is the main idea?

When an application suddenly gets **Permission denied**, SELinux may be blocking it.

But an important point is:

> **An SELinux denial does not necessarily mean SELinux is broken. It often means SELinux is working correctly.**

SELinux uses **labels + policies** to decide what a process is allowed to access.

Think of it like:

**Process → Resource → Action**

For example:

```text
httpd_t → httpd_sys_content_t → read
```

The SELinux policy determines whether this action is allowed.

If the policy has no rule allowing the action:

```text
DENY
 ↓
AVC message generated
 ↓
Logged in audit.log
```

---

# 2. How SELinux decides access

Every important object has an SELinux **context/label**.

For example:

```text
Process:
system_u:system_r:httpd_t:s0

File:
system_u:object_r:httpd_sys_content_t:s0
```

The important parts here are:

* `httpd_t` → process type
* `httpd_sys_content_t` → file type

SELinux policy contains rules such as:

```text
httpd_t
   |
   | read
   ↓
httpd_sys_content_t
```

Therefore Apache can read files labeled `httpd_sys_content_t`.

If the file has the wrong label:

```text
httpd_t
   |
   | read
   ↓
admin_home_t
```

SELinux may deny the access.

---

# 3. Three things to remember

For an SELinux access decision, remember:

### Process

**Who is trying to access?**

Example:

```text
httpd_t
```

### Resource

**What is being accessed?**

Example:

```text
/var/www/html/mypage
```

with context:

```text
admin_home_t
```

### Action

**What is the process trying to do?**

Examples:

```text
read
write
getattr
execute
```

So an AVC denial can essentially be read as:

```text
httpd_t
    ↓
wants getattr
    ↓
file labeled admin_home_t
    ↓
DENIED
```

---

# 4. What is an AVC?

**AVC = Access Vector Cache**

When SELinux denies an action, it generates an **AVC denial message**.

The main audit log is:

```bash
/var/log/audit/audit.log
```

Example:

```text
avc: denied { getattr }
```

This means SELinux denied the requested operation.

---

# 5. Important SELinux troubleshooting tools

The most important commands are:

| Command             | Purpose                              |
| ------------------- | ------------------------------------ |
| `sealert`           | Analyze SELinux alerts               |
| `ausearch`          | Search audit logs                    |
| `restorecon`        | Restore correct SELinux contexts     |
| `semanage fcontext` | Define persistent file-context rules |
| `ls -Z`             | View SELinux context                 |
| `ps -Z`             | View process context                 |
| `getsebool`         | View Boolean settings                |
| `setsebool`         | Change Boolean settings              |
| `audit2allow`       | Generate policy rules from denials   |

For this lesson, focus especially on:

```bash
sealert
ausearch
restorecon
```

---

# 6. `sealert`

The `setroubleshoot-server` package provides SELinux troubleshooting tools.

When SELinux detects a denial, `sealert` can provide a human-readable explanation.

### View a specific event

```bash
sealert -l UUID
```

Example:

```bash
sealert -l a0bca4b2-46a9-4252-b0cd-6de0dfcdd454
```

### Analyze all audit events

```bash
sealert -a /var/log/audit/audit.log
```

---

# 7. Understanding the Apache example

Suppose Apache normally serves:

```text
/var/www/html/
```

You create:

```bash
touch /root/mypage
```

Then move it:

```bash
mv /root/mypage /var/www/html/
```

At first glance, this looks correct.

The file is now physically located here:

```text
/var/www/html/mypage
```

You start Apache:

```bash
systemctl start httpd
```

Then:

```bash
curl http://localhost/mypage
```

You receive:

```text
403 Forbidden
```

Why?

The Linux file location is correct, but the **SELinux label is wrong**.

---

# 8. Check the SELinux context

Use:

```bash
ls -Z /var/www/html/mypage
```

The example shows:

```text
unconfined_u:object_r:admin_home_t:s0
```

The problem is:

```text
admin_home_t
```

Apache normally expects web content to have a type such as:

```text
httpd_sys_content_t
```

So:

```text
Actual:
admin_home_t

Expected:
httpd_sys_content_t
```

That's why SELinux blocks Apache.

---

# 9. Understand the AVC message

The important part is:

```text
avc: denied { getattr }
```

Let's break it down.

```text
scontext=system_u:system_r:httpd_t:s0
```

This is the **source/process context**.

So:

```text
httpd_t
```

is the process type.

Then:

```text
tcontext=unconfined_u:object_r:admin_home_t:s0
```

This is the **target/file context**.

So:

```text
admin_home_t
```

is the file type.

And:

```text
{ getattr }
```

is the denied operation.

Therefore:

> Apache (`httpd_t`) tried to perform `getattr` on a file labeled `admin_home_t`, and SELinux denied it.

---

# 10. The correct solution: `restorecon`

The file is in the **correct location**.

The problem is its label.

Therefore, don't create a new SELinux policy.

Simply restore the expected label:

```bash
restorecon -v /var/www/html/mypage
```

You can then check:

```bash
ls -Z /var/www/html/mypage
```

It should now have the appropriate web-content type:

```text
httpd_sys_content_t
```

Then:

```bash
curl http://localhost/mypage
```

Apache should be able to access it.

---

# 11. Why did `restorecon` fix it?

SELinux maintains default file-context rules.

For example, the policy knows:

```text
/var/www/html(/.*)?
        ↓
httpd_sys_content_t
```

When you run:

```bash
restorecon /var/www/html/mypage
```

SELinux checks the default context for that path and applies it.

Conceptually:

```text
Wrong label
admin_home_t
      ↓
restorecon
      ↓
Correct label
httpd_sys_content_t
```

---

# 12. Fixing multiple files

If many files have incorrect contexts:

```bash
restorecon -R /var/www/html/
```

`-R` means **recursive**.

You can use:

```bash
restorecon -Rv /var/www/html/
```

where:

* `-R` = recursive
* `-v` = verbose

---

# 13. `ausearch`

`ausearch` searches the audit log.

To find AVC events:

```bash
ausearch -m AVC
```

To find recent AVC events:

```bash
ausearch -m AVC -ts recent
```

This is very useful when you suspect SELinux is causing a problem.

Typical troubleshooting flow:

```text
Application fails
       ↓
Check audit log
       ↓
ausearch -m AVC -ts recent
       ↓
Find AVC denial
       ↓
Analyze with sealert
       ↓
Determine root cause
       ↓
Fix context / Boolean / policy
```

---

# 14. Don't blindly use `audit2allow`

This is a **very important exam/interview point**.

You might see `sealert` recommend:

```bash
ausearch -c 'httpd' --raw | audit2allow -M my-httpd
```

and:

```bash
semodule -X 300 -i my-httpd.pp
```

But **do not automatically do this**.

Why?

Because `audit2allow` creates a policy allowing the denied action.

If the actual problem is:

```text
Wrong file context
```

then creating a new policy is the wrong solution.

### Correct approach

If:

```text
File is in wrong location
```

→ Move it to the correct location.

If:

```text
File has wrong SELinux context
```

→ Use `restorecon`.

If:

```text
Optional SELinux behavior is disabled
```

→ Check the appropriate Boolean.

If:

```text
Application genuinely requires a new SELinux permission
```

→ Investigate and potentially create a custom policy.

---

# 15. SELinux Booleans

Some SELinux policies have optional features controlled by **Booleans**.

Check Booleans:

```bash
getsebool -a
```

Set a Boolean temporarily:

```bash
setsebool boolean_name on
```

Make the change persistent:

```bash
setsebool -P boolean_name on
```

For example, some services have policies that allow optional behaviors through Booleans.

The important idea is:

```text
SELinux Boolean
      ↓
Optional policy behavior
```

Before changing a Boolean, check the relevant SELinux documentation/man page.

---

# 16. SELinux does NOT replace Linux permissions

This is another important concept.

SELinux is an **additional security layer**.

Access must satisfy both:

```text
Linux permissions / ACL
          AND
SELinux policy
```

For example:

```text
Linux permissions = ALLOW
SELinux = DENY
------------------
Final = DENY
```

Or:

```text
Linux permissions = DENY
SELinux = ALLOW
------------------
Final = DENY
```

Therefore:

> **SELinux cannot override normal Linux file permissions.**

---

# 17. Web Console troubleshooting

RHEL's web console also provides SELinux troubleshooting.

Navigate to:

```text
SELinux
```

The interface shows:

* Current SELinux state
* SELinux access-control errors
* Event details
* Suggested solutions

You can expand an event and select:

```text
Solution details
```

to see the explanation and recommended action.

In appropriate cases, you can use:

```text
Apply this solution
```

After fixing the issue, the alert should disappear.

---

# 18. Most common SELinux problem

One of the most common problems is:

> **Incorrect SELinux context on a new, copied, or moved file.**

For example:

```bash
touch /root/mypage
mv /root/mypage /var/www/html/
```

The file came from `/root`, so it may retain a context associated with its original location.

Check:

```bash
ls -Z /var/www/html/mypage
```

Then fix:

```bash
restorecon -v /var/www/html/mypage
```

This is why **copying/moving files can sometimes cause SELinux problems**.

---

# 19. Simple troubleshooting decision tree

When an application fails and you suspect SELinux:

```text
Application fails
       |
       v
Check whether SELinux is involved
       |
       v
ausearch -m AVC -ts recent
       |
       +---- No AVC?
       |       |
       |       └── Investigate other causes
       |
       v
AVC found
       |
       v
sealert -l <UUID>
       |
       v
Understand:
Process?
File?
Action?
Contexts?
       |
       v
Is the file in the correct location?
       |
    +--+--+
    |     |
   No    Yes
    |     |
 Move    Check context
 file      |
            v
       restorecon
            |
            v
      Test application
```

---

# 20. Commands you should memorize

### Check file context

```bash
ls -Z filename
```

### Check process context

```bash
ps -Z
```

### Search recent AVC denials

```bash
ausearch -m AVC -ts recent
```

### Analyze one SELinux alert

```bash
sealert -l UUID
```

### Analyze all audit events

```bash
sealert -a /var/log/audit/audit.log
```

### Restore file context

```bash
restorecon -v filename
```

### Restore recursively

```bash
restorecon -Rv directory/
```

### View Booleans

```bash
getsebool -a
```

### Enable Boolean persistently

```bash
setsebool -P boolean_name on
```

---

# 21. Interview answer: "How do you troubleshoot an SELinux issue?"

A strong answer is:

> **First, I check whether SELinux is enforcing and look for AVC denials using `ausearch -m AVC -ts recent`. I then use `sealert -l <UUID>` to understand the denied process, resource, action, and SELinux contexts. I check the file and process labels using `ls -Z` and `ps -Z`. If the issue is an incorrect file context, I use `restorecon` rather than creating a custom policy. If the service requires an optional SELinux feature, I check the relevant Boolean. I use `audit2allow` only after confirming that the denied action is legitimate and that a labeling or configuration issue is not the real cause.**

### The key principle

```text
DON'T:
AVC denial
   ↓
audit2allow immediately ❌

DO:
AVC denial
   ↓
Understand the denial
   ↓
Check location + context + permissions + Boolean
   ↓
Fix the actual root cause
```

**In one sentence:**

> **SELinux troubleshooting is mainly about identifying what was denied, why it was denied, and correcting the underlying context/configuration rather than simply disabling SELinux or allowing every denial.**
