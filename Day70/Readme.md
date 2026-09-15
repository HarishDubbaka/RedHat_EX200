# Running Containers with Podman

## 📌 Overview

**Podman** is an open-source container management tool used to:

* Run containers
* Manage containers
* Download container images
* Build container images
* Work with container registries
* Expose container applications through ports

Podman is available by default in **Red Hat Enterprise Linux (RHEL) 10**. 

---

# 1. What is Podman?

Podman is a tool that allows us to work with **containers and container images**.

A simple way to understand it:

```text
Container Image
      ↓
   Podman
      ↓
   Container
      ↓
 Application runs
```

For example:

```text
httpd image
     ↓
podman run
     ↓
Apache web server container
```

Podman can find, run, build, and deploy **OCI (Open Container Initiative)** container images and containers. 

---

## 2. Why Podman?

Some container tools use a **daemon** to manage containers.

A daemon is a background service that handles container operations.

Podman is different.

### Podman is daemonless

```text
Traditional approach:

User → Container CLI → Daemon → Container


Podman:

User → Podman → Container
```

Podman interacts directly with containers, images, and registries.

### Advantages

* No central daemon is required
* Avoids a single daemon becoming a single point of failure
* Can be used without requiring elevated privileges in many scenarios
* Suitable for production environments



---

# 3. Check Podman Version

Use:

```bash
podman -v
```

Example:

```bash
user@host:~$ podman -v
podman version 5.4.0
```

The exact version can be different depending on your system. 

### Remember

```bash
podman -v
```

= Check Podman version

---

# 4. Ways to Use Podman

Podman provides three ways to interact with containers:

1. **Podman CLI**
2. **RESTful API**
3. **Podman Desktop**

This lesson mainly focuses on the **Podman CLI**. 

---

# 5. What is a Container Image?

A **container image** is a packaged version of an application.

It contains:

* Application
* Required dependencies
* Required files
* Configuration needed to run the application

Think of an image as a **template**.

```text
Image = Template
Container = Running instance of the image
```

For example:

```text
httpd image
     ↓
Create container
     ↓
Apache web server running
```

---

# 6. Container Image Registries

A **container registry** is a place where container images are stored.

Common registries include:

* Red Hat Registry
* Quay.io
* Docker Hub
* Amazon Elastic Container Registry (ECR)



### Simple analogy

Think about a registry like a **software warehouse**.

```text
Registry
   │
   ├── httpd image
   ├── nginx image
   ├── python image
   └── database image
```

Podman downloads images from these registries when required.

---

# 7. Red Hat Container Registries

Red Hat provides three important registries.

| Registry                      | Authentication                 |
| ----------------------------- | ------------------------------ |
| `registry.redhat.io`          | Required                       |
| `registry.access.redhat.com`  | Not required                   |
| `registry.connect.redhat.com` | Red Hat Partner Connect images |



### Important

```text
registry.redhat.io
        ↓
Authentication required


registry.access.redhat.com
        ↓
Authentication not required
```

---

# 8. Red Hat Ecosystem Catalog

The **Red Hat Ecosystem Catalog** can be used to search for container images.

It provides information such as:

* Available images
* Image versions
* Containerfile information
* Installed packages
* Security scanning information
* Image tags



---

# 9. Quay.io

**Quay.io** is a container image registry.

It can be used to store custom container images.

For example:

```text
Developer
   ↓
Build container image
   ↓
Push image
   ↓
Quay.io
   ↓
Other systems pull image
```

Public images can be stored in Quay.io for free, while additional features are available for paying customers. 

---

# 10. Login to a Container Registry

Use:

```bash
podman login <registry>
```

Example:

```bash
podman login registry.redhat.io
```

Podman asks for:

```text
Username:
Password:
```

Example:

```text
Username: provide_username
Password: provide_password
Login Succeeded!
```



---

## 🔐 Password Security

When entering the password:

* The password is not displayed on the screen.
* Type the password normally.
* Press **Enter** when finished.



---

# 11. Login to a Registry Without Authentication

Some registries do not require credentials.

Example:

```bash
podman login registry.access.redhat.com
```

When prompted:

```text
Username:
Password:
```

Press **Enter** for both.

Example:

```text
Username: Enter
Password: Enter

Login Succeeded!
```



---

# 12. Check Configured Registries

Use:

```bash
podman info
```

This command provides information about:

* Operating system
* Hardware
* Registries
* Plugins
* Podman version
* Registry search configuration



You may see registries such as:

```text
registry.access.redhat.com
registry.redhat.io
docker.io
quay.io
```



---

# 13. Important Registry Configuration Files

Podman mainly uses two locations for registry configuration and credentials.

### Registry configuration

```text
/etc/containers/registries.conf
```

or:

```text
/etc/containers/registries.conf.d/
```

This contains registry configuration. 

### Authentication information

```text
${XDG_RUNTIME_DIR}/containers/auth.json
```

This stores registry authentication information for the current user.



---

# 14. Running a Container

The main command is:

```bash
podman run <image>
```

For example:

```bash
podman run registry.redhat.io/ubi10 echo 'Hello World!'
```

What happens?

```text
podman run
     ↓
Check image locally
     ↓
Image not found?
     ↓
Pull image from registry
     ↓
Create container
     ↓
Start container
     ↓
Run command
```



Output:

```text
Hello World!
```

---

# 15. What Happens After the Command Finishes?

Consider:

```bash
podman run registry.redhat.io/ubi10 echo 'Hello World!'
```

The container executes:

```bash
echo 'Hello World!'
```

After `echo` finishes, there is no longer a process keeping the container running.

Therefore:

```text
Container starts
     ↓
echo executes
     ↓
Hello World!
     ↓
Command finishes
     ↓
Container stops
```



### Important

**Stopped does not mean removed.**

The container still exists unless you remove it.

---

# 16. Run a Container in Detached Mode

Sometimes we want the container to continue running without blocking our terminal.

Use:

```bash
-d
```

or:

```bash
--detach
```

Example:

```bash
podman run -d registry.redhat.io/ubi10 sleep infinity
```



### Meaning

```text
-d
↓
Run container in background
↓
Return terminal prompt
```

---

# 17. List Running Containers

Use:

```bash
podman ps
```

Example:

```bash
podman ps
```

It displays information such as:

* Container ID
* Image
* Command
* Creation time
* Status
* Ports
* Container name



Example:

```text
CONTAINER ID   IMAGE                         COMMAND
8699...1cae    registry.redhat.io/ubi10     sleep infinity
```

---

# 18. List All Containers

By default:

```bash
podman ps
```

shows only **running containers**.

To display both running and stopped containers:

```bash
podman ps -a
```

or:

```bash
podman ps --all
```



### Remember

```bash
podman ps
```

→ Running containers

```bash
podman ps -a
```

→ Running + stopped containers

---

# 19. Container ID

Every container has an identifier.

Podman can use:

* Short container ID
* Long container ID

The short ID contains 12 characters, while the long identifier contains 64 characters. 

For example:

```text
8699...1cae
```

You can generally use the container name or ID when managing the container.

---

# 20. Give a Container a Name

If you don't specify a name, Podman generates a random name.

Example:

```text
bold_chebyshev
nervous_nobel
```

Instead, it is better to give your container a meaningful name.

Use:

```bash
--name
```

Example:

```bash
podman run --name my_python \
registry.redhat.io/ubi9/python-312 which python
```



Now the container is called:

```text
my_python
```

You can verify it:

```bash
podman ps -a
```



---

# 21. Restart a Container

Use:

```bash
podman restart <container>
```

Example:

```bash
podman restart nginx
```



It can restart a running container or start a stopped container.

---

# 22. Stop a Container

Use:

```bash
podman stop <container>
```

Example:

```bash
podman stop 1b982aeb75dd
```



### What happens?

Podman sends:

```text
SIGTERM
```

This gives the application an opportunity to perform cleanup before stopping.

---

# 23. Stop All Running Containers

Use:

```bash
podman stop --all
```

or:

```bash
podman stop -a
```

Example:

```bash
podman stop --all
```

This stops all currently running containers. 

---

# 24. Forcefully Stop a Container

If a container does not respond to `SIGTERM`, Podman eventually sends:

```text
SIGKILL
```

The default wait time is **10 seconds**. 

You can forcefully stop a container using:

```bash
podman kill <container>
```

Example:

```bash
podman kill httpd
```



---

# 25. Remove a Container

To remove a stopped container:

```bash
podman rm <container>
```

Example:

```bash
podman rm c58cfd4b90df
```



### Important

You normally cannot remove a running container.

First:

```bash
podman stop <container>
```

Then:

```bash
podman rm <container>
```

---

# 26. Force Remove a Container

You can force removal with:

```bash
podman rm -f <container>
```

Example:

```bash
podman rm c58cfd4b90df --force
```



---

# 27. Automatically Remove a Container

Sometimes we don't want a container to remain after it finishes.

Use:

```bash
--rm
```

Example:

```bash
podman run --rm registry.redhat.io/ubi10 \
echo 'Hello World!'
```



The process is:

```text
Create container
      ↓
Run command
      ↓
Command finishes
      ↓
Container exits
      ↓
Podman automatically removes it
```

Therefore:

```bash
podman ps -a
```

will not show that container afterward. 

---

# 28. Exposing Container Ports

Many applications need to accept network connections.

Examples:

* Web servers
* APIs
* Databases

A container has its own network environment.

To make an application accessible from the host, we can map a host port to a container port.

Use:

```bash
-p
```

Syntax:

```bash
-p HOST_PORT:CONTAINER_PORT
```



---

# 29. Port Mapping Example

Suppose Apache is listening on port `8080` inside the container.

We can map:

```text
Host port 8080
       ↓
Container port 8080
```

Command:

```bash
podman run -p 8080:8080 \
registry.redhat.io/rhel10/httpd-24:latest
```



Now traffic sent to:

```text
localhost:8080
```

can reach the service inside the container.

---

# 30. Test the Web Server

Use:

```bash
curl 127.0.0.1:8080
```

Example:

```bash
user@host:~$ curl 127.0.0.1:8080
```

If Apache is running correctly, it returns HTML content.



---

# 31. Run the Web Server in Background

The previous command runs in the foreground.

To avoid blocking your terminal, use:

```bash
-d
```

Example:

```bash
podman run -d -p 8080:8080 \
registry.redhat.io/rhel10/httpd-24:latest
```



Now:

```text
Terminal
   ↓
podman run -d
   ↓
Container runs in background
   ↓
Terminal remains available
```

---

# 32. Important Podman Commands

| Command          | Purpose                                    |
| ---------------- | ------------------------------------------ |
| `podman -v`      | Check Podman version                       |
| `podman info`    | Display Podman/system/registry information |
| `podman login`   | Log in to a container registry             |
| `podman run`     | Create and run a container                 |
| `podman ps`      | List running containers                    |
| `podman ps -a`   | List all containers                        |
| `podman restart` | Restart/start a container                  |
| `podman stop`    | Gracefully stop a container                |
| `podman stop -a` | Stop all running containers                |
| `podman kill`    | Forcefully stop a container                |
| `podman rm`      | Remove a stopped container                 |
| `podman rm -f`   | Forcefully remove a container              |

---

# 33. Important Podman Options

| Option   | Meaning                                       |
| -------- | --------------------------------------------- |
| `-d`     | Run container in detached/background mode     |
| `--name` | Assign a name to the container                |
| `-a`     | All                                           |
| `--all`  | All                                           |
| `--rm`   | Automatically remove container after it exits |
| `-p`     | Publish/map a port                            |
| `-f`     | Force operation                               |

---

# 34. Container Lifecycle

A simple container lifecycle looks like this:

```text
             Container Image
                    │
                    ▼
              podman run
                    │
                    ▼
             Container Created
                    │
                    ▼
               Container
                 Running
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
      podman stop         Application
          │                finishes
          ▼                   │
       Stopped                ▼
          │                 Exited
          │                   │
          └─────────┬─────────┘
                    ▼
                podman rm
                    │
                    ▼
               Container
                 Removed
```

---

# 35. Practical Example

## Step 1: Check Podman

```bash
podman -v
```

---

## Step 2: Login to Registry

```bash
podman login registry.redhat.io
```

Enter your credentials.

---

## Step 3: Run a Container

```bash
podman run registry.redhat.io/ubi10 echo 'Hello World!'
```

The image is pulled if it is not already available locally.

---

## Step 4: Check Containers

```bash
podman ps
```

The container may not appear because the command already finished.

Use:

```bash
podman ps -a
```

to see stopped containers.

---

## Step 5: Run a Long-Running Container

```bash
podman run -d --name mycontainer \
registry.redhat.io/ubi10 sleep infinity
```

---

## Step 6: Check It

```bash
podman ps
```

You should see:

```text
mycontainer
```

---

## Step 7: Stop It

```bash
podman stop mycontainer
```

---

## Step 8: Remove It

```bash
podman rm mycontainer
```

---

# 36. Web Server Example

Run Apache:

```bash
podman run -d --name myweb \
-p 8080:8080 \
registry.redhat.io/rhel10/httpd-24:latest
```

Check:

```bash
podman ps
```

Test:

```bash
curl 127.0.0.1:8080
```

Stop:

```bash
podman stop myweb
```

Remove:

```bash
podman rm myweb
```

---

# 37. Key Concepts to Remember

### Image

A packaged application and its dependencies.

```text
Image = Template
```

### Container

A running instance created from an image.

```text
Container = Running instance
```

### Registry

A location where container images are stored.

```text
Registry = Image storage
```

### Podman

Tool used to manage images and containers.

```text
Podman = Container management tool
```

---

# 38. Most Important Commands for RHCSA

For practical administration, remember these first:

```bash
podman -v
```

```bash
podman info
```

```bash
podman login <registry>
```

```bash
podman run <image>
```

```bash
podman run -d <image>
```

```bash
podman run --name <name> <image>
```

```bash
podman ps
```

```bash
podman ps -a
```

```bash
podman restart <container>
```

```bash
podman stop <container>
```

```bash
podman kill <container>
```

```bash
podman rm <container>
```

```bash
podman rm -f <container>
```

```bash
podman run --rm <image>
```

```bash
podman run -p HOST_PORT:CONTAINER_PORT <image>
```

---

# 39. Quick Memory Trick

Remember the basic lifecycle as:

```text
LOGIN
  ↓
RUN
  ↓
PS
  ↓
STOP
  ↓
RM
```

### Meaning

```text
podman login
      ↓
podman run
      ↓
podman ps
      ↓
podman stop
      ↓
podman rm
```

For a background container:

```text
podman run -d
```

For a temporary container:

```text
podman run --rm
```

For a named container:

```text
podman run --name mycontainer
```

For networking:

```text
podman run -p 8080:8080
```

---

# 40. Exam-Focused Questions

### Q1. What is Podman?

Podman is an open-source, daemonless tool for managing OCI containers and container images.

### Q2. Is Podman daemonless?

**Yes.**

### Q3. Which command checks the Podman version?

```bash
podman -v
```

### Q4. Which command runs a container?

```bash
podman run
```

### Q5. Which command shows running containers?

```bash
podman ps
```

### Q6. Which command shows stopped as well as running containers?

```bash
podman ps -a
```

### Q7. How do you run a container in the background?

```bash
podman run -d
```

### Q8. How do you assign a name?

```bash
podman run --name mycontainer <image>
```

### Q9. How do you stop a container gracefully?

```bash
podman stop <container>
```

### Q10. How do you forcefully stop a container?

```bash
podman kill <container>
```

### Q11. How do you remove a stopped container?

```bash
podman rm <container>
```

### Q12. How do you automatically remove a container after it exits?

```bash
podman run --rm <image>
```

### Q13. How do you map a host port to a container port?

```bash
podman run -p HOST_PORT:CONTAINER_PORT <image>
```

Example:

```bash
podman run -p 8080:8080 <image>
```

---

# 41. Final Summary

Podman provides a simple way to manage containers on RHEL.

The most important flow is:

```text
Container Registry
       │
       │ podman login
       ▼
     Image
       │
       │ podman run
       ▼
   Container
       │
       ├── podman ps
       │
       ├── podman restart
       │
       ├── podman stop
       │
       ├── podman kill
       │
       └── podman rm
```

### ⭐ Core commands

```bash
podman login
podman run
podman ps
podman ps -a
podman restart
podman stop
podman kill
podman rm
```

### ⭐ Core options

```bash
-d       # Run in background
--name   # Give container a name
--rm     # Automatically remove after exit
-p       # Map ports
-f       # Force operation
```

The key idea is:

> **Image → Run → Container → Manage → Stop → Remove**

This is the basic foundation for working with containers using Podman on RHEL. 

This version is ready to save directly as **`README.md`**. It keeps the terminology and command examples from your uploaded RHEL 10 material while adding clearer explanations and practical flow.
