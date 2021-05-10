# DevOps Documentation

## Infrastructure Architecture Diagram

![image](https://cdn.discordapp.com/attachments/695225022226104391/841232350897635338/Screen_Shot_2564-05-10_at_15.36.46.png)

## Prerequisites

- Before running these project's container, you need to create docker-network named `int221-network` for this project

```
docker network create -d bridge int221-network
```

## Frontend Configuration

- For this Frontend container configuration, we used both `Dockerfile` and `docker-compose.yml` to configure the container
- We used `Dockerfile` to configure how to build the image for this container
- We used `docker-compose.yml` to configure the container's properties such as container's network, image's name etc.

### Dockerfile

```
FROM node:14.16.1-alpine3.10 as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY ./ .
RUN npm run build

FROM nginx:1.19.10-alpine as production-stage
RUN mkdir /app
COPY --from=build-stage /app/dist /usr/share/nginx/html/frontend/
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

- First, we build the project with `node:14.16.1-alpine3.10`
- After finished building the project, we deploy it with `nginx` as a `web-server` by using `nginx.conf` that we prepared for this container

### nginx.conf for frontend

```
server{
	listen 80;
	server_name localhost;

	location / {
        root /usr/share/nginx/html/frontend/;
        index index.html;
		try_files $uri $uri/ /index.html;
    }
}
```

- Configure an nginx to root of `index.html`

### docker-compose.yml

```
services:
    frontend:
        build: .
        image: ghcr.io/yakruay/int221-frontend:dev
        environment:
            SERVICE_VERSION: v1
networks:
    default:
        external:
            name: int221-network
```

- Define an image's name as `ghcr.io/yakruay/int221-frontend:dev`
- Define docker network as `int221-network`

## Backend Configuration

- For this Backend container configuration, we used both `Dockerfile` and `docker-compose.yml` to configure the container
- We used `Dockerfile` to configure how to build the image for this container
- We used `docker-compose.yml` to configure the container's properties such as container's network, image's name etc.

### Dockerfile

```
FROM maven:3.6.0-jdk-11-slim AS build
WORKDIR /backend
COPY src ./src
COPY pom.xml ./
RUN mvn clean install

FROM adoptopenjdk/openjdk11:jdk-11.0.10_9-alpine
ARG JAR_FILE=/backend/target/int221-backend-0.0.1-SNAPSHOT.jar
COPY --from=build ${JAR_FILE} int221-backend-0.0.1-SNAPSHOT.jar
ENTRYPOINT ["java","-jar","int221-backend-0.0.1-SNAPSHOT.jar"]
```
- First, we build the project with `maven:3.6.0-jdk-11-slim`
- After finished building the project, we deploy it by copying `.jar file` to docker container

### docker-compose.yml

```
services:
    backend:
        build: .
        image: ghcr.io/yakruay/int221-backend:dev
        environment:
            SERVICE_VERSION: v1
        volumes:
            - ~/backendImages:/src/images
networks:
    default:
        external:
            name: int221-network
```

- Define an image's name as `ghcr.io/yakruay/int221-backend:dev`
- Define docker network as `int221-network`
- Define volume for storing image from backend as `~/backendImages:/src/images`

## Database Configuration

- For this Database container configuration, we used only `docker-compose.yml` to configure the container
- We don't need `Dockerfile` because we don't have anything to build

### docker-compose.yml

```
services:
    mysqldb:
        image: mysql/mysql-server
        restart: always
        environment:
            MYSQL_DATABASE: sandalsshop
            MYSQL_USER: backend
            MYSQL_PASSWORD: YakRuay@2X64
            MYSQL_ROOT_PASSWORD: YakRuay@2X64
        volumes:
            - ~/mysql:/var/lib/mysql
            - ./initdb.sql:/initdb.sql
networks:
    default:
        external:
            name: int221-network
```

- These environments that we defined in `docker-compose.yml` is being used to create database, user, password and root password
- Define volume to persist all the data and configuration from docker container to `~/mysql`
- Define volume to store `initdb.sql` script to docker container
- Create database and insert data with `initdb.sql` 

```
# Connect to database docker container
docker exec -it [container_name] bash

# Connect to mysql server with initdb.sql to create table and insert data
mysql -u root -p < initdb.sql
```

- Grant permission to `backend` user on `sandalsshop` database

```
GRANT SELECT, INSERT, UPDATE, DELETE ON sandalsshop.Products TO backend@'%';
GRANT SELECT, INSERT, UPDATE, DELETE ON sandalsshop.ProductColors TO backend@'%';
GRANT SELECT ON sandalsshop.Colors TO backend@'%';
GRANT SELECT ON sandalsshop.Brands TO backend@'%';
```

## Reverse proxy Configuration

 - For this Reverse Proxy container configuration, we used only `docker-compose.yml` to configure the container
- We don't need `Dockerfile` because we don't have anything to build

### docker-compose.yml

```
services:
    nginx:
        image: nginx:1.19.10-alpine
        container_name: production_nginx
        volumes:
            - ./nginx.conf:/etc/nginx/conf.d/default.conf
        ports:
            - 80:80
networks:
    default:
        external:
            name: int221-network
```

- For reverse proxy container, we set it to expose on `port 80`
- Replace nginx `default.conf` with `nginx.conf` that we prepared

### nginx.conf

```
server {
    listen      80;
    server_name localhost;

    location / {
    	proxy_pass http://frontend;
    }

	location /backend {
		proxy_pass http://backend:5000/;
		proxy_set_header Host $host;
	}
}
```

- For `nginx.conf`, we set it to listen on `port 80`
- For location `/`, we set it proxy_pass to `frontend container` on `port 80` which has an internal hostname as `http://frontend`
- For location `/backend`, we set it proxy_pass to `backend container` on `port 5000` which has an internal hostname as `http://backend` (we set backend `server.port=5000`)
- These hostname come from `service_name` that we defined under `services` in `docker-compose.yml` for each service

## GitHub Action CI/CD

### Prerequisites

- All the docker image must have naming pattern like this

```
ghcr.io/$GITHUB_USER/[repository_name]:[tag]
```
- Enable container support in GitHub account
- Generate Personal Access Token (PAT) for GitHub account
- Generate `ssh-key` for GitHub
- Register `ssh-key` to repository's secrets as `PRIVATE_KEY`
- `mkdir -p .github/workflows` in git repositories
- Store `workflow` file in `.github/workflows`

### int221-frontend-dev.yml workflow

```
name: int221-frontend-dev
on:
  push:
    branches:
      - dev

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    
    - name: Login to Github Docker Registry
      uses: docker/login-action@v1
      with:
        registry: ghcr.io
        username: ${{ github.repository_owner }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push
      uses: docker/build-push-action@v2
      with:
        push: true
        tags: ghcr.io/yakruay/int221-frontend:dev

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Execute deploy SSH commmands on remote server
      uses: JimCronqvist/action-ssh@master
      with:
        hosts: 'rew@52.187.108.86'
        privateKey: ${{ secrets.PRIVATE_KEY }}
        command: |
          cd int221-frontend
          git checkout dev
          git pull
          docker pull ghcr.io/yakruay/int221-frontend:dev
          docker-compose down
          docker-compose up -d
```

### int221-backend-dev.yml workflow

```
name: int221-backend-dev
on:
  push:
    branches:
      - dev

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Login to Github Docker Registry
      uses: docker/login-action@v1
      with:
        registry: ghcr.io
        username: ${{ github.repository_owner }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push
      uses: docker/build-push-action@v2
      with:
        push: true
        tags: ghcr.io/yakruay/int221-backend:dev

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Execute deploy SSH commmands on remote server
      uses: JimCronqvist/action-ssh@master
      with:
        hosts: 'rew@52.187.108.86'
        privateKey: ${{ secrets.PRIVATE_KEY }}
        command: |
          cd int221-backend
          git checkout dev
          git pull
          docker pull ghcr.io/yakruay/int221-backend:dev
          docker-compose down
          docker-compose up -d
```

### Workflow process

- These workflow will work everytime we push the commit to GitHub repository on `dev` branch
- In the build stage, we build docker-image on GitHub Action machine and push it to GitHub Container Reistry (GHCR)
- In the deploy stage, we ssh to remote server and execute listed command in that step