# Distro Images in Docker

Distro images are Docker images that contain a full Linux distribution environment.

Instead of only an application runtime, they include the base OS utilities, package manager, and system libraries.

---

## Simple Definition

A distro image = Docker image based on a Linux distribution like:

- Ubuntu
- Debian
- Alpine
- CentOS
- Fedora

These images provide the operating system layer for containers.

---

## Example Distro Images

### Ubuntu

    FROM ubuntu:22.04

### Debian

    FROM debian:12

### Alpine (very popular in containers)

    FROM alpine:3.19

These contain:

- Linux filesystem
- Package manager
- Basic system utilities

---

## Example: Using a Distro Image

    FROM ubuntu:22.04

    RUN apt update && apt install -y curl

    CMD ["bash"]

Here:

- `ubuntu:22.04` = distro image
- You install packages using `apt`.

---

## Why Distro Images Are Used

They give you:

- Package manager
- System libraries
- Debugging tools
- Shell utilities

### Example tools available

    bash
    apt
    curl
    vim
    net-tools
    ps
    top

---

## Common Distro Images in Docker

| Image | Size | Package Manager |
|---|---|---|
| Ubuntu | ~70MB | apt |
| Debian | ~50MB | apt |
| Alpine | ~7MB | apk |
| CentOS | ~200MB | yum |
| Fedora | ~200MB | dnf |

Alpine is most popular for containers because it is very small.

---

## Distro vs Runtime Images

### Distro Image

Contains a full OS environment.

Examples:

- Ubuntu
- Debian
- Alpine
- CentOS

### Runtime Image

Contains only the runtime for a language.

Examples:

- Node
- Python
- OpenJDK
- Golang

### Example

    FROM node:20-alpine

Here:

- `node` = runtime
- `alpine` = distro

---

## Container Filesystem Example

    Docker Image
    │
    ├── Linux distro (Alpine)
    ├── System libraries
    ├── Node runtime
    └── Application code
