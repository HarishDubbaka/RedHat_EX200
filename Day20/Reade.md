# Querying and Extracting RPM Packages Before Installation

When you download an RPM package from a third-party source, it's a good security practice to **inspect the package before installing it**. This helps you verify:

* What files will be installed.
* Package information (name, version, architecture, etc.).
* Installation scripts that will run.
* Documentation included in the package.

---

# 1. Query an RPM Package Before Installation

Normally, the `rpm` command queries packages that are **already installed**.

To inspect a package file **without installing it**, use the **`-p` (package)** option.

## Syntax

```bash
rpm [query-options] -p package.rpm
```

The `-p` option tells RPM:

> "Read information directly from this RPM file instead of the installed package database."

---

## List Files Inside an RPM

### Command

```bash
rpm -qlp package.rpm
```

### Breakdown

| Option | Meaning              |
| ------ | -------------------- |
| `-q`   | Query package        |
| `-l`   | List installed files |
| `-p`   | Query a package file |

---

### Example

```bash
ls -l podman-5.4.0-1.el10.x86_64.rpm
```

Output

```text
-rw-r--r-- 1 user user 16494682 Jun 9 23:43 podman-5.4.0-1.el10.x86_64.rpm
```

Now inspect it:

```bash
rpm -qlp podman-5.4.0-1.el10.x86_64.rpm
```

Output

```text
/usr/bin/podman
/usr/lib/.build-id
/usr/share/man/man1/podman.1.gz
...
```

### What it shows

All files that would be installed if you install the package.

---

# Other Useful RPM Queries

## Package Information

```bash
rpm -qip package.rpm
```

Example output

```text
Name        : podman
Version     : 5.4.0
Release     : 1.el10
Architecture: x86_64
Summary     : Container Management Tool
```

---

## Documentation

```bash
rpm -qdp package.rpm
```

Shows documentation files like:

```text
README
LICENSE
CHANGELOG
```

---

## Installation Scripts

```bash
rpm -q --scripts -p package.rpm
```

Shows scripts that run during installation or removal, such as:

* `%pre`
* `%post`
* `%preun`
* `%postun`

Example

```bash
rpm -q --scripts -p package.rpm
```

Output

```bash
postinstall scriptlet:
systemctl daemon-reload
```

This is useful because RPM packages can execute commands as **root**.

---

# Why Inspect an RPM?

Imagine downloading software from an unknown website.

Before installing, you should verify:

* Which files it installs.
* Whether it contains suspicious scripts.
* Whether it places files in unexpected locations.
* Whether documentation and licensing look legitimate.

This is similar to checking the contents of a ZIP archive before extracting it.

---

# 2. Installing an RPM Package

RPM can install packages directly.

## Command

```bash
rpm -ivh package.rpm
```

### Options

| Option | Meaning                             |
| ------ | ----------------------------------- |
| `-i`   | Install                             |
| `-v`   | Verbose output                      |
| `-h`   | Show progress with hash (`#`) marks |

---

### Example

```bash
sudo rpm -ivh podman-5.4.0-1.el10.x86_64.rpm
```

Output

```text
Verifying...          ################################# [100%]
Preparing...          ################################# [100%]
Updating / installing...
podman-6:5.4.0-1      ################################# [100%]
```

---

# Important Limitation of RPM

The RPM command **does not resolve dependencies**.

Suppose the package requires:

* libc
* openssl
* glib2

If one is missing:

```bash
sudo rpm -ivh package.rpm
```

Output

```text
error: Failed dependencies:
libxyz.so is needed by package.rpm
```

Installation stops.

---

# Why Use DNF Instead?

DNF automatically downloads and installs required dependencies.

Example

```bash
sudo dnf install package.rpm
```

DNF will:

* Check dependencies
* Download missing packages
* Install everything in the correct order

This is why **DNF is recommended over RPM for package installation**.

---

# Security Warning

Be careful with third-party RPM packages because they can contain:

* Malware
* Outdated software
* Untrusted code
* Scripts executed with root privileges

Always inspect packages before installing them.

---

# 3. Extract Files Without Installing

Sometimes you only need one file from an RPM.

Instead of installing the package, extract its contents.

This is done using:

```text
rpm2cpio
```

---

## What is rpm2cpio?

It converts an RPM package into a **CPIO archive**.

Think of it like:

```text
RPM file
      │
      ▼
rpm2cpio
      │
      ▼
CPIO archive
      │
      ▼
cpio command extracts files
```

---

# Extract Everything

### Command

```bash
rpm2cpio httpd.rpm | cpio -idv
```

---

## Breakdown

### rpm2cpio

Converts

```text
httpd.rpm
```

into

```text
cpio archive
```

---

### Pipe

```bash
|
```

Sends the archive directly to the `cpio` command.

---

### cpio Options

| Option | Meaning            |
| ------ | ------------------ |
| `-i`   | Extract files      |
| `-d`   | Create directories |
| `-v`   | Verbose output     |

---

Example

```bash
rpm2cpio httpd-2.4.63-1.el10.x86_64.rpm | cpio -idv
```

Output

```text
./etc/httpd/conf.modules.d/00-brotli.conf
./etc/httpd/conf.modules.d/00-systemd.conf
./usr/lib/systemd/system/httpd.service
...
```

---

After extraction

```bash
ls
```

Output

```text
etc/
usr/
httpd-2.4.63-1.el10.x86_64.rpm
```

The package is **not installed**.

The files are simply copied into the current directory.

---

# Extract Only One File

Instead of extracting everything:

```bash
rpm2cpio package.rpm | cpio -id "/path/to/file"
```

Example

```bash
rpm2cpio httpd-2.4.63-1.el10.x86_64.rpm | \
cpio -id "/etc/httpd/conf.modules.d/00-systemd.conf"
```

Output

```text
121 blocks
```

Verify

```bash
ls etc/httpd/conf.modules.d/
```

Output

```text
00-systemd.conf
```

Only the requested file is extracted.

---

# List Files Without Extracting

To see the contents of an RPM without installing or extracting:

```bash
rpm2cpio package.rpm | cpio -tv
```

### Options

| Option | Meaning         |
| ------ | --------------- |
| `-t`   | List contents   |
| `-v`   | Verbose listing |

Example

```bash
rpm2cpio httpd-2.4.63-1.el10.x86_64.rpm | cpio -tv
```

Output

```text
-rw-r--r-- root root    ./etc/httpd/conf.modules.d/00-brotli.conf
-rw-r--r-- root root    ./etc/httpd/conf.modules.d/00-systemd.conf
-rw-r--r-- root root    ./usr/lib/systemd/system/httpd.service
...
```

This is similar to:

```bash
tar -tvf archive.tar
```

for TAR files.

---

# RPM vs DNF vs rpm2cpio

| Feature                          | `rpm`          | `dnf`   | `rpm2cpio`     |
| -------------------------------- | -------------- | ------- | -------------- |
| Install package                  | ✅              | ✅       | ❌              |
| Resolve dependencies             | ❌              | ✅       | ❌              |
| Query installed packages         | ✅              | Limited | ❌              |
| Query local RPM file             | ✅ (`-p`)       | ❌       | ❌              |
| Extract files without installing | ❌              | ❌       | ✅              |
| List package contents            | ✅ (`rpm -qlp`) | ❌       | ✅ (`cpio -tv`) |

---

# Quick Command Cheat Sheet

| Task                               | Command                                            |
| ---------------------------------- | -------------------------------------------------- |
| Show package information           | `rpm -qip package.rpm`                             |
| List files in an RPM               | `rpm -qlp package.rpm`                             |
| Show documentation                 | `rpm -qdp package.rpm`                             |
| Show install/remove scripts        | `rpm -q --scripts -p package.rpm`                  |
| Install an RPM                     | `sudo rpm -ivh package.rpm`                        |
| Install with dependency resolution | `sudo dnf install package.rpm`                     |
| Extract all files                  | `rpm2cpio package.rpm \| cpio -idv`                |
| Extract a specific file            | `rpm2cpio package.rpm \| cpio -id "/path/to/file"` |
| List files without extracting      | `rpm2cpio package.rpm \| cpio -tv`                 |

## Key Takeaways

* Use **`rpm -p`** to inspect an RPM file before installation.
* **`rpm -qlp`** lists all files inside the package.
* **`rpm -ivh`** installs an RPM but **does not resolve dependencies**.
* Prefer **`dnf install package.rpm`** for automatic dependency resolution.
* Use **`rpm2cpio | cpio`** to extract or inspect files without installing the package.
* Always inspect RPMs from third-party sources to reduce security risks.
