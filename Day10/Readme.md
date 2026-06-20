# 🐧 Linux Shell Expansions & Pattern Matching – Quick Reference Guide

## 🎯 Objective

Learn how Bash expands commands before execution and how to use shell expansions to efficiently work with multiple files.

---

# 🔄 What Are Shell Expansions?

Before executing a command, the Bash shell processes it through several expansions:

| Expansion Type                | Purpose                                    |
| ----------------------------- | ------------------------------------------ |
| Pathname Expansion (Globbing) | Match files and directories using patterns |
| Brace Expansion               | Generate multiple strings                  |
| Tilde Expansion               | Expand to user home directories            |
| Variable Expansion            | Replace variables with their values        |
| Command Substitution          | Replace commands with their output         |

---

# 1️⃣ Pathname Expansion (Globbing)

Pathname expansion uses special wildcard characters to match files and directories.

## Common Wildcards

| Pattern       | Matches                         |
| ------------- | ------------------------------- |
| `*`           | Zero or more characters         |
| `?`           | Exactly one character           |
| `[abc]`       | Any one character: a, b, or c   |
| `[!abc]`      | Any character except a, b, or c |
| `[^abc]`      | Any character except a, b, or c |
| `[[:alpha:]]` | Alphabetic character            |
| `[[:lower:]]` | Lowercase letter                |
| `[[:upper:]]` | Uppercase letter                |
| `[[:alnum:]]` | Letter or digit                 |
| `[[:digit:]]` | Number 0-9                      |
| `[[:space:]]` | Whitespace character            |

---

## Practice Setup

```bash
mkdir glob
cd glob

touch alfa bravo charlie delta echo able baker cast dog easy
```

```bash
ls
```

Output:

```text
able  alfa  baker  bravo  cast  charlie  delta  dog  easy  echo
```

---

## Examples

### Match files starting with "a"

```bash
ls a*
```

Output:

```text
able  alfa
```

---

### Match files containing "a"

```bash
ls *a*
```

Output:

```text
able  alfa  baker  bravo  cast  charlie  delta  easy
```

---

### Match files starting with "a" or "c"

```bash
ls [ac]*
```

Output:

```text
able  alfa  cast  charlie
```

---

### Match exactly 4-character filenames

```bash
ls ????
```

Output:

```text
able  cast  easy  echo
```

---

### Match exactly 5-character filenames

```bash
ls ?????
```

Output:

```text
baker  bravo  delta
```

---

# 2️⃣ Brace Expansion

Brace expansion generates multiple strings automatically.

## Examples

### Create multiple log filenames

```bash
echo {Sunday,Monday,Tuesday,Wednesday}.log
```

Output:

```text
Sunday.log Monday.log Tuesday.log Wednesday.log
```

---

### Numeric sequence

```bash
echo file{1..3}.txt
```

Output:

```text
file1.txt file2.txt file3.txt
```

---

### Alphabetic sequence

```bash
echo file{a..c}.txt
```

Output:

```text
filea.txt fileb.txt filec.txt
```

---

### Multiple combinations

```bash
echo file{a,b}{1,2}.txt
```

Output:

```text
filea1.txt filea2.txt fileb1.txt fileb2.txt
```

---

### Nested braces

```bash
echo file{a{1,2},b,c}.txt
```

Output:

```text
filea1.txt filea2.txt fileb.txt filec.txt
```

---

## Practical Example

Create multiple directories at once:

```bash
mkdir RHEL{8,9,10}
```

Result:

```text
RHEL8
RHEL9
RHEL10
```

---

# 3️⃣ Tilde Expansion (~)

The tilde (`~`) represents a user's home directory.

## Current User Home Directory

```bash
echo ~
```

Output:

```text
/home/user
```

---

## Another User's Home Directory

```bash
echo ~devops
```

Output:

```text
/home/devops
```

---

## Access Files from Anywhere

```bash
ls ~/scripts/run-app.sh
```

Equivalent to:

```bash
ls /home/user/scripts/run-app.sh
```

---

## Non-Existent User Example

```bash
ls ~dev1/scripts/run-app.sh
```

Output:

```text
ls: cannot access '~dev1/scripts/run-app.sh': No such file or directory
```

---

# 4️⃣ Variable Expansion

Variables store values that can be reused.

## Create a Variable

```bash
USERNAME=operator
```

---

## Display Variable Value

```bash
echo $USERNAME
```

Output:

```text
operator
```

---

## Recommended Syntax

```bash
echo ${USERNAME}
```

Output:

```text
operator
```

---

## Variable Naming Rules

✅ Allowed:

```text
USERNAME
user_name
USER1
```

❌ Not Allowed:

```text
1USER
user-name
user name
```

---

# 5️⃣ Command Substitution

Command substitution replaces a command with its output.

## Basic Example

```bash
echo Today is $(date +%A).
```

Output:

```text
Today is Wednesday.
```

---

## Multiple Command Substitutions

```bash
echo The time is $(date +%M) minutes past $(date +%I%p).
```

Output:

```text
The time is 26 minutes past 11AM.
```

---

## Real-World Example

Create a backup file with today's date:

```bash
cp app.conf app.conf.$(date +%F).bak
```

Result:

```text
app.conf.2026-06-20.bak
```

---

# 🧪 Practice Challenges

### Challenge 1

List all files starting with `b`.

```bash
ls b*
```

---

### Challenge 2

List all files with exactly 4 characters.

```bash
ls ????
```

---

### Challenge 3

Create directories:

```bash
mkdir Project{1..5}
```

---

### Challenge 4

Display your home directory.

```bash
echo ~
```

---

### Challenge 5

Create a variable and print it.

```bash
COURSE=RHCSA
echo $COURSE
```

---

# 💡 Memory Trick

```text
*  → Many characters
?  → One character
{} → Generate combinations
~  → Home directory
$VAR → Variable value
$(cmd) → Command output
```

---

# 😂 Linux Admin Joke

> "Why type 100 filenames manually when `*` can do the overtime for free?"
>
> — Every Linux Administrator Ever 🐧🚀
