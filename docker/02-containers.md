# Containers

> **Why this matters:** Containers are the thing Docker actually runs — understanding their lifecycle means you can start, stop, inspect, and clean them up with confidence.

---

## Before You Read This

This document assumes you know what an image is.
If you haven't read it yet: [docker/01-images.md](01-images.md) — What an image is and how it acts as a blueprint for containers.

---

## Running a Container

A **container** — a lightweight, isolated process — is created when you tell Docker to run an image.

```bash
docker run nginx
```

This command tells Docker: "Find the `nginx` image (download it if needed) and start a container from it."

The container runs in the foreground by default. To run it in the background (detached mode):

```bash
docker run -d nginx
```

> Analogy: The image is the recipe. Running a container is cooking the dish. The dish exists independently — you can make another one from the same recipe without affecting the first.

### Check yourself
- [x] I know the command to run a container from an image
- [x] I understand what the `-d` flag does and why I would use it

---

## Naming and Finding Containers

When you run a container, Docker assigns it a random name (like `happy_einstein`) and a unique ID. You can give it a name yourself:

```bash
docker run -d --name my-nginx nginx
```

To see all running containers:

```bash
docker ps
```

To see all containers — including stopped ones:

```bash
docker ps -a
```

### Check yourself
- [ ] I know how to give a container a custom name
- [ ] I know the command to list running containers
- [ ] I know how to see stopped containers too

---

## The Container Lifecycle

A container moves through these states:

```
Created → Running → Stopped → Removed
```

| Command | What it does |
|---------|-------------|
| `docker run` | Creates and starts a container |
| `docker stop my-nginx` | Gracefully stops a running container |
| `docker start my-nginx` | Restarts a stopped container |
| `docker rm my-nginx` | Permanently removes a stopped container |
| `docker logs my-nginx` | Shows the output (logs) from a container |
| `docker exec -it my-nginx bash` | Opens a terminal inside a running container |

> Analogy: A container is like a rental car. You pick it up (run), drive it (use it), return it (stop), and eventually the lot disposes of it (remove). The car model (image) still exists — you can rent another one anytime.

### Check yourself
- [ ] I can name the four lifecycle states of a container
- [ ] I know the commands to stop, start, and remove a container
- [ ] I know how to open a terminal inside a running container

---

## Isolation: What Containers Cannot See

Each container runs in its own isolated space. By default:

- It **cannot see** files on your host machine
- It **cannot see** other containers
- Its processes are **not visible** from outside
- When it stops, any files written inside it are **gone**

That last point is important: **containers do not save their own data by default.**
This is why Docker has Volumes — covered in the next document.

### Check yourself
- [ ] I understand what "isolated" means in practice for a container
- [ ] I know that data written inside a container disappears when it's removed
- [ ] I understand why this isolation is useful (predictability, security)

---

**What's next:** [docker/03-volumes.md](03-volumes.md) — How to keep data alive when a container stops.
