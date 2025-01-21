---
type: slides
date: 2025-01-19
theme: real-black
customTheme: real-black
highlightTheme: tomorrow-night-bright
transition: fade
controls: false
progress: false
enableMenu: false
enableChalkboard: false
enableTitleFooter: false
enableZoom: false
enableSearch: false
overview: true
---
# Bootable
# Containers

---

> [!NOTE]
>  All material, including source, presentation, build workflow, containers, disk images, etc. are published on
>  https://github.com/gbraad-redhat/fedora-bootc-example


---

![](https://qr.io/qr-svg/gpo0BA.svg?1737436393812)

---

## From `Containerfile` 
## to Virtual Machine

####  Gerard Braad <me@gbraad.nl>
<!-- element class="fragment fade-in" -->

---

# `Containerfile`

---
### describes content
#### `FROM`
#### `RUN`, `USER`
#### `COPY`, `ADD`

---

### what runs

#### `CMD`
#### `ENTRYPOINT`

---

### or describes runtime use

#### `EXPOSE`
#### `VOLUME`

---

## Example 
# `CMD`

---

```Dockerfile
FROM quay.io/fedora/fedora

RUN dnf install -y cowsay

CMD [ "cowsay", "Moo" ]
```

---

```shell
$ podman build -t cowsay -f Containerfile .
```
```shell
$ podman run --rm cowsay
```
<!-- element class="fragment fade-in" -->


---

```text
	 _____
	< Moo >
	 -----
	        \   ^__^
	         \  (oo)\_______
	            (__)\       )\/\
	                ||----w |
	                ||     ||
```

---

```shell
$ podman run -it --rm cowsay bash
```
```shell
bash-5.2#
```
<!-- element class="fragment fade-in" -->


---

## Example
# `ENTRYPOINT`

---

```Dockerfile
FROM quay.io/fedora/fedora

RUN dnf install -y cowsay

ENTRYPOINT [ "cowsay" ]
```

---

```shell
$ podman build -t cowsay -f Containerfile .
```

```shell
$ podman run --rm cowsay Moo
```
<!-- element class="fragment fade-in" -->

---

# Application Containers

---

### Single process

### Long running


>  supervisor process

---

# System Containers

---

```Dockerfile
FROM ...

RUN dnf install -y systemd

ENTRYPOINT [ "/sbin/init" ]
```
---

```shell
$ podman run -d --name system system
```
```shell
$ podman exec system cowsay Moo
```
<!-- element class="fragment fade-in" -->

---

![](https://brand.systemd.io/assets/svg/systemd-light-vertical-color.svg)
#### System and Service Manager

---

# Still a container

### No 'container'-provided kernel
<!-- element class="fragment fade-in" -->

---

# What if ...

---

## we include a kernel ?

### and call it a Machine Container ?
<!-- element class="fragment fade-in" -->

---

# Still a container

### the container runtime
<!-- element class="fragment fade-in" -->
### runs 'isolated' processes
<!-- element class="fragment fade-in" -->
### using a namespace
<!-- element class="fragment fade-in" -->

---

# Bootable Container

---
```Dockerfile
FROM quay.io/fedora/fedora-bootc

RUN useradd -m -d /var/home/admin -G wheel admin \
    && echo "admin:password" | chpasswd

...
```

---

# Still a container

### therefore easy to build and test
<!-- element class="fragment fade-in" -->
<i class="fas fa-cog fa-spin fa-3x"></i>
<!-- element class="fragment fade-in" -->

---

> [!IMPORTANT]
> The full container image name is:
> `ghcr.io/gbraad-redhat/fedora-bootc-example/fedora-bootc:latest`
>  In most slides this will be shortened

---

```shell
$ podman build -t ghcr.io/.../fedora-bootc \
	-f Containerfile .
```

---

```shell
$ podman push ghcr.io/.../fedora-bootc
```

---

 ```shell
$ podman run -it ghcr.io/.../fedora-bootc bash
```

---

```shell
bash-5.2# cat /etc/os-release
NAME="Fedora Linux"
VERSION="41.20250119.0 (Forty One)"
...
OSTREE_VERSION='41.20250119.0'
```

---

# Wait a minute

<i class="fas fa-lightbulb fa-3x  fa-fade"></i>

---

# bootable
# machines

---

# run
# `/sbin/init`

---

```shell
$ podman run -d --name test ghcr.io/.../fedora-bootc:latest
52cae56640ff...4840bff9
```

---

```shell
$ podman ps
CONTAINER ID  IMAGE           COMMAND     STATUS        NAMES
52cae56640ff  ghcr.io/...est  /sbin/init  Up 4 seconds  test
```

---

# success ?

---

```shell
$ podman exec -it test bash
```

```shell
bash-5.2# uname -na
```
<!-- element class="fragment fade-in" -->
```shell
Linux 52cae56640ff 5.15.167.4-microsoft-standard-WSL2
```
<!-- element class="fragment fade-in" -->

---

# bootable 
# machines


---

# start a
# bootloader

---

# We do not
# 'boot' containers
<!-- element class="fragment fade-in" -->

---

# We instead
# 'run' containers
<!-- element class="fragment fade-in" -->

---

# We just ran a
# system container
<!-- element class="fragment fade-in" -->

---

# What if ...

<i class="fas fa-lightbulb fa-3x fa-fade"></i>

---

# We create a
# disk image
<!-- element class="fragment fade-in" -->

---

```shell
 $ sudo podman run \
    -v ./output:/output \
	quay.io/centos-bootc/bootc-image-builder
	  --target-arch amd64 \
	  --type qcow2 \
	  --rootfs xfs \
	  ... \
	  ghcr.io/gbraad-redhat/fedora-bootc-example/fedora-bootc
```

---

# and start a 
# Virtual Machine
<!-- element class="fragment fade-in" -->

---

```shell
$ sudo virt-install \
    --name fedora-bootc \
    --cpu host \
    --vcpus 2 \
    --memory 2048 \
    --import --disk ./output/qcow2/disk.qcow2 \
    --os-variant fedora-eln
```
---

# Login

---

### Use console or SSH

> [!NOTE]
>  Username: `admin`
>  Password: `password`
>  Groups: `wheel`

---

# Recap

---

### We used a

# `Containerfile`

### to describe the contents

---

### We produced an 

# `OCI`

### container-image

---

### We converted this into a

# `qcow2`

### disk image

---

## run a
# container-image

---

## as a
# virtual machine

---

# Why does
# this matter ?

---

```shell
$ sudo bootc status
No staged image present
Current booted image: ghcr.io/.../fedora-bootc:latest
    Image version: 41.2025019.0 (...)
    Image digest: sha256:4074...
No rollback image present
```

---

## Forgot to include
# `cowsay`

---

```shell
$ sudo dnf install -y cowsay
Updating and loading repositories:

```
<!-- element class="fragment fade-in" -->
```shell
...
Transaction Summary:
 Installing:         1 package
...
```
<!-- element class="fragment fade-in" -->
```shell
Running transction
Transaction failed: Rpm transaction failed.
  - installing package cowsay needs 376KB more space on the / filesystem
```
<!-- element class="fragment fade-in" -->

---

# Immutable

---

## As a virtual machine

### this runs as an immutable environment

---

### `ostree`
### Version control system on an entire filesystem

---

### `/etc`, `/var`
## to store configuration, state, ...

---

# Instead we 
# update the
# `Containerfile`

---

```Dockerfile
FROM quay.io/fedora/fedora-bootc

...

RUN dnf install -y cowsay
```

---

# ... build the new container image

---

```shell
$ podman build -t ghcr.io/.../fedora-bootc \
	-f Containerfile .
```

---

# ... we push this new container image

---

```shell
$ podman push ghcr.io/.../fedora-bootc
```

---

# ... and update the virtual machine

---

```shell
$ sudo bootc update
layers already present: 65; layers needed: 2 (117.6 MB)


```

---

```shell
$ sudo bootc update
layers already present: 65; layers needed: 2 (117.6 MB)
Fetching layers ██████████░░░░░░░░░░ 1/2
 └ Fetching ███░░░░░░░░░░░░░░░░░ 18.11 MiB/112.12 MiB
```

---

```shell
$ sudo bootc update
layers already present: 65; layers needed: 2 (117.6 MB)
Fetching layers ████████████████████ 2/2 
 └ Fetching ████████████████████ 112.12 MiB/112.12 MiB
```

---

```shell
$ sudo bootc update
layers already present: 65; layers needed: 2 (117.6 MB)
Fetched layers: 112.12 MiB in 21 seconds (5.39 MiB/s)
⠖ Deploying
```

---

```shell
$ sudo bootc update
layers already present: 65; layers needed: 2 (117.6 MB)
Fetched layers: 112.12 MiB in 21 seconds (5.39 MiB/s)
  Deploying: done (4 seconds)
Queued for next boot: ghcr.io/.../fedora-bootc:latest
  Version: 41.20250119.0
  Digest: sha256:990c...
Total new layers: 67    Size: 948.3 MB
Removed layers:   1     Size: 1.8 kB
Added layers:     2     Size: 117.6 MB
```

---

# Reboot

---

```shell
$ sudo bootc status
Current staged image: ghcr.io/.../fedora-bootc:latest
    Image version: 41.2025019.0 (...)
    Image digest: sha256:990c...
...
$ sudo reboot
```

---

# Success!

---

```shell
$ cowsay Moo
```

```text
	 _____
	< Moo >
	 -----
	        \   ^__^
	         \  (oo)\_______
	            (__)\       )\/\
	                ||----w |
	                ||     ||
```

---

# Rollback

```shell
$ sudo bootc status
No staged image present
Current booted image: ghcr.io/.../fedora-bootc:latest
    Image version: 41.2025019.0 (...)
    Image digest: sha256:990c...
Current rollback image: ghcr.io/.../fedora-bootc:latest
    Image version: 41.2025019.0 (...)
    Image digest: sha256:4074...
```

---

# Do I need a 
# non-bootc
# container ?

---

```shell
$ sudo bootc status
System is not deployed via bootc.
```

---

# Mutable

---

`.devcontainer/devcontainer.json`
```json
{
    "name": "Fedora bootc example",
    "image": "ghcr.io/.../fedora-bootc:latest",

    "runArgs": [  ],

    "overrideCommand": false,
	"privileged": true,
    "remoteUser": "admin"
}
```

---

# There is more

---

### export different disk image types
<!-- element class="fragment fade-in" -->
### target different architecture
<!-- element class="fragment fade-in" -->
### `config.toml` for customization
<!-- element class="fragment fade-in" -->
### Fedora, CentOS, RHEL
<!-- element class="fragment fade-in" -->
### ...
<!-- element class="fragment fade-in" -->

---

# Resources

- https://github.com/gbraad-redhat/fedora-bootc-example
- https://github.com/gbraad-docs/presentation-podman-libvirt

- https://containers.github.io/bootc/
- https://osbuild.org/docs/bootc/
- https://gitlab.com/fedora/bootc/examples
- https://gitlab.com/fedora/bootc/base-images
