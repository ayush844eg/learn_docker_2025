# **environment variables in Docker containers**


## **1️⃣ What are Environment Variables in Docker?**

* Environment variables (**ENV vars**) are **key-value pairs** available inside a container.
* They allow you to **configure container behavior without changing the image**.

Example uses:

* Set **database credentials** (`DB_USER`, `DB_PASSWORD`)
* Control **application mode** (`NODE_ENV=production`)
* Configure **ports, API keys, feature flags**

> **Why use ENV vars?**
>
> * Avoid hardcoding secrets into images.
> * Easily change configs across dev/staging/prod without rebuilding.

---

## **2️⃣ Ways to Set Environment Variables**

### **2.1 Using `-e` flag in `docker run`**

```bash
docker run -it \
  -e MY_NAME=Ayush \
  -e APP_ENV=production \
  ubuntu bash
```

Inside the container:

```bash
root@container:/# echo $MY_NAME
Ayush
root@container:/# env | grep APP_ENV
APP_ENV=production
```

---

### **2.2 Using `--env-file` for multiple variables**

1. Create a file `.env`:

```
DB_HOST=mysql
DB_USER=root
DB_PASSWORD=secret
```

2. Run container with:

```bash
docker run -it --env-file .env ubuntu bash
```

* All variables in `.env` are injected into the container.

---

### **2.3 Using Dockerfile `ENV` Instruction**

Inside **Dockerfile**:

```dockerfile
FROM node:20
ENV NODE_ENV=production
ENV PORT=3000
```

* These variables are **baked into the image**.
* Can be overridden at runtime using `-e`.

---

### **2.4 Using `docker-compose` (Recommended for multi-service apps)**

`docker-compose.yml`:

```yaml
version: "3"
services:
  web:
    image: node:20
    environment:
      - NODE_ENV=production
      - PORT=3000
    env_file:
      - .env
```

Then:

```bash
docker compose up -d
```

---

## **3️⃣ Inspecting Environment Variables in a Container**

1. **Using `docker exec` + `printenv`:**

```bash
docker exec <container> printenv
```

2. **Using `docker inspect`:**

```bash
docker inspect <container_id> --format='{{json .Config.Env}}'
```

Outputs a JSON array of all environment variables.

---

## **4️⃣ Overriding & Precedence**

1. **Dockerfile `ENV`** → Base default
2. **`docker run -e` or `--env-file`** → Overrides default
3. **docker-compose `environment`** → Overrides all

> ⚡ You can override environment variables **without rebuilding** the image.

---

## **5️⃣ Security Notes**

* Avoid putting **secrets in Dockerfiles** → they become part of the image.
* Use `--env-file` or **Docker Secrets** for production.
* Don’t `docker inspect` on production containers with secrets if logs are exposed.

---

## **6️⃣ Real-World Example**

You want to run a MySQL container with a custom password:

```bash
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=supersecret \
  -p 3306:3306 \
  mysql:8.0
```

* MySQL reads the password from **env variable** at container start.
* Connect via `mysql -u root -p` using that password.

---

## **7️⃣ Key Commands Summary**

```bash
# Set env inline
docker run -e KEY=value image

# Load env from file
docker run --env-file .env image

# Show env variables inside container
docker exec <container> printenv

# Inspect container environment
docker inspect <container> | grep -i env
```

