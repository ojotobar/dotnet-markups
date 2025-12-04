Here’s a **clean, beginner-friendly Docker basics guide** with a **cheat sheet of common commands** and a **simple setup walkthrough**.
Perfect for getting comfortable with Docker quickly.

---

# 📌 **1. What is Docker (Basics)**

Docker is a platform that lets you package applications into **containers**.

A **container** is:

* Lightweight
* Has everything your app needs: OS libraries, dependencies, runtime
* Runs the same everywhere (your PC, server, cloud)

A **Docker image** = blueprint
A **Docker container** = a running instance of an image

---

# 📌 **2. Key Docker Concepts**

| Concept            | Meaning                                               |
| ------------------ | ----------------------------------------------------- |
| **Image**          | A template/blueprint (e.g., Ubuntu, MySQL, your app). |
| **Container**      | A running instance of an image.                       |
| **Dockerfile**     | A file describing how to build an image.              |
| **Docker Hub**     | Public store of images.                               |
| **Volume**         | Persistent file storage for containers.               |
| **Network**        | Allow containers to communicate.                      |
| **Docker Compose** | Runs multiple containers together.                    |

---

# 📌 **3. Install & Setup Docker**

### 👉 **Windows / Mac**

Install **Docker Desktop**:
[https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

### 👉 **Linux**

```bash
sudo apt update
sudo apt install docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

Verify installation:

```bash
docker --version
docker run hello-world
```

---

# 📌 **4. Dockerfile Basics (Very Important)**

Example Dockerfile for a .NET app:

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /app

COPY . .
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/runtime:8.0
WORKDIR /app
COPY --from=build /app/out .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

Build the image:

```bash
docker build -t myapp .
```

Run the container:

```bash
docker run -p 8080:8080 myapp
```

---

# 📌 **5. Common Docker Commands (Cheat Sheet)**

### 🔹 **Images**

```bash
docker pull imageName           # pull from Docker Hub
docker images                   # list images
docker rmi imageName            # remove image
docker build -t name .          # build image from Dockerfile
docker tag old new              # rename/tag image
```

---

### 🔹 **Containers**

```bash
docker run image                # run container
docker run -d image             # run in background
docker run -it image bash       # open terminal inside container
docker ps                       # list running containers
docker ps -a                    # list all containers
docker stop containerID         # stop a container
docker start containerID        # start container
docker rm containerID           # delete container
docker logs containerID         # view logs
```

---

### 🔹 **Ports**

```bash
docker run -p HOST:CONTAINER image
# Example: docker run -p 8080:80 nginx
```

---

### 🔹 **Volumes (Persistent Storage)**

```bash
docker volume create myvol
docker run -v myvol:/data image
docker volume ls
docker volume rm myvol
```

---

### 🔹 **Copy Files**

```bash
docker cp file.txt container:/path
docker cp container:/path/file.txt .
```

---

### 🔹 **Exec (Run inside a running container)**

```bash
docker exec -it containerID bash
```

---

# 📌 **6. Docker Compose Basics**

Create **docker-compose.yml**:

```yaml
version: '3.9'

services:
  app:
    image: myapp
    ports:
      - "8080:8080"
  redis:
    image: redis
    ports:
      - "6379:6379"
```

Start all:

```bash
docker compose up
```

Stop all:

```bash
docker compose down
```

Rebuild + start:

```bash
docker compose up --build
```

---

# 📌 **7. Docker Common Real-World Use Cases**

| Use Case           | Docker Benefit                             |
| ------------------ | ------------------------------------------ |
| Running a local DB | No installations needed                    |
| Deploying apps     | Same environment everywhere                |
| Microservices      | Each service isolated in its own container |
| CI/CD pipelines    | Stable build environments                  |
| Testing            | Quickly reset environments                 |

---

# 📌 **8. Simple Example: Run a MySQL Database**

```bash
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=1234 \
  -p 3306:3306 \
  mysql:8
```

Open MySQL shell:

```bash
docker exec -it mysql bash
mysql -u root -p
```

---

# 📌 **9. Simple Example: Run Nginx Web Server**

```bash
docker run -d -p 8080:80 nginx
```

Visit:
[http://localhost:8080](http://localhost:8080)

---

# 📌 **10. Want a Printable Cheat Sheet PDF?**

I can generate a clean PDF version of this cheat sheet if you'd like.

---

If you want, I can also give you:

* Docker networking basics
* Multi-stage builds
* Best practices for .NET in Docker
* Kubernetes basics (since you're prepping KCNA)