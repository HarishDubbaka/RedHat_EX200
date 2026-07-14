# 🐧 Managing Flatpak Remote Repositories

## 🎯 Objective

After completing this guide, you will be able to:

- Add a new Flatpak remote repository
- Configure repositories for system-wide or user-only access
- View configured repositories
- List applications available in a repository
- Remove a Flatpak remote repository
- Disable and enable repositories without uninstalling applications

---

# 📚 What is a Flatpak Remote Repository?

A **Flatpak Remote Repository (Remote)** is a location where Flatpak stores applications and runtimes.

Think of it as an **App Store for Linux**.

Examples include:

- Red Hat Flatpak Repository
- Fedora Flatpak Repository
- Flathub

Without a remote repository, Flatpak has nowhere to download applications.

---

# 🏗️ Why Add Another Remote Repository?

By default, RHEL uses the **Red Hat Flatpak Repository**.

However, you may want to add additional repositories to access more applications.

Common repositories include:

| Repository | Description |
|------------|-------------|
| **Red Hat** | Official enterprise repository |
| **Fedora** | Community-maintained repository |
| **Flathub** | Largest collection of Flatpak applications |

---

# ➕ Adding a Flatpak Remote Repository

Use the `flatpak remote-add` command.

## Syntax

```bash
flatpak remote-add [OPTIONS] <repository-name> <repository-url>
```

### Options

| Option | Description |
|---------|-------------|
| `--if-not-exists` | Prevents overwriting an existing repository |
| `--user` | Adds the repository only for the current user |
| `--system` | Adds the repository system-wide (default) |

---

# Example 1: Add the Fedora Repository

The Fedora Project provides a Flatpak repository that can be used by many Linux distributions.

```bash
flatpak remote-add --if-not-exists fedora \
oci+https://registry.fedoraproject.org
```

### Explanation

| Part | Description |
|------|-------------|
| `remote-add` | Adds a new repository |
| `--if-not-exists` | Avoids duplicate entries |
| `fedora` | Local repository name |
| `oci+https://registry.fedoraproject.org` | Fedora repository URL |

---

# Verify the Repository

List configured repositories.

```bash
flatpak remotes
```

Example output:

```text
Name      Options

fedora    system,oci
rhel      system,oci,no-gpg-verify
```

This confirms that both repositories are available.

---

# 🌍 System Repository vs 👤 User Repository

Flatpak supports two installation modes.

## 1. System Mode (Default)

Available to **all users** on the system.

```bash
flatpak remote-add --if-not-exists fedora \
oci+https://registry.fedoraproject.org
```

Equivalent to:

```bash
flatpak remote-add --system ...
```

---

## 2. User Mode

Available **only to the current user**.

Example:

```bash
flatpak remote-add --if-not-exists --user \
flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

Now only the logged-in user can install applications from Flathub.

---

# View System Repositories

Display repositories configured for all users.

```bash
flatpak remotes --system
```

Example:

```text
Name      Options

fedora    oci
rhel      oci,no-gpg-verify
```

---

# View User Repositories

Display repositories configured only for the current user.

```bash
flatpak remotes --user
```

Example:

```text
Name

flathub
```

---

# List Applications Available in a Repository

Use the `remote-ls` command.

```bash
flatpak remote-ls fedora --app
```

Example output:

```text
Dialect
Multiplication Puzzle
pw3270
Dconf Editor
AbiWord
ConvertAll
...
```

### Explanation

| Command | Description |
|---------|-------------|
| `remote-ls` | Lists repository contents |
| `fedora` | Repository name |
| `--app` | Show only applications |

Without `--app`, Flatpak also lists runtimes.

---

# 🗑️ Removing a Flatpak Remote Repository

Use:

```bash
flatpak remote-delete fedora
```

Example:

```text
The following refs are installed from remote 'fedora':

runtime/org.fedoraproject.Platform
runtime/org.fedoraproject.Platform.Locale
app/org.filezillaproject.Filezilla

Remove them? [y/n]:
```

If you enter:

```text
y
```

Flatpak removes:

- Repository
- Applications
- Runtimes

⚠️ **Important:** Removing a repository also removes applications installed from that repository.

---

# 🚫 Disable a Repository Instead of Removing It

Sometimes you don't want to uninstall applications.

Instead, simply disable the repository.

```bash
flatpak remote-modify --disable fedora
```

Benefits:

- Applications continue to work
- Repository becomes unavailable for new installations
- No software is removed

---

# ✅ Enable the Repository Again

Later, if needed:

```bash
flatpak remote-modify --enable fedora
```

Now the repository is active again.

---

# Remove vs Disable

| Feature | Remove Repository | Disable Repository |
|----------|-------------------|--------------------|
| Repository removed | ✅ Yes | ❌ No |
| Applications removed | ✅ Yes | ❌ No |
| Runtimes removed | ✅ Yes | ❌ No |
| Can install new apps | ❌ No | ❌ No |
| Can enable later | ❌ Must add again | ✅ Yes |

---

# 💼 Real-Time Production Scenario

## Scenario

You are a Linux Administrator managing multiple RHEL workstations.

The development team requests **Visual Studio Code**, which is available in the Fedora Flatpak repository.

### Steps

1. Add the Fedora repository.

```bash
flatpak remote-add --if-not-exists fedora \
oci+https://registry.fedoraproject.org
```

2. Verify the repository.

```bash
flatpak remotes
```

3. Search for available applications.

```bash
flatpak remote-ls fedora --app
```

4. Install the required application.

5. Later, the Fedora repository is no longer needed.

Instead of deleting it and uninstalling applications, disable it.

```bash
flatpak remote-modify --disable fedora
```

If future updates are required:

```bash
flatpak remote-modify --enable fedora
```

---

# 📝 Summary

| Command | Purpose |
|----------|---------|
| `flatpak remote-add` | Add a repository |
| `flatpak remotes` | List configured repositories |
| `flatpak remotes --system` | Show system repositories |
| `flatpak remotes --user` | Show user repositories |
| `flatpak remote-ls` | List applications in a repository |
| `flatpak remote-delete` | Remove a repository and its applications |
| `flatpak remote-modify --disable` | Disable a repository |
| `flatpak remote-modify --enable` | Re-enable a repository |

---

# 🎯 Key Takeaways

- A **Flatpak Remote** is a repository containing applications and runtimes.
- Use **`flatpak remote-add`** to add new repositories.
- Use **`--system`** for all users and **`--user`** for the current user.
- Use **`flatpak remote-ls`** to browse available applications.
- **Removing** a repository also removes its installed applications and runtimes.
- **Disabling** a repository is safer when you want to keep installed applications while preventing new installations.

> 💡 **RHCSA EX200 Tip:** Be familiar with `flatpak remote-add`, `flatpak remotes`, `flatpak remote-ls`, `flatpak remote-delete`, and `flatpak remote-modify` commands, along with the difference between **system** and **user** repositories.
````
