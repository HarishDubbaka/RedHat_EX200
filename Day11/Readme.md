# 🐧 Editing Text Files with Vim (RHCSA EX200 Notes)

## 🎯 Objective

Create, view, and edit text files from the command line using the **Vim Editor**.

---

# Why Learn Vim?

* Linux stores most configurations in **text files**.
* Vim is available on almost every Linux/Unix system.
* Useful when working on:

  * Remote servers (SSH)
  * Terminal-only environments
  * System configuration files

---

# Installing Vim

### Minimal Version

```bash
sudo dnf install vim-minimal
```

Open file:

```bash
vi filename
```

### Enhanced Version

```bash
sudo dnf install vim-enhanced
```

Open file:

```bash
vim filename
```

---

# Vim Modes

| Mode                  | Purpose                 |
| --------------------- | ----------------------- |
| Command Mode          | Navigation and commands |
| Insert Mode           | Type and edit text      |
| Visual Mode           | Select text             |
| Extended Command Mode | Save, quit, search      |

### Mode Switching

```text
Command Mode
      |
      | i
      ↓
Insert Mode
      |
      | Esc
      ↓
Command Mode
      |
      | v
      ↓
Visual Mode
      |
      | :
      ↓
Extended Command Mode
```

> 💡 Press `Esc` anytime to return safely to Command Mode.

---

# Basic Vim Commands

## Enter Insert Mode

```bash
i
```

## Append Text

```bash
a
```

Adds text after the cursor.

---

# Navigation Keys

| Key | Action     |
| --- | ---------- |
| h   | Move Left  |
| j   | Move Down  |
| k   | Move Up    |
| l   | Move Right |

---

# Editing Commands

| Command | Description      |
| ------- | ---------------- |
| x       | Delete character |
| u       | Undo last change |

---

# Save and Quit

| Command | Action              |
| ------- | ------------------- |
| :w      | Save file           |
| :wq     | Save and Quit       |
| :q!     | Quit without saving |

---

# Copy and Paste (Yank & Put)

### Copy Text

1. Enter Visual Mode:

   ```bash
   v
   ```
2. Select text.
3. Press:

   ```bash
   y
   ```

### Paste Text

```bash
p
```

---

# Visual Modes

| Key       | Mode           |
| --------- | -------------- |
| v         | Character Mode |
| Shift + V | Line Mode      |
| Ctrl + V  | Block Mode     |

### Character Mode

```bash
v
```

Select individual characters.

### Line Mode

```bash
V
```

Select complete lines.

### Block Mode

```bash
Ctrl + V
```

Select columns of text.

---

# Vim Configuration Files

| File       | Purpose                |
| ---------- | ---------------------- |
| /etc/vimrc | System-wide settings   |
| ~/.vimrc   | User-specific settings |

### Example ~/.vimrc

```bash
autocmd FileType yaml setlocal ts=2
set number
```

### What it Does

* YAML files use **2-space tabs**
* Displays **line numbers**

---

# RHCSA Exam Important Commands

```bash
i       # Insert mode
Esc     # Command mode
h j k l # Navigation
x       # Delete character
u       # Undo
a       # Append text
v       # Visual mode
y       # Copy (Yank)
p       # Paste
:w      # Save
:wq     # Save and Exit
:q!     # Exit without saving
```

## 🚀 RHCSA Exam Tip

**Remember this sequence:**

```text
i  → Type Text
Esc → Command Mode
:wq → Save & Exit
:q! → Exit Without Saving
u → Undo
```

> 🐧 "In Linux, if you can edit with Vim, you can survive on almost any server!" 😄

