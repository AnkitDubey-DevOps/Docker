# Most Asked Docker Interview Questions (with Easy Answers)

## Basics

### 1. What is Docker?

Docker is a tool that packages your application along with everything it needs (code, libraries, settings) into one box called a container. That box runs the same way on any machine: your laptop, a server, or the cloud. It solves the “it works on my machine” problem.

### 2. What is a container?

A container is a running, lightweight, isolated process that holds your app and its dependencies. It shares the host’s operating system kernel, so it starts in seconds and uses very little memory.

### 3. What is the difference between a container and a virtual machine?

- **VM:** Has its own full guest OS on top of a hypervisor. It is heavy (GBs) and slow to boot.
- **Container:** Shares the host OS kernel. It is light (MBs) and starts in seconds.

Simple line to say: “A VM virtualizes hardware, a container virtualizes the operating system.”

### 4. What is a Docker image?

An image is a read-only template used to create containers. It is built in layers. Think of it as a recipe or blueprint, and the container is the dish made from it.

### 5. Image vs Container?

An image is the blueprint (static). A container is a running instance of that image (live). You can create many containers from one image.

### 6. What is a Dockerfile?

A text file with step-by-step instructions to build an image, for example which base image to use, what to copy, what to install, and which command to run.

### 7. What is Docker Hub / a registry?

A registry is a place to store and share images. Docker Hub is the public one. Companies also use private registries like AWS ECR, Nexus, or Harbor.

### 8. What is the Docker architecture?

- **Docker Client:** The CLI where you type commands (`docker run`).
- **Docker Daemon (`dockerd`):** The background service that builds, runs, and manages containers.
- **Registry:** Where images are stored.

The client talks to the daemon through a REST API.

## Dockerfile Questions

### 9. Common Dockerfile instructions?

| Instruction | Meaning |
|---|---|
| `FROM` | Base image to start from |
| `WORKDIR` | Sets the working folder inside the container |
| `COPY` | Copies files from your machine into the image |
| `ADD` | Like COPY, but can also extract tar files and download from URLs |
| `RUN` | Runs a command while building the image |
| `CMD` | Default command when the container starts |
| `ENTRYPOINT` | The fixed main command of the container |
| `ENV` | Sets environment variables |
| `EXPOSE` | Documents which port the app uses |
| `ARG` | Build-time variable |
| `VOLUME` | Creates a mount point for persistent data |

### 10. CMD vs ENTRYPOINT?

`CMD` gives a default command that can be easily overridden when you run the container.

`ENTRYPOINT` is the main executable that always runs. Whatever you pass at runtime becomes its arguments.

Common practice: use ENTRYPOINT for the program and CMD for its default arguments.

### 11. COPY vs ADD?

Use `COPY` for simple copying. It is clear and predictable. `ADD` has extra magic (auto-extract tar, download from URL), so use it only when you need that.

### 12. RUN vs CMD?

`RUN` executes at build time and its result is saved in the image (for example installing packages). `CMD` executes at run time, when the container starts.

### 13. What is a multi-stage build and why use it?

You use multiple `FROM` statements in one Dockerfile. The first stage builds the app (with compilers and tools), and the final stage copies only the result into a small clean image.

**Benefit:** Much smaller and more secure images. For example, a Java app builds with Maven in stage 1 and runs with only a JRE in stage 2.

### 14. How do you reduce Docker image size?

- Use small base images (alpine, slim, distroless).
- Use multi-stage builds.
- Combine RUN commands to reduce layers.
- Clean up cache in the same layer (`apt-get clean`).
- Use a `.dockerignore` file.

### 15. What is `.dockerignore`?

Like `.gitignore`. It tells Docker which files not to send to the build (`node_modules`, `.git`, logs). This makes builds faster and images smaller.

### 16. What are image layers and how does caching work?

Every instruction in a Dockerfile creates a layer. Docker caches layers, and if nothing changed in a step, it reuses the cache. Once one layer changes, all layers after it are rebuilt.

Tip: Put things that rarely change (installing dependencies) before things that change often (copying source code).

## Networking

### 17. What network types does Docker have?

- **bridge (default):** Containers on the same host talk to each other.
- **host:** The container uses the host’s network directly (no isolation).
- **none:** No networking.
- **overlay:** Connects containers across multiple hosts (used in Swarm).
- **macvlan:** Gives the container its own MAC/IP on the network.

### 18. How do containers communicate with each other?

Put them on the same user-defined bridge network. They can then reach each other by container name (Docker has built-in DNS).

### 19. What is port mapping?

`-p 8080:80` means host port 8080 forwards to container port 80. This is how the outside world reaches your container.

### 20. EXPOSE vs -p?

`EXPOSE` is just documentation. It says “this app uses port 80.” `-p` actually publishes the port to the host.

## Storage

### 21. How do you persist data in Docker?

Container data is deleted when the container is removed. To keep it, use:

- **Volumes:** Best, managed by Docker.
- **Bind mounts:** Map a host folder into the container.
- **tmpfs:** Temporary, stored in memory.

### 22. Volume vs Bind Mount?

- **Volume:** Managed by Docker, safer, portable, preferred for production.
- **Bind mount:** You pick an exact host path. It is good for development (live code changes).

## Commands

### 23. Most-used Docker commands?

    docker build -t myapp:1.0 .      # build an image
    docker run -d -p 8080:80 myapp   # run in background with port mapping
    docker ps                        # running containers (-a for all)
    docker images                    # list images
    docker stop <id>                 # stop a container
    docker rm <id>                   # remove a container
    docker rmi <image>               # remove an image
    docker logs -f <id>              # view logs
    docker exec -it <id> bash        # get a shell inside a running container
    docker pull / docker push        # download / upload images
    docker inspect <id>              # detailed info
    docker system prune              # clean unused data

### 24. docker stop vs docker kill?

`stop` sends SIGTERM, which is a polite request to shut down, and waits about 10 seconds before forcing it. `kill` sends SIGKILL, which stops it immediately.

### 25. docker run vs docker start?

`run` creates a new container from an image and starts it. `start` restarts an existing stopped container.

### 26. docker exec vs docker attach?

`exec` opens a new process (such as a shell) inside a running container. `attach` connects to the container’s main process.

### 27. How do you see why a container crashed?

    docker logs <container>
    docker inspect <container> (check the exit code and state)
    docker ps -a (see the exit status)

Exit code 137 usually means killed (often out of memory). Exit code 0 means a normal exit.

## Docker Compose

### 28. What is Docker Compose?

A tool to define and run multi-container apps using one YAML file (`docker-compose.yml`). With one command (`docker compose up`) you start your app, database, and cache together.

### 29. Sample Compose file?

    services:
      web:
        build: .
        ports:
          - "8080:80"
        depends_on:
          - db
      db:
        image: postgres:16
        environment:
          POSTGRES_PASSWORD: secret
        volumes:
          - dbdata:/var/lib/postgresql/data
    volumes:
      dbdata:

### 30. Does depends_on wait until the database is ready?

No. It only controls start order, not readiness. To wait until a service is truly ready, use a healthcheck with `condition: service_healthy`.

## Orchestration and Advanced

### 31. What is Docker Swarm?

Docker’s built-in orchestration tool. It manages a cluster of machines, scales containers, and handles failures. (Most companies use Kubernetes instead.)

### 32. Docker Swarm vs Kubernetes?

Swarm is simpler and easier to set up. Kubernetes is more powerful, has a bigger ecosystem, and is the industry standard.

### 33. How do you handle secrets in Docker?

Never hardcode passwords in a Dockerfile or image. Use:

- Docker secrets (Swarm).
- Environment variables passed at runtime (less secure).
- External tools like Vault, AWS Secrets Manager, or Kubernetes Secrets.

### 34. How do you secure Docker containers?

- Use official and minimal base images.
- Run as a non-root user (`USER` instruction).
- Scan images for vulnerabilities (Trivy, Docker Scout).
- Keep images updated.
- Limit resources and capabilities.
- Don’t run in `--privileged` mode.
- Never store secrets in images.

### 35. How do you limit container resources?

    docker run --memory=512m --cpus=1 myapp

This stops one container from consuming all of the host’s resources.

### 36. What is a Docker healthcheck?

A command Docker runs periodically to check whether the app inside is actually working, not just whether the process is alive.

    HEALTHCHECK --interval=30s CMD curl -f http://localhost/ || exit 1

### 37. What are restart policies?

They control what happens when a container stops: `no` (default), `on-failure`, `always`, `unless-stopped`.

### 38. What happens to data when a container is deleted?

Anything written inside the container’s writable layer is lost. Data in volumes survives.

### 39. How do you tag and push an image to a registry?

    docker tag myapp:1.0 myregistry.com/team/myapp:1.0
    docker login myregistry.com
    docker push myregistry.com/team/myapp:1.0

### 40. What is the difference between docker-compose and docker compose?

`docker-compose` (with a hyphen) is the older standalone tool. `docker compose` is the newer version built into the Docker CLI.

## Scenario Questions (Interviewers Love These)

### 41. Your container keeps exiting immediately. What do you do?

Check `docker logs` first, then `docker ps -a` for the exit code. Common causes: the main process finished (containers live only as long as their main process), a wrong CMD, a missing environment variable, or out of memory. Run it with `docker run -it <image> sh` to debug interactively.

### 42. Your Docker image is 2 GB. How do you shrink it?

Switch to a slim or alpine base, use multi-stage builds, remove build tools and caches, add a `.dockerignore`, and merge RUN commands.

### 43. How do you debug a running container?

`docker logs -f`, `docker exec -it <id> sh`, `docker inspect`, `docker stats` (live CPU and memory), and `docker top`.

### 44. How do you do zero-downtime deployment with Docker?

Use an orchestrator (Swarm or Kubernetes) with rolling updates and healthchecks, or put a load balancer in front and replace containers one at a time.

### 45. Where does Docker fit in a CI/CD pipeline?

Code is pushed, then the pipeline builds a Docker image, runs tests inside it, scans it, pushes it to a registry (such as Nexus or ECR), and finally deploys that exact image to servers or Kubernetes. The same image moves through every environment.

## Quick Revision Cheat-Lines

- Image = blueprint, Container = running instance.
- Containers share the host kernel, so they are lighter than VMs.
- Use volumes for persistent data.
- Use multi-stage builds for small images.
- CMD can be overridden, ENTRYPOINT is the fixed command.
- Put rarely changing steps first in the Dockerfile to use the cache.
- Never run as root and never store secrets in images.


# Most Asked Scenario-Based Docker Interview Questions

## 1. Your container starts and exits immediately. What do you do?

A container lives only as long as its main process. If that process finishes or crashes, the container stops.

**Steps:**

- `docker logs <container>` to see the error.
- `docker ps -a` to check the exit code.
- Run `docker run -it <image> sh` to debug inside.

**Common causes:** Wrong `CMD`, missing environment variable, app crashed, or the app runs in the background so Docker thinks it finished.

---

## 2. Your Docker image is 2 GB. How do you reduce it?

- Use a smaller base image (`alpine`, `slim`, `distroless`).
- Use **multi-stage builds** so build tools don’t end up in the final image.
- Combine `RUN` commands and clean the cache in the same layer.
- Add a `.dockerignore` file.

*Example:* A Java app goes from about 700 MB (full JDK) to about 200 MB (JRE only).

---

## 3. The app works on your laptop but fails inside the container. Why?

Usually the environment is different. Check:

- Missing environment variables or config files.
- App listening on `localhost` (127.0.0.1) instead of `0.0.0.0`.
- Missing OS packages in the image.
- Wrong file paths or permissions.
- Different versions of dependencies.

---

## 4. You can’t reach your app from the browser even though the container is running.

**Check in this order:**

1. Did you publish the port? (`-p 8080:80`)
2. Is the app listening on `0.0.0.0` and not `127.0.0.1`? (Very common mistake)
3. Is the host firewall or cloud security group blocking the port?
4. Is the port number inside the container correct?

---

## 5. Container A can’t connect to Container B. How do you fix it?

- Make sure both are on the **same user-defined network**.
- Use the **container/service name** as the hostname, not `localhost`.
- Check that the target app is actually running and listening.
- Test with `docker exec -it A ping B`.

*Key point:* `localhost` inside a container means that container itself.

---

## 6. You deleted a container and lost your database data. How do you prevent this?

Container storage is temporary. Use a **volume**:

    docker run -v dbdata:/var/lib/postgresql/data postgres

The data now lives outside the container and survives deletion.

---

## 7. Your container is using too much memory and affecting other containers.

Set limits:

    docker run --memory=512m --cpus=1 myapp

Monitor with `docker stats`. If it gets killed with **exit code 137**, it ran out of memory (OOMKilled), so either raise the limit or fix the memory leak.

---

## 8. Builds are very slow. How do you speed them up?

- Order the Dockerfile so rarely changing steps come first (install dependencies before copying source code).
- Use `.dockerignore`.
- Use multi-stage builds.
- Use BuildKit and a cache.
- Use smaller base images.

---

## 9. A small code change makes Docker reinstall all dependencies every time.

This is a **layer cache** problem. Fix the order:

    COPY package.json .
    RUN npm install
    COPY . .

Now `npm install` re-runs only when `package.json` changes, not on every code change.

---

## 10. How do you handle passwords and API keys?

Never write them in the Dockerfile or image, because anyone with the image can read them.

**Better options:** Docker secrets, runtime environment variables, or tools like Vault and AWS Secrets Manager.

---

## 11. Your image has security vulnerabilities. What do you do?

- Scan with **Trivy**, Docker Scout, or Snyk.
- Update the base image.
- Use minimal images.
- Remove unnecessary packages.
- Add scanning to the CI pipeline so bad images never reach production.

---

## 12. The container runs as root. Is that a problem?

Yes. If someone breaks into the app, they get root power. Fix:

    RUN adduser --disabled-password appuser
    USER appuser

---

## 13. Your disk is full because of Docker. How do you clean up?

- `docker system df` to see what uses space.
- `docker system prune` removes stopped containers, unused networks, and dangling images.
- `docker image prune -a` removes unused images.
- `docker volume prune` removes unused volumes (**careful**, check for data first).

---

## 14. How do you update a running app with zero downtime?

Use **rolling updates** with an orchestrator (Swarm or Kubernetes): new containers start, pass their healthcheck, and only then do old ones stop. Without an orchestrator, run the new version beside the old one and switch the load balancer.

---

## 15. You pushed an image but the server still runs the old version.

Likely causes:

- You reused the same tag (like `latest`), so the server used its cached copy and needs `docker pull`.
- The container wasn’t recreated.
- The wrong tag was deployed.

**Best practice:** Use **unique version tags** (`app:1.4.2` or the git commit ID) instead of only `latest`.

---

## 16. Why is using the `latest` tag in production a bad idea?

It is a moving target. You can’t tell which version is running, rollbacks are hard, and two servers may pull different versions. Always pin a specific version.

---

## 17. A container is running but the app inside is frozen. How do you detect it?

Add a **HEALTHCHECK**. Docker checks the app regularly and marks the container `unhealthy` if it fails. Orchestrators can then restart it automatically.

---

## 18. The container should restart automatically if it crashes.

Use a restart policy:

    docker run --restart unless-stopped myapp

Options: `no`, `on-failure`, `always`, `unless-stopped`.

---

## 19. Your app needs to start after the database is ready, but `depends_on` doesn’t wait.

`depends_on` only controls **start order**, not readiness. Use a **healthcheck** on the database and `condition: service_healthy`, or add retry logic in the app.

---

## 20. How do you debug a running container in production?

- `docker logs -f <id>` for live logs.
- `docker exec -it <id> sh` to get inside.
- `docker stats` for CPU and memory.
- `docker inspect <id>` for config and state.
- `docker top <id>` for running processes.

---

## 21. Logs are filling the disk. What do you do?

By default, Docker stores logs without limits. Set log rotation:

    --log-opt max-size=10m --log-opt max-file=3

Or better, send logs to a central system (ELK, Loki, CloudWatch).

---

## 22. Developers want code changes to show instantly without rebuilding.

Use a **bind mount** in development:

    docker run -v $(pwd):/app myapp

Your local code folder is mirrored inside the container, so changes appear live.

---

## 23. How do you run a database, backend, and frontend together locally?

Use **Docker Compose**. Define all three services in one `docker-compose.yml`, then start everything with `docker compose up`. Compose also creates a shared network so they can talk by service name.

---

## 24. How do you use Docker in a CI/CD pipeline?

1. Developer pushes code.
2. Pipeline builds the image.
3. Runs tests inside the container.
4. Scans the image for vulnerabilities.
5. Pushes it to a registry (Nexus, ECR, Docker Hub).
6. Deploys that same image to the server or Kubernetes.

*Key idea:* The **same image** moves through dev, test, and production, so behavior stays consistent.

---

## 25. How do you roll back if a new deployment breaks production?

Redeploy the **previous image tag**. This is why versioned tags matter. With Kubernetes, `kubectl rollout undo` does it in one command. Also keep old images in your registry and use healthchecks to catch bad releases early.

---

## Tips for Answering Scenario Questions in an Interview

Use this simple pattern every time:

1. **Understand the problem:** “First I would check the logs and exit code.”
2. **Find the root cause:** Name 2 or 3 likely causes.
3. **Fix it:** Give the specific command or change.
4. **Prevent it:** Mention a best practice (healthchecks, scanning, pinned tags).
