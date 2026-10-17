## 1. Docker Volume

Definition: A Docker volume is a Docker-managed storage mechanism used to store data outside a container's writable layer. The data can remain available even after the container is removed.

Command: Create a volume and use it

    # Create a volume
    docker volume create app_data

    # Run a container using the volume
    docker run -d --name app1 \
      -v app_data:/data \
      alpine sleep infinity

    # Write data to the volume
    docker exec app1 sh -c 'echo "Hello DevOps" > /data/test.txt'

    # Read the data
    docker exec app1 cat /data/test.txt

Expected output:

    Hello DevOps

Interview answer: Docker volumes are used to persist application and database data independently of container lifecycle.

## 2. Named Volume

Definition: A named volume is a Docker-managed volume with a user-defined name. It is useful for storing persistent application or database data.

Commands:

    # Create
    docker volume create mysql_data

    # List volumes
    docker volume ls

    # Inspect volume
    docker volume inspect mysql_data

    # Attach to a container
    docker run -d --name db1 \
      -v mysql_data:/var/lib/mysql \
      mysql:8

Here, `mysql_data` is the volume name and `/var/lib/mysql` is the path inside the container.

Interview question: What happens if you delete the container?

Answer: The named volume normally remains, and a new container can reuse it.

    docker rm -f db1

    # The volume still exists
    docker volume ls

Note: This example illustrates mounting storage. A real MySQL deployment should also configure credentials and other required settings.

## 3. Anonymous Volume

Definition: An anonymous volume is a Docker-managed volume that has an automatically generated name rather than a name chosen by the user.

Command:

    docker run -d --name app2 \
      -v /data \
      alpine sleep infinity

Docker creates an anonymous volume and mounts it at `/data`.

Inspect the mount:

    docker inspect app2

Interview answer: Named volumes have user-defined names, whereas anonymous volumes receive generated identifiers. Both can persist after ordinary container removal.

## 4. Bind Mount

Definition: A bind mount maps a specific file or directory from the host machine into a container.

Command:

    # Create a host directory
    mkdir -p ~/docker-project

    # Create a file on the host
    echo "Hello from host" > ~/docker-project/index.html

    # Mount it into the container
    docker run -d --name web \
      --mount type=bind,source="$HOME/docker-project",target=/usr/share/nginx/html \
      nginx:alpine

Verify the file:

    docker exec web cat /usr/share/nginx/html/index.html

Expected output:

    Hello from host

Interview answer: Bind mounts are commonly used in development to share source code, configuration files, or other host files with a container.

Important difference: A volume is managed by Docker; a bind mount uses a specific host path.

## 5. Volume Persistence

Definition: Volume persistence means that data remains available after the container that wrote it has been removed, provided the volume itself has not been deleted.

Practical commands:

    # Create a named volume
    docker volume create persistent_data

    # Start a container
    docker run -d --name test1 \
      -v persistent_data:/data \
      alpine sleep infinity

    # Write a file
    docker exec test1 sh -c 'echo "Persistent data" > /data/file.txt'

    # Remove the container
    docker rm -f test1

    # Start a new container with the same volume
    docker run --rm \
      -v persistent_data:/data \
      alpine cat /data/file.txt

Expected output:

    Persistent data

Interview answer: Docker volumes allow data to survive container removal, making them useful for databases and stateful applications.

## 6. Volume Sharing Between Containers

Definition: Volume sharing means mounting the same volume into multiple containers so that they can access the same stored data.

Commands:

    # Create the shared volume
    docker volume create shared_data

    # Start the first container
    docker run -d --name app1 \
      -v shared_data:/data \
      alpine sleep infinity

    # Write data
    docker exec app1 sh -c 'echo "Shared content" > /data/file.txt'

    # Start the second container
    docker run -d --name app2 \
      -v shared_data:/data \
      alpine sleep infinity

    # Read the same data
    docker exec app2 cat /data/file.txt

Expected output:

    Shared content

Interview answer: Multiple containers can mount the same volume, but applications must handle concurrent access and file permissions correctly.

## 7. Volume Backup and Restore

Definition: A volume backup is a copy of stored data saved separately for recovery. Restoring means extracting that copy into a volume.

### Backup a volume

Assume the volume `persistent_data` already contains files.

    docker run --rm \
      --mount type=volume,source=persistent_data,target=/data,readonly \
      --mount type=bind,source="$(pwd)",target=/backup \
      alpine \
      tar -czf /backup/backup.tar.gz -C /data .

This creates `backup.tar.gz` in the current host directory.

### Restore into a new volume

    # Create the destination volume
    docker volume create restored_data

    # Restore the backup
    docker run --rm \
      --mount type=volume,source=restored_data,target=/data \
      --mount type=bind,source="$(pwd)",target=/backup,readonly \
      alpine \
      tar -xzf /backup/backup.tar.gz -C /data

Interview answer: Docker volumes do not automatically provide backups. We must create and test a backup and recovery process separately.

For databases, use database-consistent backup procedures when necessary.

## 8. Docker Volume Commands You Must Remember

| Command | Purpose |
|---|---|
| `docker volume create mydata` | Create a volume |
| `docker volume ls` | List all volumes |
| `docker volume inspect mydata` | View volume details |
| `docker volume rm mydata` | Delete an unused volume |
| `docker volume prune` | Remove unused anonymous volumes by default |
| `docker volume prune -a` | Remove unused named and anonymous volumes |
| `docker inspect mycontainer` | Inspect container mount information |
| `docker rm -f mycontainer` | Force-remove a container |
| `docker exec -it mycontainer sh` | Open a shell in a running container |

Warning: Volume pruning and deletion can permanently destroy data. Verify which volumes will be removed before confirming.

## 9. Bonus: Docker Compose Volume

Definition: Docker Compose allows you to define volumes in a YAML file alongside the services that use them.

Example `compose.yaml`:

    services:
      app:
        image: nginx:alpine
        volumes:
          - app_data:/usr/share/nginx/html

    volumes:
      app_data:

Commands:

    # Start the service
    docker compose up -d

    # List volumes
    docker volume ls

    # Remove containers and the project network
    docker compose down

The named volume normally remains after `docker compose down`.

    # Remove the Compose-managed volumes too
    docker compose down -v

Be careful: `down -v` can delete the stored data.
