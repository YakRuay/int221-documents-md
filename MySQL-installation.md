# Install MySQL using Docker

## Download MySQL Server Docker Image

```
docker pull mysql/mysql-server
```

## Create docker-compose configuration

- Create `~/mysql` directory to persist all the data, configuration and passwords etc
- Create `~/int221-database` directory to store this `docker-compose.yml`
- Store `initdb.sql` in `~/int221-database` directory
- `docker-compose.yml` configuration:

```
services:
    mysqldb:
        image: mysql/mysql-server
        restart: always
        ports:
            - 3306:3306
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

## Run MySQL Container

```
docker-compose up -d
```

## Create database from mounted initdb.sql

```
docker exec -it [container_name] bash
mysql -u root -p < initdb.sql
```

## Grant permission on database to backend user

```
GRANT SELECT, INSERT, UPDATE, DELETE ON sandalsshop.Products TO backend@'%';
GRANT SELECT, INSERT, UPDATE, DELETE ON sandalsshop.ProductColors TO backend@'%';
GRANT SELECT ON sandalsshop.Colors TO backend@'%';
GRANT SELECT ON sandalsshop.Brands TO backend@'%';
```