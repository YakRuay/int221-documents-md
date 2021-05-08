# DevOps Documentation

## Infrastructure Architecture Diagram

```

```

## Prerequisite

- Create docker-network named `int221-network` for this project

```
docker network create -d bridge int221-network
```

## Frontend Configuration

- For this frotnend container configuration, we used both `Dockerfile` and `docker-compose.yml` to configure the container
- We used `Dockerfile` to configure how to build the image for this container
- We used 

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

- In `docker-compose.yml`, we used it to define the container's properties such as container's network, container's image, build file location (Dockerfile) etc.
- For docker network, we need to create it in an instance before we deploy by using this command

## Backend Configuration

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

## Database Configuration

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

## Reverse proxy Configuration

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
            - 443:443
networks:
    default:
        external:
            name: int221-network
```

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