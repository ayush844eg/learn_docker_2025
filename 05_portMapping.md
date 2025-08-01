# PORT mapping


## **1️⃣ Why Port Mapping is Needed**

* **Containers are isolated from the host machine**:

  * Each container has its **own private network namespace**.
  * Services running inside a container (like Nginx on port 80) **cannot be accessed** from your host machine by default.

* **Port Mapping connects container ports to the host**:

  * Host listens on a port (like 8080) → forwards traffic to the container port (like 80).
  * Syntax:

    ```
    docker run -p <host_port>:<container_port> <image>
    ```

> Example:
> `docker run -d -p 8080:80 nginx`
>
> * Host port **8080** → Container port **80**
> * Access: `http://localhost:8080`

---

## **2️⃣ Docker Port Mapping Syntax**

```
-p <host_port>:<container_port>
```

---

## **1️⃣ Why Port Mapping is Needed**

* **Containers are isolated from the host machine**:

  * Each container has its **own private network namespace**.
  * Services running inside a container (like Nginx on port 80) **cannot be accessed** from your host machine by default.

* **Port Mapping connects container ports to the host**:

  * Host listens on a port (like 8080) → forwards traffic to the container port (like 80).
  * Syntax:

    ```
    docker run -p <host_port>:<container_port> <image>
    ```

> Example:
> `docker run -d -p 8080:80 nginx`
>
> * Host port **8080** → Container port **80**
> * Access: `http://localhost:8080`

---

## **2️⃣ Docker Port Mapping Syntax**

```
-p <host_port>:<container_port>
```

* **Host Port:** Port on your machine (accessible externally)
* **Container Port:** Port your app/service inside container listens on

**Example:**

```bash
docker run -d -p 7001:7001 eg-image-resizer-image-resizer
```

* Maps **host:7001 → container:7001**
* Anyone accessing `localhost:7001` hits the container service.

---

### **2.1 Multiple Ports**

Map multiple ports using multiple `-p` flags:

```bash
docker run -d \
  -p 8080:80 \
  -p 8443:443 \
  nginx
```

* Maps **80 → 8080** and **443 → 8443**

---

### **2.2 Random Host Port (Ephemeral)**

Let Docker assign a **random free port**:

```bash
docker run -d -P nginx
```

* `-P` = Publish **all exposed ports** to **random host ports**
* Check with:

```bash
docker ps
```

Example output:

```
PORTS
0.0.0.0:49153->80/tcp
```

---

## **3️⃣ Understanding Host Binding**

You can bind the container to:

* **All interfaces (default)** → `0.0.0.0:8080:80`
* **Specific interface (host IP)**:

```bash
docker run -d -p 127.0.0.1:8080:80 nginx
```

* Only accessible from **localhost**, not from other devices.

---

## **4️⃣ Checking Port Mappings**

```bash
docker ps
```

Example:

```
CONTAINER ID   IMAGE     PORTS
7d2a961e19fc   nginx     0.0.0.0:8080->80/tcp
```

* **Host 8080** → **Container 80**

Also, use:

```bash
docker port <container_id>
```

---

## **5️⃣ Common Port Mapping Examples**

1. **Web Server (Nginx/Apache)**

```bash
docker run -d -p 8080:80 nginx
# Access at http://localhost:8080
```

2. **MySQL Database**

```bash
docker run -d -p 3306:3306 mysql:8.0
# Connect to MySQL using host: 127.0.0.1:3306
```

3. **Multiple Services**

```bash
docker run -d -p 7001:7001 image-resizer
docker run -d -p 7002:7002 face-recognition
```

---

## **6️⃣ Quick Diagram**

```
[ Host Machine Port 8080 ] <---> [ Docker NAT ] <---> [ Container Port 80 ]
         ↑
     Access via browser
```

* Docker uses **NAT (iptables)** to forward traffic.

---

## **7️⃣ Key Tips**

* If you **don’t map ports**, the container app is **not accessible from outside**.
* Use `-P` for **random mapping**; `-p` for **manual mapping**.
* Mapping can be **host\:container** or **host\_ip\:host\_port\:container\_port**.

---

