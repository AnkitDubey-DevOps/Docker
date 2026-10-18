## 1. What is a Docker Multi-Stage Build?

Definition: A Docker multi-stage build is a technique in which a Dockerfile uses multiple `FROM` instructions to create separate build stages. Artifacts from one stage can be copied into another stage, allowing the final image to contain only the files and dependencies required to run the application.

Simple example: Suppose you build a Java application. You need Maven and the JDK to compile the code, but you may need only a Java runtime and the compiled JAR to run it. A multi-stage build separates these steps.

Stage 1: Build

Source code + compiler + build tools

Copy build artifact

Compiled JAR, binary, or static assets

Stage 2: Final image

Runtime + application artifact only

Interview answer: Multi-stage builds help create smaller, cleaner production images by separating build dependencies from runtime dependencies.

## 2. Why do we use multi-stage builds?

The most important benefits are:

1. Smaller image size: Build tools that are not needed at runtime can be excluded.
2. Better security: Fewer unnecessary tools and packages are included in the final image.
3. Cleaner production images: Source code and compilers can be left out when not required at runtime.
4. Simplified Dockerfiles: Build and runtime steps can be organized in one Dockerfile.
5. Improved deployment efficiency: Smaller images can reduce image transfer and storage costs.
6. Reproducible builds: Build stages can specify their toolchain and dependencies consistently.

Important: Multi-stage builds do not automatically guarantee security or reproducibility. You still need to manage dependencies, secrets, base images, and build inputs correctly.

## 3. Important Dockerfile instructions

These are the main instructions you should understand for interviews.

| Instruction | Easy definition |
|---|---|
| `FROM` | Specifies the base image and starts a new build stage. |
| `AS` | Gives a stage a name that can be referenced later. |
| `WORKDIR` | Sets the working directory for subsequent instructions. |
| `COPY` | Copies files from the build context or from another stage into the current stage. |
| `RUN` | Executes commands during image building. |
| `CMD` | Specifies the default command to run when a container starts. |
| `ENTRYPOINT` | Specifies the main executable for the container. |
| `EXPOSE` | Documents the port the application is expected to use. It does not publish the port. |

### Most important instructions for multi-stage builds

`FROM` starts a stage.

    FROM maven:3.9-eclipse-temurin-21 AS build

`AS build` names the stage `build`.

`COPY --from=build` copies files from a previous stage into the current stage.

    COPY --from=build /app/target/app.jar /app/app.jar

Interview tip: `COPY --from` is one of the most important instructions to remember for multi-stage builds.

## 4. Practical example: Java multi-stage build

Suppose you have a Java application that uses Maven and generates a JAR file.

Project structure

    my-java-app/
    ├── Dockerfile
    ├── pom.xml
    └── src/
        └── main/
            └── java/

### Dockerfile

    # Stage 1: Build the application
    FROM maven:3.9-eclipse-temurin-21 AS build

    WORKDIR /app

    COPY pom.xml .
    COPY src ./src

    RUN mvn clean package -DskipTests

    # Stage 2: Create the runtime image
    FROM eclipse-temurin:21-jre

    WORKDIR /app

    COPY --from=build /app/target/*.jar app.jar

    EXPOSE 8080

    CMD ["java", "-jar", "app.jar"]

### Explanation

- `FROM ... AS build`: Creates the build stage with Maven and the JDK.
- `WORKDIR /app`: Sets the working directory.
- `COPY pom.xml .` and `COPY src ./src`: Copy the project files into the build stage.
- `RUN mvn clean package -DskipTests`: Compiles and packages the application without running tests.
- Second `FROM`: Starts a new, independent runtime stage.
- `COPY --from=build`: Copies the generated JAR from the build stage.
- `CMD`: Starts the Java application when the container runs.

The final image does not include the Maven build environment merely because it was used in the first stage.

Note: This example assumes the project produces an executable JAR under `target/`. Skipping tests is only for demonstrating the build; production CI should run the appropriate tests.

### Commands to build and run

    # Build the final image
    docker build -t my-java-app .

    # Run the container
    docker run -d --name java-app \
      -p 8080:8080 \
      my-java-app

    # Check logs
    docker logs java-app

    # Stop and remove the container
    docker rm -f java-app

## 5. What happens without a multi-stage build?

A single-stage Dockerfile might look like this:

    FROM maven:3.9-eclipse-temurin-21

    WORKDIR /app

    COPY pom.xml .
    COPY src ./src

    RUN mvn clean package -DskipTests

    CMD ["java", "-jar", "target/app.jar"]

This image is based on the Maven image, so it includes build tools even though the application only needs Java to run.

With a multi-stage build, the final image uses a separate Java runtime image and copies only the application artifact.

| Single-stage build | Multi-stage build |
|---|---|
| Build and runtime share one stage | Build and runtime use separate stages |
| Build tools may remain in the final image | Unneeded build tools can be excluded |
| Can produce a larger image | Often produces a smaller image |
| Simpler for basic applications | Useful for production-oriented builds |

Interview answer: A multi-stage build separates compilation from execution so that the final image contains the runtime and required application files rather than the entire build environment.

## 6. Multi-stage build using Node.js

This is another common DevOps example: building a frontend application.

    # Stage 1: Build the frontend
    FROM node:22-alpine AS build

    WORKDIR /app

    COPY package*.json ./
    RUN npm ci

    COPY . .
    RUN npm run build

    # Stage 2: Serve the static files
    FROM nginx:alpine

    COPY --from=build /app/dist /usr/share/nginx/html

    EXPOSE 80

    CMD ["nginx", "-g", "daemon off;"]

How it works:

1. The first stage installs Node.js dependencies and builds the frontend.
2. The build generates static files in `/app/dist`.
3. The second stage copies those files into the Nginx image.
4. Nginx serves the static website without including Node.js or its build dependencies.

This example assumes the frontend build output is `dist`, as with a typical Vite application. Other frameworks may use a different output directory.

Commands:

    docker build -t frontend-app .
    docker run -d --name frontend \
      -p 8080:80 frontend-app

Open `http://localhost:8080` on the Docker host to access the website.

## 7. Named build stages

Definition: A named build stage is a stage assigned an identifier using `AS`, allowing later stages to reference its files or allowing you to select that stage as a build target.

Example:

    FROM golang:alpine AS build

    WORKDIR /app
    COPY . .
    RUN go build -o app .

    FROM alpine:3.22 AS production

    WORKDIR /app
    COPY --from=build /app/app .
    CMD ["./app"]

Here:

- `AS build` names the compilation stage.
- `AS production` names the final stage.
- `COPY --from=build` copies the compiled binary into the production stage.

Interview answer: Naming stages makes multi-stage Dockerfiles easier to read and allows files to be copied from specific stages by name.

## 8. What is `--target` in a multi-stage build?

Definition: The `--target` option tells Docker to build a specific stage rather than the default final stage.

Example Dockerfile:

    FROM node:22-alpine AS build

    WORKDIR /app
    COPY package*.json ./
    RUN npm ci
    COPY . .
    RUN npm run build

    FROM nginx:alpine AS production

    COPY --from=build /app/dist /usr/share/nginx/html

Build the final production image:

    docker build --target production -t my-frontend .

If you define a separate testing stage, you can build it directly:

    FROM build AS test

    RUN npm test

    docker build --target test -t my-frontend-test .

Interview answer: `--target` is useful for building or testing a specific stage, such as a development, testing, or production stage.

## 9. Multi-stage builds and Docker layer caching

Definition: Docker layer caching allows Docker to reuse the results of unchanged build steps instead of executing them again.

A common optimization is to copy dependency files before application source code.

    FROM node:22-alpine AS build

    WORKDIR /app

    COPY package*.json ./
    RUN npm ci

    COPY . .
    RUN npm run build

Why is this useful?

- Dependency installation is performed before the source code is copied.
- If the source code changes but the dependency files do not, Docker can often reuse the cached dependency-installation layer.
- This can make subsequent builds faster.

Interview tip: Multi-stage builds and layer caching solve different problems. Multi-stage builds help control what goes into the final image; layer caching helps reduce repeated build work.

## 10. Multi-stage build best practices

For a DevOps interview, remember these points:

1. Use a build image containing the required compilers and build tools.
2. Use an appropriate runtime image for the final stage.
3. Copy only the required binaries, JARs, or static assets into the final image.
4. Choose trusted, maintained base images and pin versions appropriately.
5. Use `.dockerignore` to avoid sending unnecessary files in the build context.
6. Never pass secrets through Dockerfile `ARG` or `ENV` if they could be exposed in image metadata or layers; use BuildKit secret mounts where appropriate.
7. Run the application as a non-root user when practical.
8. Run tests in CI or a dedicated build stage.

Example `.dockerignore`:

    .git
    node_modules
    .env
    coverage

This helps exclude unnecessary files from the build context. Ensure any required files are handled appropriately and secrets are not copied into the image.
