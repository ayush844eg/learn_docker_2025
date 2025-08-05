

# **Docker Compose**

---

## **1️⃣ What is Docker Compose?**

* **Docker Compose** is a tool for **defining and running multi-container applications**.
* Instead of manually running multiple `docker run` commands, you:

  1. **Describe** all your services in a `docker-compose.yml` file.
  2. Start them all with a single command:

     ```bash
     docker compose up
     ```
* Handles:

  * **Multiple containers** (services)
  * Networking between services
  * Environment variables
  * Volumes
  * Dependencies (e.g., run DB before app)

---

## **2️⃣ Why Use Docker Compose?**

Without Compose:

```bash
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=pass mysql:8
docker run -d --name myapp --link mysql:mysql myapp:latest
```

With Compose:

```yaml
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: pass
  app:
    image: myapp:latest
    depends_on:
      - db
```

Then just:

```bash
docker compose up
```

---

## **3️⃣ The `docker-compose.yml` Structure**

Basic sections:

```yaml
version: "3.9"   # Compose file format version

services:        # All containers
  service_name:
    image:       # Image from Docker Hub or custom build
    build:       # Or build from Dockerfile
    ports:       # Port mappings
    environment: # Env vars
    volumes:     # Persistent storage
    depends_on:  # Start order dependencies

volumes:         # Named volumes (optional)

networks:        # Custom networks (optional)
```

---

## **4️⃣ Key Elements Explained with Examples**

### **4.1 `version`**

* Specifies Docker Compose file format version.
* Common: `"3"`, `"3.8"`, `"3.9"` for modern Docker.

### **4.2 `services`**

Each service is a container.

Example:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
```

* Runs Nginx accessible at `localhost:8080`.

---

### **4.3 `image` vs `build`**

* **`image`**: Pulls from registry.
* **`build`**: Builds locally from Dockerfile.

Example:

```yaml
  backend:
    build: ./backend
    ports:
      - "5000:5000"
```

---

### **4.4 `ports`**

* Maps host\:container ports.

```yaml
ports:
  - "8080:80"
```

---

### **4.5 `environment`**

* Set environment variables.

```yaml
environment:
  - NODE_ENV=production
  - DB_HOST=db
```

---

### **4.6 `volumes`**

* Persist data or mount code for development.

```yaml
volumes:
  - mydata:/var/lib/mysql
  - ./app:/usr/src/app
```

---

### **4.7 `depends_on`**

* Ensure service starts after dependencies (not health check, just order).

```yaml
depends_on:
  - db
```

---

### **4.8 `networks`**

* Compose creates a default network so services can talk via service name.

```yaml
networks:
  mynet:
```

---

## **5️⃣ Example: Node.js + MySQL with Compose**

```yaml
version: "3.9"

services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: mydb
    volumes:
      - dbdata:/var/lib/mysql
    ports:
      - "3306:3306"

  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db
      DB_USER: root
      DB_PASSWORD: rootpass
    depends_on:
      - db

volumes:
  dbdata:
```

**Run:**

```bash
docker compose up
```

* Both DB and App start together.
* App can reach DB using `db:3306` because they share the same network.

---

## **6️⃣ Common Docker Compose Commands**

| Command                               | Description                                           |
| ------------------------------------- | ----------------------------------------------------- |
| `docker compose up`                   | Start all services (build if needed)                  |
| `docker compose up -d`                | Start in detached mode                                |
| `docker compose down`                 | Stop and remove containers, networks, default volumes |
| `docker compose build`                | Build or rebuild images                               |
| `docker compose ps`                   | List running services                                 |
| `docker compose logs`                 | Show logs from all services                           |
| `docker compose logs -f`              | Follow logs                                           |
| `docker compose stop`                 | Stop containers without removing                      |
| `docker compose restart`              | Restart services                                      |
| `docker compose exec <service> <cmd>` | Run command in a running container                    |

---

## **7️⃣ How Networking Works in Compose**

* All services in the same `docker-compose.yml` share a **default network**.
* They can reach each other by **service name**.
* Example:

  * Service `app` can connect to MySQL using `DB_HOST=db` instead of `localhost`.

---

## **8️⃣ Best Practices**

* Use **`.env` files** to store secrets/configs instead of hardcoding.
* Keep production and development compose files separate (`docker-compose.prod.yml`).
* Use `depends_on` for order, but for true readiness checks, implement **healthchecks**.

---

## **9️⃣ Example Workflow**

```bash
# Start services in background
docker compose up -d

# Check logs
docker compose logs -f app

# Run shell inside app service
docker compose exec app bash

# Stop and clean up
docker compose down
```

