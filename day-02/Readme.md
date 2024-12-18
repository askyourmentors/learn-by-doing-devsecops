# DevSecOps Bootcamp - Day 2: Introduction to Containerization with Docker

## Table of Contents
1. [Introduction to Containerization](#introduction-to-containerization)
2. [Docker Architecture and Components](#docker-architecture-and-components)
3. [Basic Docker Commands](#basic-docker-commands)
4. [Creating and Managing Docker Images](#creating-and-managing-docker-images)
5. [Docker Compose, Networking, and Volumes](#docker-compose-networking-and-volumes)
6. [Running Applications in Docker](#running-applications-in-docker)
   - [Node.js Application](#nodejs-application)
   - [Python Application](#python-application)
7. [Multi-Stage Docker Build](#multi-stage-docker-build)
8. [Securing Docker Images Against Attacks](#securing-docker-images-against-attacks)
9. [Hands-on Practice](#hands-on-practice)
10. [Conclusion](#conclusion)

---

## Introduction to Containerization
Containerization is a lightweight alternative to full machine virtualization that involves encapsulating an application and its dependencies into a container. This allows for consistent environments across development, testing, and production.

![Containerization Diagram](path/to/image.png)

---

## Docker Architecture and Components
Docker consists of several key components:
- **Docker Engine**: The core component that runs and manages containers.
- **Docker Daemon**: The background service that manages Docker containers.
- **Docker CLI**: The command-line interface for interacting with Docker.
- **Docker Hub**: A cloud-based registry for sharing Docker images.

![Docker Architecture](path/to/image.png)

---

## Basic Docker Commands
Here are some basic Docker commands to get you started:

```bash
# Check Docker version
docker --version

# List running containers
docker ps

# List all containers
docker ps -a

# Build an image from a Dockerfile
docker build -t my-image .

# Run a container
docker run -d -p 80:80 my-image
```

---

## Creating and Managing Docker Images
To create a Docker image, you need a Dockerfile. Below is an example of a simple Dockerfile for a Python application:

### Sample Python Application

**app.py**
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, World from Python!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

**requirements.txt**
```
Flask==2.0.1
```

**Dockerfile for Python**
```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Run app.py when the container launches
CMD ["python", "app.py"]
```

### Build and Run Python Application
```bash
# Build the Docker image
docker build -t my-python-app .

# Run the Docker container
docker run -p 80:80 my-python-app
```

### Outcome
After running the above commands, you can access the Python application by navigating to `http://localhost` in your web browser, and you should see "Hello, World from Python!".

---

### Sample Node.js Application

**app.js**
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.send('Hello, World from Node.js!');
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

**package.json**
```json
{
  "name": "my-node-app",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

**Dockerfile for Node.js**
```dockerfile
# Use the official Node.js image
FROM node:14

# Set the working directory
WORKDIR /usr/src/app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application code
COPY . .

# Expose the application port
EXPOSE 3000

# Command to run the application
CMD ["node", "app.js"]
```

### Build and Run Node.js Application
```bash
# Build the Docker image
docker build -t my-node-app .

# Run the Docker container
docker run -p 3000:3000 my-node-app
```

### Outcome
After running the above commands, you can access the Node.js application by navigating to `http://localhost:3000` in your web browser, and you should see "Hello, World from Node.js!".

---

## Multi-Stage Docker Build
Multi-stage builds allow you to optimize your Docker images by reducing their size. Here’s an example using the previous Node.js application:

### Multi-Stage Dockerfile for Node.js
```dockerfile
# Stage 1: Build
FROM node:14 AS build

WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .

# Stage 2: Production
FROM node:14-slim
WORKDIR /usr/src/app
COPY --from=build /usr/src/app .

EXPOSE 3000
CMD ["node", "app.js"]
```

### Build and Run Multi-Stage Node.js Application
```bash
# Build the Docker image
docker build -t my-node-app-multi-stage .

# Run the Docker container
docker run -p 3000:3000 my-node-app-multi-stage
```

### Outcome
The multi-stage build reduces the final image size by excluding unnecessary build dependencies, resulting in a cleaner and more efficient image.

---

## Securing Docker Images Against Attacks
To secure Dockerfiles without relying on image scanners, it's essential to adopt best practices that focus on minimizing vulnerabilities and enhancing the security posture of your containers. Here’s a summary of effective strategies and techniques for writing secure Dockerfiles:

### Best Practices for Securing Dockerfiles

#### Use Non-Root Users
Running containers as non-root users is a fundamental security practice. By default, Docker containers run as the root user, which can lead to significant security risks if compromised. To mitigate this, specify a non-root user in your Dockerfile:
```dockerfile
FROM ubuntu

# Create a non-root user
RUN useradd -m myuser

# Switch to the non-root user
USER myuser
```

#### Minimize the Base Image
Select minimal base images to reduce the attack surface. For example, using Alpine Linux instead of a full Debian or Ubuntu image can significantly decrease the number of unnecessary packages and potential vulnerabilities:
```dockerfile
FROM alpine:latest
```

#### Limit Package Installation
Avoid installing unnecessary packages in your Dockerfile. Each additional package increases the attack surface. Use multi-stage builds to separate build dependencies from runtime dependencies:
```dockerfile
# Stage 1: Build
FROM golang:alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: Run
FROM alpine:latest
COPY --from=builder /app/myapp /usr/local/bin/
```

#### Set File Permissions Properly
Ensure that files and directories within the container have appropriate permissions set. This can be done using `RUN chown` and `chmod` commands:
```dockerfile
COPY --chown=myuser:myuser app /home/myuser/app
RUN chmod -R 755 /home/myuser/app
```

#### Use Environment Variables for Secrets
Avoid hardcoding sensitive information like API keys directly in the Dockerfile. Instead, use environment variables or external secret management tools:
```dockerfile
ENV API_KEY=${API_KEY}
```

#### Regularly Update Base Images
Keep your base images updated with the latest security patches. Regularly rebuild your images to incorporate updates from upstream sources:
```bash
docker pull ubuntu:latest
```

---

## Hands-on Practice

Explore the concepts by creating Dockerfiles, running containers, and optimizing images using multi-stage builds. Practice securing Docker images using the best practices outlined above.

---

## Conclusion
Docker is a powerful tool for containerization, enabling consistent and efficient application deployment. By understanding its architecture, mastering basic commands, and adopting secure practices, you can harness the full potential of Docker while ensuring the security of your applications.
