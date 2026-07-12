# Managing Flatpak Remote Repositories (Beginner-Friendly Guide)

![Image](https://images.openai.com/static-rsc-4/ICF15gLF3EShqUPRPGE0883tK57arXVrMHHaYjL1vg0XgAcEPg0YaLoAUGMo44MxY-rIESxo1-QNyUpgQUiRWSbAPM6qLc8SBW9KLMgTJcBWR6IHWCCPba_XwMOeNakM5shZcjX0K0nXjAX7CXn6x9kAUOcpnIj5aHWXO09-1mxxeqG5c-OO8Lwlg38PjkCz?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/w61m0vTJH88QfoGAbcDUFpNJFjIn5qR4hVLRH-Hyf8DP960l0YsP2qRMIHcbz2FLcvSpdEAEeq1Br_tpNEHuLNrgdoBuLfe-2EU-giDzKWc_i-ufa6rywQTwf-SG6NBee9AOqJVfI3NLQd8F5HjxSMDAZ3YsHPUeuES_5Pa0P3KK7TLyTKzU3K__PwIP-F1p?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ccg3s9q7w9SzO8IYY-Dtx439abZtwQWVZOQ0TFMgdKSTjHjzRMVZ_byZL0WZFl6DbFwb-P1koycHQYFMK55nIXtlaZLxY4szNMgVVq5z7dkdcW00DtwLbZwEnnef8YWBuxcYQMkXmVzk6MAw_QiDPtqUl6XpR_ynKMLx7H8CngLWGqi19LJbjSdx8PXfiXqL?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/wAuJ8Lon7lW3kfN12zsO87a1vWLAYCzZukWoWkHabeiNc5IOiSbJjO8CDUCGsrIxP77cG7pQDjWOUCsUSSJ359t1_3U6hPsqhwlNl-HR2ckaLb3bg5mGt6p2AIvBh68pjjwMO-fXVPyXSTOPAE4pM8k2yWg7ly08Z4h8XW4kNQxWXeELWkpS9Gts3wXHTeNw?purpose=fullsize)

## Learning Objectives

After completing this lesson, you will be able to:

* Understand what a **Flatpak Remote Repository (Remote)** is.
* Configure Flatpak remotes.
* Authenticate to the Red Hat Flatpak repository.
* View configured repositories.
* List available applications.
* Understand the difference between **RHEL Flatpak Repository** and **Flathub**.

---

# What is a Flatpak Remote Repository?

A **Remote** is simply a **repository that stores Flatpak applications and runtimes**.

Think of it like:

| Package Manager | Repository            |
| --------------- | --------------------- |
| DNF             | BaseOS, AppStream     |
| APT             | Ubuntu Repository     |
| Docker          | Docker Hub            |
| **Flatpak**     | **Remote Repository** |

Without a remote, Flatpak has **nowhere to download applications from**.

---

# Real-Life Analogy

Imagine you're shopping online.

* **Amazon** → Stores products
* **Flipkart** → Stores products
* **Myntra** → Stores products

Similarly:

* **Flathub** → Stores Flatpak apps
* **Red Hat Repository** → Stores Red Hat-certified Flatpak apps
* **Fedora Repository** → Stores Fedora Flatpak apps

The repository is just the **store**, and Flatpak is the **customer** that downloads applications.

---

# Types of Flatpak Repositories

## 1. Red Hat Flatpak Repository

Supported by Red Hat.

✔ Official

✔ Enterprise-tested

✔ Stable

Ideal for production environments.

---

## 2. Flathub

Flathub

The largest Flatpak repository.

Contains thousands of applications like:

* Firefox
* VLC
* Spotify
* Discord
* LibreOffice
* GIMP

**Note:**

⚠️ Red Hat does **not** officially support applications installed from Flathub.

---

## 3. Fedora Flatpak Repository

Maintained by the Fedora Project.

Contains community-maintained Flatpak packages.

---

# Initial Flatpak Setup

Flatpak is installed by default on **RHEL 10** with GNOME Software.

Verify:

```bash
flatpak --version
```

Example:

```text
Flatpak 1.16.0
```

If Flatpak isn't installed:

```bash
sudo dnf install flatpak
```

---

# Why Login is Required for Red Hat Repository

Unlike Flathub, the Red Hat Flatpak repository is protected.

You must authenticate using your **Red Hat Customer Portal** credentials.

Login command:

```bash
podman login registry.redhat.io
```

Example:

```text
Username: your_username
Password: ********

Login Succeeded!
```

Why?

Because Red Hat verifies that you have an active subscription before allowing downloads.

---

# Save Authentication Permanently

By default, login credentials expire when the session ends.

To avoid logging in every time:

### User-specific

```bash
cp $XDG_RUNTIME_DIR/containers/auth.json \
$HOME/.config/flatpak/oci-auth.json
```

Only the current user can access the repository.

---

### System-wide

```bash
sudo cp $XDG_RUNTIME_DIR/containers/auth.json \
/etc/flatpak/oci-auth.json
```

Allow all users to read it:

```bash
sudo chmod 644 /etc/flatpak/oci-auth.json
```

Now every user can access the Red Hat Flatpak repository.

---

# View Configured Repositories

List all remotes:

```bash
flatpak remotes
```

Example:

```text
Name

rhel
```

Meaning:

Your system currently has one configured repository named **rhel**.

---

# Display Detailed Information

```bash
flatpak remotes -d
```

Example output:

```text
Name        rhel

Title       Red Hat Flatpak Repository

URL         oci+https://registry.redhat.io

Priority    1
```

This command provides:

* Repository name
* URL
* Collection ID
* Priority
* Options

---

# List Available Applications

View all applications:

```bash
flatpak remote-ls --app
```

Example:

```text
Thunderbird

Firefox

LibreOffice

GIMP
```

Notice the `--app` option.

Without it, Flatpak also displays **Runtimes**, which most users don't need to see.

---

# How Flatpak Downloads Applications

```text
User

     │

flatpak install

     │

Remote Repository

     │

Downloads

Application

+

Runtime

+

Dependencies

     │

Installed

Inside Sandbox
```

---

# Difference Between RHEL Repository and Flathub

| Feature                         | RHEL Repository | Flathub                     |
| ------------------------------- | --------------- | --------------------------- |
| Officially Supported by Red Hat | ✅ Yes           | ❌ No                        |
| Enterprise Ready                | ✅ Yes           | Community Supported         |
| Authentication Required         | ✅ Yes           | ❌ No                        |
| Number of Applications          | Moderate        | Very Large                  |
| Best for Production             | ✅ Yes           | Mostly Desktop/Personal Use |

---

# Real-Time Production Scenario

## Situation

You are a Linux Administrator managing RHEL 10 workstations.

A development team requests **Thunderbird** for secure email access.

### Steps

1. Verify Flatpak is installed.
2. Authenticate to the Red Hat registry using `podman login`.
3. Save the authentication token.
4. Confirm the **rhel** remote is configured.
5. List available applications with:

```bash
flatpak remote-ls --app
```

6. Verify Thunderbird is available.
7. Install it using Flatpak.
8. Test the application and inform the user.

### Result

* Application installed successfully.
* Supported Red Hat repository used.
* Secure authentication configured.
* Future updates managed through Flatpak.

---

# Key Takeaways

* A **Flatpak Remote** is a repository that stores Flatpak applications and runtimes.
* **RHEL 10** includes a default Red Hat Flatpak repository.
* Accessing the Red Hat repository requires authentication with `podman login`.
* Authentication tokens can be saved per-user or system-wide.
* Use `flatpak remotes` to view configured repositories.
* Use `flatpak remotes -d` to inspect repository details.
* Use `flatpak remote-ls --app` to browse available applications.
* **Flathub** offers the largest collection of Flatpak apps but is **not supported by Red Hat** for enterprise environments.

💡 **Exam Tip (RHCSA EX200):** Be familiar with `flatpak --version`, `flatpak remotes`, `flatpak remotes -d`, `flatpak remote-ls --app`, and the difference between official Red Hat repositories and community repositories like Flathub.
