# Docker Volume Commands in Detail

## 6. Create, list, inspect and remove volumes

These commands form the foundation of day-to-day Docker volume administration.

### 6.1 Create a volume

    docker volume create app_data

With labels:

    docker volume create \
      --label project=college \
      --label environment=development \
      app_data

Labels provide metadata for organization and automation. They do not enforce access control by themselves.

### 6.2 List volumes

    docker volume ls

Only display volume names or identifiers:

    docker volume ls -q

Filter by driver:

    docker volume ls --filter driver=local

Filter by name:

    docker volume ls --filter name=app

Find unused volumes:

    docker volume ls --filter dangling=true

Unused does not necessarily mean unimportant. A volume that is not currently attached to a container can still contain valuable historical data.

### 6.3 Inspect a volume

    docker volume inspect app_data

Extract selected fields:

    docker volume inspect \
      --format '{{ .Name }}' app_data

    docker volume inspect \
      --format '{{ .Driver }}' app_data

    docker volume inspect \
      --format '{{ .Mountpoint }}' app_data

The `Mountpoint` field is useful for troubleshooting, especially on a Linux Docker host. It is not a guarantee that manually modifying the underlying directory is safe or supported.

### 6.4 Inspect container mounts

    docker inspect app1

To display only mount information:

    docker inspect app1 \
      --format '{{json .Mounts}}'

Check whether a particular volume is referenced:

    docker ps -a --filter volume=app_data

These commands help answer questions such as:

- Is the expected volume attached?
- Is the mount target correct?
- Is the mount read-only?
- Which containers reference the volume?

### 6.5 Remove a volume

    docker volume rm app_data

Docker normally refuses to remove a volume that is still referenced by a container.

Check references first:

    docker ps -a --filter volume=app_data

After validating that the data is no longer required and the volume can be safely retired, remove it.

### 6.6 Prune unused volumes

    docker volume prune

To include unused named volumes:

    docker volume prune -a

Production warning: Volume pruning is destructive. Before using it, verify ownership, retention requirements, backup status and whether the data is still required by any application.
