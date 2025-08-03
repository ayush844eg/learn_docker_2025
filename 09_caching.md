

## **1️⃣ How Docker Caching Works**

* When you run `docker build`, **Docker executes the Dockerfile instructions in order**, creating a **layer** for each instruction (`FROM`, `RUN`, `COPY`, etc.).
* Before running each instruction, Docker checks:

  1. **Is the instruction exactly the same as before?**
  2. **Has any file it depends on changed?**
* If both answers are **No changes**, Docker **reuses the cached layer** instead of rebuilding it.

---

## **2️⃣ Why Order Matters**

Docker caching works **top to bottom**, so if an instruction changes, **all instructions after it are rebuilt**.

Example from your Dockerfile:

```dockerfile
COPY package.json package-lock.json main.js ./
RUN npm install
```

Here’s what happens:

* If **any** of `package.json`, `package-lock.json`, or `main.js` changes → cache for this COPY step is invalidated → `npm install` will run again, even if dependencies didn’t change.
* That’s why we usually split it:

```dockerfile
COPY package.json package-lock.json ./
RUN npm install
COPY main.js ./
```

Now:

* If `main.js` changes but `package.json` doesn’t → `npm install` step is cached and skipped.
* Build is much faster.

---

## **3️⃣ Example Build Flow**

### First Build

```dockerfile
FROM node:18
COPY package.json package-lock.json ./
RUN npm install
COPY . .
ENTRYPOINT ["node", "main.js"]
```

**Layers created:**

1. Base image (Node 18)
2. Copy package manifests
3. `npm install` result
4. Copy source code
5. Entrypoint

### Second Build (only code changed)

* Docker sees **package.json** unchanged → skips `npm install`
* Only rebuilds:

  * `COPY . .`
  * ENTRYPOINT layer (trivial)

Result: **Much faster build** because dependencies are reused from cache.

---

## **4️⃣ Caching Do’s & Don’ts**

✅ **Do:**

* Copy dependency files (`package*.json`) **before** app source.
* Use `.dockerignore` to avoid sending unnecessary files (especially `node_modules`).
* Keep OS/package installs in one `RUN` to reduce cache invalidations.

❌ **Don’t:**

* Combine `package.json` and all source files in the same `COPY` if you want `npm install` to be cached.
* Change apt commands unnecessarily (changing `RUN apt-get install` order invalidates cache).

---

## **5️⃣ Cache Invalidating Example**

Say you have:

```dockerfile
COPY package.json package-lock.json ./
RUN npm install
COPY main.js ./
```

* **Change in `main.js` only** → Cache hit for `COPY package.json...` and `RUN npm install`, only `COPY main.js` is rebuilt.
* **Change in `package.json`** → Cache miss for `COPY package.json...`, `RUN npm install` is rebuilt, then `COPY main.js` is rebuilt.

---

## **6️⃣ Tip for Node.js Projects**

If you run `npm install` in the same layer as `COPY main.js`:

* Every time you change your app code, Docker will reinstall dependencies — slow and unnecessary.
* Always **separate dependency installation from source code copy**.

---
