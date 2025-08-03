# **Publishing custom Docker images to Docker Hub**

## **1. Prerequisites**

* Docker installed and working locally.
* A Docker Hub account: sign up at hub.docker.com.
* (Recommended) Create a **personal access token** on Docker Hub to use instead of your password for `docker login` (more secure).

---

## **2. Log in to Docker Hub from CLI**

```bash
docker login
```

* You’ll be prompted for your Docker Hub **username** and **password** (or personal access token in place of password).
* After successful login, credentials are cached (e.g., in `~/.docker/config.json`).

To log out:

```bash
docker logout
```

---

## **3. Build your image locally**

Example (from your Node.js app):

```bash
docker build -t ayush844eg_nodejsapp .
```

This creates an image tagged `ayush844eg_nodejsapp` locally.

---

## **4. Tag the image for Docker Hub**

Docker Hub images must be named with your username (or org) as prefix:

```bash
docker tag ayush844eg_nodejsapp yourdockerhubusername/ nodejsapp:latest
```

Example:

```bash
docker tag ayush844eg_nodejsapp ayush844eg/nodejsapp:latest
```

You can also use versioned tags:

```bash
docker tag ayush844eg_nodejsapp ayush844eg/nodejsapp:v1.0.0
```

---

## **5. Create the repository on Docker Hub**

* Go to Docker Hub web UI.
* Create a new repository named `nodejsapp` (or whatever you tagged).

  * Choose visibility: **public** (anyone can pull) or **private** (you control access).

*Note:* If it’s private, ensure your plan supports private repos.

---

## **6. Push the image to Docker Hub**

```bash
docker push ayush844eg/nodejsapp:latest
```

or for versioned tag:

```bash
docker push ayush844eg/nodejsapp:v1.0.0
```

* Docker will upload each layer; subsequent pushes that reuse layers are faster.

---

## **7. Verify & Pull**

On any machine (or after logout), you can pull the image:

```bash
docker pull ayush844eg/nodejsapp:latest
```

Run it:

```bash
docker run -it ayush844eg/nodejsapp:latest
```

---

## **8. Updating the Image**

1. Make changes locally (e.g., update `main.js` or dependencies).
2. Rebuild, optionally with a new tag:

   ```bash
   docker build -t ayush844eg/nodejsapp:v1.0.1 .
   docker push ayush844eg/nodejsapp:v1.0.1
   ```
3. You can also retag and push `latest` if you want the mutable “latest” pointer to move:

   ```bash
   docker tag ayush844eg/nodejsapp:v1.0.1 ayush844eg/nodejsapp:latest
   docker push ayush844eg/nodejsapp:latest
   ```

---

## **9. Optional: Automate Builds via GitHub/GitLab**

* In Docker Hub, you can link a repository (GitHub/GitLab) and set up **automated builds**.
* Push your Dockerfile and code to that git repo.
* Docker Hub will build on each commit/tag based on rules you define and publish images automatically.

---

## **10. Best Practices**

* **Use a `.dockerignore`** to exclude unnecessary files (e.g., `node_modules`, `.git`, temp files) from the build context.
* **Avoid baking secrets** into images. Use environment variables or secret management at runtime.
* **Use versioned tags** (e.g., `v1.2.3`) instead of relying only on `latest` for reproducibility.
* **Scan images**: use `docker scan ayush844eg/nodejsapp:latest` to detect vulnerabilities.
* **Multistage builds**: keep final image small by building in one stage and copying only what's needed into the runtime image.

---

## **11. Cleaning Up Old Local Tags (optional)**

After pushing, you can remove local tags you don’t need:

```bash
docker image rm ayush844eg/nodejsapp:latest
```

---

## **12. Example Full Workflow**

```bash
# Login
docker login

# Build locally
docker build -t mynodeapp .

# Tag for Docker Hub
docker tag mynodeapp ayush844eg/nodejsapp:v1.0.0

# Push
docker push ayush844eg/nodejsapp:v1.0.0

# Optionally update latest
docker tag ayush844eg/nodejsapp:v1.0.0 ayush844eg/nodejsapp:latest
docker push ayush844eg/nodejsapp:latest

# Pull & run elsewhere
docker pull ayush844eg/nodejsapp:latest
docker run -it ayush844eg/nodejsapp:latest
```

---

## **13. Permissions & Visibility**

* Public repo: anyone can `docker pull`.
* Private repo: you must be authenticated; you can grant collaborator access if needed.

---

