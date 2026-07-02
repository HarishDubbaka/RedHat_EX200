# Lab Summary: Managing Group Directory Permissions in Linux

## Objective

Create a shared directory for the **consultants** group so that:

* Group members can create and delete files.
* Other users cannot access the directory.

---

## Step 1: Connect to `servera` and Become Root

```bash
ssh student@servera
```

Switch to the root user:

```bash
sudo -i
```

Password:

```
student
```

---

## Step 2: Create the Shared Directory

```bash
mkdir /home/consultants
```

---

## Step 3: Change Group Ownership

Assign the directory to the **consultants** group.

```bash
chown :consultants /home/consultants
```

Verify:

```bash
ls -ld /home/consultants
```

Output:

```text
drwxr-xr-x. 2 root consultants ... /home/consultants
```

---

## Step 4: Give the Group Write Permission

Initially, the group has only:

```
r-x
```

Add write permission using the **symbolic method**:

```bash
chmod g+w /home/consultants
```

Verify:

```bash
ls -ld /home/consultants
```

Output:

```text
drwxrwxr-x.
```

### Meaning

| Permission | Description              |
| ---------- | ------------------------ |
| r          | Read directory contents  |
| w          | Create/Delete files      |
| x          | Enter (cd) the directory |

Now the group has:

```
rwx
```

---

## Step 5: Remove Access for Others

Use the **octal method**:

```bash
chmod 770 /home/consultants
```

Verify:

```bash
ls -ld /home/consultants
```

Output:

```text
drwxrwx---.
```

### Permission Breakdown

| Owner | Group | Others |
| ----- | ----- | ------ |
| rwx   | rwx   | ---    |

**770 means:**

* **7** = rwx (Owner)
* **7** = rwx (Group)
* **0** = --- (Others)

---

## Step 6: Test as `consultant1`

Exit root:

```bash
exit
```

Switch user:

```bash
su - consultant1
```

Password:

```
redhat
```

Go to the directory:

```bash
cd /home/consultants
```

Create a file:

```bash
touch consultant1.txt
```

Verify:

```bash
ls
```

Output:

```text
consultant1.txt
```

---

## Step 7: Test as `consultant2`

Exit:

```bash
exit
```

Switch user:

```bash
su - consultant2
```

Password:

```
redhat
```

Go to the directory:

```bash
cd /home/consultants
```

Create another file:

```bash
touch consultant2.txt
```

Verify:

```bash
ls
```

Output:

```text
consultant1.txt  consultant2.txt
```

Both users can create files because both belong to the **consultants** group.

---

## Step 8: Exit

Return to the workstation:

```bash
exit
exit
```

---

# Permission Commands Used

| Command                                | Purpose                                        |
| -------------------------------------- | ---------------------------------------------- |
| `mkdir /home/consultants`              | Create the directory                           |
| `chown :consultants /home/consultants` | Change group ownership                         |
| `chmod g+w /home/consultants`          | Give write permission to the group             |
| `chmod 770 /home/consultants`          | Allow owner and group full access; deny others |
| `touch consultant1.txt`                | Create a file as `consultant1`                 |
| `touch consultant2.txt`                | Create a file as `consultant2`                 |

---

# Key Concepts

* **Group ownership (`chown :group`)** allows a group to share access to files/directories.
* **`chmod g+w`** (symbolic mode) adds write permission to the group without changing other permissions.
* **`chmod 770`** (octal mode) grants full access to the owner and group while denying all access to others.
* **Directory permissions** differ slightly from file permissions:

  * **r** = list directory contents.
  * **w** = create, delete, or rename files.
  * **x** = enter (`cd`) the directory and access its contents.
