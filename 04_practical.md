# **Docker CLI notesheet**


## **1️⃣ Container Management**

### **List Containers**

```bash
docker ps                     # List running containers (short)
docker container ls           # Same as docker ps
docker ps -a                  # List all containers (including stopped)
docker container ls -a        # Same with explicit command
```

### **Start / Stop / Restart Containers**

```bash
docker start <container>      # Start a stopped container
docker stop <container>       # Stop a running container
docker restart <container>    # Restart container
docker container prune        # Remove all stopped containers
```

> 🔹 Tip: Use `container name` (like `affectionate_ride`) or `container ID` (like `8150f1402602`).

---

### **Accessing and Running Commands in Containers**

```bash
docker exec <container> <cmd>         # Run a command (e.g., ls)
docker exec -it <container> bash      # Open interactive bash shell
docker attach <container>             # Attach to container main process
```

> **`-it`** → interactive + terminal
> **`exec`** → runs a new process in a running container

---

### **Run New Containers**

```bash
docker run -it ubuntu bash            # Create & run container interactively
docker run -d -p 8080:80 nginx        # Detached mode, map port 8080->80
docker run --name myapp ubuntu        # Run container with a name
```

* **`-it`** → Interactive mode
* **`-d`** → Detached (run in background)
* **`-p host:container`** → Port mapping

---

### **Remove Containers**

```bash
docker rm <container>                 # Remove stopped container
docker rm -f <container>              # Force remove (running or stopped)
```

---

## **2️⃣ Image Management**

### **List, Pull, Remove Images**

```bash
docker images                         # List images
docker pull ubuntu                     # Download image from Docker Hub
docker rmi ubuntu                      # Remove an image
docker image prune                     # Remove dangling/unused images
docker system prune -a                 # Remove all unused images & containers
```

---

### **Build & Tag Images**

```bash
docker build -t myimage:1.0 .          # Build image from Dockerfile
docker tag myimage:1.0 myimage:latest  # Add a new tag
docker push myusername/myimage:1.0     # Push to Docker Hub
```

---

## **3️⃣ Useful Inspection Commands**

```bash
docker logs <container>                # Show container logs
docker logs -f <container>             # Follow logs live
docker inspect <container_or_image>    # Detailed JSON info
docker top <container>                 # Processes running inside container
docker stats                           # Resource usage of running containers
```

---

## **4️⃣ Volumes & Persistent Data**

```bash
docker volume ls                       # List volumes
docker volume create myvolume          # Create a named volume
docker run -v myvolume:/data ubuntu    # Mount volume to container
docker volume rm myvolume              # Remove volume
docker volume prune                    # Remove all unused volumes
```

---

## **5️⃣ Networking (Optional)**

```bash
docker network ls                      # List networks
docker network create mynet            # Create custom network
docker run --network=mynet ubuntu      # Connect container to network
docker network inspect mynet           # Inspect network details
docker network rm mynet                 # Remove network
```

---

## **6️⃣ Cleanup Commands (Handy for Development)**

```bash
docker system df                       # Disk usage by Docker
docker system prune                     # Remove stopped containers & cache
docker system prune -a                  # Remove everything unused
```

---

### **🧠 Memory Trick for Daily Usage**

* **Check containers** → `docker ps` / `docker ps -a`
* **Start / Stop** → `docker start` / `docker stop`
* **Access** → `docker exec -it`
* **Images** → `docker images` / `docker pull` / `docker rmi`
* **Clean** → `docker system prune`

---

