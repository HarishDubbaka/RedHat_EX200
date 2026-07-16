# Installing, Updating, and Managing Flatpak Objects (Branches, Updates & Removal)

Flatpak allows you to install applications and runtimes from **different branches**, similar to **Git branches**. Each branch represents a specific version or release channel of an application.

Examples of branches:

* **stable** → Recommended production version
* **beta** → Testing version with upcoming features
* **nightly** → Daily development builds
* **Version branches** → `1.0`, `1.1`, `el8`, `el9`, `el10`

---

## Flatpak Branch Concept

```text
                  Remote Repository
                        │
      ┌─────────────────┼──────────────────┐
      │                 │                  │
   Stable            Beta             Nightly
      │                 │                  │
 Latest Stable     Testing Build     Daily Build
```

> Similar to Git, once you install from a branch, Flatpak continues updating only within that branch.

---

## 1. Check Available Branches

Before installing, you can check which branches exist.

### Syntax

```bash
flatpak remote-info <remote> <application>
```

### Example

```bash
flatpak remote-info rhel com.redhat.Platform
```

### Output

```text
error: Multiple branches available for com.redhat.Platform,
you must specify one of:

com.redhat.Platform/x86_64/el10
com.redhat.Platform/x86_64/el9
com.redhat.Platform/x86_64/el8
```

### Explanation

This tells you that the runtime exists in **three different branches**.

| Branch | Meaning         |
| ------ | --------------- |
| el8    | RHEL 8 runtime  |
| el9    | RHEL 9 runtime  |
| el10   | RHEL 10 runtime |

---

## 2. Install from a Specific Branch

Once you know the branch, install using the **identifier triple**.

### Syntax

```bash
flatpak install <application>/<architecture>/<branch>
```

### Example

```bash
flatpak install com.redhat.Platform/x86_64/el8
```

Example:

```text
Installing...

com.redhat.Platform

Branch:
el8

Architecture:
x86_64

Installation complete.
```

---

## Installation Flow

```text
Check Branches
       │
       ▼
flatpak remote-info
       │
       ▼
Choose Branch
       │
       ▼
flatpak install app/x86_64/stable
```

---

# 3. Updating Flatpak Objects

Flatpak updates installed applications to the newest release **within the same branch**.

### Update Everything

```bash
flatpak update
```

Example

```text
Looking for updates...

Updating...

Done.
```

---

### Update One Application

```bash
flatpak update com.redhat.Platform
```

Example

```text
Looking for updates...

Nothing to do.
```

This means:

* already running latest version
* no updates available in current branch

---

## Important Note

Suppose you installed

```text
Firefox Stable
```

Later:

```text
Firefox Beta
```

becomes available.

Running

```bash
flatpak update
```

**will NOT upgrade Stable → Beta**

It updates only:

```text
Stable → New Stable
```

Exactly like Git:

```text
Git Branch

main
 ├── commit
 ├── commit
 └── commit

You remain on main
```

Flatpak behaves the same way.

---

## Update Workflow

```text
Installed App
      │
      ▼
Current Branch
      │
      ▼
Check Remote
      │
      ▼
New version available?
      │
 ┌────┴────┐
 │         │
Yes        No
 │         │
 ▼         ▼
Update   Nothing
```

---

# 4. Prevent Updates (Mask)

Sometimes you do **not** want an application or runtime to receive updates.

Flatpak provides the **mask** feature.

---

## Mask an Application

```bash
flatpak mask com.redhat.Platform
```

Now Flatpak ignores updates for that object.

Example

```text
Object masked.

Future updates skipped.
```

---

## Remove Mask

```bash
flatpak mask --remove com.redhat.Platform
```

Example

```text
Mask removed.
```

Now updates resume normally.

---

## Mask Workflow

```text
Application
      │
      ▼
flatpak mask
      │
      ▼
No Future Updates
      │
      ▼
flatpak mask --remove
      │
      ▼
Updates Enabled
```

---

# 5. Uninstall Flatpak Objects

Use:

```bash
flatpak uninstall
```

---

## Remove an Application

```bash
flatpak uninstall thunderbird
```

Example

```text
Found installed ref

org.mozilla.Thunderbird

Proceed? Y
```

---

## Remove Application + User Data

Normally Flatpak removes only the application.

User data remains in

```text
~/.var/app/
```

To remove everything:

```bash
flatpak uninstall --delete-data thunderbird
```

Example

```text
Delete data for org.mozilla.Thunderbird?

y
```

Now both:

* application
* configuration
* cache
* user files

are deleted.

---

## Runtime Dependency Warning

Applications depend on runtimes.

Example:

```text
Firefox
        │
        ▼
Freedesktop Runtime
```

If you try removing the runtime:

```bash
flatpak uninstall org.freedesktop.Platform
```

Flatpak reports:

```text
Runtime required by installed applications.
```

You can force removal:

```bash
flatpak uninstall --force-remove org.freedesktop.Platform
```

Use this only when you're certain it won't break installed applications.

---

## Uninstall Workflow

```text
Installed Application
          │
          ▼
flatpak uninstall
          │
          ▼
Application Removed
          │
          ▼
Delete Data?
     │
 ┌───┴────┐
 │        │
Yes      No
 │        │
 ▼        ▼
Everything   Config remains
```

---

# 6. Managing Flatpak with GNOME Software

If you prefer a graphical interface, **GNOME Software** provides full Flatpak management.

![Image](https://images.openai.com/static-rsc-4/KZlOIJh5CX5h0z8TvRCMDIEjJXoKXbIweZ6d4b49Vr5v4fq8Yi5qIAKVtE6fbuqtxAfp2LonCHUcGHfsXpH9GyY8y9HLFpbV5GtRS1DmDTrXkN-GR9S1yTPPICM3o2SEMkL34Ydp3_ZN9qWHTxb8G9acwaliQ_icNYBmCYGihP1Pe4K3Hm5cD2x4PW9amFok?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/0dPoHOXXUDLszwHBPKh-Ns34wavqfe6z4COxJBg5bAyexCKManUl7r6yBmVXVPmxm-KFxgpAe6N0-EE_7DvWCxKkq62y1ppi3i218QpHR1Pi_RS4tDTF-cwk43NNF_gsDj6Ayy67KPYC_gGV9-uQWLxeJbpX72hX5XCF_whWnB0kexw0YIYDR3syrPVAJmoj?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/Bhmhxy52xjuyy_xl3FlbXxlLn0CvoZzKZripuXOlVg9O2MxHkSXMkSTTJoDWM7BQRiUOh92APUM5PZmd1DsFNOxmfrpfLH3HINKY-yW04n3a-Dsb5ZQEsJaLDcszcODIig1VbHQCY98T0xQl3S57zVKdMGbXOsPuVaJRZzi9IB94BJ_dq-pFg4hDkglNwXMl?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/LENi-JhD9veNUMDu71vqYRfkAqgrQiFPBE3snXXKf1xXK0sihZcmNiCyNHEdmkeFBa0GFI02lRtnYQZGaHT9wAOkwSXR3AqTC-aNSwB5jorZRoaERSn9XBjOxdbi-E1QRuaSGgu6ykfthEgHaiBB7g7ZTpzm7mn3666O2sureO--WlMZAnAL7dvbKM7IzRK8?purpose=fullsize)

Launch it by:

* Opening **Activities → Software**
* Using **Show Applications → Software**
* Running:

```bash
gnome-software
```

---

## What You Can Do

* Search applications
* Install Flatpak apps
* Install DNF packages
* Update applications
* Remove applications
* Browse by category
* View application screenshots and descriptions

GNOME Software seamlessly integrates **DNF packages** and **Flatpak applications**, so both appear together in search results.

---

## Installing Through GNOME Software

1. Open **Software**
2. Search for an application.
3. Select the application.
4. If multiple branches are available, choose the desired branch.
5. Click **Install**.
6. Dependencies are resolved automatically.

---

## Removing Through GNOME Software

1. Open **Installed**.
2. Select the application.
3. Click **Uninstall**.

---

# Common Flatpak Commands

| Task                      | Command                                          |
| ------------------------- | ------------------------------------------------ |
| Search applications       | `flatpak search firefox`                         |
| List repositories         | `flatpak remotes`                                |
| List installed apps       | `flatpak list`                                   |
| Show application info     | `flatpak info firefox`                           |
| View available branches   | `flatpak remote-info rhel com.redhat.Platform`   |
| Install application       | `flatpak install flathub org.mozilla.Firefox`    |
| Install a specific branch | `flatpak install com.redhat.Platform/x86_64/el8` |
| Update all objects        | `flatpak update`                                 |
| Update one object         | `flatpak update <app>`                           |
| Mask updates              | `flatpak mask <app>`                             |
| Remove mask               | `flatpak mask --remove <app>`                    |
| Uninstall application     | `flatpak uninstall <app>`                        |
| Uninstall and delete data | `flatpak uninstall --delete-data <app>`          |

---

# Directory Locations

| Location                  | Purpose                                     |
| ------------------------- | ------------------------------------------- |
| `/var/lib/flatpak`        | System-wide Flatpak installations           |
| `~/.local/share/flatpak`  | User-specific Flatpak installations         |
| `~/.var/app/`             | Per-application user data and configuration |
| `/etc/flatpak/remotes.d/` | Flatpak repository configuration            |

---

# Summary

* **Branches** let you choose specific versions or release channels (Stable, Beta, Nightly, or version-specific).
* Use **`flatpak remote-info`** to discover available branches.
* Install a specific branch with the **identifier triple** (`app/architecture/branch`).
* **`flatpak update`** only updates within the currently installed branch.
* **`flatpak mask`** prevents selected applications or runtimes from receiving updates until unmasked.
* **`flatpak uninstall`** removes applications, while **`--delete-data`** also deletes user data stored in `~/.var/app/`.
* **GNOME Software** offers an easy graphical interface to search, install, update, and remove both Flatpak and DNF applications.
