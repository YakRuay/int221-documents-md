# Install MySQL using Docker

## Download MySQL Server Docker Image

```
docker pull mysql/mysql-server
```

## Connect to MySQL Docker Container

```
# Install mysql-client
apt-get install mysql-client
# Start mysql-client inside container
docker exec -it [container_name] mysql -u root -p
```


## Create docker-compose configuration

- Create `~/mysql` directory to persist all the data, configuration and passwords etc
- Create `~/int221-database` directory to store this `docker-compose.yml`
- Store `data.sql` in `~/int221-database` directory
- `docker-compose.yml` configuration:

```
services:
    mysqldb:
        image: mysql/mysql-server
        restart: always
        environment:
            MYSQL_DATABASE: db
            MYSQL_USER: rew
            MYSQL_PASSWORD: YakRuay@2X64
            MYSQL_ROOT_PASSWORD: YakRuay@2X64
        ports:
            - "3306:3306"
        volumes:
            - ~/mysql:/var/lib/mysql
            - ./data.sql:/data.sql
networks:
    default:
        external:
            name: int221-network
```

## Run MySQL Container

```
docker-compose up -d
```

## Create database from mounted data.sql

```
docker exec -it [container_name] bash
mysql -u root -p < data.sql
```