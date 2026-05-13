# Docker Images

> **Why this matters:** Before Docker can run a container, it needs a blueprint — images are that blueprint, and understanding them lets you control exactly what runs inside your containers.

---

## What an Image Is

An **image** is a read-only blueprint that describes everything a container needs: the operating system layer, the language runtime, your app code, and any configuration.

You do not edit an image while it is running. It is frozen. Every container created from the same image starts from an identical state.

> Analogy: An image is a recipe. The recipe itself never changes — but you can cook it (run it) as many times as you want, and each time you get the same dish.

### Check yourself
- [ ] I can explain what "read-only" means for an image
- [ ] I understand that many containers can be created from a single image
- [ ] I can explain the difference between an image and a container in one sentence

---

## Where Images Come From

Images come from two places:

**1. Docker Hub** — a public registry (library) of pre-built images. If you want to run a PostgreSQL database, an Nginx web server, or a Python app, someone has already built and published that image. You download it with one command:

```bash
docker pull nginx
```

**2. You build one yourself** — using a file called a `Dockerfile`.

> Analogy: Docker Hub is like a cookbook store. You can grab a published recipe (pull an image) or write your own from scratch (build an image).

### Check yourself
- [ ] I know what Docker Hub is
- [ ] I know the command to download an image from Docker Hub
- [ ] I understand that I can also build my own images

---

## What a Dockerfile Is

A **Dockerfile** is a plain text file with step-by-step instructions for building your own image.

Each line adds a layer to the image — base OS, then dependencies, then your code. Here is a minimal example:

```dockerfile
FROM python:3.11          # start from an official Python image
WORKDIR /app              # set the working directory inside the container
COPY . .                  # copy your code into the image
RUN pip install -r requirements.txt   # install dependencies
CMD ["python", "app.py"]  # the command to run when the container starts
```

You build this into an image with:

```bash
docker build -t my-app .
```

The `-t` flag gives the image a name (`my-app`). The `.` means "look for the Dockerfile in the current directory."

### Check yourself
- [ ] I can describe what a Dockerfile does in my own words
- [ ] I understand what each line in the example above does at a high level
- [ ] I know the command to build an image from a Dockerfile

---

**What's next:** [docker/02-containers.md](02-containers.md) — How containers run, start, stop, and stay isolated from each other.
