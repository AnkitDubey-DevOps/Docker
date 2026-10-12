# CMD vs ENTRYPOINT in Docker

The easiest way to remember it:

- **CMD = default command / default arguments**
- **ENTRYPOINT = main process / fixed executable**

Think like this:

- CMD says: "run this by default"
- ENTRYPOINT says: "this container is built to run this"

---

## 1) CMD Example — Default Command

### Dockerfile

    FROM alpine
    CMD ["echo", "Hello from CMD"]

### Build

    docker build -t cmd-demo .

### Run

    docker run --rm cmd-demo

### Output

    Hello from CMD

Here:

- CMD provides the default command
- If you do not give anything extra, Docker runs it

### Try to override CMD

    docker run --rm cmd-demo date

### Output

    Fri Mar 7 10:00:00 UTC 2026

Now CMD is overridden.

So with CMD:

- no extra command → default runs
- extra command provided → default is replaced

---

## 2) Another CMD Example

### Dockerfile

    FROM python:3.12-slim
    CMD ["python", "--version"]

### Run

    docker run --rm python-cmd-demo

### Output

    Python 3.12.x

### Override it

    docker run --rm python-cmd-demo python -c "print('Hi')"

### Output

    Hi

Again, CMD is just the default.

---

## 3) Multiple CMD in Dockerfile

### Dockerfile

    FROM alpine
    CMD ["echo", "First CMD"]
    CMD ["echo", "Second CMD"]

### What happens?

Only the last CMD works.

### Run

    docker build -t multi-cmd-demo .
    docker run --rm multi-cmd-demo

### Output

    Second CMD

### Rule

If you write multiple CMD instructions:

- Docker ignores earlier ones
- only the last CMD is used

---

## 4) ENTRYPOINT Example — Main Process

### Dockerfile

    FROM alpine
    ENTRYPOINT ["echo", "Hello from ENTRYPOINT"]

### Build

    docker build -t entrypoint-demo .

### Run

    docker run --rm entrypoint-demo

### Output

    Hello from ENTRYPOINT

Here ENTRYPOINT defines the main process.

This container is basically built to run:

    echo "Hello from ENTRYPOINT"

### Try to override ENTRYPOINT normally

    docker run --rm entrypoint-demo date

### Output

    Hello from ENTRYPOINT date

### Why?

Because with ENTRYPOINT, extra values passed to docker run are treated as arguments to ENTRYPOINT, not as a replacement.

So Docker runs something like:

    echo "Hello from ENTRYPOINT" date

---

## 5) Another ENTRYPOINT Example

### Dockerfile

    FROM alpine
    ENTRYPOINT ["ping", "-c", "2"]

### Run

    docker run --rm ping-demo google.com

This becomes:

    ping -c 2 google.com

Here:

- ENTRYPOINT = main executable + fixed options
- runtime value = appended argument

Very useful when the container is meant to behave like a specific command-line tool.

---

## 6) Multiple ENTRYPOINT in Dockerfile

### Dockerfile

    FROM alpine
    ENTRYPOINT ["echo", "First ENTRYPOINT"]
    ENTRYPOINT ["echo", "Second ENTRYPOINT"]

### What happens?

Only the last ENTRYPOINT is used.

### Run

    docker build -t multi-entry-demo .
    docker run --rm multi-entry-demo

### Output

    Second ENTRYPOINT

### Rule

If multiple ENTRYPOINT instructions are present:

- earlier ones are discarded
- only the last ENTRYPOINT applies

---

## 7) How to override ENTRYPOINT

Normal extra command does not replace it.

To override ENTRYPOINT, use:

    docker run --rm --entrypoint date entrypoint-demo

Now Docker ignores the Dockerfile ENTRYPOINT and runs:

    date

### Another example

    docker run --rm --entrypoint sh entrypoint-demo

This replaces the original entrypoint with sh.

So:

- CMD override → just add another command after image name
- ENTRYPOINT override → use `--entrypoint`

---

## 8) CMD + ENTRYPOINT together

This is the most important real-world pattern.

### Dockerfile

    FROM alpine
    ENTRYPOINT ["echo"]
    CMD ["Hello from CMD"]

### Run

    docker build -t cmd-entry-demo .
    docker run --rm cmd-entry-demo

### Output

    Hello from CMD

### What Docker actually runs:

    echo "Hello from CMD"

Here:

- ENTRYPOINT = main process
- CMD = default arguments to that process

### Override CMD in CMD + ENTRYPOINT case

    docker run --rm cmd-entry-demo "Overridden text"

### Output

    Overridden text

Now Docker runs:

    echo "Overridden text"

So in this pattern:

- ENTRYPOINT stays fixed
- CMD can be replaced easily

This is why people say:

- ENTRYPOINT = executable
- CMD = default parameters

---

# ARG vs ENV (Simple Difference)

| Feature | ARG | ENV |
|---|---|---|
| Scope | Build time | Build time + Runtime |
| Available in container | ❌ No | ✅ Yes |
| Used for | Build configuration | Application environment variables |
| Defined with | ARG | ENV |
| Passed with | `--build-arg` | `-e` or defined in Dockerfile |

## Example Dockerfile (Different Values)

    FROM node:20-alpine

    # Build-time variable
    ARG BUILD_VERSION=1.0

    # Runtime variable (different value)
    ENV APP_ENV=production

    RUN echo "Build version: $BUILD_VERSION"

    CMD ["sh", "-c", "echo Runtime environment: $APP_ENV"]

## Build the Image

    docker build --build-arg BUILD_VERSION=2.5 -t myapp .

During build you will see:

    Build version: 2.5

This proves ARG is used during build.

## Run the Container

    docker run --rm myapp

### Output:

    Runtime environment: production

This shows ENV is used during runtime.

## Check Inside the Container

Run interactive shell:

    docker run -it myapp sh

Then check variables:

    echo $APP_ENV

### Output:

    production

Try checking ARG:

    echo $BUILD_VERSION

### Output:

    (empty)

Because ARG does not exist inside the running container.

## Key Idea

| Variable | Value | Visible When |
|---|---|---|
| ARG | `BUILD_VERSION=2.5` | During docker build |
| ENV | `APP_ENV=production` | Inside container runtime |

## Important Rule

- ARG → build-time only
- ENV → runtime environment variable

## 4️⃣ Typical Use Cases

### Use ARG for

- selecting base image versions
- build configuration
- build secrets (temporary)

Example:

    ARG NODE_VERSION=20
    FROM node:${NODE_VERSION}-alpine

### Use ENV for

- application configuration
- database URLs
- runtime configs

Example:

    ENV DB_HOST=db
    ENV PORT=3000

---

# ADD vs COPY (Quick Difference)

| Feature | COPY | ADD |
|---|---|---|
| Basic file copy | ✅ Yes | ✅ Yes |
| Download from URL | ❌ No | ✅ Yes |
| Auto extract tar files | ❌ No | ✅ Yes |
| Predictability | ✅ Simple | ⚠ Less predictable |
| Best practice | ✅ Preferred | ⚠ Use only when needed |

## 1️⃣ COPY (Recommended)

COPY simply copies files from host → container.

### Example

    COPY app.py /app/

### Meaning:

    Host: ./app.py
        ↓
    Container: /app/app.py

Another example:

    COPY . .

Copies everything from current directory → container working directory.

---

## 2️⃣ ADD (More Powerful)

ADD can do extra things automatically.

### Example 1 — Copy file

    ADD app.py /app/

Same behavior as COPY.

### Example 2 — Download file from URL

    ADD https://example.com/file.tar.gz /app/

Docker will:

    download → copy to container

⚠ Not recommended in most cases.

### Example 3 — Auto extract tar

    ADD archive.tar.gz /app/

Docker automatically:

    extracts archive.tar.gz into /app

With COPY, extraction does not happen.

---

## Example Comparison

### Using COPY

    COPY app.tar.gz /app
    RUN tar -xzf /app/app.tar.gz

### Using ADD

    ADD app.tar.gz /app

Docker extracts automatically.

---

## Best Practice (Important)

Docker documentation recommends:

**Use COPY unless you specifically need ADD features.**

Reasons:

- More predictable
- Clear behavior
- Better security
- Easier to understand builds

---

## DevOps Production Rule

Use:

    COPY .

Avoid:

    ADD .

unless you need:

- URL download
- tar extraction
