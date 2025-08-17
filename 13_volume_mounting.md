
## 📦 What Is Volume Mounting in Docker?

**Volume mounting** is how Docker **persists data** or shares files between:

* **Host ↔ Container**
* **Containers ↔ Containers**

By default, data written inside a container is **lost** when the container is removed. Volumes solve this by mounting external storage to containers.

---

## 🧱 1. Why Use Volumes?

* **Persistence**: Data survives container restarts, updates, or deletions.
* **Sharing**: Share data between containers.
* **Isolation**: Separate data from application logic.
* **Performance**: Docker-managed volumes are optimized and more efficient than bind mounts.
* **Portability**: Volumes can be backed up, restored, or moved.

---

## 📁 2. Types of Mounts in Docker

There are **3 main types** of mounts:

| Type             | Description                                              |
| ---------------- | -------------------------------------------------------- |
| **Volumes**      | Docker-managed data stored in `/var/lib/docker/volumes/` |
| **Bind mounts**  | Direct mapping between host and container path           |
| **tmpfs mounts** | In-memory mount; does not persist after container stops  |

---

## 🔹 2.1. Volumes (Recommended)

### How it works:

* Docker creates and manages the storage
* Stored under `/var/lib/docker/volumes/`
* Not directly tied to a host path (but still on the host FS)

### Commands:

```bash
# Create a named volume
docker volume create mydata

# Use it in a container
docker run -d -v mydata:/data --name app alpine
```

### Behind the scenes:

* Volume is created: `/var/lib/docker/volumes/mydata/_data/`
* Inside the container, `/data` maps to that directory

### Best for:

* Portability
* Docker-managed data
* Shared data between containers

---

## 🔹 2.2. Bind Mounts

### How it works:

* Mounts a specific file or folder from the host to the container
* Gives the container direct access to host files

```bash
docker run -v /host/path:/container/path myapp
```

> Changes in one are reflected immediately in the other.

### Pros:

* Powerful for development (live reloading, logs)
* Total control over file paths

### Cons:

* Less portable
* Security risks (host files exposed)
* Prone to breaking across OSes or environments

### Example:

```bash
docker run -v $(pwd)/app:/usr/src/app node
```

* Maps local `./app` into container's `/usr/src/app`

---

## 🔹 2.3. tmpfs Mounts (Ephemeral)

* Mounts a **RAM-based** file system
* Data is stored **in memory only**, never written to disk
* Useful for **sensitive data** (e.g., passwords, keys) or **fast temporary files**

```bash
docker run --tmpfs /app/tmp:rw,size=100m alpine
```

---

## 🧬 3. Syntax Differences

You can mount volumes in two ways:

### Short syntax (`-v` or `--volume`)

```bash
docker run -v volume-name:/container/path
```

Format:

```
-v [host path | volume name]:[container path]:[options]
```

Example:

```bash
-v /host/data:/container/data:ro
```

### Long syntax (`--mount`)

```bash
docker run --mount type=volume,source=mydata,target=/app/data
```

Format:

```
--mount type=[volume|bind|tmpfs],source=...,target=...,options
```

> `--mount` is more verbose but more flexible and readable.

---

## 🏗️ 4. Container ↔ Container Volume Sharing

Two containers can share the same volume:

```bash
docker volume create shared

docker run -d -v shared:/data --name writer alpine
docker run -d -v shared:/data --name reader alpine
```

Both containers read/write the same `/data`.

---

## 🧼 5. Cleaning Up Volumes

* **List volumes**:

  ```bash
  docker volume ls
  ```
* **Inspect a volume**:

  ```bash
  docker volume inspect mydata
  ```
* **Remove a volume**:

  ```bash
  docker volume rm mydata
  ```
* **Remove unused volumes**:

  ```bash
  docker volume prune
  ```

---

## 🧠 6. Under the Hood

### Where are volumes stored?

* Linux:

  ```
  /var/lib/docker/volumes/<volume-name>/_data
  ```

### Mounting process:

* Docker uses **mount namespaces** and the Linux **mount()** syscall.
* It creates a directory inside the host’s Docker volume store.
* That directory is then mounted into the container's FS namespace.

---

## ⚠️ 7. Permissions and Gotchas

* Containers may not have permission to write to bind-mounted directories unless properly configured.
* UID/GID mismatches can cause access issues.
* SELinux and AppArmor may block mounts if not configured correctly.

Example fix for SELinux:

```bash
-v /host/data:/container/data:Z
```

---

## 💡 8. Practical Use Cases

### ✅ Persisting database data

```bash
docker run -d \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

### ✅ Mounting configuration files

```bash
docker run -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf nginx
```

### ✅ Development with live code updates

```bash
docker run -v $(pwd)/src:/app python
```

### ✅ Sharing cache/build artifacts between steps

In multi-stage builds or CI/CD pipelines.

---

## 🧾 Summary Table

| Mount Type | Managed by | Persist? | Best For                       |
| ---------- | ---------- | -------- | ------------------------------ |
| **Volume** | Docker     | ✅ Yes    | Data persistence, portability  |
| **Bind**   | User       | ✅ Yes    | Development, config sharing    |
| **tmpfs**  | Kernel     | ❌ No     | Sensitive or fast temp storage |

---
