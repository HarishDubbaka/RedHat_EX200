## RHCSA Practice Questions and Answers – Password Aging and User Management

### 1. How do you log in to the `serverb` machine as the `student` user from the workstation?

**Answer:**

```bash
ssh student@serverb
```

---

### 2. How do you switch to the root user on `serverb`?

**Answer:**

```bash
sudo -i
```

---

### 3. Which file controls the default password aging policy for newly created users?

**Answer:**

```bash
/etc/login.defs
```

---

### 4. How do you configure newly created users to change their passwords every 30 days?

**Answer:**

Edit `/etc/login.defs` and set:

```bash
PASS_MAX_DAYS 30
```

---

### 5. What does the `PASS_MAX_DAYS` parameter define?

**Answer:**

It specifies the maximum number of days a password can be used before it expires.

---

### 6. How do you create a group named `consultants` with GID `35000`?

**Answer:**

```bash
groupadd -g 35000 consultants
```

---

### 7. How do you verify that the `consultants` group was created successfully?

**Answer:**

```bash
grep consultants /etc/group
```

Example output:

```bash
consultants:x:35000:
```

---

### 8. How do you grant sudo privileges to all members of the `consultants` group?

**Answer:**

Create a file in `/etc/sudoers.d`:

```bash
vim /etc/sudoers.d/consultants
```

Add:

```bash
%consultants ALL=(ALL) ALL
```

---

### 9. Why is it recommended to use files in `/etc/sudoers.d/` instead of editing `/etc/sudoers` directly?

**Answer:**

It prevents accidental corruption of the main sudoers file and simplifies administration.

---

### 10. How do you create the user `consultant1` and add it to the `consultants` supplementary group?

**Answer:**

```bash
useradd -G consultants consultant1
```

---

### 11. How do you create multiple consultant users?

**Answer:**

```bash
useradd -G consultants consultant1
useradd -G consultants consultant2
useradd -G consultants consultant3
```

---

### 12. How do you set a password for a user?

**Answer:**

```bash
passwd consultant1
```

---

### 13. What password was assigned to the consultant users in this exercise?

**Answer:**

```text
redhat
```

---

### 14. Why does the system display the warning `BAD PASSWORD: The password is shorter than 8 characters`?

**Answer:**

Because the password `redhat` contains only 6 characters and does not meet the recommended minimum length.

---

### 15. How can you determine a date 90 days from today?

**Answer:**

```bash
date -d "+90 days" +%F
```

Example:

```bash
2025-08-12
```

---

### 16. How do you set an account expiration date for a user?

**Answer:**

```bash
chage -E YYYY-MM-DD username
```

Example:

```bash
chage -E 2025-08-12 consultant1
```

---

### 17. Which command displays a user's password aging information?

**Answer:**

```bash
chage -l consultant1
```

---

### 18. How do you configure the `consultant2` user to change the password every 15 days?

**Answer:**

```bash
chage -M 15 consultant2
```

---

### 19. What does the `-M` option of the `chage` command do?

**Answer:**

It sets the maximum number of days a password remains valid.

---

### 20. How do you force a user to change the password at the next login?

**Answer:**

```bash
chage -d 0 username
```

Example:

```bash
chage -d 0 consultant1
```

---

### 21. What does the `-d 0` option of the `chage` command do?

**Answer:**

It sets the last password change date to zero, forcing the user to change the password during the next login.

---

### 22. How do you force all consultant users to change their passwords at first login?

**Answer:**

```bash
chage -d 0 consultant1
chage -d 0 consultant2
chage -d 0 consultant3
```

---

### 23. How do you verify the account expiration date and password aging settings for a user?

**Answer:**

```bash
chage -l consultant1
```

---

### 24. Which command exits the root shell and returns to the student user?

**Answer:**

```bash
exit
```

---

### 25. How do you disconnect from `serverb` and return to the workstation?

**Answer:**

```bash
exit
```

---

## RHCSA Exam Quick Commands

```bash
# Set default password expiration
vim /etc/login.defs
PASS_MAX_DAYS 30

# Create group
groupadd -g 35000 consultants

# Configure sudo access
echo "%consultants ALL=(ALL) ALL" > /etc/sudoers.d/consultants

# Create users
useradd -G consultants consultant1
useradd -G consultants consultant2
useradd -G consultants consultant3

# Set passwords
passwd consultant1
passwd consultant2
passwd consultant3

# Find date after 90 days
date -d "+90 days" +%F

# Set account expiration
chage -E YYYY-MM-DD consultant1
chage -E YYYY-MM-DD consultant2
chage -E YYYY-MM-DD consultant3

# Password expires every 15 days
chage -M 15 consultant2

# Force password change at first login
chage -d 0 consultant1
chage -d 0 consultant2
chage -d 0 consultant3

# Verify settings
chage -l consultant1
```

These are common RHCSA EX200-style questions that frequently appear in user and password management tasks.
