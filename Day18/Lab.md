# Lab: Control Access to Files (Complete Explanation)

This lab demonstrates how to create a **secure collaborative directory** where multiple users can work together while protecting each other's files.

---

# Lab Objective

Create a collaborative directory `/home/techdocs` with these requirements:

* ✅ Only members of the **techdocs** group can access it.
* ✅ Every new file automatically belongs to the **techdocs** group.
* ✅ Users can edit each other's files.
* ✅ Users cannot delete another user's files.
* ✅ Other users have no access.
* ✅ `dev1` and `dev2` always create files with group write permission.

---

# Step 1: Start the Lab

```bash
lab start perms-review
```

Creates

```
dev1
dev2
dbadmin1
```

Group membership

| User     | Group           |
| -------- | --------------- |
| dev1     | techdocs        |
| dev2     | techdocs        |
| dbadmin1 | Not in techdocs |

---

# Step 2: Login

```bash
ssh student@serverb
```

Become root

```bash
sudo -i
```

---

# Step 3: Create Collaborative Directory

```bash
mkdir /home/techdocs
```

---

# Step 4: Change Group Ownership

```bash
chown :techdocs /home/techdocs
```

Verify

```bash
ls -ld /home/techdocs
```

Group should be

```
techdocs
```

---

# Step 5: Set Permissions

```bash
chmod 770 /home/techdocs
```

Permission

```
770

Owner  -> rwx
Group  -> rwx
Others -> ---
```

Meaning

| User           | Access    |
| -------------- | --------- |
| Owner          | Full      |
| techdocs group | Full      |
| Others         | No access |

---

# Step 6: Enable SGID

```bash
chmod g+s /home/techdocs
```

Verify

```bash
ls -ld /home/techdocs
```

Output

```
drwxrws---
```

Notice

```
s
```

instead of

```
x
```

## Why SGID?

Without SGID

```
dev1 creates file

↓

Group = dev1
```

With SGID

```
dev1 creates file

↓

Group = techdocs
```

Every new file automatically belongs to the **techdocs** group.

---

# Step 7: Enable Sticky Bit

```bash
chmod o+t /home/techdocs
```

Verify

```bash
ls -ld /home/techdocs
```

Output

```
drwxrws--T
```

Notice

```
T
```

## Why Sticky Bit?

Without Sticky Bit

```
Anyone with write permission

↓

Can delete everyone's files
```

With Sticky Bit

Only

* File owner
* Directory owner
* root

can delete files.

---

# Step 8: Configure dev1

Become dev1

```bash
su - dev1
```

Set umask

```bash
umask 007
```

Persist it

```bash
echo "umask 007" >> ~/.bashrc
```

## Why 007?

Default permissions

Directories

```
777
```

Subtract

```
007
```

Result

```
770
```

Files

```
666
```

Subtract

```
007
```

Result

```
660
```

Meaning

```
Owner

rw

Group

rw

Others

---
```

Perfect for collaboration.

---

# Step 9: Create dev1 Directory

```bash
cd /home/techdocs
```

Create folder

```bash
mkdir dev1
```

Create files

```bash
touch dev1/dev1{.txt,.log,.cfg}
```

Verify

```bash
ls -l dev1
```

Output

```
-rw-rw---- dev1 techdocs
```

Notice

Group is

```
techdocs
```

because of SGID.

Permissions

```
660
```

because of umask 007.

---

# Step 10: Configure dev2

Exit

```bash
exit
```

Switch

```bash
su - dev2
```

Set umask

```bash
umask 007
```

Persist

```bash
echo "umask 007" >> ~/.bashrc
```

Create directory

```bash
cd /home/techdocs

mkdir dev2
```

Create files

```bash
touch dev2/dev2{.txt,.log,.cfg}
```

Verify

```bash
ls -l dev2
```

Output

```
-rw-rw---- dev2 techdocs
```

---

# Step 11: Collaboration Test

As dev2

Append text

```bash
echo "Logging started" >> dev1/dev1.log
```

Display

```bash
cat dev1/dev1.log
```

Output

```
Logging started
```

## Why can dev2 edit it?

Because

```
Group = techdocs

Permission = rw
```

dev2 belongs to

```
techdocs
```

So writing is allowed.

---

# Step 12: Delete Test

Still as dev2

Try

```bash
rm -rf dev1
```

Output

```
Operation not permitted
```

### Why?

Sticky Bit

Even though dev2 has write permission on the directory, only the owner of `dev1` (or the directory owner/root) can delete it.

---

# Step 13: Unauthorized User Test

Exit

```bash
exit
```

Become

```bash
su - dbadmin1
```

Try

```bash
cd /home/techdocs
```

Output

```
Permission denied
```

### Why?

Directory permission

```
770
```

Others have

```
---
```

`dbadmin1` is not in the `techdocs` group.

---

# Final Permission Check

```bash
ls -ld /home/techdocs
```

Output

```
drwxrws--T
```

Breakdown

| Character | Meaning                                              |
| --------- | ---------------------------------------------------- |
| d         | Directory                                            |
| rwx       | Owner permissions                                    |
| rws       | Group permissions + SGID                             |
| --T       | Sticky bit enabled, no execute permission for others |

---

# Complete Command Summary

```bash
# Become root
sudo -i

# Create directory
mkdir /home/techdocs

# Change group
chown :techdocs /home/techdocs

# Permissions
chmod 770 /home/techdocs

# Enable SGID
chmod g+s /home/techdocs

# Enable Sticky Bit
chmod o+t /home/techdocs

# Configure dev1
su - dev1
umask 007
echo "umask 007" >> ~/.bashrc

cd /home/techdocs
mkdir dev1
touch dev1/dev1{.txt,.log,.cfg}
exit

# Configure dev2
su - dev2
umask 007
echo "umask 007" >> ~/.bashrc

cd /home/techdocs
mkdir dev2
touch dev2/dev2{.txt,.log,.cfg}

echo "Logging started" >> dev1/dev1.log

rm -rf dev1     # Fails due to Sticky Bit

exit

# Unauthorized user test
su - dbadmin1
cd /home/techdocs   # Permission denied
exit
```

## Key Concepts for Exams

| Feature                  | Command                         | Purpose                                                             |
| ------------------------ | ------------------------------- | ------------------------------------------------------------------- |
| **Group ownership**      | `chown :techdocs dir`           | Assigns the collaborative group                                     |
| **Standard permissions** | `chmod 770 dir`                 | Only owner and group can access                                     |
| **SGID**                 | `chmod g+s dir`                 | New files inherit the directory's group                             |
| **Sticky Bit**           | `chmod o+t dir`                 | Only file owners (or root/directory owner) can delete files         |
| **umask 007**            | `umask 007`                     | New files: `660`, new directories: `770`; no permissions for others |
| **Persistent umask**     | `echo "umask 007" >> ~/.bashrc` | Applies the umask automatically in future Bash sessions             |
