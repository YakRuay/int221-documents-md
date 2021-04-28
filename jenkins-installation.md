# Install Jenkins using Docker

## Download Jenkins Docker Image

```
docker pull jenkins/jenkins:lts
```

## Create docker-compose configuration

- Create `~/jenkins` directory to persist all the data, configuration, pligins, pipelines and passwords etc
- Create an empty directory to store `docker-compose.yml`
- `docker-compose.yml` configuration:
```
services:
  jenkins:
    image: jenkins/jenkins:lts.
    privileged: true
    user: root
    ports:
      - 8081:8080
      - 50000:50000
    container_name: jenkins
    volumes:
      - ~/jenkins:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/local/bin/docker:/usr/local/bin/docker
```

## Run Jenkins Container

```
docker-compose up
```

## View the initial password

```
docker exec -it [container_name] bash
cat /var/jenkins_home/secrets/initialAdminPassword
```