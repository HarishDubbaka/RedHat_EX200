# Managing SSH Host Keys 

Think of an **SSH host key as the server's identity card**.

When you connect:

```bash
ssh hostb
```

SSH wants to make sure:

> **"Am I really connecting to hostb, or is someone pretending to be hostb?"**

---

## 1. What is an SSH Host Key?

An SSH server has a pair of host keys:

* **Private host key** → stays securely on the server.
* **Public host key** → presented to SSH clients.

The client remembers the server's public key in:

```bash
~/.ssh/known_hosts
```

There can also be a system-wide file:

```bash
/etc/ssh/ssh_known_hosts
```

### Simple example

First time:

```text
Client                         Server
  |                              |
  |-------- SSH connection ----->|
  |                              |
  |<------- Host public key -----|
  |                              |
  |  "I don't know this key"     |
  |                              |
  |  Ask user for confirmation   |
```

After you accept it, the key is stored in:

```bash
~/.ssh/known_hosts
```

Next time:

```text
Client                         Server
  |                              |
  |-------- SSH connection ----->|
  |                              |
  |<------- Host public key -----|
  |                              |
  | Compare with known_hosts     |
  |                              |
  |      MATCH ✅                |
  |                              |
  |-------- Connection --------->|
```

---

# 2. Why is this important?

Suppose you normally connect to:

```bash
ssh server1
```

An attacker could potentially place themselves between your computer and `server1`.

This is called a:

**Man-in-the-Middle (MITM) attack**

Without host-key verification, you might unknowingly connect to the attacker's machine.

SSH protects against this by checking:

```text
Received server key
       ↓
Compare with known_hosts
       ↓
     Match?
     /    \
   YES     NO
   ↓        ↓
Connect   Warning/Reject
```

---

# 3. What happens during the first connection?

Example:

```bash
ssh hostb
```

You may see:

```text
The authenticity of host 'hostb (192.168.250.11)' can't be established.

ED25519 key fingerprint is
SHA256:Qit7NeebQCgP2otXSaVXbw1rGmUVMlybXyV68qaND/M.

Are you sure you want to continue connecting
(yes/no/[fingerprint])?
```

SSH is basically asking:

> **"I've never seen this server before. Do you trust this server's identity?"**

If you enter:

```text
yes
```

SSH stores the host key in:

```bash
~/.ssh/known_hosts
```

---

# 4. What if the host key changes?

Suppose you previously connected to:

```text
server1
```

and its key was:

```text
KEY-A
```

Later, the server presents:

```text
KEY-B
```

SSH says:

> 🚨 Something changed!

This could happen for legitimate reasons, such as:

* Server reinstallation
* SSH host keys regenerated
* Server replacement
* IP address reused by another server

But it could also indicate:

* Man-in-the-middle attack
* Wrong server
* DNS/IP configuration problem

So **don't blindly accept a changed key**.

---

# 5. `StrictHostKeyChecking`

This parameter controls how SSH behaves when dealing with host keys.

Configuration files:

### System-wide

```bash
/etc/ssh/ssh_config
```

### User-specific

```bash
~/.ssh/config
```

User configuration generally takes precedence over system-wide settings.

You can also specify it directly:

```bash
ssh -o StrictHostKeyChecking=yes hostb
```

---

# 6. Four Important Values

## `StrictHostKeyChecking=yes`

🔐 **Most secure**

SSH requires the host key to already exist in `known_hosts`.

If it doesn't exist:

```text
Connection refused
```

If it exists but doesn't match:

```text
Connection refused
```

So you need to manually add trusted keys.

### Think:

> **"I trust only servers whose identity I already know."**

---

## `StrictHostKeyChecking=ask`

⭐ **Default**

For a new server:

```text
Are you sure you want to continue connecting?
```

You answer:

```text
yes
```

Then SSH stores the key.

If an existing host key changes, SSH refuses the connection.

### Think:

> **"Ask me before trusting a new server."**

---

## `StrictHostKeyChecking=accept-new`

Useful when you want automation but still want protection against changed keys.

For a **new server**:

```text
New key → Automatically accepted → stored
```

For a **changed existing key**:

```text
Changed key → ❌ Connection refused
```

### Think:

> **"Automatically trust new servers, but NEVER silently trust a changed identity."**

---

## `StrictHostKeyChecking=no`

⚠️ **Least secure**

SSH automatically accepts new keys.

Worse, it can also continue despite changed host keys.

### Think:

> **"Don't bother checking carefully."**

This can expose you to MITM attacks.

Avoid using this casually, especially in production.

---

# 7. Easy Comparison

| Setting      | New Host | Changed Host | Security   |
| ------------ | -------- | ------------ | ---------- |
| `yes`        | ❌ Reject | ❌ Reject     | 🔐 Highest |
| `ask`        | ❓ Ask    | ❌ Reject     | ⭐ High     |
| `accept-new` | ✅ Accept | ❌ Reject     | ⭐ High     |
| `no`         | ✅ Accept | ⚠️ Accept    | 🔓 Lowest  |

### Interview shortcut

Remember:

**YES → Ask → Accept-new → NO**

Actually, for security:

```text
yes
 ↓
accept-new / ask
 ↓
no
```

---

# 8. Important Commands

### View known hosts

```bash
cat ~/.ssh/known_hosts
```

### Search for a particular host

```bash
ssh-keygen -F hostb
```

### Remove an old host key

If a server was legitimately rebuilt:

```bash
ssh-keygen -R hostb
```

Then reconnect:

```bash
ssh hostb
```

SSH will ask you to accept the new key.

### View SSH client configuration

```bash
cat /etc/ssh/ssh_config
```

or:

```bash
cat ~/.ssh/config
```

---

# 9. Real-Time Example

Imagine your SAP production server is:

```text
sapprd01
```

First connection:

```bash
ssh sapprd01
```

SSH doesn't know the server.

```text
New server
    ↓
Check known_hosts
    ↓
Not found
    ↓
Ask user
    ↓
yes
    ↓
Store host key
```

Later:

```bash
ssh sapprd01
```

SSH receives the key again:

```text
Received key
     ↓
Compare with known_hosts
     ↓
Match ✅
     ↓
Connection allowed
```

But if the server suddenly presents a different key:

```text
Received KEY-B
Stored KEY-A
       ↓
   MISMATCH ❌
       ↓
Potential security issue
       ↓
Connection blocked
```

---

# 10. One Important Correction

The **host-key check happens before the SSH session is trusted**, but it is useful to distinguish two concepts:

* **Host authentication:** "Is this really the server I intended to contact?"
* **User authentication:** "Is this user allowed to log in?"

For example:

```text
1. Connect to SSH server
        ↓
2. Verify server host key
        ↓
3. Establish secure cryptographic session
        ↓
4. Authenticate user
        ↓
5. Shell/session starts
```

So don't confuse **SSH host keys** with **user SSH keys**.

### Host key

Identifies the **server**.

```text
Server → Client
```

### User key

Authenticates the **user**.

```text
User → Server
```

---

## 🎯 Interview Answer

If asked **"What is SSH host key verification?"**, you can say:

> **SSH host key verification is a security mechanism used by the SSH client to verify the identity of the SSH server. The server's public host key is compared with the key stored in the client's `known_hosts` file. If the keys match, the connection proceeds. If a known host's key changes, SSH warns or rejects the connection depending on `StrictHostKeyChecking`, helping protect against man-in-the-middle attacks.**

**Easy memory trick:**

> 🔑 **Host key = Server's ID card**
> 📁 **known_hosts = Client's list of trusted server IDs**
> ⚙️ **StrictHostKeyChecking = How strictly SSH checks those IDs**
