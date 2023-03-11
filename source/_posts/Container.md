---
title: Containers
date: 2023-03-08 16:29:12
categories:
  - [计算机]
---

Docker is a platform for developers and sysadmins to develop, deploy, and run applications with containers.

<!-- more -->

# Overview

## Terminology

- **Images**: The **file system** and **configuration** of our application which are used to create containers.
- **Containers**
  - Running instances of Docker images (containers run the actual applications).
  - A container includes an application and all of its dependencies.
  - It shares the kernel with other containers, and runs as an isolated process in user space on the host OS.
- **Docker daemon**: The background service running on the host that manages building, running and distributing Docker containers.
- **Docker client**: The command line tool that allows the user to interact with the Docker daemon.
- **Docker Hub**: A directory of available Docker images.

## Basic Flow of Containerizing an App

1. **Build**: App + Dockerfile --- `docker image build` ---> Image.
2. **Ship**: Push image.
3. **Run**: Pull image and run.

# Example: Run a Python Flask App

## A Python Flask App

`app.py`

```python
import os
import socket
from flask import Flask,request,jsonify

app = Flask(__name__)

@app.route("/")
def main():
    return "Welcome Python flask!"

@app.route('/about')
def about():
    return 'I am '+socket.gethostname()

@app.route('/users')
def get_users():
    json_data = [{"name":"alice","age":18},{"name":"bob", "age": 22}]
    return jsonify(json_data),200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True, use_reloader=True)   
```

`requirements.txt`

```text
Flask
```

We can create a new virtual environment for running the app.

1. install `virtualenv`
   ```bash
   pip3 install virtualenv
   ```

2. Init virtual environment called `env`
   ```bash
   virtualenv venv
   ```

3. Activate `venv`
   ```bash
   source venv/bin/activate
   # The shortcut for the above command is 
   # . venv/bin/activate
   ```

4. If you wan to exit `venv`:

   ```bash
   deactivate
   ```

And then install flask:

```bash
pip install flask
```

Run the flask app:

```bash
python3 app.py
```

## Pulling a Python Docker Image

Pull a python docker image with `slim` tag from Dockerhub:

```bash
docker pull python:slim
```

Check the image just pulled:

```bash
docker image ls
```

Run the image with python interpreter:

```bash
docker run -it python:slim
```

Run the image with bash:

```bash
docker run -it python:slim bash
```

## Containerizing the Python App

Create a `Dockerfile`:

```dockerfile
# Use an official Python runtime as a parent image
FROM python:slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Run app.py when the container launches
CMD ["python", "app.py"]
```

Build a docker image:

```bash
# -t specify the docker image's name
docker build . -t myapp
```

Check that the `myapp` image is created:

```bash
docker image ls
```

Run the image:

```bash
# -p is used to map the port 5000 inside to the container host's port 15000
docker run -p 15000:5000 myapp
```

## Push to Dockerhub

```bash
docker login
```

Tag your image with your Dockerhub ID:

```bash
docker tag myapp:latest [your Dockerhub ID]/myapp:latest
```

Push image to Dockerhub:

```bash
docker push [your Dockerhub ID]/myapp
```

And now you can run your app at any computer:

```bash
docker run -p 15000:500 [your Dockerhub ID]/myapp
```

## Explanations

### RUN vs CMD in Dockerfile

`RUN`  execute **once at build time** and get written into your Docker image as a new layer.

`CMD` lets you define a default command to run **when your container starts**.

-  Defines a default command to run when none is given.
- Each `CMD` will replace and override the previous one. (Defining multiple `CMD` lines is useless).

- May provided append commands to `docker run` to override the default `CMD`.

### Running Multiple Containers

Use `-d` to specify the docker image in daemon mode (background).

```bash
docker run --name app1 -d -p 1082:8080 myapp
docker run --name app2 -d -p 1083:8080 myapp
docker run --name app3 -d -p 1084:8080 myapp
```

## Best Practices

[Best Practices in Docker Docs.](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

[Docker with CI.](https://docs.docker.com/build/ci/)

- Create ephemeral containers (don't try to keep state inside the container, since it will be reset when the container restart).
- Exclude with `.dockerignore`.
- Don't install unnecessary packages.

- Decouple applications.
- Minimize the number of layers.
- Sort multi-line arguments.
- Leverage build cache.
- If you rely on some other libs, specify the tag (don't use latest).

# Docker Layers

Each container is **an image** with **a readable / writable layer** on the top of a bunch of **read-only layers**.

These layers (also called intermediate images) are generated when the commands in the Dockerfile are executed during the Docker image build.

Multiple containers can **share** some or all file system layers from one or many images.

# Example: Run a Redis Database

Start the Redis server:

```bash
docker run -p 6379:6379 redis
```

Check the container logs:

```bash
docker logs [container_id]
```

Launch a shell into the container:

```bash
docker exec -it [container_id] bash
```

## Persistence in Docker

If the running container modifies an existing file, the file is copied out of the underlying read-only layer and into the top-most read-write layer where changes are applied.

When a Docker container is deleted, relaunching the image will start a fresh container, without any of the changes made in the previously running container.

### Bind Mount

A file or directory on the host machine is mounted into a container.

```bash
# Windows Only
docker run -d -p 6379:6379 -v d:\redis_data:/data redis
# Linux/MacOS
docker run -d -p 6379:6379 -v /redis_data:/data redis
```

`d:\redis_data` is host directory, and `/redis_data` is container path.

### Docker Volume

Volumes may be created when you run containers, and may remain after your containers are stopped and removed.

```
docker run -d -p 6379:6379 -v redis_data:/data redis
```

`redis_data` is a volume, you can check all volumes like:

```bash
docker volume ls
```

# Docker Networking

- Each docker container will be assigned an IP address

- Use `docker inspect` to find the IP address of a container:
  ```bash
  docker inspect --format '{{.NetworkSettings.IPAddress}}' <ContainerId>
  ```

- When running on Linux, we can even ping the IP address directly
  ```bash
  docker run busybox ping 172.17.0.3
  ```

## Create a Docker Network

To allow containers to communicate using "domain names", you have to create a Docker network:

```bash
docker network create my-network
```

To show the various networks in docker:

```bash
docker network ls
```

Start a nginx container and use the `--net` option to connect the container to the created netwrok:

```bash'
docker run --name myws --net my-network nginx
```

Start another container with python image and connect to the created network:

```bash
docker run --net my-network -it python
```

In the python container's shell, access the nginx's container website using its **container name** as domain name:

```bash
curl myws
```

# Example: Python Flask with Redis

In `app.py`:

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

In `requirements.txt`:

```text
flask
redis
```

In `Dockerfile`:

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

Start:

```bash
# Create docker netwrok
docker network create my-network
# Build image
docker build . -t myapp
# Run redis container
docker run -d --name redis --net my-network redis
# Run python container
docker run -d --name web -p 5000:5000 --net my-network myapp
```

Access python flask app: `http://localhost:5000`

Check container logs `docker logs redis`

## Docker-compose

A tool for defining and running multi-containers docker application (application stack).

Create a `docker-compose.yml` file:

```yaml
version: "3.9"
services:
  web:
    image: myapp
    ports:
      - "8000:5000"
  redis:
    image: "redis:alpine"
```

Run the containers:

```bash
docker compose up
```

## Pruning

```bash
# Stop and remove containers
docker container rm -f [container_id]
# Use prune command to cleanup all the stopped containers
docker container prune

# Remove docker image
docker image rm [image_id]
# Force removal
docker image rm -f [image_id]
# Remove dangling images (layers that have no relationship to any tagged image)
docker image prune

# Clean up the unused volumes
docker volume prune
docker system df
docker system df -a
```

