# DOCKER COMMANDS


### **1️⃣ Check Installation & System Info**

```bash
docker --version        # Check Docker version
docker info             # Detailed info about Docker engine
docker ps               # List running containers
docker ps -a            # List all containers (including stopped)
docker images           # List downloaded images
docker version          # Client & server version info
```

---

### **2️⃣ Images (Download, Build, Remove)**

```bash
docker pull nginx                # Download image from Docker Hub
docker images                    # List all images
docker rmi nginx                 # Remove an image
docker image prune               # Remove unused images
docker build -t myapp:1.0 .       # Build image from Dockerfile
docker tag myapp:1.0 myapp:latest # Tag image
docker push username/myapp        # Push image to Docker Hub
```

---

### **3️⃣ Containers (Run, Stop, Logs)**

```bash
docker run hello-world                  # Quick test
docker run -it ubuntu bash              # Run interactive Ubuntu shell
docker run -d -p 8080:80 nginx          # Run Nginx detached & map ports
docker exec -it <container_id> bash     # Access a running container
docker logs -f <container_id>           # View logs live
docker stop <container_id>              # Stop a running container
docker start <container_id>             # Start a stopped container
docker restart <container_id>           # Restart container
docker rm <container_id>                # Remove container
docker container prune                  # Remove all stopped containers
```

---

### **4️⃣ Volumes & Data Persistence**

```bash
docker volume ls                         # List volumes
docker volume create myvolume            # Create volume
docker run -d -v myvolume:/data nginx    # Attach volume to container
docker volume rm myvolume                # Remove volume
docker volume prune                      # Remove unused volumes
```

---

### **5️⃣ Networks**

```bash
docker network ls                        # List networks
docker network create mynet              # Create custom bridge network
docker run -d --network=mynet nginx      # Attach container to network
docker network inspect mynet             # Inspect network
docker network rm mynet                   # Remove network
```

---

### **6️⃣ Docker Compose (If Installed)**

* Start services:

```bash
docker compose up -d
```

* Stop services:

```bash
docker compose down
```

* View logs:

```bash
docker compose logs -f
```

---

### **7️⃣ Clean Up Docker**

```bash
docker system prune             # Remove stopped containers, networks, cache
docker system prune -a          # Remove everything unused (images too)
docker builder prune            # Clean up build cache
```

---
