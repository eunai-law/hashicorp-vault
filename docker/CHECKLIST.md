# Docker: Master Self-Assessment Checklist

Check these off as you **understand** them — not just as you read them.
It is okay to re-read a document before checking a box. That is the point.

---

## 00 — What Is Docker
[Full document](00-what-is-docker.md)

- [ ] I can explain the "it works on my machine" problem
- [ ] I can explain how Docker solves that problem
- [ ] I can name the two main pieces of Docker (Engine and CLI)
- [ ] I can describe a container in one sentence

**I understand what Docker is and why it exists.**

---

## 01 — Images
[Full document](01-images.md)

- [ ] I can explain what a Docker image is
- [ ] I understand that images are read-only blueprints
- [ ] I know what Docker Hub is
- [ ] I know the command to download an image (`docker pull`)
- [ ] I know what a Dockerfile is
- [ ] I know the command to build an image from a Dockerfile (`docker build`)

**I understand what images are and where they come from.**

---

## 02 — Containers
[Full document](02-containers.md)

- [ ] I know the command to run a container (`docker run`)
- [ ] I know the `-d` flag and what it does
- [ ] I know how to give a container a name (`--name`)
- [ ] I can list running containers (`docker ps`)
- [ ] I can list all containers including stopped ones (`docker ps -a`)
- [ ] I know the four lifecycle states: Created → Running → Stopped → Removed
- [ ] I know how to stop, start, and remove a container
- [ ] I know how to view container logs
- [ ] I know how to open a terminal inside a running container
- [ ] I understand what container isolation means in practice
- [ ] I know that data inside a container is lost when it is removed

**I can manage the full lifecycle of a container.**

---

## 03 — Volumes
[Full document](03-volumes.md)

- [ ] I can explain what "ephemeral" means for a container
- [ ] I understand why data loss is by design, not a bug
- [ ] I know what a Docker volume is
- [ ] I know how to create a volume (`docker volume create`)
- [ ] I can read the `-v volume-name:path` syntax
- [ ] I understand what a bind mount is
- [ ] I know when to use a volume vs a bind mount
- [ ] I know how to list and remove volumes

**I can persist data across container restarts and removals.**

---

## 04 — Networking
[Full document](04-networking.md)

- [ ] I understand why containers cannot communicate by default
- [ ] I know what a Docker network is
- [ ] I know how to create a network and connect containers to it
- [ ] I understand that containers on the same network find each other by name
- [ ] I understand what port mapping is
- [ ] I can read the `-p host-port:container-port` syntax
- [ ] I know why port mapping is needed to reach a container from a browser

**I can connect containers to each other and to the outside world.**

---

## 05 — Docker Compose
[Full document](05-docker-compose.md)

- [ ] I can explain why running containers manually is a problem at scale
- [ ] I know what a `docker-compose.yml` file does
- [ ] I can read the `services`, `volumes`, and `ports` sections
- [ ] I understand what `depends_on` does
- [ ] I know the command to start all services (`docker compose up -d`)
- [ ] I know the difference between `docker compose down` and `docker compose down -v`
- [ ] I know how to tell Compose to build a local image
- [ ] I know when to use `--build`

**I can define and run a multi-container system with a single command.**

---

## All Done?

If every box above is checked, you have a solid working understanding of Docker.

Continue to: [vault/00-what-is-vault.md](../vault/00-what-is-vault.md) — The secrets problem Vault solves and what Vault is at its core.
