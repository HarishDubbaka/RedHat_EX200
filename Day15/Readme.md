# Managing Local Group Accounts 

## Objectives

* Create local groups.
* Modify existing groups.
* Delete groups.
* Manage primary and supplementary group memberships.
* Temporarily change the primary group.

---

# 1. Creating Groups

## Create a Regular Group

```bash
groupadd group01
```

Verify:

```bash
grep group01 /etc/group
```

Example:

```bash
group01:x:10000:
```

---

## Create a Group with a Specific GID

```bash
groupadd -g 10000 group01
```

Verify:

```bash
tail /etc/group
```

Output:

```bash
group01:x:10000:
```

---

## Create a System Group

System groups use GIDs from the system GID range defined in:

```bash
/etc/login.defs
```

Create:

```bash
groupadd -r group02
```

Verify:

```bash
tail /etc/group
```

Example:

```bash
group02:x:988:
```

---

# 2. Modifying Groups

## Rename a Group

Syntax:

```bash
groupmod -n NEW_GROUP OLD_GROUP
```

Example:

```bash
groupmod -n group0022 group02
```

Verify:

```bash
grep group0022 /etc/group
```

Output:

```bash
group0022:x:988:
```

---

## Change Group GID

Syntax:

```bash
groupmod -g NEW_GID GROUP
```

Example:

```bash
groupmod -g 20000 group0022
```

Verify:

```bash
grep group0022 /etc/group
```

Output:

```bash
group0022:x:20000:
```

---

# 3. Deleting Groups

Syntax:

```bash
groupdel GROUPNAME
```

Example:

```bash
groupdel group0022
```

---

## Important Note

You cannot delete a group if it is the primary group of an existing user.

Check users using a group:

```bash
grep group01 /etc/group
```

or

```bash
getent group group01
```

---

# 4. Primary and Supplementary Groups

## Primary Group

A user can have only **one** primary group.

Check:

```bash
id user02
```

Example:

```bash
uid=1006(user02) gid=1008(user02) groups=1008(user02)
```

Primary group:

```bash
gid=1008(user02)
```

---

## Supplementary Groups

A user can belong to multiple supplementary groups.

Example:

```bash
uid=1007(user03) gid=1009(user03) groups=1009(user03),10000(group01)
```

Supplementary group:

```bash
group01
```

---

# 5. Change User's Primary Group

Syntax:

```bash
usermod -g GROUP USER
```

Example:

```bash
usermod -g group01 user02
```

Before:

```bash
id user02
```

Output:

```bash
uid=1006(user02) gid=1008(user02) groups=1008(user02)
```

After:

```bash
id user02
```

Output:

```bash
uid=1006(user02) gid=10000(group01) groups=10000(group01)
```

---

## Important

Changing the primary group:

```bash
usermod -g
```

does **not**:

* Add the old primary group as a supplementary group.
* Change ownership of existing files.

---

# 6. Add Users to Supplementary Groups

## Method 1: Using groupmod

Syntax:

```bash
groupmod -aU USER GROUP
```

Example:

```bash
groupmod -aU user03 group01
```

Verify:

```bash
id user03
```

Output:

```bash
uid=1007(user03) gid=1009(user03) groups=1009(user03),10000(group01)
```

---

## Warning

Always use:

```bash
-a
```

with:

```bash
-U
```

Without `-a`, existing members of the group are replaced.

Correct:

```bash
groupmod -aU user03 group01
```

Incorrect:

```bash
groupmod -U user03 group01
```

---

## Method 2: Using usermod

Syntax:

```bash
usermod -aG GROUP USER
```

Example:

```bash
usermod -aG group01 user03
```

Multiple groups:

```bash
usermod -aG group01,group02,group03 user03
```

Verify:

```bash
id user03
```

---

# 7. Temporarily Change Primary Group

The current primary group determines the group ownership of newly created files.

Use:

```bash
newgrp GROUP
```

Example:

Current membership:

```bash
id
```

Output:

```bash
uid=1007(user03) gid=1009(user03) groups=1009(user03),10000(group01)
```

Switch primary group:

```bash
newgrp group01
```

Verify:

```bash
id
```

Output:

```bash
uid=1007(user03) gid=10000(group01) groups=1009(user03),10000(group01)
```

---

## Exit Temporary Group

```bash
exit
```

or log out and log back in.

The user's original primary group is restored.

---

# Useful Files

| File              | Purpose                         |
| ----------------- | ------------------------------- |
| `/etc/group`      | Stores group information        |
| `/etc/passwd`     | Stores user account information |
| `/etc/login.defs` | Default UID/GID settings        |

---

# RHCSA Exam Commands to Remember

```bash
groupadd group01
groupadd -g 10000 group01
groupadd -r group02

groupmod -n newgroup oldgroup
groupmod -g 20000 group01

groupdel group01

usermod -g group01 user01
usermod -aG group01 user01

groupmod -aU user01 group01

id user01
groups user01

newgrp group01
```

## Exam Tip

For RHCSA, remember the difference:

```bash
usermod -g
```

➡ Changes **primary group**

```bash
usermod -aG
```

➡ Adds **supplementary group**

```bash
newgrp
```

➡ Temporarily changes the current shell's primary group.

