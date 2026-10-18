## 1. What is Docker Networking?

Definition: Docker networking allows containers to communicate with one another, access external networks, and expose application services to the host machine or outside users.

Example: Suppose you have a frontend container and a backend container. Docker networking allows the frontend to communicate with the backend API.

Frontend container

Web application

Docker Network

Container-to-container communication

Backend container

API service

Interview answer: Docker networking provides connectivity between containers and allows controlled communication between containers, the host, and external networks.

## 2. The 4 important Docker network drivers

Docker has several network drivers, but these are the four you should prioritize for interviews.

### 1. Bridge network

Definition: A bridge network connects containers through a virtual network on a single Docker host. User-defined bridge networks support automatic DNS-based container discovery.

Use case: Connecting a frontend and backend on the same host.

Priority: Very high

### 2. Host network

Definition: Host networking allows a container to use the host's network namespace directly, rather than receiving its own isolated network namespace.

Use case: Workloads that need direct access to the host's network interfaces.

Priority: High

### 3. None network

Definition: The `none` network driver disables normal container networking, leaving the container without ordinary external network connectivity.

Use case: Workloads that should not require network access.

Priority: Medium

### 4. Overlay network

Definition: An overlay network connects containers across multiple Docker hosts. It is commonly used with Docker Swarm services.

Use case: Applications distributed across multiple hosts.

Priority: High

### Commands for network drivers

    # Create a bridge network
    docker network create --driver bridge my_bridge

    # Run a container using host networking (Linux)
    docker run --network host nginx

    # Run a container without normal networking
    docker run --network none alpine

    # Create an overlay network for Swarm
    docker swarm init
    docker network create --driver overlay my_overlay

Important: Overlay networking for Swarm requires a Swarm setup. Host networking also behaves differently across operating systems; on Docker Desktop, it has platform-specific limitations.

## 3. Bridge network — most important for interviews

Definition: A bridge network is a virtual network that allows containers on the same Docker host to communicate with each other.

Docker has a default bridge network and supports user-defined bridge networks.

### Create and use a bridge network

    # Create a user-defined bridge network
    docker network create my_bridge

    # Run the first container
    docker run -dit --name app1 \
      --network my_bridge \
      alpine sh

    # Run the second container
    docker run -dit --name app2 \
      --network my_bridge \
      alpine sh

Test communication:

    # Execute a command inside app1
    docker exec app1 ping -c 3 app2

If `ping` is available and ICMP traffic is permitted, `app2` should resolve by its container name and respond.

Interview answer: A user-defined bridge network allows containers to communicate using container names through Docker's embedded DNS.

### Default bridge vs user-defined bridge

| Default bridge | User-defined bridge |
|---|---|
| Created automatically by Docker | Created by the user |
| Containers generally need IP addresses or legacy linking for discovery | Containers can discover each other by name |
| Less flexible for application isolation | Supports better network organization |
| Suitable for basic networking | Recommended for most applications on one host |

Remember: Containers on the same user-defined bridge can communicate without publishing their ports to the host, provided the application is listening and firewall rules allow the traffic.

## 4. Port mapping: `-p`

Definition: Port mapping publishes a container's port on a host port so that applications outside the container can access the service.

Example:

    docker run -d --name web \
      -p 8080:80 \
      nginx

Here:

- `8080` is the host port.
- `80` is the container port.
- Nginx listens on port `80` inside the container.
- You can access the service through `http://localhost:8080` on the Docker host.

Host machine

## Port 8080

Docker container

## Port 80

Nginx web server

### More port-mapping commands

    # Map host port 8080 to container port 80
    docker run -d -p 8080:80 nginx

    # Map a specific host IP and port
    docker run -d -p 127.0.0.1:8080:80 nginx

    # Show published ports
    docker ps

    # Inspect container port mappings
    docker port web

Interview question: What is the difference between `EXPOSE` and `-p`?

- `EXPOSE` documents the port the application is expected to use; it does not publish the port by itself.
- `-p` publishes a container port on the host.

Example Dockerfile:

    FROM nginx:alpine
    EXPOSE 80

Even if the Dockerfile contains `EXPOSE 80`, you can still need `-p 8080:80` to make the service accessible through the host's port.

## 5. Host networking

Definition: Host networking allows a container to share the host's network namespace instead of using a separate container network namespace.

Example on a compatible Linux Docker Engine:

    docker run -d --name host-nginx \
      --network host \
      nginx

In this mode, Nginx uses the host's network stack and listens on its configured port. The usual `-p` port publishing is unnecessary and is ignored for host-networked containers.

Advantages:

- Avoids the separate container network namespace.
- Can be useful for applications that need direct host-network access.

Disadvantages:

- Less network isolation.
- Port conflicts with host services are possible.
- Platform support varies.

Interview answer: Bridge networking provides network isolation for containers, while host networking shares the host's network namespace and does not use the usual port-mapping mechanism.

## 6. None networking

Definition: The `none` network driver gives a container no normal Docker network connectivity.

Command:

    docker run -dit --name isolated \
      --network none \
      alpine sh

Inspect the container:

    docker inspect isolated

Use case: Workloads that need to operate without normal network connectivity.

Interview answer: The `none` driver is used when a container does not need normal networking. It is different from a bridge network because it does not provide ordinary connectivity to other containers or external networks.

## 7. Overlay networking

Definition: An overlay network enables communication between containers or services running on different Docker hosts.

It is particularly relevant to Docker Swarm.

Example:

    # Initialize Swarm on the manager
    docker swarm init

    # Create an overlay network
    docker network create \
      --driver overlay \
      --attachable \
      app_overlay

    # Create a service on that network
    docker service create \
      --name web \
      --network app_overlay \
      --replicas 2 \
      nginx

Use case: Multi-host container deployments where services must communicate across machines.

Interview answer: Bridge networking is primarily for containers on one host, whereas overlay networking supports communication across multiple hosts in a Swarm environment.

Note: A standalone overlay network generally requires Swarm initialization. With `--attachable`, standalone containers can also join the network.

## 8. Docker network commands you must know

| Command | Definition |
|---|---|
| `docker network ls` | Lists Docker networks |
| `docker network create mynet` | Creates a bridge network by default |
| `docker network inspect mynet` | Displays network configuration and connected containers |
| `docker network connect mynet app1` | Connects a running container to a network |
| `docker network disconnect mynet app1` | Disconnects a container from a network |
| `docker network rm mynet` | Removes an unused network |
| `docker network prune` | Removes unused custom networks |
| `docker inspect app1` | Shows container configuration, including networking details |
| `docker exec -it app1 sh` | Opens a shell inside a running container |

### Quick practical example

    # Create a network
    docker network create backend_net

    # Start two containers
    docker run -dit --name app1 \
      --network backend_net alpine sh

    docker run -dit --name app2 \
      --network backend_net alpine sh

    # Inspect the network
    docker network inspect backend_net

    # Disconnect app2
    docker network disconnect backend_net app2

    # Remove containers
    docker rm -f app1 app2

    # Remove the unused network
    docker network rm backend_net

## 9. Container-to-container communication

Definition: Container-to-container communication allows applications running in different containers to exchange data over a shared Docker network.

Example: A frontend container communicates with a backend container.

    docker network create app_net

    docker run -d --name backend \
      --network app_net \
      nginx:alpine

    docker run --rm \
      --network app_net \
      curlimages/curl \
      http://backend

The second container sends an HTTP request to the backend using its container name.

Interview point: Containers on a user-defined bridge network can communicate by container name. They do not need published host ports for this internal communication.

## 10. Docker DNS

Definition: Docker DNS resolves container and service names into IP addresses on networks that support Docker's embedded DNS.

For example, if the backend container is named `backend`, another container on the same user-defined bridge network can access it using `http://backend`.

    docker network create app_net

    docker run -d --name backend \
      --network app_net \
      nginx:alpine

    docker run --rm \
      --network app_net \
      busybox nslookup backend

Interview question: Why use container names instead of IP addresses?

Answer: Container IP addresses can change when containers are recreated. Using a stable service or container name avoids hardcoding an IP address.

## 11. Docker Compose networking

Definition: Docker Compose creates and manages networks for services declared in a Compose file. By default, services in the same Compose project can communicate through a shared project network.

Example `compose.yaml`:

    services:
      frontend:
        image: nginx:alpine
        ports:
          - "8080:80"

      backend:
        image: nginx:alpine

Start the services:

    docker compose up -d

The frontend can reach the backend using the service name `backend` on its listening port.

    http://backend:80

Check the networks:

    docker network ls
    docker compose ps

Important: `localhost` inside a container refers to that same container, not another container. To connect to the backend, use its Compose service name.

Interview answer: Docker Compose simplifies networking by creating a project network and providing service-name-based discovery between services.

## 12. Docker network troubleshooting

These are common practical problems asked in DevOps interviews.

| Problem | Possible reason | What to check |
|---|---|---|
| Containers cannot communicate | They are on different networks | `docker network inspect` |
| Website is not accessible | Port not published or wrong port | `docker ps`, `docker port` |
| Backend hostname does not resolve | Wrong service name or no shared network | `docker exec`, `nslookup` |
| Connection refused | Application not listening on the expected port | Application logs and listening ports |
| Container cannot access the internet | DNS, routing, firewall, or host network issue | Container DNS, host connectivity, firewall |
| Container IP changes | Container was recreated | Use service names rather than fixed IPs |

Useful commands:

    # View all networks
    docker network ls

    # Inspect network configuration
    docker network inspect app_net

    # View container IP and network details
    docker inspect backend

    # Check container logs
    docker logs backend

    # Test DNS inside a container
    docker exec backend cat /etc/resolv.conf

Interview tip: First verify that the containers share a network, then check name resolution, the application's listening port, and firewall rules.
