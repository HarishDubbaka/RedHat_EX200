# Managing Containers with Podman

## 1. What is a Container?

A **container** is a lightweight, isolated environment used to run an application along with everything the application needs.

For example:

```text
Application
   +
Libraries
   +
Configuration
   +
Runtime dependencies
   ↓
Container
```

The container uses the **host's Linux kernel**, but the application and its required libraries are packaged separately.

### Simple example

Suppose you have a Python application.

Without containers:

```text
Linux Server
 ├── Python
 ├── Python libraries
 ├── Application
 └── Configuration
```

With a container:

```text
Host Linux
 └── Container
      ├── Python
      ├── Python libraries
      ├── Application
      └── Configuration
```

This makes the application easier to move between environments.

---

# 2. Container vs Virtual Machine

This is an important interview question.

![Image](https://images.openai.com/static-rsc-4/AXUF_s_Oxd9idgEpuq6CGL3eS1qyLOjOzjlzjSCi6uOieLEROc4vND-ZVXkwkjuKWvN9L2ayDuDxTJ3Hg966SEiftl_Ky8Qtr6cxC1ixgLDSDSKJFdtiiNb8XXSkXjEh47GGCNlrHHRMWi9a8au_4t95Wq02cXya967WL7iafix52rBmpHKtyc4RvVu06iOJ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/fnFdCdn48Pm7fRvCTDYYTFNL3Qnj-aXa_xGQiVUDF2gYz3lktgTHgoKFdY2b7rzHnmYsaGQ5kUKBkQM17MK2Y0USdz7Ywvju6OFH8fdLPgfKp_39LR-I2mUyylJFgKEVDS0Nve9i516huZcqpCJMBArMRmzYckCIaqQNV4W8yQ8ky8pQMtSqexUWHN2qwD6V?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ziDSb6CoFg-VMxBtqUAfo0i9OAe7JrxXsuJzm9N7iMnzbZiv0DtawNkugzpqymYS8cSzpgAKU1aCphdmyiVGvHhtKainz3V0EcQynTqcKtY8KiS42xwbn59ZVPFNo2Ydbvcozv6Sb5ae4k5SkH5k_6dMtl7_5z3F_TKfZy_-IGuQ26dWnFUHsx4Q5u6u5_ra?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/LT1YMl7QpHsW8EpWEAbmOcfhkJB8jIj0OtpKmFsJqJwScNqD4XePkePGEMjJLlXcTtq1FmNjGQGOMddmOR8h3ImuoFlZgjO8_TSA2dML-uhhndMvoHlk_w6jSKkwm13D8XCbHYC4WH-o2ZQmtHSKEOdqAsDdzZibT9Nw494Wgx15HsTISanayqCCyX3sVHjC?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/7NLSIAt1HifkptQJHd9SOtWa_Fr4rNPpw3imIkDo5owNmtcgr9X4jNL7d0ObUDi8jDgG9MVMNR1i9B4Jo7AawOPj75l4cppGnL8L92KLo1nDidJVZ1lSduu_ku5C6H03zDZukzXgsaU6C0zFGOyS4Ynias9gkhXxLw0hwNXj3ZbKr7eT_49W14J40RrdVcQP?purpose=fullsize)

### Virtual Machine

```text
Physical Server
      |
   Hypervisor
      |
 ┌──────────────┐
 │ VM           │
 │ Guest OS     │
 │ Application  │
 └──────────────┘
```

Every VM generally contains its own **guest operating system/kernel**.

### Container

```text
Physical Server
      |
    Linux
    Kernel
      |
 ┌──────────────┐
 │ Container    │
 │ Application  │
 │ Libraries    │
 └──────────────┘
```

Containers share the host's kernel.

### Key difference

| VM                        | Container               |
| ------------------------- | ----------------------- |
| Uses hypervisor           | Uses container engine   |
| Has guest OS              | Shares host kernel      |
| Larger                    | Lightweight             |
| Usually GBs               | Often MBs               |
| Slower startup            | Very fast startup       |
| Strong OS-level isolation | Process-level isolation |
| KVM, VMware, Hyper-V      | Podman, Docker          |

**Interview answer:**

> A VM virtualizes an entire operating system, while a container virtualizes/isolate the application process and its dependencies while sharing the host kernel.

---

# 3. What is Podman?

**Podman** is a container management tool used to:

* Run containers
* Stop containers
* Remove containers
* Download images
* Build images
* Manage container networks
* Manage container volumes
* Run containers without root

Unlike traditional Docker architecture, Podman is **daemonless**.

```text
Podman
  |
  ├── Images
  ├── Containers
  ├── Networks
  └── Volumes
```

A major advantage is **rootless containers**.

For example, a normal user can run:

```bash
podman run ...
```

without necessarily requiring root privileges.

---

# 4. Container Image vs Container

This distinction is extremely important.

### Image

An **image is a template**.

Example:

```text
nginx image
```

It contains:

```text
OS files
Libraries
Nginx
Configuration
Application files
```

### Container

A **container is a running/created instance of an image**.

Think:

```text
Image = Class
Container = Object
```

For example:

```text
nginx image
     |
     +---- Container 1
     |
     +---- Container 2
     |
     +---- Container 3
```

One image can create many containers.

---

# 5. Container Image Layers

Container images are built using **layers**.

For example:

```text
Application layer
-----------------
Nginx layer
-----------------
Library layer
-----------------
Base OS layer
-----------------
```

These image layers are generally **immutable/read-only**.

When a container runs, a writable layer is added:

```text
Container
-----------------
Writable layer
-----------------
Application layer
-----------------
Nginx layer
-----------------
Library layer
-----------------
Base OS layer
```

If you remove the container, its writable layer is normally removed too.

That's why containers are considered **ephemeral by default**.

---

# 6. Why Use Containers?

### Advantages

**Fast startup**

Containers can start very quickly.

**Lightweight**

They consume fewer resources than VMs.

**Portable**

The same container image can be used across:

```text
Development
     ↓
Testing
     ↓
Production
```

**Isolation**

Applications can be isolated from each other.

**Scalability**

You can easily create multiple instances:

```text
Application
   |
   +-- Container 1
   +-- Container 2
   +-- Container 3
   +-- Container 4
```

---

# 7. Challenges with Containers

Containers aren't automatically a solution for everything.

### Data persistence

Container storage is ephemeral.

If the container is deleted:

```text
Container
   ↓
Writable data
   ↓
Deleted
```

For persistent data, use **volumes** or external storage.

```text
Container
    |
    +---- Volume
             |
             +---- Persistent data
```

### Networking

Networking becomes more complicated when multiple containers communicate.

### Security

Containers share the host kernel, so security configuration is important.

Linux provides security technologies such as:

* SELinux
* Namespaces
* cgroups
* seccomp

---

# 8. Linux Technologies Used by Containers

This is very important for Red Hat exams.

## Namespaces

Namespaces provide **isolation**.

They allow processes inside a container to have an isolated view of resources.

For example:

```text
Host
 |
 +-- Container 1
 |     PID namespace
 |
 +-- Container 2
       PID namespace
```

Processes in one container don't normally see processes in another container in the same way as host processes do.

---

## cgroups

**Control groups (cgroups)** control resource usage.

For example:

```text
Container
   |
   +-- CPU limit
   +-- Memory limit
   +-- I/O limit
```

Example concept:

```text
Container A → 1 CPU
Container B → 2 CPUs
```

This prevents one container from consuming all available resources.

---

## SELinux

SELinux provides an additional security boundary.

It helps control what containers are allowed to access on the host.

---

## seccomp

**Secure computing mode (seccomp)** restricts the system calls that a process can make.

In simple terms:

> seccomp limits which kernel operations a container process can request.

---

# 9. OCI

**OCI = Open Container Initiative**

OCI defines standards for containers.

Two important specifications are:

```text
OCI Image Specification
        ↓
Defines container images

OCI Runtime Specification
        ↓
Defines how containers are executed
```

Because Podman and other modern container tools follow OCI standards, container images are portable between compliant tools.

---

# 10. Container Registry

A **container registry** stores container images.

Think of it as:

```text
Container Registry
       |
       +-- Image 1
       +-- Image 2
       +-- Image 3
       +-- Image 4
```

Examples from Red Hat:

```text
registry.redhat.io
registry.access.redhat.com
registry.connect.redhat.com
```

### Important difference

| Registry                    | Authentication              |
| --------------------------- | --------------------------- |
| registry.redhat.io          | Required                    |
| registry.access.redhat.com  | Generally no authentication |
| registry.connect.redhat.com | Partner Connect images      |

---

# 11. `/etc/containers/registries.conf`

Podman uses container registry configuration.

Important file:

```bash
/etc/containers/registries.conf
```

It defines/configures registries used by container tools.

You may encounter it when troubleshooting image pulls.

---

# 12. Basic Podman Commands

Now let's move to the practical part.

## Check Podman version

```bash
podman --version
```

or:

```bash
podman version
```

---

## Search for an image

```bash
podman search nginx
```

Example:

```bash
podman search httpd
```

---

## Pull an image

```bash
podman pull nginx
```

Better to specify a registry when appropriate:

```bash
podman pull registry.access.redhat.com/ubi9
```

---

# 13. List Images

```bash
podman images
```

or:

```bash
podman image ls
```

Example:

```text
REPOSITORY       TAG       IMAGE ID
nginx            latest    xxxxx
```

---

# 14. Run a Container

Basic command:

```bash
podman run nginx
```

Run in background:

```bash
podman run -d nginx
```

`-d` means:

> Detached mode

---

# 15. Give the Container a Name

```bash
podman run -d --name web nginx
```

Now the container is called:

```text
web
```

Instead of remembering a generated container ID.

---

# 16. List Containers

### Running containers

```bash
podman ps
```

### All containers

```bash
podman ps -a
```

Remember:

```text
podman ps
     ↓
running containers

podman ps -a
     ↓
all containers
```

---

# 17. Stop a Container

```bash
podman stop web
```

Start it again:

```bash
podman start web
```

Restart:

```bash
podman restart web
```

---

# 18. Remove a Container

```bash
podman rm web
```

If it is running, you may need:

```bash
podman rm -f web
```

---

# 19. View Container Logs

Very important for troubleshooting:

```bash
podman logs web
```

Follow logs continuously:

```bash
podman logs -f web
```

`-f` = follow.

---

# 20. Inspect a Container

```bash
podman inspect web
```

This provides detailed information such as:

* Container configuration
* Network
* Mounts
* Environment variables
* Image
* IDs
* Runtime configuration

For troubleshooting, `inspect` is very useful.

---

# 21. Execute a Command Inside Container

For example:

```bash
podman exec web ls
```

Open a shell:

```bash
podman exec -it web /bin/bash
```

If Bash isn't available:

```bash
podman exec -it web /bin/sh
```

Meaning:

```text
-i → interactive
-t → terminal
```

---

# 22. Port Mapping

Suppose the application inside the container listens on port 80.

You want to access it through port 8080 on the host.

```bash
podman run -d --name web -p 8080:80 nginx
```

The format is:

```text
-p HOST_PORT:CONTAINER_PORT
```

So:

```text
Host                  Container
8080  ----------------> 80
```

Then access:

```text
http://host:8080
```

---

# 23. Container Images: Create Your Own

Podman can build images using a **Containerfile** or **Dockerfile**.

Example:

```text
Containerfile
```

Contents:

```dockerfile
FROM registry.access.redhat.com/ubi9

RUN dnf install -y nginx

CMD ["nginx", "-g", "daemon off;"]
```

Then build:

```bash
podman build -t myweb .
```

Explanation:

```text
podman build
      |
      +-- -t myweb → image name
      |
      +-- . → current directory
```

Check:

```bash
podman images
```

---

# 24. Containerfile Important Instructions

You should know these:

| Instruction  | Purpose                      |
| ------------ | ---------------------------- |
| `FROM`       | Base image                   |
| `RUN`        | Execute command during build |
| `COPY`       | Copy files into image        |
| `ADD`        | Add/copy files               |
| `WORKDIR`    | Set working directory        |
| `ENV`        | Environment variable         |
| `EXPOSE`     | Documents container port     |
| `CMD`        | Default command              |
| `ENTRYPOINT` | Main executable              |

Example:

```dockerfile
FROM registry.access.redhat.com/ubi9

WORKDIR /app

COPY app.sh /app/

RUN chmod +x /app/app.sh

CMD ["/app/app.sh"]
```

Build:

```bash
podman build -t myapp .
```

Run:

```bash
podman run --name myapp-container myapp
```

---

# 25. Image Management Commands

List:

```bash
podman images
```

Pull:

```bash
podman pull IMAGE
```

Remove:

```bash
podman rmi IMAGE
```

Tag:

```bash
podman tag SOURCE_IMAGE TARGET_IMAGE
```

Inspect:

```bash
podman inspect IMAGE
```

---

# 26. Container Lifecycle

Remember this flow:

```text
        podman pull
             ↓
          IMAGE
             ↓
       podman run
             ↓
        CONTAINER
             ↓
    ┌────────┴────────┐
    ↓                 ↓
 podman stop      podman restart
    ↓
 STOPPED
    ↓
 podman rm
    ↓
 REMOVED
```

The image can still remain after the container is removed.

---

# 27. Image vs Container — Interview Question

### Question:

**What is the difference between an image and a container?**

### Simple answer:

> A container image is an immutable template containing the application and its dependencies. A container is a runtime instance created from that image.

Example:

```text
nginx IMAGE
    |
    +---- web1 CONTAINER
    +---- web2 CONTAINER
    +---- web3 CONTAINER
```

---

# 28. Why Are Containers Ephemeral?

Because the container's writable layer belongs to the container.

For example:

```text
Image
 ↓
Read-only layers
 ↓
Writable container layer
```

If you delete the container:

```text
Container
    ↓
Writable layer
    ↓
Deleted
```

Therefore, important application data should be stored using a **volume** or external persistent storage.

---

# 29. Rootless Containers

One of Podman's major strengths is the ability to run containers as a regular Linux user.

For example:

```bash
podman run -d --name myweb nginx
```

You don't necessarily need:

```bash
sudo
```

This reduces the security risk associated with giving users root privileges.

---

# 30. Important Commands to Memorize

For your **RHCSA / Red Hat container lab**, focus on these first:

```bash
podman --version

podman search IMAGE

podman pull IMAGE

podman images

podman run IMAGE

podman run -d --name NAME IMAGE

podman ps

podman ps -a

podman stop NAME

podman start NAME

podman restart NAME

podman rm NAME

podman rm -f NAME

podman logs NAME

podman inspect NAME

podman exec -it NAME /bin/bash

podman build -t IMAGE .

podman rmi IMAGE

podman tag SOURCE TARGET
```

---

# 31. One Practical Example

Let's run an Nginx container from start to finish:

### Step 1 — Pull image

```bash
podman pull nginx
```

### Step 2 — Verify image

```bash
podman images
```

### Step 3 — Run container

```bash
podman run -d --name web -p 8080:80 nginx
```

### Step 4 — Check

```bash
podman ps
```

### Step 5 — Check logs

```bash
podman logs web
```

### Step 6 — Test container

```bash
curl http://localhost:8080
```

### Step 7 — Stop

```bash
podman stop web
```

### Step 8 — Start again

```bash
podman start web
```

### Step 9 — Remove

```bash
podman rm -f web
```

The **image still exists**:

```bash
podman images
```

You can create another container from it.

---

# 32. The Big Picture

Remember this architecture:

```text
                 CONTAINER REGISTRY
                        |
                    podman pull
                        ↓
                 ┌──────────────┐
                 │    IMAGE     │
                 │              │
                 │ immutable    │
                 │ layers       │
                 └──────┬───────┘
                        │
                    podman run
                        ↓
                 ┌──────────────┐
                 │  CONTAINER   │
                 │              │
                 │ writable     │
                 │ runtime      │
                 │ network      │
                 │ processes    │
                 └──────┬───────┘
                        │
              Linux Kernel Features
                        |
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
   Namespaces         cgroups        SELinux
   Isolation       Resources         Security
```

## ⭐ Most important points for interviews

1. **Podman is a daemonless container engine.**
2. **Containers share the host's Linux kernel.**
3. **Images are templates; containers are instances.**
4. **Image layers are immutable.**
5. **Containers have a writable runtime layer.**
6. **Containers are ephemeral by default.**
7. **Use volumes for persistent data.**
8. **Namespaces provide isolation.**
9. **cgroups manage resources.**
10. **SELinux provides an additional security boundary.**
11. **OCI defines container standards.**
12. **`podman ps` shows running containers; `podman ps -a` shows all containers.**
13. **`podman logs` and `podman inspect` are important troubleshooting commands.**
14. **Podman supports rootless containers.**
15. **Container registries store container images.**
