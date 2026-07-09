# Configuring Repositories by Using RPM Packages (Easy Explanation)

When you want to install software from a new repository, you have **two options**:

1. **Manually create a `.repo` file** (more work).
2. **Install a Repository RPM package** (easier and recommended).

Most software vendors, such as **EPEL, Docker, Microsoft, HashiCorp, PostgreSQL, and Google**, provide a **repository RPM package** that automatically configures everything for you.

---

# What is a Repository RPM Package?

A **Repository RPM** is a small RPM package whose job is **not to install software**, but to **configure a software repository** on your system.

Think of it as a **setup package** for a repository.

Instead of manually creating files and importing keys, you install one RPM, and it does everything automatically.

---

## Real-Life Example

Imagine buying a new Wi-Fi router.

There are two ways to connect it:

### Method 1 (Manual)

You manually:

* Configure the IP address
* Configure the DNS server
* Configure the gateway
* Configure security

This takes time and is prone to mistakes.

### Method 2 (Automatic Setup)

You run the manufacturer's setup wizard.

The wizard configures everything for you.

A **Repository RPM** works the same way—it automatically sets up the repository.

---

# What Does the Repository RPM Do?

When you install a repository RPM, it performs several tasks automatically:

✅ Creates the `.repo` file in:

```text
/etc/yum.repos.d/
```

✅ Copies the repository configuration.

✅ Adds the repository URL.

✅ Installs the repository's GPG public key.

✅ Configures repository settings.

After that, DNF can download packages from the new repository.

---

# Installation Process

```text
Repository RPM
       │
       ▼
Install RPM
       │
       ▼
Creates .repo file
       │
       ▼
Imports GPG Key
       │
       ▼
Repository is Ready
       │
       ▼
Install Packages Using DNF
```

---

# Example: Installing the EPEL Repository

Suppose you want to install packages from the **EPEL (Extra Packages for Enterprise Linux)** repository.

---

## Step 1: Import the GPG Key

```bash
rpm --import https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-10
```

### What happens?

This command downloads the **public GPG key** and stores it in the RPM database.

Now RPM can verify that packages from EPEL are genuine and have not been modified.

---

## Why Is This Important?

Imagine downloading software from the internet.

How do you know it wasn't changed by a hacker?

Linux uses **digital signatures** to verify packages.

The GPG key acts like a **trusted signature**.

If the signature matches, the package is installed.

If it doesn't, the installation is stopped.

---

## Step 2: Install the Repository RPM

```bash
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-10.noarch.rpm
```

### What happens behind the scenes?

DNF downloads the RPM package.

Then it automatically:

* Creates the repository file
* Configures the repository
* Adds the repository information
* Makes it available for package installation

You don't have to create the `.repo` file yourself.

---

# What Files Are Created?

After installation, check the repository directory:

```bash
ls /etc/yum.repos.d/
```

Example output:

```text
epel.repo
redhat.repo
```

The **epel.repo** file was created automatically.

---

# Viewing the Repository File

```bash
cat /etc/yum.repos.d/epel.repo
```

Example:

```ini
[epel]
name=Extra Packages for Enterprise Linux $releasever - $basearch

metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-$releasever&arch=$basearch

enabled=1

gpgcheck=1

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever
```

Let's understand each line.

---

# `[epel]`

```ini
[epel]
```

This is the **Repository ID**.

DNF uses this name internally.

Example:

```bash
dnf --enablerepo=epel install htop
```

---

# `name`

```ini
name=Extra Packages for Enterprise Linux
```

This is the **display name** shown when you run:

```bash
dnf repolist
```

Example:

```text
repo id     repo name

epel        Extra Packages for Enterprise Linux
```

---

# `metalink`

```ini
metalink=https://mirrors.fedoraproject.org/...
```

Instead of downloading packages from a single server, DNF contacts the metalink URL.

The metalink server returns a list of the **best and fastest mirrors**.

Then DNF downloads packages from the nearest available mirror.

### Why use a metalink?

* Faster downloads
* Automatic failover if one mirror is unavailable
* Better reliability

---

# `enabled=1`

```ini
enabled=1
```

This means the repository is **enabled**.

DNF will search this repository whenever you install or update packages.

If it is:

```ini
enabled=0
```

The repository exists but is ignored until you enable it.

---

# `gpgcheck=1`

```ini
gpgcheck=1
```

This tells DNF to verify the package signature before installation.

Always keep this enabled on production systems.

---

# `gpgkey`

```ini
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-10
```

This tells DNF where the trusted GPG public key is stored on your system.

DNF uses this key to verify downloaded packages.

---

# What If the GPG Key Is Missing?

Suppose you try to install a package.

```bash
dnf install htop
```

If the GPG key has not been imported, DNF displays an error such as:

```text
Public key for package.rpm is not installed

GPG check FAILED
```

The installation stops because Linux cannot verify that the package is from a trusted source.

---

# Can We Ignore GPG Verification?

Yes, by using:

```bash
dnf install --nogpgcheck package-name
```

### What does it do?

It skips signature verification and installs the package anyway.

⚠️ **Warning:** This is not recommended for production systems because it may allow installation of malicious or tampered packages.

---

# Multiple Repositories in One `.repo` File

A single `.repo` file can contain more than one repository.

Example:

```ini
[epel]
enabled=1

[epel-source]
enabled=0
```

---

## Repository 1: `[epel]`

Contains **binary RPM packages** that you install normally.

Example:

```bash
dnf install htop
```

---

## Repository 2: `[epel-source]`

Contains **source RPM (SRPM)** packages.

These are mainly used by developers to:

* Study source code
* Debug software
* Rebuild RPM packages

Since most administrators don't need source packages daily, it is disabled by default:

```ini
enabled=0
```

---

# Temporary Repository Enablement

You can enable a repository for just one command without changing the configuration file.

```bash
dnf --enablerepo=epel-source install package-name
```

Only this command uses the repository.

After it finishes, the repository becomes disabled again.

---

# Permanent Repository Enablement

To enable a repository permanently:

```bash
dnf config-manager --set-enabled epel-source
```

To disable it permanently:

```bash
dnf config-manager --set-disabled epel
```

These commands modify the `.repo` file.

---
