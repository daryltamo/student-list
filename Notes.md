# Pozos API - Build and Deployment Guide

## Environment Setup

### 0- Spinning up a centos build server

```bash
cd eazytraining/cursus-devops/vagrant/docker/docker_ubuntu22 && vagrant up --provision

vagrant ssh

```



![1769601847062](image/Notes/1769601847062.png)





## Building and Testing

### 1- Building the API Image

We use a multi-stage `Dockerfile` to keep the runtime image lean

```bash
git clone https://github.com/daryltamo/student-list.git

cd student-list/simple_api/

docker build -t pozos_api:1.0 .

docker images

```



![1769602251253](image/Notes/1769602251253.png)





### 2- Network Configuration

```bash
docker network create pozos_network --driver=bridge

docker network ls

```



![1769602358243](image/Notes/1769602358243.png)





### 3- Starting the Backend (API)

when starting the backend API we need to mount the student data and attach the container to our network as follow

```bash
 cd student-list
 
docker run --rm -d \
  --name pozos_api \
  --network pozos_network \
  -v ./simple_api/student_age.json:/data/student_age.json \
  pozos_api:1.0

docker ps -a

```



![1769602500851](image/Notes/1769602500851.png)





### 4- Starting the Frontend (webapp)

We first update the PHP configuration to point to the API container, then launch the Apache-based frontend:

```bash
# Update the API  endpoint in index.php
sed -i 's\<api_ip_or_name:port>\pozos_api:5000\g' ./website/index.php

# Then launch the frontend
docker run --rm -d \
  --name pozos_frontend \
  -p 80:80 \
  --network pozos_network \
  -v ./website/:/var/www/html \
  -e USERNAME=toto -e PASSWORD=python \
  php:apache

docker ps -a

```



![1769602787919](image/Notes/1769602787919.png)



### 5- Testing and Verification

From Command Line

```
docker exec pozos_frontend curl -u toto:python -X GET http://pozos_api:5000/pozos/api/v1.0/get_student_ages

```



![1769602835485](image/Notes/1769602835485.png)


From Browser

![1769602919320](image/Notes/1769602919320.png)



### 6- Cleaning the workspace

```
docker stop pozos_api pozos_frontend

docker rm pozos_api pozos_frontend

docker network rm pozos_network

docker network ls
  
docker ps

```



![1769602984337](image/Notes/1769602984337.png)



The API's Dockerfile makes use of multi-stage build which optimizes the final image by:

- separating build dependencies (stage 1) from runtime dependencies (stage 2)
- Discarding large build tools after compilation

## Production Deployment (Docker Compose)

### 1- Full Stack Orchestration

Our docker compose file's name is ***pozos-app.docker-compose.yml***
Use the pre-configured Compose file for a one-command deployment.

```bash
docker compose -f pozos-app.docker-compose.yml up -d
```



![1769603113053](image/Notes/1769603113053.png)



Accessing the app from a browser




### 2- Private Registry Setup

Creating pozos' container registry from docker-compose file ***pozos-registry.docker-stack.yml***

```
# Start Registry
docker compose -f pozos-registry.docker-stack.yml up -d

```







Accessing the registry ui through our browser




4- Tagging and pushing the image

```
docker login localhost:5000

docker image tag pozos-api:1.0 localhost:5000/pozos/pozos-api:1.0

docker images

docker image push localhost:5000/pozos/pozos-api:1.0

```







### 3- CleanUp

```
docker compose down

```







## Security Notes

- Non-root user not enforced (Flask requires setup)
- Build tools removed from runtime image
- Consider adding health checks in production
- Pin Python version (3.13) for reproducibility
