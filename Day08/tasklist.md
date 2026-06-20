# 🐧 Linux File Management Practice Lab

## 🎯 Objective

Practice essential Linux file management commands:

* `mkdir`
* `touch`
* `mv`
* `cp`
* `cp -r`
* `rm -rf`
* `tree`

By the end of this lab, you will be able to create, organize, copy, move, and delete files and directories from the Linux command line.

---

# 🖥️ Lab Scenario

You are organizing media files on a Linux server.

Create directories for music, pictures, and videos, then organize files into folders. Later, create backup copies for friends and family, archive them into a work folder, and finally clean up the environment.

---

# 📋 Task 1: Create Media Directories

Create three directories in your home directory:

* Music
* Pictures
* Videos

### Command

```bash
mkdir Music Pictures Videos
```

### Verify

```bash
ls -l
```

---

# 📋 Task 2: Create Practice Files

Create the following files:

### Music Files

```bash
touch song1.mp3 song2.mp3
```

### Picture Files

```bash
touch snap1.jpg snap2.jpg
```

### Video Files

```bash
touch film1.avi film2.avi
```

### Verify

```bash
ls -l
```

Expected files:

```text
song1.mp3
song2.mp3
snap1.jpg
snap2.jpg
film1.avi
film2.avi
```

---

# 📋 Task 3: Organize Files

Move files into their appropriate directories.

### Move Music Files

```bash
mv song1.mp3 song2.mp3 Music
```

### Move Picture Files

```bash
mv snap1.jpg snap2.jpg Pictures
```

### Move Video Files

```bash
mv film1.avi film2.avi Videos
```

### Verify

```bash
ls -l Music Pictures Videos
```

---

# 📋 Task 4: Create Project Directories

Create three new directories using a single command:

* friends
* family
* work

### Command

```bash
mkdir friends family work
```

### Verify

```bash
ls -l
```

---

# 📋 Task 5: Copy Files to Friends Directory

Copy all files containing the number **1** into the `friends` directory.

### Navigate

```bash
cd friends
```

### Copy Files

```bash
cp ~/Music/song1.mp3 ~/Pictures/snap1.jpg ~/Videos/film1.avi .
```

### Verify

```bash
ls -l
```

Expected:

```text
song1.mp3
snap1.jpg
film1.avi
```

---

# 📋 Task 6: Copy Files to Family Directory

Navigate to the family directory.

```bash
cd ../family
```

Copy all files containing the number **2**.

```bash
cp ~/Music/song2.mp3 ~/Pictures/snap2.jpg ~/Videos/film2.avi .
```

### Verify

```bash
ls -l
```

Expected:

```text
song2.mp3
snap2.jpg
film2.avi
```

---

# 📋 Task 7: Create Work Backup

Navigate to the work directory.

```bash
cd ../work
```

Copy the entire family and friends directories.

```bash
cp -r ~/family ~/friends .
```

### Verify

```bash
ls -l
```

Expected:

```text
family/
friends/
```

---

# 📋 Task 8: Verify Directory Structure

Return to your home directory.

```bash
cd ~
```

Display the complete directory tree.

```bash
tree
```

Expected structure:

```text
.
├── Music
│   ├── song1.mp3
│   └── song2.mp3
├── Pictures
│   ├── snap1.jpg
│   └── snap2.jpg
├── Videos
│   ├── film1.avi
│   └── film2.avi
├── family
│   ├── film2.avi
│   ├── snap2.jpg
│   └── song2.mp3
├── friends
│   ├── film1.avi
│   ├── snap1.jpg
│   └── song1.mp3
└── work
    ├── family
    │   ├── film2.avi
    │   ├── snap2.jpg
    │   └── song2.mp3
    └── friends
        ├── film1.avi
        ├── snap1.jpg
        └── song1.mp3
```

---

# 📋 Task 9: Clean Up the Environment

Delete the following directories and all their contents:

* family
* friends
* work

### Command

```bash
rm -rf family/ friends/ work/
```

### Verify

```bash
ls -l
```

Expected:

```text
Music/
Pictures/
Videos/
```

---

# 🔍 Challenge Tasks

Try completing the following without looking at the solution:

### Challenge 1

Create a directory structure:

```text
Projects/
├── Linux
├── AWS
└── DevOps
```

### Challenge 2

Create files:

```text
linux_notes.txt
aws_notes.txt
devops_notes.txt
```

Move each file to its corresponding directory.

### Challenge 3

Create a backup directory and copy all project folders into it.

### Challenge 4

Delete the backup directory recursively.

---

# 📝 Commands Used in This Lab

| Command  | Description                    |
| -------- | ------------------------------ |
| `mkdir`  | Create directories             |
| `touch`  | Create empty files             |
| `ls`     | List files and directories     |
| `mv`     | Move files and directories     |
| `cp`     | Copy files                     |
| `cp -r`  | Copy directories recursively   |
| `tree`   | Display directory structure    |
| `rm -rf` | Delete directories recursively |
| `cd`     | Change directory               |

---

# 💡 Key Learning Points

✅ Create multiple directories with a single command

✅ Move files between directories

✅ Copy files from multiple source locations

✅ Copy entire directory structures recursively

✅ Verify directory layouts using `tree`

✅ Safely remove directories using `rm -rf`

---

Mastering Linux file management is one of the most important skills for Linux Administrators, DevOps Engineers, Cloud Engineers.
