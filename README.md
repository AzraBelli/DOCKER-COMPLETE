# 📓 Docker Complete — Learning Notes

A small Node.js/Express project I built to practice the fundamentals of Docker: writing a `Dockerfile`, building an image, running a container, and exposing a port. This README is written as a set of learning notes rather than a formal doc — a record of what the project does and what each piece is for.

---

## 🗂 What's in here

```
.
├── Dockerfile        # Image definition for the app
├── app.mjs            # Express server entry point
├── helpers.mjs        # Dummy database connection helper
└── package.json       # Project metadata and dependencies
```

---

## 🧠 The app, in plain words

It's a bare-bones Express server:

- One route, `GET /`, that responds with `<h2>Hi there!</h2>`.
- Before it starts listening, it calls `connectToDatabase()` from `helpers.mjs` and waits for it to resolve.
- `connectToDatabase()` doesn't actually talk to a database — it's a stand-in `Promise` wrapped around a `setTimeout`, just there to simulate "waiting for a connection" before the server boots.

So the interesting part isn't the app logic — it's what surrounds it: the `Dockerfile`.

---

## 🐳 Step 1 — Reading the Dockerfile

```dockerfile
FROM node:14
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "app.mjs"]
```

Notes on each line:

- `FROM node:14` — starts from an official image that already has Node.js 14 installed. No need to install Node manually.
- `WORKDIR /app` — every following instruction runs relative to `/app` inside the image. Also creates that folder if it doesn't exist.
- `COPY package.json .` — copies **only** the manifest first, before the rest of the source. This is a deliberate ordering trick.
- `RUN npm install` — installs dependencies inside the image.
- `COPY . .` — now copies everything else (the actual source code).
- `EXPOSE 3000` — documents that the container listens on port 3000. This is informational; it doesn't actually publish the port.
- `CMD ["node", "app.mjs"]` — the default command that runs when a container starts from this image.

**Why copy `package.json` before the rest of the code?**
Docker caches each layer. If only application code changes (and dependencies don't), Docker can reuse the cached `npm install` layer instead of reinstalling everything from scratch. Copying `package.json` separately, first, is what makes that caching possible. If `COPY . .` came before `RUN npm install`, any code change would invalidate the cache and force a full reinstall every time.

---

## 🏗 Step 2 — Building the image

```bash
docker build -t docker-complete .
```

- `-t docker-complete` tags the image with a readable name instead of a random ID.
- `.` tells Docker to use the current directory as the build context (where it looks for the `Dockerfile` and the files to `COPY`).

---

## ▶️ Step 3 — Running the container

```bash
docker run -p 3000:3000 docker-complete
```

- `-p 3000:3000` maps port 3000 on the host machine to port 3000 inside the container (`EXPOSE` alone doesn't do this — it only documents the port).
- Without `-p`, the app would run inside the container but wouldn't be reachable from the host.

Then visit **http://localhost:3000** — should show `Hi there!`.

Useful follow-up commands while testing:

```bash
docker ps                  # list running containers
docker logs <container_id> # see the app's console output
docker stop <container_id> # stop it
```

---

## 🧪 Running it without Docker (for comparison)

```bash
npm install
npm start
```

Same result at `http://localhost:3000` — this is basically what happens *inside* the container, just run directly on the host instead.

---

## 📌 Key takeaways

- A `Dockerfile` is a recipe: base image → workdir → dependencies → source code → exposed port → start command.
- Instruction order matters for build caching — copy dependency manifests before source code.
- `EXPOSE` is documentation; `-p` on `docker run` is what actually publishes a port to the host.
- `helpers.mjs` shows a common pattern: awaiting an async setup step (like a DB connection) before the server starts accepting requests — even when it's just a dummy `Promise`.

---

## 🔧 Tech Stack

- Node.js 14
- Express 4

## License

ISC
