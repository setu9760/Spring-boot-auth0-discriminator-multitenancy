# Inventory

Inventory is an example Spring Boot application on how to properly set up a SaaS implementation by following the best practices as well having support for Multi Tenancy. 

## Highlights

- Discriminator based Multi Tenancy based on Hibernate Filter & Hibernate Interceptors with Auth0 integration  
 -- Closely working together with Spring JPA thanks to Spring AOP  
 -- Default CLOSED! No annotations, no "reminders to enable multi tenancy" -> You don't need to worry about multi tenancy anymore!! (except for one JPA method - FindById -> See more why in section: [Spring JPA](#spring-jpa)  
 -- Integrated with Auth0 identity provider with dynamically tenant resolving thanks to the JWT token  
 -- Easily applied to an existing project! See section [Upgrade my Spring boot app](#upgrade-my-spring-boot-application) on how to!  
 -- Full test coverage to show that it actually works!  
 -- Explained what it exactly does, so you don't need to search on other resources how this code works.  

## Initial Start

- JVM version should be Java 11 (or higher)
- Docker is necessary to be able to run the tests through TestContainer + local development thanks to the prepared docker commands below.

### Initial Database setup
Postgres is chosen for demo purposes, but other databases can be configured just as well. Details can be found in [Application.yml](/src/main/resources/application.yml)

For the development environment locally, we choose the use the docker internal volume management to persist our local database. (See: [Docker hub postgres for more info](https://hub.docker.com/_/postgres/))  
* Docker own internal volume management [Volumes](https://docs.docker.com/storage/volumes/)
    * Easier to share database data between docker applications
    * OS independent

> To keep the data persistent, we are going to use a data volume container.  

```bash
docker create -v /var/lib/postgresql/data --name inventory-postgres-data-dev busybox
```
Explanation of the command:
* Create a new data container from the busybox image (Time of writing: Version 1.32) 
* The -v option specifies the location of the persisted data inside the docker container. 
* The --name option provides a friendly name to the data container.

> Now we will start the postgres image as in its own container together with the previous created data volume container.

```bash
docker run --name inventory-postgreSQL-dev -p 5432:5432 -e POSTGRES_PASSWORD=inventory-docker-test-password -e POSTGRES_INITDB_ARGS="--data-checksums" -d --volumes-from inventory-postgres-data-dev postgres:13-alpine
```

Explanation of the command:  
* Create and start a new container from the postgres:13-alpine image.   
* For this container, we are providing the name: `inventory-postgreSQL-dev`
* Start the database on port `5432` internally and expose it through the same port.
* We are providing parameters (-e) to the database like:  
    * POSTGRES_PASSWORD : Database password  
    * POSTGRES_USER : Database user, not defined for development, so (default `postgres`)  
    * POSTGRES_INITDB_ARGS : Additional arguments like --data-checksums.  
        * --data-checksums adds checksums to each page of data when it is written to the HDD. This can detect data corruption. 
* By using -d, we are running the container in detached mode
* We are mounting the busybox container as a volume storage by adding --volumes-from  

To be able to start the application, we need to add a new database called `inventory`
         
```bash
docker run -it --link inventory-postgreSQL-dev:postgres --rm postgres:13-alpine sh -c 'exec psql -h "$POSTGRES_PORT_5432_TCP_ADDR" -p "$POSTGRES_PORT_5432_TCP_PORT" -U postgres'
```

This will ask for the password which can be found in [Application.yml](/src/main/resources/application.yml) -> datasource password  

> Create the required database.

```postgresql
CREATE DATABASE inventory
```

That's it! You can now properly run the application without any issues

## Start / stop the local development environment.

Afterwards to start or stop the container, you can use

> Start the database container

```bash
docker start inventory-postgreSQL-dev
```
> Stop the database container

```bash
docker stop inventory-postgreSQL-dev
```

# Spring JPA

explain why

# Upgrade my spring boot application

todo, upgrade db, migrate config, copy fwk subfolder