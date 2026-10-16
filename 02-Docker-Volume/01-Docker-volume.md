## 1. What is a Docker Volume?

A Docker volume is a Docker-managed storage object that can persist data independently of a container's writable layer.

In simple terms, a container is where an application runs, and a volume is where the application can store data that must survive container replacement.

## 2. Docker Storage Architecture

### Layer 1: Image Layers

Read-only filesystem layers containing the application, runtime, libraries and base operating system.

### Layer 2: Container Writable Layer

Container-specific changes, including files created or modified during execution.

### Layer 3: Mounted Storage

Volumes, bind mounts or other mounts that expose storage at a container path.

### 2.1 Image layers

Docker images use layered filesystems. The layers are generally read-only and shared between containers created from the same image.

For example, an image may contain:

- A base operating system.
- Installed packages.
- A Python or Node.js runtime.
- Application source code.
- Default configuration files.

### 2.2 Container writable layer

When Docker creates a container, it adds a writable layer above the image layers.

Run the following commands:

    docker run -d --name storage-demo alpine sleep infinity

Create a file:

    docker exec storage-demo sh -c \
      'echo "Docker storage test" > /tmp/demo.txt'

Read the file:

    docker exec storage-demo cat /tmp/demo.txt

Remove the container:

    docker rm -f storage-demo

The file is lost because it was stored in the container's writable layer.

### 2.3 Mounted storage

Create a volume:

    docker volume create app_data

Start a container with the volume:

    docker run -d \
      --name storage-demo \
      --mount type=volume,source=app_data,target=/data \
      alpine sleep infinity

Write data:

    docker exec storage-demo sh -c \
      'echo "Persistent data" > /data/demo.txt'

Remove the container and recreate access to the volume:

    docker rm -f storage-demo

    docker run --rm \
      --mount type=volume,source=app_data,target=/data,readonly \
      alpine cat /data/demo.txt

Expected output:

    Persistent data

This demonstrates the separation between the container lifecycle and the data lifecycle.
