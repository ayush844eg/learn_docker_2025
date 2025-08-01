# **Docker Images** and **Containers**

## **1️⃣ Docker Image**

### **What is an Image?**

* A **read-only blueprint/template** for creating containers.
* Contains:

  * **App code or binary** (e.g., `nginx`, `node`, `python`)
  * **Dependencies** (libraries, runtime)
  * **File system snapshot** (like a mini Linux OS)
  * **Startup command** (e.g., `CMD` in Dockerfile)

Think of it like a **class in programming** or a **recipe**—you can create many instances (containers) from the same image.

---

### **Image Properties**

* **Immutable:** Once created, it doesn’t change.

* **Layered Architecture:**

  * Docker images are **built in layers** based on the Dockerfile instructions.
  * Example for `ubuntu` image:

    ```
    Layer 1: base Ubuntu filesystem
    Layer 2: security updates
    Layer 3: installed packages
    ```
  * Layers are **cached and shared** between images → saves disk space.

* **Identified by:**

  * **Name + Tag** → `ubuntu:20.04`
  * **Image ID** → `d1a364dc548d`

* **Stored locally** after you `docker pull` or `docker build`.

**Commands:**

```bash
docker images           # list all local images
docker pull ubuntu      # download image from Docker Hub
docker rmi ubuntu       # remove image
```

---

## **2️⃣ Docker Container**

### **What is a Container?**

* A **runtime instance of an image**.
* Includes:

  * The **image’s filesystem**
  * A **writable layer on top** (to store changes)
  * **Container metadata** (ID, name, network, volumes)
  * **Process running inside** (the app or service)

Think of it like an **object instance of a class** in programming.

---

### **Container Properties**

* **Ephemeral by default:**

  * If you delete the container, its data (not in volumes) is gone.
* **Isolated:**

  * Has its own filesystem, network, process tree.
  * Uses **Linux namespaces and cgroups** for isolation.
* **Lifecycle States:**

  * **Created → Running → Stopped → Removed**

**Commands:**

```bash
docker ps                   # running containers
docker ps -a                # all containers
docker run -it ubuntu bash  # create & run interactive container
docker exec -it <id> bash   # exec into running container
docker stop <id>            # stop container
docker rm <id>              # remove container
```

---

## **3️⃣ Image vs Container (Summary)**

| Feature   | Docker Image                  | Docker Container                               |
| --------- | ----------------------------- | ---------------------------------------------- |
| Nature    | Read-only blueprint           | Running instance                               |
| Lifecycle | Permanent (unless deleted)    | Temporary (can restart)                        |
| Changes   | Immutable                     | Writable layer (changes lost if not committed) |
| Count     | 1 image → many containers     | Container unique per run                       |
| Storage   | Stored locally in image cache | Uses image + writable diff                     |

---

### **4️⃣ Relationship Example**

1. **Pull image:**

   ```bash
   docker pull ubuntu
   ```

   * Downloads the `ubuntu` image.

2. **Create container:**

   ```bash
   docker run -it ubuntu bash
   ```

   * Creates a new container from `ubuntu` image.
   * Running in interactive mode with a bash shell.
   * Inside container: `root@<container_id>:/#`

3. **Modify container (optional):**

   ```bash
   apt update && apt install curl
   ```

   * Changes are **inside writable container layer**, not in the original image.

4. **Commit container to new image (optional):**

   ```bash
   docker commit <container_id> my-ubuntu:with-curl
   ```

   * Creates a new image snapshot.

---

### **5️⃣ Quick Diagram**

```
[Docker Image]  --->  docker run  --->  [Docker Container]
(read-only)          creates              (image + writable layer)
```

* **Image = blueprint**
* **Container = living instance**

---

