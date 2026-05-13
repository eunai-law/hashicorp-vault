# Docker Networking

> **Why this matters:** Most real applications have multiple parts (an app server, a database, a cache) — networking is how those parts find and talk to each other inside Docker.

---

## Before You Read This

This document assumes you understand containers and isolation.
If you haven't read it: [docker/02-containers.md](02-containers.md) — How containers run, start, stop, and stay isolated from each other.

---

## The Default: Containers Are Cut Off

Because containers are isolated, they cannot communicate with each other by default. Container A cannot reach Container B even if they are both running on the same machine.

This is safe — but most apps need their pieces to talk. Docker solves this with **networks**.

> Analogy: Each container starts in its own soundproof room. A Docker network is a hallway that connects specific rooms so the people inside can talk to each other.

### Check yourself
- [ ] I understand why containers cannot communicate by default
- [ ] I understand that Docker networks are how you change that

---

## Docker Networks

A **Docker network** is a private, virtual network that you create. Containers connected to the same network can reach each other by name.

```bash
docker network create my-network

docker run -d --name db --network my-network postgres
docker run -d --name app --network my-network my-app-image
```

Now, inside the `app` container, you can connect to the database using the hostname `db` — exactly the container's name. Docker handles the DNS automatically.

```
app container → connects to → "db" → Docker resolves → db container's IP
```

> Analogy: A Docker network is a private LAN (local area network) in an office. Every device on the LAN can find the others by their hostname. Devices outside the office can't see in.

### Check yourself
- [ ] I know the command to create a Docker network
- [ ] I know how to connect a container to a network using `--network`
- [ ] I understand that containers on the same network can find each other by name

---

## Port Mapping: Opening a Door to the Outside

By default, even if a container runs a web server on port 80, you cannot reach it from your browser. The container's network is private.

**Port mapping** punches a hole: it maps a port on your machine to a port inside the container.

```bash
docker run -d -p 8080:80 --name web nginx
```

The `-p 8080:80` flag says: "Forward traffic from my machine's port 8080 into the container's port 80."

Now `http://localhost:8080` in your browser reaches the Nginx server running inside the container.

```
Your browser → localhost:8080 → Docker → container port 80 → Nginx
```

### Check yourself
- [ ] I understand why port mapping is necessary
- [ ] I can read the `-p host-port:container-port` syntax
- [ ] I know what happens if I forget to add `-p` when running a web server

---

## Common Networking Commands

```bash
docker network ls                          # list all networks
docker network inspect my-network          # see which containers are connected
docker network connect my-network my-container   # add a container to a network
docker network disconnect my-network my-container  # remove a container from a network
docker network rm my-network               # delete a network
```

### Check yourself
- [ ] I know how to list and inspect Docker networks
- [ ] I know how to add or remove a container from a network after it is running

---

**What's next:** [docker/05-docker-compose.md](05-docker-compose.md) — How to define and run multiple containers together as one system.
