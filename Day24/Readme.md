# Configuring DNF Repositories in RHEL 

A **DNF repository** is a location that stores software packages (RPM files) along with metadata. Whenever you install, update, or remove packages using the `dnf` command, DNF checks the configured repositories to find the required packages.

Think of a repository like an **app store** for Linux.

* **Package** = Application
* **Repository** = App Store
* **DNF** = Package Manager that downloads applications from the repository

---

# Types of Repositories

There are mainly two types of repositories:

1. **Red Hat Repositories**

   * Provided by Red Hat
   * Available after registering the system

2. **Third-Party Repositories**

   * Created by organizations like Fedora (EPEL), Docker, HashiCorp, etc.
   * Used to install software not available in Red Hat repositories.

---

# How DNF Knows About Repositories

DNF reads repository information from two locations.

## 1. Repository Files (Recommended)

```
/etc/yum.repos.d/
```

Each repository is stored as a separate file ending with `.repo`.

Example:

```
/etc/yum.repos.d/rhel.repo
/etc/yum.repos.d/epel.repo
/etc/yum.repos.d/docker.repo
```

This is the **recommended method** by Red Hat because each repository has its own configuration file.

---

## 2. dnf.conf File

```
/etc/dnf/dnf.conf
```

This file contains global DNF settings.

You can also define repositories here, but Red Hat recommends using it only for global configurations.

Example:

```
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
```

---

# Which Configuration Takes Priority?

If the same repository exists in both places:

```
/etc/yum.repos.d/
```

and

```
/etc/dnf/dnf.conf
```

then the **.repo file takes precedence**.

---

# Method 1: Manually Create a Repository

Suppose you want to configure the EPEL repository manually.

Create a new repository file.

```
sudo vi /etc/yum.repos.d/epel.repo
```

Add the following content.

```ini
[EPEL]
name=Extra Packages for Enterprise Linux 10
baseurl=https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-10
```

Save the file.

Now verify.

```
dnf repolist
```

Output

```
repo id       repo name
EPEL          Extra Packages for Enterprise Linux 10
```

The repository is now available.

---

# Understanding Each Parameter

## 1. Repository ID

```ini
[EPEL]
```

This is the repository identifier.

It must be unique.

Example

```
[EPEL]
```

or

```
[Docker]
```

---

## 2. name

```ini
name=Extra Packages for Enterprise Linux 10
```

This is a human-readable name.

Displayed when running:

```
dnf repolist
```

Example output

```
repo id     repo name

EPEL        Extra Packages for Enterprise Linux 10
```

---

## 3. baseurl

```ini
baseurl=https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/
```

This tells DNF where packages are stored.

Whenever you install software,

```
dnf install htop
```

DNF downloads packages from this URL.

---

## 4. enabled

```ini
enabled=1
```

Repository is enabled.

```
1 = Enabled
0 = Disabled
```

Disabled repositories exist but won't be used until enabled.

Enable later:

```
dnf config-manager --set-enabled EPEL
```

Disable:

```
dnf config-manager --set-disabled EPEL
```

---

## 5. gpgcheck

```ini
gpgcheck=1
```

This enables signature verification.

```
1 = Verify packages

0 = Do not verify packages
```

Always keep this enabled in production.

---

## 6. gpgkey

```ini
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-10
```

This points to the public GPG key.

DNF uses this key to verify packages.

---

# Method 2: Configure Repository Using Command Line

Instead of manually creating the file,

DNF can create it automatically.

Command:

```bash
dnf config-manager \
--add-repo=https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/
```

Output

```
Updating Subscription Management repositories.

Adding repo from:

https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/
```

DNF automatically creates a `.repo` file.

---

# Verify the Created File

Move to repository directory.

```bash
cd /etc/yum.repos.d/
```

Display the file.

```bash
cat dl.fedoraproject.org_pub_epel_10_Everything_x86_64_.repo
```

Output

```ini
[dl.fedoraproject.org_pub_epel_10_Everything_x86_64_]

name=created by dnf config-manager

baseurl=https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/

enabled=1
```

DNF generated this file automatically.

---

# Verifying Signed Packages (GPG)

## Why Verify Packages?

Imagine downloading software from the Internet.

How do you know it wasn't modified by an attacker?

The answer is:

**Digital Signature**

Every trusted package is signed using a private key.

DNF verifies the signature using the public GPG key.

If the signatures match,

the package is genuine.

---

## Simple Analogy

Think about a university certificate.

* University = Repository
* Signature = GPG Signature
* Certificate = RPM Package

If the signature is valid,

you trust the certificate.

If not,

it may be fake.

---

# Where Are GPG Keys Stored?

Normally under

```
/etc/pki/rpm-gpg/
```

Example

```
RPM-GPG-KEY-redhat-release

RPM-GPG-KEY-EPEL-10
```

---

# Import a GPG Key

Download the key first.

Then import it.

```bash
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-10
```

Verify imported keys.

```bash
rpm -qa gpg-pubkey*
```

Example

```
gpg-pubkey-12345678
```

---

# Why Use a Local GPG Key?

Instead of

```
gpgkey=https://website/key
```

Red Hat recommends

```
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-10
```

### Benefits

* Faster verification
* No Internet dependency during installation
* Better security
* Prevents attackers from replacing the key online

---

# Useful Repository Commands

## List enabled repositories

```bash
dnf repolist
```

---

## List all repositories

```bash
dnf repolist all
```

---

## Enable a repository

```bash
dnf config-manager --set-enabled EPEL
```

---

## Disable a repository

```bash
dnf config-manager --set-disabled EPEL
```

---

## Show repository details

```bash
dnf repoinfo
```

---

## Clean repository cache

```bash
dnf clean all
```

---

## Rebuild metadata cache

```bash
dnf makecache
```

---

## Check repository configuration

```bash
dnf repolist -v
```

---

# Real-Time Administrator Scenario (P2 Incident)

Imagine you're the on-call Linux administrator, and a customer raises a **P2 (Priority 2)** ticket requesting access to a new repository because they need to install software urgently. No other team members are available.

**Steps to resolve:**

1. Verify the repository URL provided by the customer.
2. Create a new `.repo` file under `/etc/yum.repos.d/` (or use `dnf config-manager --add-repo`).
3. Configure the repository with the correct `baseurl`, `enabled=1`, and `gpgcheck=1`.
4. Download and import the repository's GPG key into `/etc/pki/rpm-gpg/`.
5. Run:

   ```bash
   dnf clean all
   dnf makecache
   ```
6. Confirm the repository appears in:

   ```bash
   dnf repolist
   ```
7. Test by searching or installing a package:

   ```bash
   dnf search <package-name>
   ```
8. Update the ticket with the actions taken, validation results, and notify the customer that the repository is ready for use.

Following these steps ensures the issue is resolved quickly while maintaining security and compliance.
