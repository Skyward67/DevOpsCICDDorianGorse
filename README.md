# DevOps CI & CD Dorian Gorse

## TP 01

### Database

```Bash
docker build -t alpine-cpe .
docker run --name alpine -v /data:/var/lib/postgresql/data -e POSTGRES_DB="db" -e POSTGRES_USER="usr" -e POSTGRES_PASSWORD="pwd" -p 5432:5432 --network app-network alpine-cpe
```

```bash
docker run --name adminer --network app-network -p 8080:8080 adminer
```

### API

First Dockerfile:

```Dockerfile
FROM openjdk:11
WORKDIR /app
COPY Main.class /app
CMD ["java", "Main"]
```

Commands:

```bash
docker build -t my-java-app .
docker run my-java-app
```

#### 1-2 Why do we need a multistage build? And explain each step of this dockerfile

Because we need to get all the necessary packages before building, so the first part is getting all the packages referenced in the project and the second part is building the project.

### Web

#### Get the conf for modification

```Bash
docker exec web cat /usr/local/apache2/conf/httpd.conf 
```

#### Build

```Bash
docker docker build -t my-apache2 . 
docker run -dit --name web -p 80:80 my-apache2   
```

#### Why do we need a reverse proxy?

The reverse proxy is here to make us able to reach the API through the web app without exposing our api port and make us able to link to others api/services in the future if needed.

### Docker compose

#### Why is docker-compose so important?

It is important to simplify deployement, make different container work together and make specific configuration for testing.

#### Code

Build :

```YAML
version: '3.7'

services:
    database:
        build: ./database
        ports:
          - "5432:5432"
        environment:
            POSTGRES_USER: usr
            POSTGRES_PASSWORD: pwd
            POSTGRES_DB: db
        networks:
          - app-network

    backend:
        build: ./API
        ports:
          - "8080:8080"
        networks:
          - app-network
        depends_on:
          - database

    web:
        build: ./Web
        ports:
          - "80:80"
        networks:
          - app-network
        depends_on:
          - backend

networks:
    app-network:
        driver: bridge
```

Images :

```YAML
version: '3.7'

services:
    database:
        image: "skyward67/devopsdatabase:1.0"
        ports:
          - "5432:5432"
        environment:
            POSTGRES_USER: usr
            POSTGRES_PASSWORD: pwd
            POSTGRES_DB: db
        networks:
          - app-network

    backend:
        image: "skyward67/devopsbackend:1.0"
        ports:
          - "8080:8080"
        networks:
          - app-network
        depends_on:
          - database

    web:
        image: "skyward67/devopsweb:1.0"
        ports:
          - "80:80"
        networks:
          - app-network
        depends_on:
          - backend

networks:
    app-network:
        driver: bridge

```

## TP 02

### 

#### 2-1 What are testcontainers?

Test containers are docker container dedicated for testing. They build parts or the whole app to execute test and give a report.