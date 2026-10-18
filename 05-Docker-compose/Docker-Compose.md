## 1. What is Docker Compose?

Definition: Docker Compose is a tool used to define, configure, and run multi-container applications using a YAML configuration file, usually named `compose.yaml`.

Easy example: Suppose your application has three components:

- Frontend — Nginx
- Backend — Node.js
- Database — MySQL

Without Compose, you would run and configure each container separately. With Docker Compose, you define all the services in one file and start them using a single command.

compose.yaml

Services, networks, volumes, ports and configuration

Docker Compose

Creates and manages the application resources

Frontend

Backend

Database

Interview answer: Docker Compose simplifies multi-container application management by defining services, networking, storage, and configuration in one YAML file.

## 2. Why do we use Docker Compose?

The main benefits are:

1. Multiple containers: Manage several application services together.
2. Single configuration file: Define application settings in one place.
3. Easy startup: Start the application with `docker compose up -d`.
4. Networking: Services can communicate using their service names.
5. Persistent storage: Define named volumes for database and application data.
6. Repeatability: Use the same configuration across development, testing, and other environments, with appropriate environment-specific settings.
7. Simplified cleanup: Stop and remove the application's containers and network using one command.

## 3. Important Docker Compose concepts

| Concept | Easy definition |
|---|---|
| `services` | Defines the containers or application services to run. |
| `image` | Specifies the image used to create a container. |
| `build` | Specifies how to build an image from a Dockerfile. |
| `ports` | Maps host ports to container ports. |
| `environment` | Defines environment variables inside a container. |
| `volumes` | Mounts persistent storage or host paths into containers. |
| `networks` | Defines how services communicate with each other. |
| `depends_on` | Specifies service startup and dependency relationships. |
| `restart` | Defines when Docker should restart a container. |
| `healthcheck` | Defines how to check whether a service is healthy. |

These are the concepts you should understand before moving to advanced Compose configurations.

## 4. Practical example: Create a Docker Compose file

Let's run two services: Nginx and Redis.

Create a file named `compose.yaml`:

    services:
      web:
        image: nginx:alpine
        ports:
          - "8080:80"
        depends_on:
          - redis
        restart: unless-stopped

      redis:
        image: redis:7-alpine
        volumes:
          - redis_data:/data
        command: ["redis-server", "--appendonly", "yes"]
        restart: unless-stopped

    volumes:
      redis_data:

### Explanation of each important line

- `services`: Defines the application's services.
- `web` and `redis`: Service names.
- `image`: Specifies which Docker image to use.
- `ports`: Publishes host port `8080` to container port `80`.
- `depends_on`: Starts Redis before starting the web service, but does not by itself wait for Redis to become ready.
- `restart`: Restarts the container according to the specified policy.
- `volumes`: Persists Redis data in a named volume.
- `command`: Configures Redis to use append-only persistence.

Compose creates a default network for the project, so the services can communicate using their service names.

### Commands to run the application

    # Start services in the background
    docker compose up -d

    # Check running services
    docker compose ps

    # View logs
    docker compose logs

    # Follow logs continuously
    docker compose logs -f

    # Stop and remove project containers and network
    docker compose down

After starting the services, open `http://localhost:8080` on the Docker host to access Nginx.

Interview point: `docker compose up -d` starts the application in the background, whereas `docker compose down` removes the project containers and network. The named Redis volume normally remains.

## 5. Most important Docker Compose commands

| Command | Definition |
|---|---|
| `docker compose up` | Creates and starts the services. |
| `docker compose up -d` | Starts the services in the background. |
| `docker compose down` | Stops and removes project containers and networks. |
| `docker compose ps` | Shows the project's containers and status. |
| `docker compose logs` | Displays service logs. |
| `docker compose logs -f` | Streams service logs. |
| `docker compose stop` | Stops services without removing their containers. |
| `docker compose start` | Starts previously created, stopped containers. |
| `docker compose restart` | Restarts service containers. |
| `docker compose build` | Builds or rebuilds images for services with build configurations. |
| `docker compose pull` | Pulls configured images from registries. |
| `docker compose exec web sh` | Opens a shell in a running service container, if a shell is available. |
| `docker compose config` | Validates and renders the resolved Compose configuration. |

Important distinction:

- `stop` stops containers but keeps them.
- `down` removes project containers and networks.
- `down -v` also removes the project's declared named and anonymous volumes. Use it carefully because stored data can be lost.

## 6. Docker Compose vs Dockerfile

This is a very common interview question.

| Dockerfile | Docker Compose |
|---|---|
| Defines how to build an image | Defines how to configure and run multiple services |
| Contains instructions such as `FROM`, `RUN`, and `COPY` | Contains configuration such as `services`, `ports`, and `volumes` |
| Used by `docker build` | Used by `docker compose up` |
| Primarily describes image creation | Primarily describes application orchestration on a Docker host |

Interview-ready answer:

"A Dockerfile defines how to build a container image. Docker Compose defines how to run and connect application services, including their ports, networks, volumes, and environment variables."

## 7. `image` vs `build` in Docker Compose

### `image`

Uses an existing image, pulled from a registry if necessary.

    services:
      web:
        image: nginx:alpine

### `build`

Builds an image using a Dockerfile and a build context.

    services:
      app:
        build:
          context: .
          dockerfile: Dockerfile

Start the application:

    docker compose up -d --build

This tells Compose to build the configured images before starting the services.

Interview answer: Use `image` when you want to run a specified image. Use `build` when you want Compose to build an image from your application's source and Dockerfile. You can specify both when you want to build an image and assign it a particular image name.

## 8. `depends_on` in Docker Compose

Definition: `depends_on` specifies the order in which Compose starts services based on their dependencies.

Example:

    services:
      db:
        image: redis:7-alpine
        healthcheck:
          test: ["CMD", "redis-cli", "ping"]
          interval: 5s
          timeout: 3s
          retries: 5

      app:
        image: nginx:alpine
        depends_on:
          db:
            condition: service_healthy

In this example, Compose waits for the `db` service to pass its health check before starting `app`.

Very important interview question: Does `depends_on` guarantee that the database is ready?

Answer: Not by itself. Basic `depends_on` controls startup order. Using `condition: service_healthy` with a correctly configured health check allows Compose to wait for the dependency to become healthy before starting the dependent service.

The application should still handle connection failures and reconnections where appropriate.

## 9. Environment variables in Docker Compose

Definition: Environment variables configure application behavior without hardcoding every setting into the image.

Example:

    services:
      app:
        image: myapp:latest
        environment:
          APP_ENV: production
          LOG_LEVEL: info

To provide values through a `.env` file beside the Compose file:

    APP_ENV=production
    LOG_LEVEL=info

Reference them in `compose.yaml`:

    services:
      app:
        image: myapp:latest
        environment:
          APP_ENV: ${APP_ENV}
          LOG_LEVEL: ${LOG_LEVEL}

Interview point: Compose `.env` files are commonly used for variable substitution. They are not automatically a secure secrets manager. Avoid committing sensitive credentials to source control; use suitable secrets management for production.

## 10. Volumes and networking in Docker Compose

These two concepts are especially important for DevOps interviews.

### Volumes

Definition: Compose volumes allow services to store persistent data outside their container writable layers.

    services:
      redis:
        image: redis:7-alpine
        volumes:
          - redis_data:/data

    volumes:
      redis_data:

Here, `redis_data` is a named volume mounted at `/data` inside the Redis container.

### Networks

Definition: Compose networks allow services to communicate with one another.

    services:
      frontend:
        image: nginx:alpine
        networks:
          - app_net

      backend:
        image: nginx:alpine
        networks:
          - app_net

    networks:
      app_net:

Both services share the `app_net` network and can resolve each other by service name.

Interview answer: Volumes provide persistent storage, while networks provide communication between services.

## 11. Restart policies

Definition: A restart policy determines whether Docker should restart a container when it exits or when certain Docker daemon restart conditions occur.

| Policy | Meaning |
|---|---|
| `no` | Do not automatically restart the container. |
| `always` | Always restart, subject to Docker's restart behavior. |
| `unless-stopped` | Restart unless the container has been explicitly stopped. |
| `on-failure` | Restart when the container exits with a non-zero status, optionally with a retry limit. |

Example:

    services:
      app:
        image: nginx:alpine
        restart: unless-stopped

Interview point: A restart policy is not the same as a health check. A container can be running but unhealthy, and a restart policy alone does not necessarily restart it just because its health check fails.
