# SSH Host Key Management — Clear Explanation

## 1. First understand the problem

When you connect:

```bash
ssh user@hostb
```

how does your client know that **hostb is really hostb**?

Imagine:

```text
Your Server                         Remote Server
SSH Client  --------------------->  SSH Server
              "Are you really hostb?"
```

The SSH server has a **host key**.

The client remembers that host key and uses it to identify the server on future connections.

### Why is this important?

Without host-key verification, an attacker could potentially pretend to be your server.

For example:

```text
You
 |
 | ssh user@serverA
 |
 v
Attacker's Server
 |
 v
Real Server
```

This is related to a **Man-in-the-Middle (MITM)** attack.

SSH host-key checking helps prevent this.

---

# 2. What is an SSH Host Key?

An SSH server has a pair of keys:

```text
Private Key              Public Key
     |                       |
     |                       |
ssh_host_ed25519_key   ssh_host_ed25519_key.pub
```

The **private key** stays on the server.

The **public key** can be shared with clients.

On the server, these keys are normally stored under:

```bash
/etc/ssh/
```

For example:

```text
/etc/ssh/ssh_host_ed25519_key
/etc/ssh/ssh_host_ed25519_key.pub

/etc/ssh/ssh_host_ecdsa_key
/etc/ssh/ssh_host_ecdsa_key.pub

/etc/ssh/ssh_host_rsa_key
/etc/ssh/ssh_host_rsa_key.pub
```

Your source notes show these three host-key algorithms being used by default on RHEL 10. 

---

# 3. What happens during the first SSH connection?

Suppose you connect to `hostb` for the first time:

```bash
ssh hostb
```

The client doesn't know hostb yet.

SSH displays something like:

```text
The authenticity of host 'hostb' can't be established.

ED25519 key fingerprint is
SHA256:Qit7NeebQCgP2otXSaVXbw1rGmUVMlybXyV68qaND/M.

Are you sure you want to continue connecting?
```

This means:

> "I don't have this server's host key stored yet. Do you trust this key?"

If you verify the fingerprint and answer:

```text
yes
```

SSH stores the host key in:

```bash
~/.ssh/known_hosts
```

Your source demonstrates this exact first-connection flow. 

---

# 4. What is a Fingerprint?

This is one of the most important concepts.

A server's public key is long and difficult for humans to compare.

For example:

```text
Very-long-public-key....................
.........................................
.........................................
```

So SSH creates a short **fingerprint** from the public key.

Example:

```text
SHA256:Qit7NeebQCgP2otXSaVXbw1rGmUVMlybXyV68qaND/M
```

Think of it like a **digital ID card** for the server.

### Simple analogy

```text
Server Public Key
       ↓
   Hash function
       ↓
   Fingerprint
       ↓
SHA256:ABC123...
```

The fingerprint makes it easier for an administrator to verify that the server is genuine. 

---

# 5. How do I check the server fingerprint?

Log in locally to the SSH server and run:

```bash
ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key.pub
```

Example:

```text
256 SHA256:Qit7NeebQCgP2otXSaVXbw1rGmUVMlybXyV68qaND/M root@hostb (ED25519)
```

The important part is:

```text
SHA256:Qit7NeebQCgP2otXSaVXbw1rGmUVMlybXyV68qaND/M
```

Your source recommends comparing this trusted fingerprint with what the SSH client sees during the connection. 

---

# 6. Very Important: Don't blindly type `yes`

Suppose you see:

```text
ED25519 key fingerprint is
SHA256:ABC123...
```

Before typing:

```text
yes
```

you should ideally compare it with a **trusted fingerprint** obtained through another communication method.

For example:

```text
Server administrator
       |
       | Provides fingerprint
       ↓
SHA256:ABC123...
       |
       ↓
Your SSH client
       |
       ↓
SHA256:ABC123...
```

### Match

```text
Trusted: SHA256:ABC123
Client:  SHA256:ABC123

             ↓

          SAFE TO
          CONTINUE
```

### Doesn't match

```text
Trusted: SHA256:ABC123
Client:  SHA256:XYZ789

             ↓

          STOP
```

A mismatch can indicate a security problem, although legitimate server changes can also cause it. 

---

# 7. What is `StrictHostKeyChecking`?

SSH has a configuration parameter called:

```text
StrictHostKeyChecking
```

It controls how SSH behaves when the server's host key is unknown or has changed.

You can configure it in SSH configuration files, or temporarily specify it directly in the command.

Example:

```bash
ssh -o StrictHostKeyChecking=accept-new hostb
```

The command-line option takes precedence over settings in:

```text
~/.ssh/config
```

and:

```text
/etc/ssh/ssh_config
```

So you can temporarily change the behavior for **one SSH connection**. 

---

# 8. What does `accept-new` mean?

Example:

```bash
ssh -o StrictHostKeyChecking=accept-new hostb
```

If `hostb` is completely new, SSH can automatically accept and save its host key.

You may see:

```text
Warning: Permanently added 'hostb' (ED25519)
to the list of known hosts.
```

However, your source specifically warns that automatically accepting keys should generally be avoided unless you are certain about the server's authenticity. 

---

# 9. Where does the client store host keys?

There are two important locations.

## User-specific

```bash
~/.ssh/known_hosts
```

This belongs to the individual user.

Example:

```text
/home/harish/.ssh/known_hosts
```

## System-wide

```bash
/etc/ssh/ssh_known_hosts
```

This can contain trusted host keys for **all users on the system**.

Your source explains that the system-wide file is not normally present by default and must be created/populated by an administrator. 

### Easy way to remember

```text
/etc/ssh/ssh_known_hosts
        ↓
   ALL USERS

~/.ssh/known_hosts
        ↓
   ONE USER
```

---

# 10. What happens if the server's host key changes?

This is extremely important in real-world administration.

Suppose today:

```text
Server host key
SHA256:ABC123
```

Your client stores it:

```text
~/.ssh/known_hosts
```

Later, the server's host key changes:

```text
Old:
SHA256:ABC123

New:
SHA256:XYZ789
```

When you connect:

```bash
ssh user@hostb
```

SSH sees:

```text
Expected:
SHA256:ABC123

Received:
SHA256:XYZ789
```

So SSH warns:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

and may report:

```text
Host key verification failed.
```

Your source notes show this exact warning and explain that it can happen after legitimate host-key changes, while also protecting against a possible MITM attack. 

---

# 11. Why would a host key legitimately change?

One common reason is **regenerating the SSH host keys**.

For example, when creating a new server image, you may want each server instance to have unique host keys.

On the server:

```bash
rm -f /etc/ssh/ssh_host_*
```

Then:

```bash
systemctl restart sshd.service
```

The SSH service detects that the host keys are missing and regenerates them. 

### Important

Do this carefully on production systems because clients that previously trusted the old keys will see a host-key mismatch.

---

# 12. How do I remove the old key?

Instead of manually editing:

```bash
~/.ssh/known_hosts
```

you can use:

```bash
ssh-keygen -R hostb
```

Example:

```bash
ssh-keygen -R hostb
```

SSH removes the matching entry from:

```text
~/.ssh/known_hosts
```

The source notes that this is the safer way to update/remove a changed host entry. 

For the system-wide file:

```bash
ssh-keygen -R hostb -f /etc/ssh/ssh_known_hosts
```

This removes the matching entry from the system-wide known-hosts file. 

---

# 13. What is `ssh-keyscan`?

`ssh-keyscan` is used to **collect public host keys** from remote servers.

Example:

```bash
ssh-keyscan hostb >> ~/.ssh/known_hosts
```

This adds the retrieved key information to:

```text
~/.ssh/known_hosts
```

It's particularly useful for:

* Automation
* Scripts
* Provisioning
* Preparing known-hosts files

Your source specifically discusses using it to populate personal or system-wide known-hosts files. 

---

# 14. But be careful with `ssh-keyscan`

This is a very important security point.

If you do:

```bash
ssh-keyscan hostb >> ~/.ssh/known_hosts
```

you are collecting the key that the remote server presents **at that moment**.

If an attacker is pretending to be `hostb`, you could accidentally save the attacker's key.

Therefore:

```text
ssh-keyscan
     ↓
Get key
     ↓
Verify fingerprint
     ↓
Add to known_hosts
```

is safer than blindly trusting the result.

Your source explicitly warns that using `ssh-keyscan` without fingerprint verification on an untrusted network can expose you to interceptor attacks. 

---

# 15. Complete Real-World Flow

Think about the whole process like this:

```text
                 SSH SERVER
                    |
                    |
             Host Key Pair
              /          \
        Private Key     Public Key
             |              |
             |              |
          Server          Client
                            |
                            ↓
                     known_hosts
```

### First connection

```text
Client → SSH Server

"Who are you?"

Server → Public Host Key

Client → Verify Fingerprint

Fingerprint matches?
       |
     YES
       ↓
Save key
       ↓
known_hosts
```

### Future connection

```text
Client
  |
  | SSH connection
  ↓
Server
  |
  | Host Key
  ↓
Compare with known_hosts
  |
  +---- Match ----→ Continue
  |
  +---- Mismatch -→ WARNING / STOP
```

---

# 16. Commands You Should Remember

| Purpose                                 | Command                                                        |
| --------------------------------------- | -------------------------------------------------------------- |
| Check ED25519 fingerprint               | `ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key.pub`           |
| Connect with temporary host-key setting | `ssh -o StrictHostKeyChecking=accept-new hostb`                |
| Remove user's old host key              | `ssh-keygen -R hostb`                                          |
| Remove system-wide old key              | `ssh-keygen -R hostb -f /etc/ssh/ssh_known_hosts`              |
| Collect remote host key                 | `ssh-keyscan hostb`                                            |
| Add key to user's known_hosts           | `ssh-keyscan hostb >> ~/.ssh/known_hosts`                      |
| List SSH host keys                      | `ls -l /etc/ssh/ssh_host_*`                                    |
| Regenerate host keys                    | `rm -f /etc/ssh/ssh_host_*` + `systemctl restart sshd.service` |

---

## 🎯 Interview-Friendly Explanation

If an interviewer asks:

**"What is SSH host-key verification?"**

You can answer:

> SSH host-key verification is a mechanism used by an SSH client to verify the identity of an SSH server. The server presents its public host key, and the client compares its fingerprint with the trusted key stored in `known_hosts`. If the key matches, the connection proceeds. If the key changes unexpectedly, SSH displays a `REMOTE HOST IDENTIFICATION HAS CHANGED` warning to protect against possible man-in-the-middle attacks.

### The 5 things to remember

```text
1. Host Key
      ↓
Identifies the SSH server

2. Fingerprint
      ↓
Short representation of public key

3. known_hosts
      ↓
Client stores trusted host keys

4. Host key changes
      ↓
SSH gives security warning

5. ssh-keyscan
      ↓
Collects host keys, but VERIFY them first
```

This is the core of the entire lesson.
