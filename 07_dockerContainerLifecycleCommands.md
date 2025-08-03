# Docker Container Lifecycle & Commands (run, start, stop, exec)

## **1️⃣ `docker run` — Create & Start a New Container**

**Syntax:**

```bash
docker run [OPTIONS] IMAGE [COMMAND]
```

* **Creates a NEW container from an image** (never reuses existing containers).
* Starts the container immediately.
* If image not found locally → pulls it from registry.
* You can pass:

  * `-it` → Interactive terminal
  * `-d` → Detached mode
  * `--name` → Name the container
  * `-p` → Port mapping
  * `-e` → Environment variables

**Example:**

```bash
docker run -it ubuntu bash
```

* Creates a **new Ubuntu container**
* Gives it a random name unless `--name` is specified.
* Drops you into bash shell.

**When to use:**

* First time you want to create a container from an image.
* When you need multiple independent containers from the same image.

---

## **2️⃣ `docker start` — Start an Existing Container**

**Syntax:**

```bash
docker start [OPTIONS] CONTAINER
```

* **Starts a container that was previously created/stopped**.
* Does **NOT** create a new container.
* If you want to attach to it and see its output → use `-a`.
* If it’s interactive, also use `-i`.

**Example:**

```bash
docker start -ai myubuntu
```

* Starts an existing container named `myubuntu`.
* `-a` = attach to its output.
* `-i` = keep STDIN open (for interactive bash sessions).

**When to use:**

* To restart a container you already created with `docker run`.
* When you want to keep previous changes made in the container.

---

## **3️⃣ `docker stop` — Stop a Running Container**

**Syntax:**

```bash
docker stop CONTAINER
```

* Gracefully stops a running container.
* Sends `SIGTERM` first, then `SIGKILL` if it doesn’t stop within 10s.
* Container stays on disk and can be restarted later.

**Example:**

```bash
docker stop myubuntu
```

**When to use:**

* When you want to temporarily stop a container but might restart it later.
* When shutting down services without removing them.

---

## **4️⃣ `docker exec` — Run Commands Inside a Running Container**

**Syntax:**

```bash
docker exec [OPTIONS] CONTAINER COMMAND
```

* Runs a **new process inside an already running container**.
* Does not restart container — just executes a command inside it.
* `-it` for interactive commands like bash.

**Example:**

```bash
docker exec -it myubuntu bash
```

* Opens a bash shell in the running container named `myubuntu`.

**When to use:**

* To debug or inspect a running container.
* To execute additional commands without restarting it.

---

## **5️⃣ Common Lifecycle Scenarios**

### **Case 1: First Time Running a Container**

```bash
docker run -it --name myubuntu ubuntu bash
```

* Creates and runs a new container named `myubuntu`.

### **Case 2: Reuse the Same Container**

```bash
docker start -ai myubuntu
```

* Starts and attaches to `myubuntu` container again.

### **Case 3: Run Extra Commands in Running Container**

```bash
docker exec -it myubuntu bash
docker exec myubuntu ls /var
```

### **Case 4: Stop a Container**

```bash
docker stop myubuntu
```

* Gracefully stops it. Can be restarted with `docker start`.

---

## **6️⃣ Quick Comparison Table**

| Command        | Creates New Container? | Starts Existing? | Runs Command Inside? | Needs Container Running? |
| -------------- | ---------------------- | ---------------- | -------------------- | ------------------------ |
| `docker run`   | ✅                      | ✅                | ✅ (main process)     | ❌                        |
| `docker start` | ❌                      | ✅                | ❌                    | ❌ (starts stopped one)   |
| `docker stop`  | ❌                      | ❌                | ❌                    | ✅                        |
| `docker exec`  | ❌                      | ❌                | ✅                    | ✅                        |




---
---
---


Got it ✅
Here’s a **clear explanation** of each option with examples you can directly add to your notes.

---

# **Docker Run Command Options Explained**

## **1️⃣ `-it` → Interactive Terminal**

* **`-i`** → Keep STDIN open (interactive mode)
* **`-t`** → Allocate a pseudo-TTY (gives you a shell-like terminal)
* Used when you want to **interact directly** with the container (bash shell, REPL, etc.).

**Example:**

```bash
docker run -it ubuntu bash
```

* Creates a new Ubuntu container
* Opens bash shell so you can run commands inside it.

---

## **2️⃣ `-d` → Detached Mode**

* Runs container **in the background**.
* You don’t stay attached to container output.
* Useful for long-running services like web servers or databases.

**Example:**

```bash
docker run -d nginx
```

* Starts an Nginx container in the background.
* Can check with `docker ps`.

---

## **3️⃣ `--name` → Name the Container**

* Gives the container a **custom name** instead of a random one.
* Makes it easier to reference the container later (`docker start myapp`).

**Example:**

```bash
docker run -it --name myubuntu ubuntu bash
```

* Creates a container named `myubuntu`.
* Can reuse later with:

```bash
docker start -ai myubuntu
```

---

## **4️⃣ `-p` → Port Mapping**

* Maps **host machine port → container port**.
* Format:

  ```
  -p <host_port>:<container_port>
  ```
* Required if you want to access container services from outside.

**Example:**

```bash
docker run -d -p 8080:80 nginx
```

* Maps host `8080` → container `80`.
* Access Nginx at `http://localhost:8080`.

---

## **5️⃣ `-e` → Environment Variables**

* Passes key-value pairs into the container.
* Common for setting app configs, API keys, credentials, etc.

**Example:**

```bash
docker run -it -e MY_NAME=Ayush -e APP_ENV=production ubuntu bash
```

* Inside container:

```bash
echo $MY_NAME   # Ayush
echo $APP_ENV   # production
```

---

## **Quick Example Combining All Options**

```bash
docker run -it -d --name myweb -p 5000:80 -e APP_ENV=production nginx
```

* Runs Nginx in detached mode
* Interactive mode possible with `exec`
* Named container: `myweb`
* Maps host port 5000 → container port 80
* Sets environment variable `APP_ENV=production`

---
