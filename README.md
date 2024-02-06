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

### CI Pipelines

#### 2-1 What are testcontainers?

Test containers are docker container dedicated for testing. They build parts or the whole app to execute test and give a report.

#### 2-2 Document your Github Actions configurations

aThe first step will create a VM ready to accept a java project using JDK 17. Then it will get the project in the right folder and run the tests on it.

```YAML
name: CI devops 2023

on:
  push:
    branches: 
      - main

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: 'adopt'

      - name: Change directory to TP01/API
        run: cd TP01/API

      - name: Build and test with Maven
        run: mvn -f TP01/API/pom.xml test

```

### CD Pipelines

```YAML
name: CI devops 2023

on:
  push:
    branches: 
      - main

jobs:
  test-backend: 
    runs-on: ubuntu-22.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: 17
          distribution: 'adopt'

      - name: Change directory to TP01/API
        run: cd TP01/API

      - name: Build and test with Maven
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=Skyward67_DevOpsCICDDorianGorse -Dsonar.organization=skyward67 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./TP01/API/pom.xml

  # define job to build and publish docker image
  build-and-push-docker-image:
    needs: test-backend
    # run only when code is compiling and tests are passing
    runs-on: ubuntu-22.04

    # steps to perform in job
    steps:
      - name: Checkout code
        uses: actions/checkout@v2.5.0

      - name: Login to DockerHub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build image and push backend
        uses: docker/build-push-action@v3
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./TP01/API
          # Note: tags has to be all lower-case
          tags:  ${{secrets.DOCKERHUB_USERNAME}}/devopsbackend:latest
          # build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push database
        uses: docker/build-push-action@v3
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./TP01/database
          # Note: tags has to be all lower-case
          tags:  ${{secrets.DOCKERHUB_USERNAME}}/devopsdatabase:latest
          # build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/main' }}

      - name: Build image and push httpd
        uses: docker/build-push-action@v3
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./TP01/Web
          # Note: tags has to be all lower-case
          tags:  ${{secrets.DOCKERHUB_USERNAME}}/devopsweb:latest
          # build on feature branches, push only on main branch
          push: ${{ github.ref == 'refs/heads/main' }}
```
