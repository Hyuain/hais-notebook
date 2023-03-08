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

# Example

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

