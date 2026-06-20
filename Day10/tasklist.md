# 🐧 Linux Practice Lab: Shell Expansions, File Management & Links

## 🎯 Objective

Practice:

* Brace Expansion `{ }`
* Range Expansion `{1..N}`
* Wildcards `*`
* Character Matching `[ ]`
* Command Substitution `$( )`
* File and Directory Management
* Hard Links

---

## 1️⃣ Login to Server

```bash
ssh student@serverb
```

---

# 📁 Task 1: Create Project Plans

Create the `project-plans` directory and two project files.

```bash
mkdir -p ~/Documents/project-plans

touch ~/Documents/project-plans/season{1,2}-project-plan.odf
```

Verify:

```bash
ls -lR ~/Documents
```

---

# 📺 Task 2: Create TV Episode Files

Create 12 episode files for two seasons.

```bash
touch tv-season{1..2}-episode{1..6}.ogg
```

Verify:

```bash
ls tv*
```

---

# 📖 Task 3: Create Mystery Novel Chapters

Create eight chapter files.

```bash
touch mystery-chapter{1..8}.odf
```

Verify:

```bash
ls mystery*
```

---

# 📂 Task 4: Organize TV Episodes

Create season directories.

```bash
mkdir -p Videos/season{1..2}
```

Move episodes.

```bash
mv tv-season1* Videos/season1
mv tv-season2* Videos/season2
```

Verify:

```bash
ls -R Videos
```

---

# 📚 Task 5: Create Book Project Structure

Create directory hierarchy.

```bash
mkdir -p Documents/my-bestseller/chapters
```

Create additional folders.

```bash
mkdir Documents/my-bestseller/{editor,changes,vacation}
```

Verify:

```bash
ls -R Documents/my-bestseller
```

---

# 🚚 Task 6: Move Chapters into Chapters Directory

Go to chapters directory.

```bash
cd ~/Documents/my-bestseller/chapters
```

Move all chapter files.

```bash
mv ~/mystery-chapter* .
```

Verify:

```bash
ls
```

---

# ✍️ Task 7: Send Chapters to Editor

Move chapters 1 and 2.

```bash
mv mystery-chapter{1..2}.odf ../editor
```

Verify:

```bash
ls ../editor
```

---

# 🏖️ Task 8: Move Vacation Chapters

Move chapters 7 and 8.

```bash
mv mystery-chapter{7,8}.odf ../vacation
```

Verify:

```bash
ls ../vacation
```

---

# 🎬 Task 9: Copy TV Episodes to Vacation Folder

Go to Season 2.

```bash
cd ~/Videos/season2
```

Copy episode 1.

```bash
cp *episode1.ogg ~/Documents/my-bestseller/vacation
```

Change to vacation folder.

```bash
cd ~/Documents/my-bestseller/vacation
```

Verify:

```bash
ls
```

Return to previous directory.

```bash
cd -
```

Copy episode 2.

```bash
cp *episode2.ogg ~/Documents/my-bestseller/vacation
```

Return to vacation folder.

```bash
cd -
```

Verify:

```bash
ls
```

---

# 📝 Task 10: Create Chapter Copies for Changes

Go to project directory.

```bash
cd ~/Documents/my-bestseller
```

Copy chapters 5 and 6.

```bash
cp chapters/mystery-chapter[56].odf changes
```

Verify:

```bash
ls changes
```

---

# 📅 Task 11: Command Substitution Practice

Go to changes directory.

```bash
cd changes
```

Copy chapter 5 with today's date.

```bash
cp mystery-chapter5.odf mystery-chapter5-$(date +%F).odf
```

Copy chapter 5 with epoch timestamp.

```bash
cp mystery-chapter5.odf mystery-chapter5-$(date +%s).odf
```

Verify:

```bash
ls
```

---

# 🗑️ Task 12: Delete Changes Directory

Delete all files.

```bash
rm mystery*
```

Move to parent directory.

```bash
cd ..
```

Attempt removal (expected to fail).

```bash
rm changes
```

Remove correctly.

```bash
rmdir changes
```

Verify:

```bash
ls
```

---

# 🧹 Task 13: Delete Vacation Directory

```bash
rm -r vacation
```

Verify:

```bash
ls
```

Return home.

```bash
cd
```

---

# 🔗 Task 14: Create a Hard Link

Create links directory.

```bash
mkdir ~/Documents/links
```

Create hard link.

```bash
ln ~/Documents/project-plans/season2-project-plan.odf \
~/Documents/links/season2-project-plan.odf.link
```

Verify:

```bash
ls -li ~/Documents/project-plans
ls -li ~/Documents/links
```

Expected:

* Same inode number
* Link count = 2

---

# 🚪 Task 15: Logout

```bash
exit
```

---

# 🎓 Shell Expansion Cheat Sheet

| Expansion            | Example                      | Result                 |
| -------------------- | ---------------------------- | ---------------------- |
| Brace Expansion      | `{a,b,c}`                    | a b c                  |
| Numeric Range        | `{1..5}`                     | 1 2 3 4 5              |
| Multiple Expansion   | `season{1..2}-episode{1..3}` | Generates combinations |
| Wildcard             | `*.txt`                      | All txt files          |
| Character Match      | `[56]`                       | Matches 5 or 6         |
| Home Shortcut        | `~`                          | User home directory    |
| Previous Directory   | `cd -`                       | Last directory         |
| Command Substitution | `$(date +%F)`                | Current date           |

---

Happy Practicing for **RHCSA / EX200**! 💪🐧

