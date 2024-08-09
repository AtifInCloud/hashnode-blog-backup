---
title: "Dockerizing Wanderlust: A Simple Step-by-Step Tutorial"
seoTitle: "Dockerizing Wanderlust: Easy Step-by-Step Guide"
seoDescription: "Step-by-step guide to Dockerizing Wanderlust, a MERN travel blog app, resolving disk space issues, and setting up Docker Compose effectively"
datePublished: Fri Aug 09 2024 06:50:17 GMT+0000 (Coordinated Universal Time)
cuid: clzmckyos000y0al66za9cxbv
slug: dockerizing-wanderlust-a-simple-step-by-step-tutorial
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1723186036689/e608ec73-4a7e-4034-bf77-c5a197751a27.jpeg
tags: docker, devops, mern, containerization, tech-community, tech-tutorial

---

### Introduction

In this post, I'll walk you through the process of Dockerizing a full-stack application called Wanderlust.

**Wanderlust** is a simple yet powerful MERN (MongoDB, Express, React, Node.js) travel blog website designed for those passionate about travel and tech

This project involved setting up Docker for both the backend and frontend services, along with MongoDB for data persistence. During the process, I encountered a few challenges, particularly related to disk space, which I managed to resolve. Let's dive into the details.

### Step 1: Setting Up Docker

First, I needed to install Docker on my Ubuntu instance.

```bash
sudo apt-get install docker.io
```

To avoid using `sudo` for Docker commands, I added my user to the Docker group:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723182987279/3188a05d-2703-4e46-8aca-5281593e6b65.png align="center")

### Step 2: Building the Backend Docker Image

I navigated to the `backend` directory of the Wanderlust project and created a Dockerfile with the following contents:

```dockerfile
FROM node:latest

WORKDIR /app

COPY . .

RUN npm i

COPY .env.sample .env

EXPOSE 5000

CMD ["npm", "start"]
```

To build the Docker image, I ran:

```bash
docker build -t backend .
```

After building the image, I verified it with:

```bash
docker images
```

### Step 3: Setting Up the MongoDB Container

For the MongoDB service, I pulled and ran the official MongoDB Docker image:

```bash
docker run -d -p 27017:27017 --name mongo mongo:latest
```

To ensure MongoDB was running correctly, I accessed the Mongo shell inside the container:

```bash
docker exec -it mongo bash
mongosh
```

### Step 4: Running the Backend Container

Next, I started the backend container:

```bash
docker run -d -p 5000:5000 --name backend backend:latest
```

### Step 5: Building the Frontend Docker Image

In the `frontend` directory, I created another Dockerfile:

```dockerfile
DockerfileCopy codeFROM node:latest

WORKDIR /app

COPY . .

RUN npm i

COPY .env.sample .env.local

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host"]
```

I attempted to build the frontend Docker image:

```bash
docker build -t frontend .
```

### Key Issue: Disk Space Limitation

During the frontend image build, I encountered an error due to insufficient disk space. The root partition was 93% full, with only 495 MB available.

### Step 6: Managing Disk Space

To resolve this, I increased the disk size of my EC2 instance from 8GB to 16GB using the AWS Management Console. I then expanded the partition on Ubuntu to use the additional space.

### Step 7: Installing Docker Compose

I needed Docker Compose for orchestrating the services. I installed Docker Compose V2 with the following commands:

```bash
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
```

To verify the installation, I ran:

```bash
docker compose version
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723183383159/6fd9725c-f56d-4a96-b91e-3d7f7d7e69ab.png align="center")

### Step 8: Using Docker Compose

With Docker Compose installed, I created a `docker-compose.yml` file to define the services:

```yaml
version: "3.8"
services:
  mongodb:
    container_name: mongo
    image: mongo:latest
    volumes:
      - ./backend/data:/data
    ports:
      - "27017:27017"
    
  backend:
    container_name: backend
    build: ./backend
    env_file:
      - ./backend/.env.sample
    ports:
      - "5000:5000"
    volumes:
      - logs:/app/logs
      - ./backend:/app
      - /app/node_modules
    depends_on:
      - mongodb

  frontend:
    container_name: frontend
    build: ./frontend
    ports:
      - "5173:5173"
    volumes:
      - ./frontend/src:/app/src
    env_file:
      - ./frontend/.env.sample
    stdin_open: true
    tty: true
    depends_on:
      - backend

volumes:
  logs:
```

To start the services, I used:

```bash
docker compose up -d --build
```

To ensure everything was running smoothly, I checked the running containers:

```bash
docker ps
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723183618686/06f0b472-ff38-4eaf-8021-53c0761be37d.png align="center")

### Importing Sample Data: Bringing the Website to Life

After getting everything set up, I eagerly opened the website, only to find… nothing. The site was up and running, but there was no content. It was a strange mix of triumph and disappointment. But I quickly realized the issue—I hadn’t imported any sample data into MongoDB.

To fix this, I imported the sample data into the MongoDB container:

```bash
docker exec -it mongo mongoimport --db wanderlust --collection posts --file ./data/sample_posts.json --jsonArray
```

Once the import was complete, I refreshed the website, and there it was—content! Seeing the Wanderlust site come to life with real data was the icing on the cake. It was the moment I’d been working towards, and it felt amazing to see everything come together.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1723183826924/b4f3373d-dd9f-4354-8db0-122dd4ff906f.png align="center")

### Conclusion

Dockerizing the Wanderlust project was a great learning experience. I encountered and resolved various issues, particularly around disk space and container conflicts. Docker Compose made it easier to manage the different services, and expanding the disk space ensured the project could build and run smoothly.

If you're interested in exploring the project further or want to see more of my work, feel free to check out my GitHub account [AtifInCloud](https://github.com/AtifInCloud). You’ll find the Wanderlust project and many other interesting repositories there. I’d love to connect with fellow developers and tech enthusiasts, so don’t hesitate to follow and collaborate!