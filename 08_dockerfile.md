# Dockerfile

## create a file in root with name ``` Dockerfile ```


### **Writing a Dockerfile for a Node.js App**

```dockerfile
FROM ubuntu

# combine apt steps to reduce layers and ensure noninteractive
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y curl && \
    curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/*

COPY package.json package-lock.json main.js ./

RUN npm install

ENTRYPOINT [ "node", "main.js" ]

```

---

## **1. Base image selection**

```dockerfile
FROM ubuntu
```

* Starts from a minimal Ubuntu image.
* **Note:** For Node.js apps it's usually better to use the official Node image (e.g., `node:18-bullseye`) because it already has Node and npm installed, and is optimized. Using Ubuntu means you have to bootstrap Node manually.

**Alternative (better):**

```dockerfile
FROM node:18-bullseye
```

---

## **2. Environment setup**

```dockerfile
ENV DEBIAN_FRONTEND=noninteractive
```

* Prevents interactive prompts during `apt` installs, making builds non-interactive and suitable for automation.

---

## **3. Install dependencies (if using Ubuntu base)**

```dockerfile
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y curl && \
    curl -fsSL https://deb.nodesource.com/setup_18.x | bash - && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/*
```

* Combines multiple package steps into one `RUN` to reduce image layers and leverage cache efficiently.
* Installs `curl` and Node.js via NodeSource script.
* Cleans up apt cache (`/var/lib/apt/lists/*`) to keep image size smaller.

**If using official Node image, this entire block can be omitted.**

---

## **4. Copy application metadata first**

```dockerfile
COPY package.json package-lock.json ./
```

* Copying `package.json` and `package-lock.json` before the rest of the source allows Docker to cache `npm install` unless these files change. (In your provided Dockerfile you also copied `main.js` here; better to separate to preserve install-layer caching.)

---

## **5. Install Node.js dependencies**

```dockerfile
RUN npm install
```

* Installs dependencies listed in `package.json`. Cached unless `package*.json` changes.

---

## **6. Copy application code**

```dockerfile
COPY main.js ./
```

* Now copy the actual app code. Changes here won't bust the earlier `npm install` layer if the package files didn’t change.

---

## **7. Define the entry point**

```dockerfile
ENTRYPOINT [ "node", "main.js" ]
```

* Specifies the default command to run when the container starts. Uses exec form (recommended for signal handling).

---

## **8. Best-practice enhancements (suggested improvements)**

### Use a non-root user

```dockerfile
# Example for official node image
RUN useradd --user-group --create-home --shell /bin/bash appuser
USER appuser
```

* Avoids running the app as root inside the container.

### Use .dockerignore

Create `.dockerignore` in build context:

```
node_modules
npm-debug.log
.git
```

* Prevents unnecessary files from being sent in the build context, speeding up builds and keeping images clean.

### Pin versions

* Explicitly choose base image and Node version (`node:18.17.1-bullseye` if you need consistency).
* Avoid `latest` for reproducibility unless you control updates.

---

## **9. Example optimized Dockerfile (using official Node image & cache-friendly ordering)**

```dockerfile
FROM node:18-bullseye

# Create app directory and switch to non-root user (optional)
RUN useradd --user-group --create-home --shell /bin/bash appuser
WORKDIR /home/appuser/app
USER appuser

# Copy dependency manifests first for caching
COPY --chown=appuser:appuser package.json package-lock.json ./

# Install dependencies
RUN npm ci --production

# Copy application code
COPY --chown=appuser:appuser main.js ./

# Default command
ENTRYPOINT [ "node", "main.js" ]
```

* `npm ci` is preferred in CI/builds if you have a lockfile; it's deterministic and faster for fresh installs.
* `--chown` ensures file ownership matches the unprivileged user.
* `WORKDIR` sets working directory.

---

## **10. Build command**

```bash
docker build -t ayush844eg_nodejsapp .
```

* `-t` tags the image.
* `.` is the build context (current directory).

---

## **11. Notes on layers & caching**

* Docker caches each instruction; changing `main.js` alone won’t invalidate the `npm install` layer if `package*.json` stayed the same.
* Changing `package.json` or `package-lock.json` will force reinstall of dependencies.

---

## **Quick Checklist Before Building**

* [ ] `.dockerignore` exists and excludes unnecessary files.
* [ ] Dependencies are in `package.json`/`package-lock.json`.
* [ ] Using pinned base image for reproducibility.
* [ ] Entry point and working directory are correct.
* [ ] Not running as root (optional but recommended).

---
---
---




### running the custom image in container

```
ayush@lucas:~$ docker run -it -e PORT=2000 -p 2000:2000 ayush844eg_nodejsapp
server start running at port number 2000
```


### working with running container

```bash
ayush@lucas:~$ docker container ls
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                                                    NAMES
5fa49c6aea05   ayush844eg_nodejsapp   "node main.js"           21 seconds ago   Up 20 seconds   0.0.0.0:2000->2000/tcp, [::]:2000->2000/tcp              wonderful_mccarthy
e684dcb486d8   ayush844eg_nodejsapp   "node main.js"           38 minutes ago   Up 38 minutes   0.0.0.0:5000->5000/tcp, [::]:5000->5000/tcp              quirky_shockley
e73de3fec49e   ayush844eg_nodejsapp   "node main.js"           39 minutes ago   Up 39 minutes                                                            sad_meitner
bf23cb1dea22   mongo                  "docker-entrypoint.s…"   2 days ago       Up 5 hours      0.0.0.0:27017->27017/tcp, [::]:27017->27017/tcp          mongodb
9e0591ebefe8   redis:7                "docker-entrypoint.s…"   2 days ago       Up 5 hours      0.0.0.0:6379->6379/tcp, [::]:6379->6379/tcp              redis
9063fdcf33fe   mysql:8.0              "docker-entrypoint.s…"   2 days ago       Up 5 hours      0.0.0.0:3306->3306/tcp, [::]:3306->3306/tcp, 33060/tcp   mysql
ayush@lucas:~$ docker exec -it 5fa49c6aea05 bash
root@5fa49c6aea05:/# ls
bin  boot  dev  etc  home  lib  lib64  main.js  media  mnt  node_modules  opt  package-lock.json  package.json  proc  root  run  sbin  srv  sys  tmp  usr  var
root@5fa49c6aea05:/# cat main.js
const express = require('express');

const app = express();

const port = process.env.PORT || 5000;

app.get("/", (req, res) => {
    return res.json({message: "HELLO I am NodeJS inside a container"});
})


app.listen(port, () => {
    console.log(`server start running at port number ${port}`)
})root@5fa49c6aea05:/# 

```