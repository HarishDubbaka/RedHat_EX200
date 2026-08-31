# Controlling SELinux File Contexts — Explained Clearly

## 1. What is an SELinux file context?

Every important SELinux resource gets a **security context**, including:

* Files
* Directories
* Processes
* Ports

For files, you can see the context with:

```bash
ls -Z filename
```

Example:

```bash
ls -Z /var/www/html/index.html
```

You may see:

```text
system_u:object_r:httpd_sys_content_t:s0
```

Think of this as a **security label** attached to the file.

The most important part for normal administration is usually the **type**:

```text
httpd_sys_content_t
```

This tells SELinux what kind of content the file is considered to be.



---

# 2. Understand this context

Consider:

```text
system_u:object_r:httpd_sys_content_t:s0
```

It has four parts:

```text
system_u : object_r : httpd_sys_content_t : s0
   │           │              │               │
   │           │              │               └─ Sensitivity level
   │           │              └──────────────── Type
   │           └────────────────────────────── Role
   └────────────────────────────────────────── SELinux user
```

For most RHCSA work, **focus mainly on the type**.

For example:

```text
httpd_sys_content_t
```

means the file is labeled as content that Apache/httpd is allowed to serve under the relevant SELinux policy.

---

# 3. How does a new file get its SELinux context?

When you create a file, SELinux needs to decide what label it should have.

There are two important cases.

### Case 1: There is a policy for the location

For example:

```bash
touch /var/www/html/index.html
```

`/var/www/html` has an SELinux policy associated with web content.

Therefore the new file gets an appropriate context.

### Case 2: There is no policy

Suppose you create:

```bash
mkdir /virtual
touch /virtual/index.html
```

If `/virtual` has no specific SELinux policy, the file can inherit the context of its parent directory.

The file in the lesson therefore initially gets:

```text
default_t
```



---

# 4. `ls -Z` — check the context

This is the first command you should remember:

```bash
ls -Z filename
```

For directories:

```bash
ls -Zd directory
```

Example:

```bash
ls -Zd /var/www/html/
```

Output:

```text
system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
```

For multiple files:

```bash
ls -Z /var/www/html/*
```



### RHCSA memory trick

> **`-Z` = show SELinux security context**

---

# 5. Very important: `cp` vs `mv`

This is one of the most important concepts in the lesson.

Suppose:

```text
/tmp/file1
```

has:

```text
user_tmp_t
```

and:

```text
/var/www/html/
```

expects:

```text
httpd_sys_content_t
```

Now compare:

```bash
mv /tmp/file1 /var/www/html/
```

and:

```bash
cp /tmp/file2 /var/www/html/
```

They behave differently.

## `mv`

When moving within the same filesystem, `mv` normally does **not create a new inode**. It moves the existing file/inode.

Therefore the existing SELinux context is normally retained.

So:

```text
/tmp/file1
```

has:

```text
user_tmp_t
```

After:

```bash
mv /tmp/file1 /var/www/html/
```

it can still have:

```text
user_tmp_t
```

## `cp`

`cp` creates a **new file/inode**.

Therefore the new file receives the SELinux context appropriate for the destination.

So:

```bash
cp /tmp/file2 /var/www/html/
```

results in:

```text
httpd_sys_content_t
```



The lesson demonstrates exactly this behavior. 

### Easy way to remember

```text
mv → usually keeps old context
cp → new file → gets destination context
```

---

# 6. How can you preserve context when copying?

Normally:

```bash
cp source destination
```

creates a new file and its context can be determined by the destination.

But you can preserve the original SELinux context with:

```bash
cp --preserve=context source destination
```

You can also use:

```bash
cp -p
```

which preserves file attributes where possible, including the context.



---

# 7. Three commands you MUST know

The lesson introduces three important commands:

```text
semanage fcontext
restorecon
chcon
```



The difference is extremely important.

| Command             | Purpose                 | Recommended?         |
| ------------------- | ----------------------- | -------------------- |
| `ls -Z`             | View context            | ✅                    |
| `chcon`             | Directly change context | ⚠️ Temporary/testing |
| `semanage fcontext` | Create SELinux policy   | ✅                    |
| `restorecon`        | Apply policy to files   | ✅                    |

---

# 8. `chcon` — manually change the context

Example:

```bash
chcon -t httpd_sys_content_t /virtual
```

This directly changes the context.

Before:

```text
default_t
```

After:

```text
httpd_sys_content_t
```

The lesson demonstrates this behavior. 

You can verify:

```bash
ls -Zd /virtual
```

---

# 9. Why is `chcon` NOT the recommended permanent solution?

This is very important.

`chcon` changes the file's context **directly**.

It does **not** create a persistent SELinux file-context policy.

For example:

```bash
chcon -t httpd_sys_content_t /virtual
```

Then later:

```bash
restorecon -v /virtual
```

SELinux looks at its policy and says:

> "According to my policy, `/virtual` should have `default_t`."

So it changes it back:

```text
httpd_sys_content_t
        ↓
default_t
```

The lesson explicitly demonstrates this. 

### Therefore

```text
chcon = manually change now
```

but

```text
semanage + restorecon = policy-based permanent solution
```

---

# 10. `semanage fcontext` — the recommended method

Suppose you create:

```bash
mkdir /virtual
```

and want Apache to access content there.

You could do:

```bash
chcon -t httpd_sys_content_t /virtual
```

But that's not the recommended approach.

Instead create a file-context policy:

```bash
semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
```

Then apply the policy:

```bash
restorecon -Rv /virtual
```

This is the **correct administrative approach**.



---

# 11. What does `(/.*)?` mean?

This is probably the most confusing part for beginners.

You will frequently see:

```bash
'/virtual(/.*)?'
```

Break it down:

```text
/virtual
```

means the directory itself.

```text
(/.*)?
```

means:

> optionally match `/` followed by anything underneath it.

So:

```text
/virtual(/.*)?
```

matches:

```text
/virtual
/virtual/file1
/virtual/file2
/virtual/index.html
/virtual/subdir
/virtual/subdir/file.txt
```

Therefore:

```bash
semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
```

means:

> Apply `httpd_sys_content_t` to `/virtual` and everything underneath it.



### Easy memory trick

```text
/path(/.*)?
```

= **directory + everything below it**

---

# 12. `semanage fcontext -l`

To list SELinux file-context policies:

```bash
semanage fcontext -l
```

Example:

```text
/var/www(/.*)?    all files    system_u:object_r:httpd_sys_content_t:s0
```

This tells us:

```text
/var/www/
    ↓
and everything underneath it
    ↓
httpd_sys_content_t
```



---

# 13. Add a new policy

The general syntax is:

```bash
semanage fcontext -a -t TYPE 'PATH'
```

Example:

```bash
semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
```

Breakdown:

```text
semanage fcontext
        │
        └── manage file-context policy

-a
│
└── add

-t
│
└── specify SELinux type

httpd_sys_content_t
│
└── desired SELinux type

'/virtual(/.*)?'
│
└── path pattern
```



---

# 14. `restorecon` — apply the policy

After creating the policy:

```bash
semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
```

you still need to apply it to existing files:

```bash
restorecon -Rv /virtual
```

Think of it this way:

```text
semanage
   ↓
Create the rule

restorecon
   ↓
Apply the rule to the actual files
```

The lesson uses:

```bash
restorecon -RFvv /virtual
```

and both `/virtual` and `/virtual/index.html` are relabeled. 

---

# 15. Why do we need both commands?

This is a common interview/RHCSA question.

Suppose:

```bash
mkdir /virtual
```

Current:

```text
default_t
```

You run:

```bash
semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
```

Now the **policy knows**:

```text
/virtual → httpd_sys_content_t
```

But the existing directory may still physically have:

```text
default_t
```

So run:

```bash
restorecon -Rv /virtual
```

Now the actual filesystem label becomes:

```text
httpd_sys_content_t
```

### In one sentence:

> `semanage` defines the rule; `restorecon` applies the rule.

---

# 16. `restorecon -R`

If you want to recursively restore contexts:

```bash
restorecon -Rv /var/www/
```

Where:

```text
-R = recursive
-v = verbose
```

So:

```bash
restorecon -Rv /var/www/
```

means:

> Recursively restore SELinux contexts and show what was changed.

The lesson demonstrates this when fixing `/var/www/html/file1`. 

---

# 17. `semanage fcontext -d`

To delete a custom file-context policy:

```bash
semanage fcontext -d 'PATH'
```

For example:

```bash
semanage fcontext -d '/virtual(/.*)?'
```

The basic options are:

```text
-a → add
-d → delete
-l → list
```



---

# 18. `semanage fcontext -l -C`

This is another useful command:

```bash
semanage fcontext -l -C
```

`-C` shows **local customizations** to the default SELinux policy.

For example:

```text
/virtual(/.*)?    all files    system_u:object_r:httpd_sys_content_t:s0
```

This tells you that `/virtual` has a locally configured custom policy. 

---

# 19. Complete practical example

Let's say you have a website stored in:

```text
/virtual
```

### Step 1 — Create directory

```bash
mkdir /virtual
```

### Step 2 — Create web file

```bash
touch /virtual/index.html
```

### Step 3 — Check contexts

```bash
ls -Zd /virtual
ls -Z /virtual
```

You may initially see:

```text
default_t
```



### Step 4 — Create SELinux policy

```bash
semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
```

### Step 5 — Apply policy

```bash
restorecon -Rv /virtual
```

### Step 6 — Verify

```bash
ls -Zd /virtual
ls -Z /virtual
```

You should now see:

```text
httpd_sys_content_t
```

This is the exact workflow demonstrated in the lesson. 

---

# 20. The most important difference

Remember this table for RHCSA:

| Situation                           | Command                   |
| ----------------------------------- | ------------------------- |
| Check SELinux context               | `ls -Z`                   |
| Check directory context             | `ls -Zd`                  |
| Temporarily/directly change context | `chcon`                   |
| Create permanent file-context rule  | `semanage fcontext`       |
| Apply SELinux policy                | `restorecon`              |
| List file-context rules             | `semanage fcontext -l`    |
| Show custom rules                   | `semanage fcontext -l -C` |
| Add rule                            | `semanage fcontext -a`    |
| Delete rule                         | `semanage fcontext -d`    |

---

# 21. The RHCSA troubleshooting flow

If an application cannot access a file because of SELinux, use this approach:

```text
                Application can't access file
                           │
                           ▼
                     Check context
                           │
                           ▼
                      ls -Z file
                           │
                           ▼
                  Is context correct?
                    /              \
                  YES              NO
                   │                │
                   │                ▼
                   │       Is this a permanent
                   │             change?
                   │          /          \
                   │        YES          NO
                   │         │             │
                   │         ▼             ▼
                   │     semanage         chcon
                   │     fcontext
                   │         │
                   │         ▼
                   │     restorecon
                   │         │
                   └─────────┘
```

### The key principle

**Don't normally use `chcon` for permanent configuration.**

Use:

```bash
semanage fcontext
```

to define the policy, then:

```bash
restorecon
```

to apply it.

The lesson explicitly identifies `semanage fcontext` + `restorecon` as the recommended approach and `chcon` as useful mainly for testing/debugging. 

---

## ⭐ Memorize these 5 commands

If you're preparing for **RHCSA/EX200**, make these automatic:

```bash
ls -Z
```

**Check context**

```bash
chcon -t TYPE FILE
```

**Direct/manual change**

```bash
semanage fcontext -a -t TYPE 'PATH(/.*)?'
```

**Create persistent policy**

```bash
restorecon -Rv PATH
```

**Apply policy**

```bash
semanage fcontext -l -C
```

**Find your custom SELinux rules**

### One-line memory trick

> **`ls -Z` → see it | `chcon` → change it manually | `semanage` → define the rule | `restorecon` → apply the rule.**
