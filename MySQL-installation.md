# Install MySQL using Docker

## Create docker-compose configuration

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

### Prerequisite

- Create `~/mysql` directory to persist all the data, configuration and passwords etc from database container
- Create `~/int221-database` directory to store this `docker-compose.yml`
- Download `initdb.sql` to `~/int221-database` directory
- Create docker network named `int221-server` (if not exist)
  
```
docker network create -d bridge int221-network
```

## Run MySQL Container

```
docker-compose up -d
```

- `-d` for run in background

## Create database from mounted initdb.sql

```
# Connect to database docker container
docker exec -it [container_name] bash

# Connect to mysql server with initdb.sql to create table and insert data
mysql -u root -p < initdb.sql
```

## Grant permission on database to backend user

```
GRANT SELECT, INSERT, UPDATE, DELETE ON sandalsshop.Products TO backend@'%';
GRANT SELECT, INSERT, UPDATE, DELETE ON sandalsshop.ProductColors TO backend@'%';
GRANT SELECT ON sandalsshop.Colors TO backend@'%';
GRANT SELECT ON sandalsshop.Brands TO backend@'%';
```