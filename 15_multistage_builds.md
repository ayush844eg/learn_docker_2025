

# 🔹 What is a Multi-Stage Build?

* Normally, a Docker build uses **one base image** and runs all steps on top of it.
* But this often leads to **big images** because:

  * You need compilers, build tools, dev dependencies, etc., just to compile or bundle your app.
  * But those are **not needed in production**.

👉 **Multi-stage builds** solve this by letting you use **multiple FROM instructions** in one Dockerfile.

* Each `FROM` starts a **new stage**.
* You can `COPY --from=...` artifacts from one stage into another.
* This way:

  * Build happens in a "builder" stage (with all tools).
  * Final stage is minimal (only runtime + built output).

---

# 🔹 Example 1: Go Application

### ❌ Without Multi-Stage (bloated image)

```dockerfile
FROM golang:1.21

WORKDIR /app

COPY . .

RUN go build -o myapp

CMD ["./myapp"]
```

* Includes full **Go toolchain** (\~1GB image).
* You only need the compiled `myapp` binary at runtime.

---

### ✅ With Multi-Stage

```dockerfile
# Stage 1: Build
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: Run
FROM debian:bookworm-slim
WORKDIR /app
COPY --from=builder /app/myapp .
CMD ["./myapp"]
```

* Stage 1 uses heavy `golang:1.21` (with compiler).
* Stage 2 uses lightweight `debian:slim` (only needs binary).
* Final image size drops from \~1GB → \~50MB 🚀.

---

# 🔹 Example 2: Node.js App with Build Tools

Let’s say you’re building a **React app**.
It requires `npm install` + `npm run build`, but runtime only needs static files.

### ✅ Multi-Stage for Node/React

```dockerfile
# Stage 1: Build React app
FROM node:20 AS builder
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

* Stage 1: Has Node.js to compile.
* Stage 2: Only Nginx serving built static files.
* **No Node.js runtime, no dev dependencies** → lean final image.

---

# 🔹 Example 3: Python App

Some Python apps require compiling wheels with build tools.

```dockerfile
# Stage 1: Build deps
FROM python:3.12 AS builder
WORKDIR /app

COPY requirements.txt .
RUN pip wheel --wheel-dir /wheels -r requirements.txt

# Stage 2: Runtime
FROM python:3.12-slim
WORKDIR /app

COPY --from=builder /wheels /wheels
RUN pip install --no-index --find-links=/wheels /wheels/*

COPY . .
CMD ["python", "app.py"]
```

* First stage builds wheels (heavy compilers, C libs).
* Final stage installs from wheels only.
* Result: much faster runtime and smaller image.

---

# 🔹 Benefits of Multi-Stage Builds

1. **Smaller Images**

   * Production image only has what you need.

2. **Security**

   * No build tools or secrets in final image.

3. **Faster Builds**

   * Cache dependencies separately.

4. **Cleaner Dockerfiles**

   * Separate concerns: one stage for building, one for running.

---

# 🔹 Best Practices

✅ **Name stages** with `AS` for clarity:

```dockerfile
FROM node:20 AS deps
FROM node:20 AS builder
FROM nginx:alpine AS runtime
```

✅ **Copy selectively**

```dockerfile
COPY --from=builder /app/dist /usr/share/nginx/html
```

✅ **Use .dockerignore**
Exclude files like `.git`, `node_modules`, `*.log` that aren’t needed.

✅ **Use multiple build stages if needed**
Example: one for dependencies, one for tests, one for release.

---

# 🔹 Real-World Multi-Stage Example (Full CI/CD Flow)

```dockerfile
# Stage 1: Dependencies
FROM node:20 AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Stage 2: Run Tests
FROM deps AS test
COPY . .
RUN npm test

# Stage 3: Build
FROM deps AS build
COPY . .
RUN npm run build

# Stage 4: Production Runtime
FROM nginx:alpine AS prod
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

* Stage 1: Dependencies installed
* Stage 2: Run tests (won’t ship in prod)
* Stage 3: Build artifacts
* Stage 4: Final runtime image

---

✅ **In short:**
Multi-stage builds let you **compile in one stage** and **deploy in another**, giving you **small, fast, secure Docker images**.

---

