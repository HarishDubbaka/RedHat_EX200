This is a **Guided Exercise for SFTP**. The main goal is to copy the `/etc/ssh` directory from **serverb → servera** using `sftp`.

## 🔹 What is happening?

```text
serverb                              servera
--------                             --------
/etc/ssh/                            /home/student/
    │                                    │
    │        SFTP: get -r                │
    └───────────────────────────────────►│
                                         │
                                  serverbackup1/
                                      └── ssh/
```

You are logged in to **servera**, but you connect to **serverb** using SFTP as `root`.

---

## Step 1 — Log in to servera

From the workstation:

```bash
ssh student@servera
```

You should get:

```text
[student@servera ~]$
```

### Why?

You need to perform the backup **on servera**, so the destination directory will be created there.

---

## Step 2 — Create backup directories

Run:

```bash
mkdir ~/serverbackup{1,2}
```

This is a Bash **brace expansion**.

It creates both:

```text
/home/student/serverbackup1
/home/student/serverbackup2
```

You can verify:

```bash
ls -ld ~/serverbackup*
```

Expected:

```text
/home/student/serverbackup1
/home/student/serverbackup2
```

---

# Step 3 — Connect to serverb using SFTP

Run:

```bash
sftp root@serverb
```

Enter:

```text
redhat
```

You should see:

```text
Connected to serverb.
sftp>
```

### Why `root`?

The exercise specifically says that only `root` can read all contents of:

```text
/etc/ssh
```

So we connect as:

```text
root@serverb
```

---

# Step 4 — Understand `lcd`

Now you're inside the SFTP session:

```text
sftp>
```

Run:

```bash
lcd /home/student/serverbackup1/
```

### What does `lcd` mean?

**`l` = local**

**`cd` = change directory**

Therefore:

```text
lcd = change LOCAL directory
```

This changes the destination on **servera**.

It does **not** change the directory on serverb.

You can verify:

```bash
sftp> lpwd
```

You should see:

```text
Local working directory: /home/student/serverbackup1
```

---

# Step 5 — Copy `/etc/ssh` from serverb

This is the most important command:

```bash
sftp> get -r /etc/ssh
```

Break it down:

```text
get
 ↓
Download from REMOTE → LOCAL
```

and:

```text
-r
 ↓
Recursive
```

So:

```bash
get -r /etc/ssh
```

means:

> **Recursively download the `/etc/ssh` directory from serverb to the current local directory on servera.**

The direction is:

```text
serverb                                  servera
--------                                 --------
/etc/ssh
   │
   │ get -r
   └───────────────────────────────► /home/student/serverbackup1/
```

The result becomes:

```text
/home/student/serverbackup1/
└── ssh/
    ├── moduli
    ├── ssh_config
    ├── sshd_config
    ├── ssh_host_rsa_key
    ├── ssh_host_rsa_key.pub
    ├── ssh_host_ecdsa_key
    ├── ssh_host_ecdsa_key.pub
    ├── ssh_host_ed25519_key
    ├── ssh_host_ed25519_key.pub
    ├── ssh_config.d/
    └── sshd_config.d/
```

### Why does it create `ssh`?

Because you're downloading the directory:

```text
/etc/ssh
```

into:

```text
/home/student/serverbackup1
```

So SFTP creates:

```text
/home/student/serverbackup1/ssh
```

---

# Step 6 — Exit SFTP

When the transfer is complete:

```bash
sftp> exit
```

You return to the normal shell:

```text
[student@servera ~]$
```

Remember:

```text
sftp> exit
       ↓
Normal Linux shell
```

---

# Step 7 — Verify the backup

Run:

```bash
ls -lR ~/serverbackup1
```

The `-R` means **recursive**.

It displays:

```text
serverbackup1
    ↓
ssh
    ↓
files + directories
    ↓
contents of ssh_config.d
    ↓
contents of sshd_config.d
```

You should see the copied SSH configuration files.

---

# 🧠 Most Important Part to Understand

There are **two machines** involved:

```text
                SFTP SESSION
                     │
          ┌──────────┴──────────┐
          │                     │
       SERVERB               SERVERA
       REMOTE                 LOCAL
          │                     │
     /etc/ssh            /home/student/
          │               serverbackup1/
          │                     │
          └──── get -r ─────────►
```

You are physically logged into:

```text
servera
```

but SFTP connects you to:

```text
serverb
```

Therefore:

```text
serverb = REMOTE
servera = LOCAL
```

---

# ⭐ Commands You Should Remember

| Command                           | Meaning                        |
| --------------------------------- | ------------------------------ |
| `ssh student@servera`             | SSH into servera               |
| `mkdir ~/serverbackup{1,2}`       | Create two directories         |
| `sftp root@serverb`               | Start SFTP to serverb          |
| `lcd /home/student/serverbackup1` | Change local directory         |
| `lpwd`                            | Show local directory           |
| `pwd`                             | Show remote directory          |
| `get /file`                       | Download file                  |
| `get -r /directory`               | Download directory recursively |
| `put file`                        | Upload file                    |
| `put -r directory`                | Upload directory recursively   |
| `exit`                            | Exit SFTP                      |
| `ls -lR`                          | Recursively list files         |

## 🔥 RHCSA Memory Trick

```text
SFTP

put  → Upload   → LOCAL  → REMOTE
get  → Download → REMOTE → LOCAL

lcd  → Change LOCAL directory
cd   → Change REMOTE directory

lpwd → Show LOCAL directory
pwd  → Show REMOTE directory

-r   → Recursive
```

### In this exercise:

```bash
sftp root@serverb
```

➡️ Connect to **serverb**

```bash
lcd /home/student/serverbackup1
```

➡️ Set destination on **servera**

```bash
get -r /etc/ssh
```

➡️ Copy `/etc/ssh` from **serverb → servera**

```bash
exit
```

➡️ Leave SFTP

```bash
ls -lR ~/serverbackup1
```

➡️ Verify the backup.
