# Volumes

> **Why this matters:** Containers throw away their data when removed — volumes are how you save anything that needs to survive.

---

## Before You Read This

This document builds on container isolation. If you haven't read it:
[docker/02-containers.md](02-containers.md) — How containers run, start, stop, and stay isolated from each other.

---

## The Persistence Problem

Containers are ephemeral — that is the technical word for "they don't keep their data."

If you run a database inside a container and then remove that container, every row of data is gone. If you run a web app that saves uploaded files, those files vanish the next time the container restarts.

This is by design. Containers are meant to be disposable. But your *data* is not disposable.

> Analogy: A container is like a whiteboard. Great for working things out, but when you erase it, it's gone. A volume is the notebook where you write down anything worth keeping.

### Check yourself
- [ ] I can explain what "ephemeral" means for a container
- [ ] I understand why data loss on container removal is by design, not a bug

---

## What a Volume Is

A **volume** is a storage location managed by Docker that lives outside the container's filesystem.

When you attach a volume to a container, that container can read and write to it. When the container stops or is removed, the volume and its data remain. Another container can attach to the same volume and pick up where the first one left off.

```bash
docker volume create my-data

docker run -d \
  --name my-db \
  -v my-data:/var/lib/postgresql/data \
  postgres
```

The `-v my-data:/var/lib/postgresql/data` part says: "Mount the volume `my-data` at the path `/var/lib/postgresql/data` inside the container."

> Analogy: A volume is like a USB drive plugged into the rental car. You can swap the car (container) out entirely — the USB drive and everything on it stays with you.

### Check yourself
- [ ] I know the command to create a volume
- [ ] I understand the `-v` flag syntax: `volume-name:path-inside-container`
- [ ] I can explain why volumes survive container removal

---

## Bind Mounts: Your Files, Inside the Container

A **bind mount** is a second type of storage attachment. Instead of a Docker-managed volume, you point directly at a folder on your own machine.

```bash
docker run -d \
  --name my-app \
  -v /Users/you/my-project:/app \
  my-app-image
```

This mounts your local `/Users/you/my-project` folder into the container at `/app`. Changes you make on your laptop appear instantly inside the container — and vice versa.

Bind mounts are useful during development. Volumes are better for production (Docker manages them, they're more portable).

### Check yourself
- [ ] I understand the difference between a volume and a bind mount
- [ ] I know when I would use each one (bind mount = dev, volume = production)

---

## Listing and Removing Volumes

```bash
docker volume ls          # list all volumes
docker volume inspect my-data   # see details about a specific volume
docker volume rm my-data  # remove a volume (only works if no container is using it)
```

### Check yourself
- [ ] I know how to list volumes
- [ ] I know how to remove a volume when I no longer need it

---

**What's next:** [docker/04-networking.md](04-networking.md) — How containers talk to each other and to the outside world.
