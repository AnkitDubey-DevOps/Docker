##  `-v` vs. `--mount`

Docker supports two common syntaxes for mounting storage.

### Short syntax: `-v`

    docker run -d \
      --name app1 \
      -v app_data:/data \
      alpine sleep infinity

General format:

    -v SOURCE:DESTINATION[:OPTIONS]

Read-only example:

    docker run --rm \
      -v app_data:/data:ro \
      alpine ls -la /data

### Explicit syntax: `--mount`

    docker run -d \
      --name app2 \
      --mount type=volume,source=app_data,target=/data \
      alpine sleep infinity

### Comparison

| Feature | `-v` | `--mount` |
|---|---|---|
| Syntax | Short, colon-separated | Explicit key-value pairs |
| Readability | Convenient for simple commands | Clearer for complex mounts |
| Missing bind source | Can create a directory | Normally returns an error |
| Production documentation | Useful for compact commands | Often preferable for clarity |

## Sharing a volume between multiple containers

Create a volume:

    docker volume create shared_data

Start a writer:

    docker run -d \
      --name writer \
      --mount type=volume,source=shared_data,target=/data \
      alpine sleep infinity

Start a read-only reader:

    docker run -d \
      --name reader \
      --mount type=volume,source=shared_data,target=/data,readonly \
      alpine sleep infinity

Write from the first container:

    docker exec writer sh -c \
      'echo "Shared file" > /data/shared.txt'

Read from the second:

    docker exec reader cat /data/shared.txt

The reader should display:

    Shared file

### Advanced consideration: Concurrent access

Mounting a volume into multiple containers does not guarantee that concurrent writes are safe.

For example:

- Two containers appending to a log file may require coordination.
- Two database instances must not independently manipulate the same database directory unless the database architecture explicitly supports that configuration.
- Network filesystem locking and consistency semantics depend on the storage backend.

**Interview principle:** Shared storage and safe concurrent access are different concepts.
