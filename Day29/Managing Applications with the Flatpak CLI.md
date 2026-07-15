# Managing Applications with the Flatpak CLI

Flatpak provides a simple command-line interface (CLI) to **search, install, update, remove, and manage applications** along with their required runtimes.

> **Note**
>
> * You can install applications from **Red Hat repositories** and **third-party repositories** (such as Flathub).
> * **Red Hat officially supports only applications and runtimes from Red Hat repositories.**

---

## Flatpak Object Identifiers

Every Flatpak application or runtime has a **unique identifier**.

### Format

```text
domain.organization.Application
```

### Example

```text
org.mozilla.Thunderbird
```

Here:

* **org** → Domain
* **mozilla** → Organization
* **Thunderbird** → Application name

You can also use the **short name** when it is unique.

```bash
flatpak install thunderbird
```

or

```bash
flatpak install org.mozilla.Thunderbird
```

---

## Identifier Triple

Sometimes you must specify:

* Architecture
* Branch (version)

Format:

```text
ID/Architecture/Branch
```

Example:

```text
org.mozilla.Thunderbird/x86_64/stable
```

Meaning:

| Component    | Value                   |
| ------------ | ----------------------- |
| ID           | org.mozilla.Thunderbird |
| Architecture | x86_64                  |
| Branch       | stable                  |

### Specify only Architecture

```text
org.mozilla.Thunderbird/x86_64//
```

### Specify only Branch

```text
org.mozilla.Thunderbird//stable
```

---

# Searching Remote Repositories

Use the **flatpak search** command.

## Syntax

```bash
flatpak search <keyword>
```

Example:

```bash
flatpak search thunderbird
```

Verbose output:

```bash
flatpak -v search thunderbird
```

---

## Example Output

```text
Thunderbird
Thunderbird is a free email client

Application ID:
org.mozilla.Thunderbird

Branch:
stable

Remote:
rhel
```

---

## How Search Works

Flatpak searches the cached AppStream metadata.

If the cache is **less than one day old**, it uses the local cache.

Otherwise, it downloads updated metadata.

This makes searching much faster.

---

## Search Matches

Flatpak searches the keyword in:

* Application name
* Description
* Application ID
* Developer-defined keywords

The search is **case-insensitive**.

Example:

```bash
flatpak search firefox
flatpak search thunderbird
flatpak search browser
```

---

# Installing Applications

## Basic Syntax

```bash
flatpak install <application>
```

Example:

```bash
flatpak install thunderbird
```

---

## Installation Process

When multiple repositories contain the application:

```text
1) fedora
2) rhel
```

You choose one.

Example:

```text
Choose:
2
```

---

### Runtime Resolution

Flatpak automatically installs required runtimes.

Example:

```text
Application:
org.mozilla.Thunderbird

Runtime:
com.redhat.Platform
```

If the runtime is already installed:

* It is reused
* No duplicate download occurs

---

### Permission Review

Before installation, Flatpak displays permissions.

Example:

```text
network
ipc
x11
pulseaudio
home directory (read-only)
notifications
```

This shows what resources the application can access.

---

### Installation Summary

Example:

```text
Object
------------------------------
com.redhat.Platform
org.mozilla.Thunderbird

Download Size:
395 MB
147 MB
```

After confirmation:

```text
Installation complete.
```

---

# Useful Install Options

| Option             | Description                   |
| ------------------ | ----------------------------- |
| `--runtime`        | Install only runtimes         |
| `--app`            | Install only applications     |
| `--or-update`      | Update if already installed   |
| `--system`         | Install for all users         |
| `--user`           | Install only for current user |
| `-y`               | Automatically answer Yes      |
| `--no-deploy`      | Download only (don't install) |
| `--no-pull`        | Install only from cache       |
| `--noninteractive` | Silent installation           |

---

## Install for Current User

```bash
flatpak install --user thunderbird
```

Installed into

```text
~/.local/share/flatpak
```

---

## Install System-wide

```bash
sudo flatpak install --system thunderbird
```

Installed into

```text
/var/lib/flatpak
```

Available for every user.

---

## Automatic Installation

```bash
flatpak install -y thunderbird
```

No confirmation prompts.

---

# Listing Installed Applications

Use:

```bash
flatpak list
```

Example:

```text
Name                 Application ID               Branch

Red Hat Platform     com.redhat.Platform          el10
Thunderbird          org.mozilla.Thunderbird      stable
```

---

# Viewing Application Information

Use:

```bash
flatpak info <Application-ID>
```

Example:

```bash
flatpak info org.mozilla.Thunderbird
```

Example output:

```text
ID:
org.mozilla.Thunderbird

Architecture:
x86_64

Branch:
stable

Origin:
rhel

Installation:
system

Installed Size:
324 MB

Runtime:
com.redhat.Platform

SDK:
com.redhat.Sdk

Commit:
3dd6...

Date:
2025-05-27
```

---

# Where Flatpak Stores Applications

### System Installation

```text
/var/lib/flatpak
```

Applications are available for **all users**.

---

### User Installation

```text
~/.local/share/flatpak
```

Applications are available **only to the current user**.

---

# Common Flatpak Commands

| Task                     | Command                                 |
| ------------------------ | --------------------------------------- |
| Search applications      | `flatpak search firefox`                |
| Install application      | `flatpak install firefox`               |
| Install for current user | `flatpak install --user firefox`        |
| Install system-wide      | `sudo flatpak install --system firefox` |
| Install automatically    | `flatpak install -y firefox`            |
| List installed apps      | `flatpak list`                          |
| View application info    | `flatpak info org.mozilla.Firefox`      |
| Update all applications  | `flatpak update`                        |
| Remove an application    | `flatpak uninstall org.mozilla.Firefox` |
| Remove unused runtimes   | `flatpak uninstall --unused`            |

---

# Interview Questions

### 1. What is Flatpak?

Flatpak is a universal Linux package management system that installs applications in isolated sandboxes with all required dependencies.

---

### 2. What is a Flatpak runtime?

A runtime is a shared platform containing libraries and dependencies required by one or more Flatpak applications.

---

### 3. How do you search for applications?

```bash
flatpak search thunderbird
```

---

### 4. How do you install a Flatpak application?

```bash
flatpak install thunderbird
```

---

### 5. Where are system-wide Flatpak applications stored?

```text
/var/lib/flatpak
```

---

### 6. Where are user-installed Flatpak applications stored?

```text
~/.local/share/flatpak
```

---

### 7. How do you list installed Flatpak applications?

```bash
flatpak list
```

---

### 8. How do you view detailed information about an application?

```bash
flatpak info org.mozilla.Thunderbird
```

---

### 9. Why does Flatpak ask for permissions during installation?

Because Flatpak applications run in a sandbox and require explicit permissions to access system resources such as the network, files, audio, display, or notifications.

---

### 10. What is the difference between `--system` and `--user`?

* `--system`: Installs applications for **all users** in `/var/lib/flatpak`.
* `--user`: Installs applications **only for the current user** in `~/.local/share/flatpak`.
