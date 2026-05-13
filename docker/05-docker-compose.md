# Docker Compose

> **Why this matters:** Real applications are never just one container — Compose lets you define your entire multi-container system in one file and start everything with a single command.

---

## Before You Read This

Compose builds on images, containers, volumes, and networking. Make sure you've covered:
- [docker/01-images.md](01-images.md) — What an image is and how it acts as a blueprint for containers.
- [docker/03-volumes.md](03-volumes.md) — How to keep data alive when a container stops.
- [docker/04-networking.md](04-networking.md) — How containers talk to each other and to the outside world.

---

## The Problem With Running Containers Manually

Imagine your app has three pieces: a web server, a database, and a cache.
Running each one manually looks like this:

```bash
docker network create my-network
docker volume create db-data
docker run -d --name db --network my-network -v db-data:/data postgres
docker run -d --name cache --network my-network redis
docker run -d --name web --network my-network -p 8080:80 my-app
```

Five commands. Every time. Easy to forget one. Easy to type it wrong.

**Docker Compose** replaces all of that with a single file and a single command.

### Check yourself
- [ ] I can explain why running containers manually becomes a problem at scale
- [ ] I understand that Compose is a solution to that problem

---

## The docker-compose.yml File

A **`docker-compose.yml`** file describes your entire system: which containers to run, how they connect, which ports to expose, and which volumes to use.

Here is the same three-container system from above, written as Compose:

```yaml
services:
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data

  cache:
    image: redis

  web:
    image: my-app
    ports:
      - "8080:80"
    depends_on:
      - db
      - cache

volumes:
  db-data:
```

Compose automatically creates a shared network for all services — you don't need to define it manually.

> Analogy: A `docker-compose.yml` is like a blueprint for an entire building. Instead of constructing each room one-by-one, you hand the blueprint to Docker and it builds the whole thing.

### Check yourself
- [ ] I can identify the three top-level sections in a Compose file (`services`, `volumes`, and the implicit network)
- [ ] I understand what `depends_on` does
- [ ] I can read the `ports` and `volumes` syntax

---

## Running and Stopping with Compose

All Compose commands are run from the same directory as your `docker-compose.yml` file.

```bash
docker compose up -d        # start all services in the background
docker compose down         # stop and remove all containers and networks
docker compose down -v      # same, but also delete volumes (data will be lost)
docker compose logs -f      # stream logs from all services
docker compose ps           # show status of all services
```

The difference between `down` and `down -v` matters. Use `down` when you want to stop but keep your data. Use `down -v` only when you want a completely clean slate.

### Check yourself
- [ ] I know the command to start all services defined in a Compose file
- [ ] I understand the difference between `docker compose down` and `docker compose down -v`
- [ ] I know how to check the status of all running services

---

## Building Your Own Image in Compose

If your app needs a custom image (not one from Docker Hub), you can tell Compose to build it from a Dockerfile:

```yaml
services:
  web:
    build: .          # build from the Dockerfile in the current directory
    ports:
      - "8080:80"
```

Then run:

```bash
docker compose up --build   # rebuild the image before starting
```

### Check yourself
- [ ] I know how to tell Compose to build a local image instead of pulling one
- [ ] I know when to use `--build`

---

**What's next:** [docker/CHECKLIST.md](CHECKLIST.md) — Master self-assessment for everything in the Docker section.

After the checklist, continue to: [vault/00-what-is-vault.md](../vault/00-what-is-vault.md) — The secrets problem Vault solves and what Vault is at its core.
