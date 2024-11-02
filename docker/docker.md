# Docker

[Youtube link](https://www.youtube.com/watch?v=1eVy_iWrc20&list=PLrMP04WSdCjq9Nzq0aO760p90Z--yRPHr&ab_channel=PavanElthepu)

**Docker** is a tool designed to create, deploy, and run applications easoly by using containers.

**Image** is a lightweight, standalone, executable package of software that includes everything needed to run an application. It contains the code, runtime, system tools, system libraries and settings.
Image become **containers** when they run on Docker engine.

**Advantages:**
- Portability
- High Peformance
- Isolation
- Scalability
- Rapid Development

**VM vs Container**

**VM**
- Virtualizes Hardware
- Has Guest OS
- Huge in Size (GB)
- Takes time to create
- Takes time to bootup
- Consumes lot of resources

**Container**
- Virtualizes OS
- Shared Host OS
- Smaller in Size (MB)
- Can be created in seconds
- Can be started in seconds
- Consumes very few resources

### Basic Commands

Download nginx image
```bash
docker pull nginx
```
Run container
```bash
docker run nginx
```
List all the containers that is running
```bash
docker ps
```
List all the containers (running and stopped)
```bash
docker ps -a
```
Port mapping - map container port to host port. Both container can be mapped from one container port, but unique host port.
```bash
docker run -p 80:80 -d nginx 
```
Remove container
```bash
docker rm <container id>
```
Stop container
```bash
docker stop <container id>
```
Custom name to container
```bash
docker run --name test -p 80:80 -d nginx 
```
Rename existing container
```bash
docker rename <container name> test1
```
Restart container
```bash
docker start <container name>
```

List images
```bash
docker images
```
Remove image
```bash
docker rmi <image>
```

### Docker Networking & Deploying MongoDB

**Docker Networking**

Pull image mongodb
```bash
docker pull mongo:4.4.6
```
Run container with passing env variables (user, password)
```bash
docker run -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_PASSWORD=password -p 21017:21017 -d mongo:4.4.6
```
Run container mongo express, docker run automatically pulls image fails
```bash
docker run -e ME_CONFIG_MONGODB_ADMINUSERNAME=root  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password -e ME_CONFIG_MONGODB_SERVER=localhost -p 8081:8081 -d --name mongo-web mongo-express
```
Check the logs
```bash
docker logs <container id>
```
See ip address of container
```bash
docker inspect <container id> | grep IPAddress
```

List of networks
```bash
docker network ls
```

**Docker Network Types**
- **None network** - if you have container in noe network, you can't access any container except your container.
```bash
docker run -d --network none alpine sleep 500

docker exec -it <container id> sh
ping google.com # cannot access anything outside because none network
```
- **Bridge network** - this network can access external resources in the same network. Default network type
```bash
docker run -d --network bridge alpine sleep 500 

docker exec -it <container id> sh
ping google.com # able to receive
```
- **Custom bridge network** - access container using container name 
```bash
# Create custom bridge network
docker network create network mongo-net --driver bridge
# Delete all containers
docker rm $(docker ps -a)
# Create container mongodb with custom network mongo-net
docker run -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_PASSWORD=password -p 21017:21017 -d --net mongo-net --name mongodb mongo:4.4.6
# Create container for mongo-express to connect mongodb container
docker run -e ME_CONFIG_MONGODB_ADMINUSERNAME=root  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password -e ME_CONFIG_MONGODB_SERVER=mongodb -p 8081:8081 -d --name mongo-web--net mongo-net mongo-express
```
- **Host network** - you can access anything in this container. 


### Building custom image & Deploying spring boot

**Dockerfile** - building custom image, text document with a list of instructions.

To start application
```bash
java -jar target/todo-1.0.0.jar
```

Dockerfile
```dockerfile
# Download Java
FROM openjdk:18-jdk

# Copy the jar from local to image
COPY target/todo-1.0.0.jar todo-1.0.0.jar 

# Run application with java -jar
CMD ["java", "-jar", "target/todo-1.0.0.jar"]
```

Build image
```bash
docker build -t todo-api:1.0.0 .
```

ARG - used to create variables

Dockerfile
```dockerfile
# Download Java
# Create java version variable with default value
ARG JAVA_VERSION="18-jdk"
FROM openjdk:${JAVA_VERSION}

# Metadata
LABEL version="1.0.0"

# Create env variable
ENV PROJECT_NAME="todo-api"

ARG APP_HOME=/opt/deployment/

RUN mkdir ${APP_HOME}
# Copy the jar from local to image
COPY target/todo-1.0.0.jar ${APP_HOME}/todo-1.0.0.jar 

# Working directory
WORKDIR ${APP_HOME}

# Container port
EXPOSE 8000

# Run application with java -jar
CMD ["java", "-jar", "todo-1.0.0.jar"]
```
Build image
```bash
docker build -t todo-api:1.0.0 . --build-arg JAVA_VERSION=18-jdk
```

ENV - environment variable you can access even inside container. ENV overwrites ARG instruction

Build image
```bash
docker build -t todo-api:1.0.2 .
```
Run container
```bash
docker run -p 8080:8080 -d todo-api:1.0.2
```
Show logs (Here is error)
```bash
docker logs <container name>
```

Run container to connect mongodb
```bash
docker run -p 8080:8080 -d -e spring.data.mongodb.host=mongodb --net mongo-net todo-api:1.0.2
```
Show logs (success)
```bash
docker logs <container name>
```

#### Docker Best Practices & Deploying ReactJS App

.env file
```
REACT_APP_BACKEND_SERVER_URL='http://localhost:8080'
```

Run app traditionally
```bash
npm install
npm start
```

Dockerfile for ReactJS Dockerfile.dev
```dockerfile
FROM node:alpine

WORKDIR /app

COPY . ./

RUN npm install

ENTRYPOINT ["npm", "start"]
```

Build image
```bash
docker build -f Dockerfile.dev -t todo-iu:1.0.0 . 
```
Run container
```bash
docker run -p 3001:3000 -d todo-iu:1.0.0
```
Show logs 
```bash
docker logs <container name>
```

Dockerfile for ReactJS Dockerfile.dev
```dockerfile
FROM node:alpine

WORKDIR /app

COPY . ./

RUN npm install

RUN npm run build

RUN npm install -g serve

ENTRYPOINT ["serve", "-s", "build"]
```
Build image
```bash
docker build -f Dockerfile.dev -t todo-iu:1.0.1 . 
```
Run container
```bash
docker run -p 3002:3000 -d todo-iu:1.0.1
```
Show logs 
```bash
docker logs <container name>
```

Muti stage docker build Dockerfile.prod
```dockerfile
# npm run build
FROM node:apline as build

WORKDIR /app

# when package*.json not changed, cache generated will be used in the future builds
# Used not to run npm install if package not changed
COPY package*.json ./

RUN npm install

COPY . ./

RUN npm run build

# Deploying to nginx
FROM nginx:stable-alpine

COPY --from=build /app/build /usr/share/nginx/html

COPY nginx.conf /etc/nginx/conf.d/default.conf

CMD ["nginx", "-g", "daemon off;"]
```

Build image
```bash
docker build -f Dockerfile.prod -t todo-iu:1.0.2 . 
```
Run container
```bash
docker run -p 3002:3000 -d todo-iu:1.0.2
```
Show images
```bash
docker images
```
Show logs 
```bash
docker logs <container name>
```

.dockerignore
```
node_modules
build
```

## Docker Volumes

**Docker volumes** - used to persist container data, even if container deleted, mounting local directory with container directory.

Create volume
```bash
docker volume create mongo-data
```
List volumes
```bash
docker volume ls
```
Show volume mongo-data
```bash
docker volume ls -f name=mongo-data
```
Show info about mongo-data volume
```bash
docker inspect volume mongo-data
```

Run container to connect mongodb with volume
```bash
docker run -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_PASSWORD=password -p 21017:21017 -d --net mongo-net --name mongodb -v mongo-data:/data/db mongo:4.4.6
```

PostgreSQL - /var/lib/postgresql/data
MySQL - /var/lib/mysql

```bash
cd /var/lib/docker
ls
cd volumes/
ls
cd _data
```

Bind mounts - directory stored, data can be stored anywhere in your system.

```bash
docker volume rm mongo-db
```

### Docker Compose

docker-compose.yaml
```yaml

version: '3'

services:
    mongodb: 
        image: mongo
        ports:
            - 21012: 21017
        environment:
            - MONGO_INITDB_ROOT_USERNAME=root 
            - MONGO_INITDB_PASSWORD=password
        networks:
            - mongo-net
        volumes:
            - mongo-data:/data/db
        container_name: mongodb

    mongo-web:
        image: mongo-express
        ports:
            - 8081:8081
        environment:
            - ME_CONFIG_MONGODB_ADMINUSERNAME=root
            - ME_CONFIG_MONGODB_ADMINPASSWORD=password
            - ME_CONFIG_MONGODB_SERVER=mongodb
        networks:
            - mongo-net
        container_name: mongo-web
        depends_on: 
            - mongodb

    todo-api:
        image: todo-api:1.0.2
        ports:
            - 8080:8080
        environment:
            - spring.data.mongodb.host=mongodb
        networks:
            - mongo-net
        container_name: todo-api
        depends_on: 
            - mongodb

    todo-ui:
        build:
            context: .
            dockerfile: Dockerfile.prod
        image: todo-ui:1.0.2
        ports:
            - 3000:80
        container_name: todo-ui
        depends_on:
            - todo-api
            - mongodb


networks:
    mongo-net:
        name: mongo-net

volumes:
    mongo-data:
        name: mongo-data
```
Even if you not create custom 

Execute and create containers for apps
```bash
docker-compose -f docker-compose.yaml up -d
```


