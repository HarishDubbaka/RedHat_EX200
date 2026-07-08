# 📦 Day 23 – Install and Remove Packages with Dependencies (DNF)

Think of **DNF** like the **Play Store/App Store** for RHEL.

* 📥 Install software → `dnf install`
* 🗑️ Remove software → `dnf remove`
* 🔍 Search packages → `dnf search`
* ℹ️ View package details → `dnf info`
* 📦 Install software groups → `dnf group install`
* 📜 View installation history → `dnf history`

---

# Step 1: Connect to the Server

```bash
ssh student@servera
sudo -i
```

---

# Step 2: Check if Package Exists

Try running the command.

```bash
nmap
```

Output:

```bash
-bash: nmap: command not found
```

➡️ Package is not installed.

---

# Step 3: Search for the Package

```bash
dnf search nmap
```

Example:

```text
nmap.x86_64 : Network exploration tool
nmap-ncat.x86_64 : Netcat replacement
```

---

# Step 4: View Package Information

```bash
dnf info nmap
```

Shows:

* Version
* Size
* Repository
* Description

---

# Step 5: Install the Package

```bash
dnf install nmap
```

Press:

```text
y
```

DNF automatically installs:

* nmap
* Required dependencies

---

# Step 6: Verify Installation

```bash
nmap -V
```

Example:

```text
Nmap version 7.92
```

✅ Installation successful.

---

# Step 7: Remove the Package

```bash
dnf remove nmap
```

Press:

```text
y
```

DNF removes:

* nmap
* Unneeded dependencies (when applicable)

---

# Working with Package Groups

Instead of installing one package at a time, DNF can install a **group** of related packages.

Example:

**Security Tools**

Contains:

* openscap
* scap-security-guide
* xmlsec
* and other security utilities

---

## List Available Groups

```bash
dnf group list
```

Example:

```text
Security Tools
System Tools
Scientific Support
RPM Development Tools
```

---

## View Group Information

```bash
dnf group info "Security Tools"
```

Displays:

* Description
* Default packages
* Optional packages

---

## Install the Group

```bash
dnf group install "Security Tools"
```

Press:

```text
y
```

DNF installs all required packages in the group.

---

## Verify Group Installation

```bash
dnf group list --installed
```

Output:

```text
Installed Groups:
Security Tools
```

---

## Remove the Group

```bash
dnf group remove "Security Tools"
```

Press:

```text
y
```

All packages installed as part of the group are removed.

---

# DNF History

DNF records every transaction.

View history:

```bash
dnf history
```

Example:

```text
ID  Command
9   group remove Security Tools
8   group install Security Tools
7   remove nmap
6   install nmap
```

---

## View Details of a Transaction

```bash
dnf history info 9
```

Displays:

* User
* Date & Time
* Command executed
* Packages installed or removed
* Transaction status

---

# Common DNF Commands

| Command                       | Purpose                             |
| ----------------------------- | ----------------------------------- |
| `dnf search <package>`        | Search for packages                 |
| `dnf info <package>`          | Show package details                |
| `dnf install <package>`       | Install a package with dependencies |
| `dnf remove <package>`        | Remove a package                    |
| `dnf group list`              | List available package groups       |
| `dnf group info "<group>"`    | Show group details                  |
| `dnf group install "<group>"` | Install a package group             |
| `dnf group remove "<group>"`  | Remove a package group              |
| `dnf history`                 | Show transaction history            |
| `dnf history info <ID>`       | View details of a transaction       |

---

# Interview Tips

**Q1. What happens when you install a package using DNF?**
➡️ DNF automatically resolves and installs all required dependencies.

**Q2. How do you remove a package?**

```bash
dnf remove <package>
```

**Q3. How do you search for a package?**

```bash
dnf search <package>
```

**Q4. How do you check package details before installing?**

```bash
dnf info <package>
```

**Q5. What is a package group?**
➡️ A collection of related packages that can be installed or removed together.

**Q6. How do you view DNF transaction history?**

```bash
dnf history
```

---

# 🎯 Easy Memory Trick

🛒 **Think of DNF like an App Store:**

* 🔍 **Search** → Find the app (`dnf search`)
* ℹ️ **Info** → View app details (`dnf info`)
* 📥 **Install** → Download the app (`dnf install`)
* 🗑️ **Remove** → Uninstall the app (`dnf remove`)
* 📦 **Group Install** → Install a bundle of related apps (`dnf group install`)
* 📜 **History** → See everything you've installed or removed (`dnf history`)
