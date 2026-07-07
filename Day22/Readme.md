# Installing, Updating, and Removing Software with DNF 

Think of **DNF** as the **App Store or Play Store for Red Hat Enterprise Linux (RHEL)**.

* Want to install software? → Use **dnf install**
* Want to update software? → Use **dnf upgrade**
* Want to remove software? → Use **dnf remove**
* Want to search for software? → Use **dnf search**

DNF automatically downloads software from **repositories**, installs it, updates it, and resolves all required dependencies.

---

# What is DNF?

**DNF (Dandified YUM)** is the package management tool in **RHEL 8, RHEL 9, and RHEL 10**.

It manages software packages in the **RPM (Red Hat Package Manager)** format.

Think of DNF like this:

| Windows         | Linux (RHEL) |
| --------------- | ------------ |
| Microsoft Store | DNF          |
| Setup.exe       | RPM Package  |
| Windows Update  | dnf upgrade  |

---

# What is a Package?

A **package** is a compressed file that contains everything needed to install software.

A package contains:

* Executable program
* Libraries
* Configuration files
* Documentation
* Installation scripts

Example:

```text
httpd-2.4.63-1.el10.x86_64.rpm
```

Let's understand every part.

| Part   | Meaning           |
| ------ | ----------------- |
| httpd  | Package name      |
| 2.4.63 | Software version  |
| 1      | Package release   |
| el10   | Built for RHEL 10 |
| x86_64 | 64-bit processor  |
| .rpm   | RPM package       |

---

# Installing Software

Command:

```bash
sudo dnf install httpd
```

Here,

* **sudo** → Run as Administrator (root)
* **dnf** → Package manager
* **install** → Install software
* **httpd** → Apache Web Server package

---

# What Happens Internally?

When you press **Enter**, DNF performs several steps.

```
User
   │
   ▼
DNF
   │
   ▼
Repository
   │
   ▼
Dependency Check
   │
   ▼
Download Packages
   │
   ▼
Install Packages
   │
   ▼
Complete
```

Let's understand each step.

---

# Step 1: Repository Search

DNF first checks all configured repositories.

A repository is simply a **software warehouse**.

Imagine Amazon.

You search for a laptop.

Amazon finds it.

Similarly,

```
DNF
     │
Search Repositories
     │
     ▼
BaseOS
AppStream
EPEL
```

DNF searches these repositories for **httpd**.

---

# Step 2: Dependency Resolution

Suppose you buy a printer.

The printer alone is useless.

You also need

* USB cable
* Driver
* Ink cartridge

These are dependencies.

Similarly,

Apache (httpd) requires other packages.

Example:

```
httpd
│
├── apr
├── apr-util
├── httpd-core
├── httpd-tools
```

DNF automatically installs them.

That is why your output shows

```
Installing dependencies:
apr
apr-util
httpd-core
httpd-tools
```

Without these,

Apache cannot run.

---

# Required Dependencies

Your output

```
Installing dependencies:
```

means

These packages are **mandatory**.

Example

```
apr
```

APR means

**Apache Portable Runtime**

Apache uses this library internally.

Without APR,

Apache cannot even start.

---

# Weak Dependencies

Your output also shows

```
Installing weak dependencies:
```

Example

```
mod_http2

mod_lua
```

These are **recommended**, but not mandatory.

Difference:

```
Required Dependency
        │
Apache cannot work
```

```
Weak Dependency
        │
Apache works
Extra features available
```

Example

```
mod_http2
```

Adds HTTP/2 support.

Apache still works without it.

---

# Understanding the Installation Output

Now let's explain your installation output line by line.

---

## Installing

```
Installing:
httpd
```

Meaning

This is the package you requested.

---

## Installing Dependencies

```
Installing dependencies:
```

Meaning

DNF found packages that Apache requires.

Example

```
apr

apr-util

httpd-core

httpd-tools
```

These are automatically installed.

---

## Installing Weak Dependencies

```
Installing weak dependencies:
```

Meaning

Optional packages are also installed.

Example

```
mod_http2

mod_lua
```

They improve functionality.

---

## Transaction Summary

```
Transaction Summary

Install 11 Packages
```

Meaning

Total packages that will be installed:

```
1 Main Package

+

10 Supporting Packages

=

11 Packages
```

---

## Total Download Size

```
Total download size: 2.2 MB
```

Meaning

DNF downloads

```
2.2 MB
```

from repositories.

---

## Installed Size

```
Installed size: 6 MB
```

Meaning

After extraction,

packages occupy

```
6 MB
```

on disk.

Think of downloading a ZIP file.

```
ZIP

↓

Extract

↓

Larger Folder
```

Same concept.

---

## Confirmation

```
Is this ok [y/N]:
```

DNF asks permission.

```
y
```

means

Continue.

```
N
```

means

Cancel.

---

# Download Phase

```
Downloading Packages:
```

Example

```
apr

httpd

mod_lua
```

DNF downloads every RPM.

Example

```
httpd.rpm

↓

Saved temporarily

↓

Ready to install
```

---

# Transaction Check

```
Running transaction check
```

DNF checks

* Missing files
* Broken dependencies
* Package conflicts

Example

```
Package A

requires

Package B

Is B available?

Yes

Continue.
```

---

# Transaction Test

```
Running transaction test
```

DNF performs a **simulation**.

Nothing is installed yet.

It simply checks

"If I install these packages,

will anything break?"

If yes,

installation stops.

If no,

```
Transaction test succeeded
```

---

# Running Transaction

```
Running transaction
```

Now installation actually begins.

Packages are installed one by one.

```
apr

↓

apr-util

↓

httpd-core

↓

httpd
```

---

# Running Scriptlet

```
Running scriptlet
```

Some RPM packages contain scripts.

Example

Apache installation creates

```
/etc/httpd

/var/www

Apache user

Apache group
```

These scripts run automatically.

---

# Installed

```
Installed:
```

Now DNF lists every installed package.

Example

```
apr

httpd

mod_http2
```

Installation finished successfully.

---

# Complete

```
Complete!
```

Everything finished successfully.

---

# Upgrading Software

Command

```bash
sudo dnf upgrade
```

This checks every installed package.

Example

```
Installed

↓

httpd 2.4.60

↓

Repository

↓

httpd 2.4.63

↓

Download

↓

Replace old version
```

Your software becomes newer.

---

# Upgrade One Package

```
sudo dnf upgrade httpd
```

Only Apache is upgraded.

Everything else remains unchanged.

---

# Configuration Files During Upgrade

Suppose

```
/etc/httpd/httpd.conf
```

contains your custom settings.

When upgrading,

DNF tries **not** to overwrite it.

Sometimes new files are created.

Example

```
httpd.conf.rpmnew
```

New version supplied by the package.

```
httpd.conf.rpmsave
```

Backup of your previous configuration.

Administrator compares both files before making changes.

---

# Kernel Upgrade

Kernel is special.

Unlike other packages,

Linux keeps multiple kernel versions.

Example

```
Kernel 1

Kernel 2

Kernel 3
```

Why?

Suppose

Newest kernel fails.

You can still boot using

Older kernel.

This makes upgrades safer.

---

# Upgrade Kernel

```
sudo dnf upgrade kernel
```

New kernel installed.

Old kernel remains.

---

# List Installed Kernels

```
dnf list kernel
```

Example

```
Installed

kernel
6.12.0

kernel
6.12.1
```

Meaning

Two kernels are available.

---

# Check Running Kernel

```
uname -r
```

Output

```
6.12.0-55.12.1.el10.x86_64
```

This is the kernel currently running.

---

# More Kernel Information

```
uname -a
```

Shows

* Hostname
* Kernel version
* Build date
* CPU architecture

Example

```
Linux workstation
6.12.0
x86_64
GNU/Linux
```

---

# Removing Packages

Command

```bash
sudo dnf remove httpd
```

DNF removes

```
httpd

↓

Unused dependencies

↓

Configuration cleanup (if applicable)
```

---

# Why Does DNF Show a Warning?

Suppose

```
Apache

↓

PHP

↓

WordPress
```

If you remove Apache,

PHP and WordPress may also stop working.

That is why DNF warns you.

Always read the package removal list before typing **y**.

---

# Installing Software Groups

Sometimes you need many related packages.

Instead of installing them one by one,

DNF provides **groups**.

Example

```
Development Tools
```

contains

```
gcc

make

gdb

git
```

Install all with one command.

```
sudo dnf group install "Development Tools"
```

---

# View Available Groups

```
dnf group list
```

Example

```
Server

Minimal Install

Development Tools

Security Tools
```

---

# View Group Details

```
dnf group info "Security Tools"
```

Shows

```
Mandatory Packages

Default Packages

Optional Packages
```

---

# Package Categories

### Mandatory

Always installed.

```
Required
```

---

### Default

Installed automatically unless excluded.

```
Recommended
```

---

### Optional

Installed only if specifically requested.

```
Extra Features
```

---

# DNF Transaction History

Every DNF action is recorded.

View history

```bash
dnf history
```

Example

```
ID 11

Installed System Tools

ID 10

Installed httpd
```

Useful for troubleshooting and auditing.

---

# Log File

DNF also writes detailed logs.

```
/var/log/dnf.rpm.log
```

View the last few entries:

```bash
tail -5 /var/log/dnf.rpm.log
```

---

# Summary Table

| Command                                 | Purpose                                     |
| --------------------------------------- | ------------------------------------------- |
| `dnf install httpd`                     | Install a package with dependencies         |
| `dnf upgrade`                           | Update all installed packages               |
| `dnf upgrade httpd`                     | Update one package                          |
| `dnf remove httpd`                      | Remove a package                            |
| `dnf list`                              | List packages                               |
| `dnf search httpd`                      | Search for a package                        |
| `dnf info httpd`                        | Show package details                        |
| `dnf group list`                        | List package groups                         |
| `dnf group install "Development Tools"` | Install a package group                     |
| `dnf history`                           | Display package transaction history         |
| `dnf list kernel`                       | Show installed kernels                      |
| `uname -r`                              | Show the running kernel version             |
| `uname -a`                              | Show detailed kernel and system information |

## SAP BASIS Interview Questions

1. **What is DNF?**
   DNF is the package manager used in RHEL to install, update, remove, and manage RPM software packages and their dependencies.

2. **What is the difference between a required dependency and a weak dependency?**
   Required dependencies are essential for the package to function, while weak dependencies provide optional or recommended features.

3. **Why does `dnf install` automatically install other packages?**
   Because it resolves and installs all required dependencies needed by the requested package.

4. **What is the purpose of the transaction check and transaction test?**
   The transaction check verifies dependencies and conflicts, while the transaction test simulates the installation to ensure it can complete successfully without making changes.

5. **Why are multiple kernel versions kept after an upgrade?**
   To allow the system to boot using an older, working kernel if the newly installed kernel fails. This provides a safe rollback option.

