# What Is Docker?

> **Why this matters:** Without Docker, software that works on your computer often breaks on someone else's — Docker eliminates that problem permanently.

---

## The Problem Docker Solves

You write an app on your laptop. It works perfectly. You send it to a colleague.
It crashes. Why? Their machine has a different version of Python, a different
operating system, different system libraries. This is so common it has a name:
**"it works on my machine."**

Docker fixes this by packaging your app together with *everything it needs to run* — the language runtime, the libraries, the config — into one portable unit.

> Analogy: Think of it like a shipping container. The goods inside are safe and unchanged whether the container is on a truck, a ship, or a warehouse. The environment outside doesn't matter.

### Check yourself
- [x] I can explain the "it works on my machine" problem in my own words
- [x] I understand why packaging an app with its dependencies solves that problem

---

## What Docker Is

**Docker** is the tool that creates, runs, and manages containers on your machine.

It has two main pieces you'll use:
- The **Docker Engine** — the background service that actually runs containers
- The **Docker CLI** — the terminal commands you type to control it (`docker run`, `docker stop`, etc.)

You install Docker once. After that, any container anyone has built can run on your machine the same way it runs everywhere else.

> Analogy: Docker is the shipping port. It loads, unloads, and tracks containers — you don't need to know what's inside each one to move it.

### Check yourself
- [ ] I can name the two main pieces of Docker (Engine and CLI)
- [ ] I understand that Docker runs on my machine and manages containers for me

---

## What a Container Is

A **container** is a lightweight, isolated process that runs your app.

"Isolated" means it cannot see other containers, other apps on your machine, or your files — unless you explicitly allow it. Each container behaves as if it is the only thing running.

"Lightweight" means it is not a full virtual machine. It does not need to boot a whole operating system. It starts in seconds.

> Analogy: A container is like a rented studio apartment. It has its own kitchen, bathroom, and locks. The tenant inside doesn't know or care about the other apartments in the building.

### Check yourself
- [ ] I can explain what "isolated" means for a container
- [ ] I understand the difference between a container and a full virtual machine
- [ ] I can describe a container in one sentence to someone who has never heard of Docker

---

**What's next:** [docker/01-images.md](01-images.md) — What an image is and how it acts as a blueprint for containers.
