# Investigating RPM Software Packages 

---

# What is RPM?

**RPM** stands for **RPM Package Manager**.

It is the standard package management system used by **Red Hat Enterprise Linux (RHEL)** and other RPM-based Linux distributions such as **CentOS**, **Rocky Linux**, **AlmaLinux**, and **Fedora**.

An RPM package contains:

* The software files
* Package information (metadata)
* Dependencies
* Installation and removal scripts
* Digital signature (GPG)

**Benefits of RPM:**

* Easy software installation and removal
* Tracks installed files
* Checks required dependencies
* Verifies package integrity with GPG signatures
* Stores package information in the local RPM database

---

# RPM Package Name Format

Example:

```text
coreutils-9.5-6.el10.x86_64.rpm
```

It follows this format:

```text
name-version-release.architecture.rpm
```

| Part         | Example   | Meaning                        |
| ------------ | --------- | ------------------------------ |
| Name         | coreutils | Package name                   |
| Version      | 9.5       | Original software version      |
| Release      | 6.el10    | Red Hat package release number |
| Architecture | x86_64    | CPU architecture               |
| Extension    | .rpm      | RPM package file               |

---

## Example

```text
bash-5.2.26-4.el10.x86_64.rpm
```

| Part         | Value  |
| ------------ | ------ |
| Name         | bash   |
| Version      | 5.2.26 |
| Release      | 4.el10 |
| Architecture | x86_64 |

---

# RPM Repository

A **repository** is a storage location that contains RPM packages.

Instead of downloading software manually, tools like **DNF** download packages from repositories.

Example:

```bash
dnf install httpd
```

DNF automatically downloads the latest RPM package from the configured repositories.

---

# What Does an RPM Package Contain?

An RPM package contains:

### 1. Software Files

Example:

```text
/usr/bin/httpd
/usr/lib/systemd/system/httpd.service
```

---

### 2. Metadata

Metadata includes:

* Package name
* Version
* Release
* Architecture
* Description
* License
* Dependencies
* Maintainer
* Changelog

---

### 3. Scripts

Scripts can run automatically:

* Before installation
* After installation
* Before removal
* After removal

Example:

```text
Create user
Create group
Restart service
Reload daemon
```

---

### 4. GPG Signature

Every official Red Hat package is digitally signed.

Before installation RPM checks:

* Package authenticity
* Package integrity

If the signature is invalid:

```text
Installation fails
```

---

# Updating RPM Packages

When updating software:

Old version

↓

Removed

↓

Latest version installed

Example:

```
httpd-2.4.60

↓

httpd-2.4.62
```

Usually configuration files are preserved.

---

# Multiple Package Versions

Normally:

Only one version is installed.

Example:

```
bash
```

Only one version exists.

---

## Exception: Kernel

Multiple kernels can exist simultaneously.

Example:

```text
kernel-6.12.0-45
kernel-6.12.0-141
```

Reason:

If the latest kernel fails to boot, you can boot into an older kernel.

---

# RPM Commands

---

## 1. List All Installed Packages

```bash
rpm -qa
```

Meaning

```
-q
Query

-a
All
```

Example

```bash
rpm -qa
```

Output

```text
bash-5.2
vim-9.0
httpd-2.4
dnf-4.20
```

---

## 2. Find Which Package Owns a File

```bash
rpm -qf filename
```

Example

```bash
rpm -qf /etc/yum.repos.d
```

Output

```text
redhat-release-10.0-30.el10.x86_64
```

Meaning

That directory was installed by the `redhat-release` package.

---

## 3. Check Installed Version

```bash
rpm -q package
```

Example

```bash
rpm -q dnf
```

Output

```text
dnf-4.20.0-12.el10.noarch
```

---

## 4. Detailed Package Information

```bash
rpm -qi package
```

Example

```bash
rpm -qi dnf
```

Displays:

* Name
* Version
* Release
* Install date
* Size
* Vendor
* Summary
* Description
* License

---

## 5. List Installed Files

```bash
rpm -ql package
```

Example

```bash
rpm -ql dnf
```

Output

```text
/usr/bin/dnf
/usr/share/bash-completion/completions/dnf
/usr/lib/systemd/system/dnf.service
```

---

## 6. List Configuration Files

```bash
rpm -qc package
```

Example

```bash
rpm -qc openssh-clients
```

Output

```text
/etc/ssh/ssh_config
/etc/ssh/ssh_config.d/50-redhat.conf
```

Only configuration files are shown.

---

## 7. List Documentation Files

```bash
rpm -qd package
```

Example

```bash
rpm -qd openssh-clients
```

Output

```text
/usr/share/man/man1/ssh.1.gz
/usr/share/man/man1/scp.1.gz
```

Shows manuals and documentation.

---

## 8. View Installation/Removal Scripts

```bash
rpm -q --scripts package
```

Example

```bash
rpm -q --scripts openssh-server
```

Output includes scriptlets such as:

```text
Create sshd user

Create sshd group

Enable service

Restart service
```

These scripts execute automatically during package installation or removal.

---

## 9. View Package Changelog

```bash
rpm -q --changelog package
```

Example

```bash
rpm -q --changelog audit
```

Output

```text
Jan 08 2025

Rebase to 4.0.3

Fixed checkpoint bug

Updated plugins
```

Useful to see recent fixes and improvements.

---

# Sorting RPM Versions

Regular sorting:

```bash
rpm -q kernel | sort
```

Output

```text
kernel-6.12.0-141
kernel-6.12.0-45
```

This is alphabetical, not version-aware.

---

Correct RPM version sorting:

```bash
rpm -q kernel | rpmsort
```

Output

```text
kernel-6.12.0-45
kernel-6.12.0-141
```

`rpmsort` understands RPM version numbers.

---

# Common RPM Query Options

| Command                      | Purpose                                  |
| ---------------------------- | ---------------------------------------- |
| `rpm -qa`                    | List all installed packages              |
| `rpm -q package`             | Show installed package version           |
| `rpm -qi package`            | Show detailed package information        |
| `rpm -ql package`            | List files installed by a package        |
| `rpm -qc package`            | List configuration files                 |
| `rpm -qd package`            | List documentation files                 |
| `rpm -qf file`               | Find which package owns a file           |
| `rpm -q --scripts package`   | Show install/remove scripts              |
| `rpm -q --changelog package` | Display package changelog                |
| `rpm -q kernel \| rpmsort`   | Sort installed kernel versions correctly |

---

# RHCSA Exam Tips

* Use `rpm -qa` to view all installed packages.
* Use `rpm -qf <file>` to identify the package that installed a specific file.
* Use `rpm -ql <package>` to see every file provided by a package.
* Use `rpm -qc <package>` to locate configuration files.
* Use `rpm -qd <package>` to find documentation and man pages.
* Use `rpm -qi <package>` for detailed package metadata.
* Use `rpmsort` instead of `sort` when ordering RPM version strings.
