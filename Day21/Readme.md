# 📦 Managing Software Packages with DNF (RHEL 9 & RHEL 10) 
## 🎯 Learning Objectives

After completing this lesson, you will understand:

* Why Linux uses package managers
* What DNF is and how it works internally
* Why DNF replaced YUM
* Difference between RPM and DNF
* BaseOS and AppStream repositories
* Package metadata and repositories
* How to search packages efficiently
* How to inspect package information
* How to find which package owns a file
* Real-world administration examples

---

# Before Understanding DNF

Imagine Windows.

When you install Google Chrome:

1. Open browser
2. Search Google Chrome
3. Download installer
4. Double-click
5. Install

Linux works differently.

Almost every software is installed from **trusted repositories**.

Instead of downloading software manually, Linux asks its package manager.

```
User
   │
   ▼
Package Manager (DNF)
   │
   ▼
Software Repository
   │
   ▼
Downloads & Installs Packages
```

This is safer because:

* Software comes from trusted sources
* Digital signatures are verified
* Dependencies are installed automatically
* Updating all software becomes very easy

---

# What is a Package?

Before learning DNF, first understand **packages**.

A package is simply a compressed file containing everything needed for a program.

Think of it like a ZIP archive.

Example:

```
httpd.rpm
```

Inside this RPM package are:

```
Apache Program Files

├── Executables
├── Libraries
├── Configuration files
├── Documentation
├── License
├── Service files
└── Metadata
```

Every software in RHEL is distributed as an RPM package.

Examples:

```
vim.rpm
firefox.rpm
httpd.rpm
nginx.rpm
python3.rpm
```

---

# What is RPM?

RPM stands for

> **Red Hat Package Manager**

RPM is the **low-level package management tool**.

Its job is very limited.

It can:

* Install packages
* Remove packages
* Verify packages
* Query installed packages

Example

```bash
rpm -ivh httpd.rpm
```

RPM only installs the package you provide.

It does **NOT** automatically download missing packages.

---

## Problem with RPM

Suppose Apache requires four libraries.

```
httpd

Needs

├── apr
├── apr-util
├── openssl
└── pcre
```

You run

```bash
rpm -ivh httpd.rpm
```

RPM replies

```
Error:

Missing dependency:

apr
apr-util
openssl
pcre
```

Now you must install each dependency manually.

This becomes frustrating.

---

# Why DNF Exists

DNF solves this dependency problem.

Instead of installing only one RPM,

DNF looks at the package metadata.

It asks:

```
What does httpd require?
```

Repository replies

```
httpd requires

apr
apr-util
openssl
pcre
```

DNF downloads everything automatically.

```
User

dnf install httpd

        │

        ▼

Checks Repository

        │

        ▼

Find Dependencies

        │

        ▼

Downloads Packages

        │

        ▼

Installs Everything
```

This is why administrators almost always use **DNF** instead of RPM.

---

# What Does DNF Mean?

DNF stands for

> **Dandified YUM**

It is the next generation of YUM.

Introduced to provide:

* Faster dependency solving
* Better performance
* Cleaner code
* Improved repository handling
* Better plugin architecture

---

# Why Was YUM Replaced?

Earlier RHEL versions used

```
YUM
```

Starting with RHEL 9

```
DNF
```

replaced YUM.

However,

millions of administrators still type

```
yum install
```

instead of

```
dnf install
```

To avoid breaking old scripts,

Red Hat created a symbolic link.

```
yum
    │
    ▼
dnf
```

Verify

```bash
ls -l /usr/bin/yum
```

Output

```
yum -> dnf-3
```

Meaning

```
yum install httpd
```

and

```
dnf install httpd
```

execute exactly the same program.

---

# DNF Architecture

```
                User
                  │
                  ▼
             DNF Command
                  │
                  ▼
        Reads Repository Metadata
                  │
                  ▼
      Resolves Dependencies
                  │
                  ▼
      Downloads RPM Packages
                  │
                  ▼
      Verifies Digital Signatures
                  │
                  ▼
      Installs Using RPM Engine
                  │
                  ▼
        Updates RPM Database
```

Notice something important.

**DNF does NOT install packages itself.**

The actual installation is still done by **RPM**.

Think of DNF as a smart manager sitting on top of RPM.

```
DNF
 │
 ▼
RPM
 │
 ▼
Package Installation
```

---

# Repository Explained

A repository is simply a warehouse of software packages.

Imagine Amazon.

Instead of books,

the warehouse stores RPM packages.

```
Repository

├── vim
├── httpd
├── nginx
├── firefox
├── python
├── git
└── hundreds of packages
```

When you type

```bash
dnf install nginx
```

DNF contacts the repository,

downloads nginx,

downloads dependencies,

then installs everything.

---

# Why Use Repositories?

Without repositories,

you would need to

* search websites
* download RPMs
* download dependencies
* verify versions
* verify signatures

Repositories automate everything.

Advantages:

✔ Trusted software

✔ Automatic updates

✔ Dependency resolution

✔ Version management

✔ Security

---

# BaseOS Repository

RHEL 10 provides two main repositories.

The first is **BaseOS**.

BaseOS contains the operating system itself.

Examples

```
Kernel

Bash

Systemd

glibc

RPM

DNF

Coreutils

SELinux
```

Think of BaseOS as the **engine of the operating system**.

Without BaseOS,

Linux cannot boot.

```
BaseOS

├── Kernel
├── Shell
├── Libraries
├── Boot files
├── RPM
├── DNF
└── Essential Utilities
```

These packages have the same lifecycle as RHEL.

---

# Application Stream (AppStream)

AppStream contains applications.

Examples

```
Apache

Python

PHP

MariaDB

PostgreSQL

Redis

NodeJS

Git
```

Unlike BaseOS,

applications change more frequently.

```
AppStream

├── Web Servers
├── Databases
├── Programming Languages
├── Development Tools
├── Containers
└── Monitoring Tools
```

---

# RHEL 10 Change

In RHEL 8 and RHEL 9,

many applications were provided as **modules**.

Example

```
NodeJS

8

10

12

14

16

18
```

Administrators selected one stream.

```
dnf module enable nodejs:18
```

In RHEL 10,

modules have been removed.

Everything is now distributed as regular RPM packages.

---

# DNF Metadata

When you run

```bash
dnf search nginx
```

DNF does **NOT** download every package.

Instead,

it downloads **metadata**.

Metadata contains information such as:

```
Package Name

Version

Dependencies

Repository

Description

Architecture

Checksums

Size
```

Metadata is much smaller than actual packages,

making searches very fast.

---

# Listing Packages

```
dnf list
```

Lists

* Installed packages
* Available packages

Example

```
Installed Packages

bash
vim
git

Available Packages

httpd
nginx
mariadb
```

---

# Wildcards

Wildcards help search patterns.

```
*
```

means

> "Anything"

Example

```
dnf list "http*"
```

Matches

```
httpd

httpd-core

httpd-tools

httpd-devel

httpcomponents-core
```

Because every package begins with

```
http
```

---

# Searching Packages

Suppose you don't know the package name.

Instead of guessing,

search.

```
dnf search apache
```

Output

```
httpd
```

DNF searches

* Package name
* Summary

---

# Search All

Even more powerful.

```
dnf search all "web server"
```

Now DNF searches

```
Package Name

Summary

Description
```

Example

```
nginx

httpd

varnish

python3-tornado
```

Even if "web server" appears only in the description,

DNF finds it.

---

# Viewing Package Information

```
dnf info httpd
```

Example

```
Name

Version

Release

Architecture

Repository

Size

License

Description

URL
```

Let's understand each field.

---

## Name

```
httpd
```

Package name.

---

## Version

```
2.4.63
```

Software version.

---

## Release

```
1.el10
```

Packaging revision.

---

## Architecture

```
x86_64
```

CPU architecture.

Other examples

```
aarch64

ppc64le

s390x

noarch
```

---

## Repository

```
AppStream
```

Where the package comes from.

---

## Size

```
53 KB
```

Download size.

---

## Description

Explains what the package does.

Useful before installation.

---

# Finding Which Package Owns a File

One of the most useful DNF features.

Suppose you discover

```
/var/www/html
```

Which package created it?

Run

```bash
dnf provides /var/www/html
```

Output

```
httpd-filesystem
```

Meaning

```
httpd-filesystem

creates

/var/www/html
```

---

# Another Example

You know

```
/usr/bin/python3
```

but don't know the package.

```
dnf provides /usr/bin/python3
```

Output

```
python3
```

---

# Wildcard Search

Suppose you only remember

```
ifconfig
```

Run

```bash
dnf provides */ifconfig
```

Output

```
net-tools
```

Very useful during troubleshooting.

---

# Real Administrator Scenario

A developer says:

> "Apache is missing."

You don't know the package.

### Step 1

Search

```bash
dnf search apache
```

↓

Find package

```
httpd
```

---

### Step 2

Inspect

```bash
dnf info httpd
```

Learn

* Version
* Repository
* Description
* Size

---

### Step 3

Need to know which package owns the web directory

```
/var/www/html
```

Run

```bash
dnf provides /var/www/html
```

↓

```
httpd-filesystem
```

---

### Step 4

Install

```bash
sudo dnf install httpd
```

DNF automatically installs

```
httpd

httpd-core

httpd-filesystem

apr

apr-util

openssl

pcre

...
```

without manual intervention.

---

# DNF Command Summary

| Command                     | Purpose                                              |
| --------------------------- | ---------------------------------------------------- |
| `dnf help`                  | Display help and usage information                   |
| `dnf list`                  | List installed and available packages                |
| `dnf list 'http*'`          | List packages matching a wildcard                    |
| `dnf search <keyword>`      | Search by package name and summary                   |
| `dnf search all "<text>"`   | Search package name, summary, and description        |
| `dnf info <package>`        | Display detailed package information                 |
| `dnf provides <file>`       | Find which package owns a specific file or directory |
| `dnf provides */<filename>` | Find a package using a wildcard file search          |

---

# Key Takeaways

* **DNF** is the default package manager in **RHEL 9 and RHEL 10** and replaces **YUM**.
* **YUM** remains available as a symbolic link for backward compatibility.
* **RPM** is the low-level package installer, while **DNF** is a high-level package manager that uses RPM underneath.
* DNF automatically resolves dependencies, downloads required packages, verifies signatures, and maintains transaction history.
* **BaseOS** contains essential operating system components, while **AppStream** provides applications and development tools.
* Starting with **RHEL 10**, **DNF modules have been removed**; applications are delivered as standard RPM packages.
* Commands like `dnf search`, `dnf info`, and `dnf provides` are essential for daily Linux administration and troubleshooting.
