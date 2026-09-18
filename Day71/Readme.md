# Creating and Managing Container Images 

This lesson is about **how to find, download, inspect, create, run, share, and delete container images using Podman**. 

Think of it like this:

> **Container Image = Template / Blueprint**
> **Container = Running instance created from that template**

---

## 1. Container vs Container Image

A **container** is an isolated environment where an application runs as an isolated process.

A **container image** contains:

* Application
* Required libraries
* Dependencies
* Configuration/files needed to run the application

An image can exist without a container, but a container needs an image to create its runtime environment. 

### Simple example

Imagine you have a house blueprint:

```text
Container Image
      ↓
   Blueprint
      ↓
 ┌───────────┐
 │ App       │
 │ Libraries │
 │ Config    │
 └───────────┘
      ↓
   podman run
      ↓
Container
```

You can create multiple containers from the same image.

---

# 2. What can Podman do with Images?

Podman provides commands to:

| Operation | Purpose                          |
| --------- | -------------------------------- |
| Search    | Find images in a registry        |
| Pull      | Download an image                |
| List      | See local images                 |
| Inspect   | Examine image information        |
| Tag       | Give an image another name/tag   |
| Build     | Create a new image               |
| Run       | Create a container from an image |
| Push      | Upload an image to a registry    |
| Remove    | Delete an image                  |
| Prune     | Remove unused/dangling images    |

These are the main image-management operations covered in the lesson. 

---

# 3. Search for an Image

Sometimes you don't know what images are available.

Use:

```bash
podman search registry.redhat.io/rhel10
```

This searches the registry for available images. 

For example, you might see:

```text
registry.redhat.io/rhel10/buildah
registry.redhat.io/rhel10/toolbox
registry.redhat.io/rhel10/net-snmp
```

### Searching for UBI

Red Hat provides **Universal Base Images (UBI)** that can be used as foundations for container applications.

You can search:

```bash
podman search registry.redhat.io/ubi
```

This can show:

```text
registry.redhat.io/ubi8/ubi
registry.redhat.io/ubi9/ubi
registry.redhat.io/ubi10/ubi
```

UBI images are OCI-compliant and freely redistributable. 

---

# 4. Pull an Image

Once you know which image you want, download it using:

```bash
podman pull IMAGE
```

Example:

```bash
podman pull registry.redhat.io/rhel10/rhel-bootc
```

This downloads the image into your **local image storage**. 

---

# 5. What is a Tag?

An image normally has a:

```text
name:tag
```

For example:

```text
rhel-bootc:10.0
```

Here:

```text
rhel-bootc → image name
10.0       → tag/version
```

You can pull a specific version:

```bash
podman pull registry.redhat.io/rhel10/rhel-bootc:10.0
```

If you don't specify a tag:

```bash
podman pull registry.redhat.io/rhel10/rhel-bootc
```

Podman uses the `latest` tag by default. 

### Important

```text
image:10.0
```

means a specific version.

```text
image:latest
```

means the latest version associated with that tag.

---

# 6. List Local Images

After pulling an image, check your local images:

```bash
podman images
```

or:

```bash
podman image list
```

Example:

```text
REPOSITORY                 TAG       IMAGE ID       SIZE
registry.../rhel-bootc    10.0      a93f5ef0baa4   1.43 GB
registry.../rhel-bootc    latest    a93f5ef0baa4   1.43 GB
```

Notice that `10.0` and `latest` can point to the same image ID. 

---

# 7. Inspect an Image

If you want detailed information about an image:

```bash
podman image inspect IMAGE
```

Example:

```bash
podman image inspect rhel-bootc:latest
```

The output is in **JSON format**. 

It can contain information such as:

```text
Image ID
Digest
Repository tags
Repository digests
Configuration
Labels
```

---

# 8. Inspect Specific Information

You don't always need the huge JSON output.

Podman allows you to use `--format`.

### Get image name

```bash
podman image inspect rhel-bootc:latest \
--format "{{.Config.Labels.name}}"
```

Output:

```text
rhel10/rhel-bootc
```

### Get creation date

```bash
podman image inspect rhel-bootc:latest \
--format "{{.Created}}"
```

### Get description

```bash
podman image inspect rhel-bootc:latest \
--format "{{.Config.Labels.description}}"
```

The lesson demonstrates these formatted inspection examples. 

---

# 9. Tag an Image

You can give an existing image another name/tag.

Syntax:

```bash
podman tag SOURCE_IMAGE NEW_IMAGE:TAG
```

Example from the lesson:

```bash
podman tag c6222576494f fedora:latest
```

Now the image has another name/tag associated with it. 

### Think of it as

```text
Original image
      ↓
c6222576494f
      ↓
podman tag
      ↓
fedora:latest
```

Tagging does **not mean building a completely new image**; it gives an existing image another reference.

---

# 10. Creating Your Own Image

This is one of the most important parts.

You can create a container image using a **Containerfile**.

A Containerfile contains instructions describing how the image should be assembled. 

Example:

```dockerfile
FROM ubi10/ubi

RUN dnf install -y httpd

COPY index.html /var/www/html/index.html

EXPOSE 80

ENTRYPOINT ["/usr/sbin/httpd", "-DFOREGROUND"]
```

Let's understand each instruction.

---

## `FROM`

```dockerfile
FROM ubi10/ubi
```

This defines the **base image**.

Think:

> "Start my new image using UBI 10."

The lesson first pulls the UBI image:

```bash
podman image pull registry.redhat.io/ubi10/ubi
```



---

## `RUN`

```dockerfile
RUN dnf install -y httpd
```

This installs the Apache HTTP server package inside the image.

---

## `COPY`

```dockerfile
COPY index.html /var/www/html/index.html
```

This copies your local `index.html` into the image's web-server document directory.

---

## `EXPOSE`

```dockerfile
EXPOSE 80
```

This indicates that the application uses port **80**.

---

## `ENTRYPOINT`

```dockerfile
ENTRYPOINT ["/usr/sbin/httpd", "-DFOREGROUND"]
```

This defines the process that should run when the container starts.

The `-DFOREGROUND` option keeps `httpd` running in the foreground, which is important for the container process. 

---

# 11. Build the Image

Once you have:

```text
Containerfile
index.html
```

you can build your image:

```bash
podman build -t my-httpd -f Containerfile
```

Meaning:

```text
podman build
     ↓
Create image

-t my-httpd
     ↓
Give image the name "my-httpd"

-f Containerfile
     ↓
Use this Containerfile
```

The build executes the Containerfile instructions step by step. 

You eventually get:

```text
localhost/my-httpd:latest
```

---

# 12. List the New Image

After building:

```bash
podman image list
```

You should see something similar to:

```text
REPOSITORY          TAG       SIZE
localhost/my-httpd  latest    273 MB
```



---

# 13. Run the Image

Now create a running container from your image:

```bash
podman run my-httpd
```

This creates a container from the image. 

The relationship is:

```text
Containerfile
      ↓
podman build
      ↓
Container Image
      ↓
podman run
      ↓
Running Container
```

This is a very important concept.

---

# 14. Push an Image to a Registry

Suppose you created:

```text
my-httpd
```

and want other systems/people to access it.

You can push it to a container registry:

```bash
podman image push my-httpd registry.redhat.io/my-httpd
```

The `podman image push` command can push an image, manifest list, or image index to a registry. 

Think:

```text
Your machine
     │
     │ podman push
     ↓
Container Registry
     │
     ├── Other servers
     ├── Developers
     └── Kubernetes/OpenShift
```

---

# 15. Remove an Image

To remove an image:

```bash
podman rmi IMAGE_ID
```

Example:

```bash
podman rmi 4e138cd2375b
```

This removes the image and can also remove untagged/unreferenced parent images. 

---

# 16. Prune Dangling Images

Sometimes images are left behind without useful tags.

These are called **dangling images**.

Remove them with:

```bash
podman image prune
```



To remove all images that aren't being used by any container, use:

```bash
podman image prune --all
```



Be careful with cleanup commands because they can remove images you may want later.

---

# 🔥 Complete Podman Image Workflow

This is the flow I recommend remembering for exams and real work:

```text
                CONTAINER REGISTRY
                       │
                       │ podman search
                       ↓
                  Find image
                       │
                       │ podman pull
                       ↓
                 Local Image
                       │
             ┌─────────┴─────────┐
             │                   │
       podman inspect        podman tag
             │                   │
             ↓                   ↓
        Information         New image tag
                                  
                 OR
                  │
                  ↓
             Containerfile
                  │
             podman build
                  ↓
            Custom Image
                  │
             podman run
                  ↓
              Container
                  │
             podman push
                  ↓
             Registry
```

---

# 🧠 Commands to Memorize

| Command                | Meaning                |
| ---------------------- | ---------------------- |
| `podman search`        | Search registry        |
| `podman pull`          | Download image         |
| `podman images`        | List images            |
| `podman image list`    | List images            |
| `podman image inspect` | Inspect image          |
| `podman tag`           | Add another name/tag   |
| `podman build`         | Build image            |
| `podman run`           | Create/run container   |
| `podman image push`    | Upload image           |
| `podman rmi`           | Remove image           |
| `podman image prune`   | Remove dangling images |

---

## ⭐ Most Important Difference

Don't confuse these:

```bash
podman pull
```

**Registry → Local image**

```bash
podman build
```

**Containerfile → New image**

```bash
podman run
```

**Image → Container**

```bash
podman push
```

**Local image → Registry**

So the easiest way to remember it is:

> **PULL → get an image**
> **BUILD → create an image**
> **RUN → create a container**
> **PUSH → share an image**
> **RMI/PRUNE → clean images**

The lesson's main objective is precisely to manage downloaded images and create a simple container image. 
