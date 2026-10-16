# Docker Storage Types

1. Mount type: How storage is exposed to a container.
2. Volume naming: Whether a volume has an explicit name.
3. Volume driver: Which storage backend implements the volume.

## Volume vs. Bind Mount vs. tmpfs

| Feature | Volume | Bind mount | tmpfs |
|---|---|---|---|
| Storage management | Docker volume subsystem | Host filesystem | Memory-backed filesystem |
| Source location | Driver-managed storage | Explicit host path | No normal host directory |
| Persistence after container removal | Usually retained | Host files remain | Temporary data is lost |
| Typical use | Database and application data | Source code and configuration | Temporary intermediate data |
| Main operational concern | Lifecycle and backup | Host access and permissions | Memory usage and temporary lifecycle |

### 4.1 Named volume

A named volume has an explicit identifier.

    docker volume create college_data

Inspect it:

    docker volume inspect college_data

Attach it:

    docker run -d \
      --name app1 \
      --mount type=volume,source=college_data,target=/data \
      alpine sleep infinity

Write a file:

    docker exec app1 sh -c \
      'echo "Student records" > /data/records.txt'

### 4.2 Anonymous volume

An anonymous volume is a volume without an explicitly assigned name.

    docker run -d \
      --name app2 \
      --mount type=volume,target=/data \
      alpine sleep infinity

Inspect the generated volume name:

    docker inspect app2 --format '{{json .Mounts}}'

### 4.3 Bind mount

A bind mount exposes an existing host path inside the container.

    mkdir -p "$(pwd)/project-data"

    docker run --rm \
      --mount type=bind,source="$(pwd)/project-data",target=/workspace \
      alpine sh -c 'echo "Created by container" > /workspace/example.txt'

Check the file on the host:

    cat project-data/example.txt

The host and container are accessing the same underlying file.

### 4.4 tmpfs mount

A tmpfs mount provides temporary memory-backed filesystem storage on supported Linux configurations.

    docker run -d \
      --name temp-app \
      --mount type=tmpfs,target=/run/temp,tmpfs-size=32m,tmpfs-mode=0700 \
      alpine sleep infinity

Write and read a file:

    docker exec temp-app sh -c \
      'echo "Temporary data" > /run/temp/example.txt'

    docker exec temp-app cat /run/temp/example.txt

Remove the container:

    docker rm -f temp-app

The tmpfs data does not survive the mount's lifecycle.
