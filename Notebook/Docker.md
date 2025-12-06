### Container

A way to package applications with all the necessary dependencies and configuration.  
This package can be shared which makes development and deployment more efficient.

Containers are stored in a container Repository. There are private repos for each company and the **DockerHub(**[https://hub.docker.com/](https://hub.docker.com/)**)** which is the public repo.

**Container Image ⇒** It refers to the container for storing specific type of files.

  

### Why Containers ⇒ Application Development

Multiple developers working on the same project each have different systems and need to install same software everywhere.

With containers you dont have to install anything on your OS. The container in itself is an isolated environment with all the resources and config needed to run your application.

These also allow 2 versions of the same application running on the same OS with different containers.

### Why Containers ⇒ Application Deployment

Traditionally we have files and a set of instructions on how to install them with a set of dependencies. This will lead to issues when the instructions are not follwed or there are dependency conflicts when installing.

With Containers, you dont have to configure the system. Just pull the image into the server and run the container.

Another advantage of containers is that if you pull newer version only the images that have changed need to be downloaded.

### What are images?

THese are a set of read only files which allow the docker container to run as an isolated environment.

> [!important] A container is made of several images stacked together. THe outer most image is the Linux base image while the application image lies on top. It can be called as the instance of an image on a system.

> [!important] A container is an instance of the image on a system. It is the environment where the image runs.

### Docker vs Virtual Machine(VM)

Any OS has two layers :

1) Application Layer ⇒ It runs all the applications and is based on the OS Kernel.  
2) OS Kernel. ⇒ it is the layer that communicates with hardware and the application layer.

VM virtualizes both the application layer and the OS Kernel. This means that it has its own OS systems and applications running on those systems.

Docker on the other hand does not have its own OS Kernel. It just virtualizes the application layer and creates an isolated environment to run application using the system’s kernel.  
Thus for Docker, there are seperate images based on the OS that you are on.

![[Capture.png]]

  

  

## Basic Docker Commands

- `docker pull __appName` ⇒ pulls the latest image
- `docker pull _appName : _version` ⇒ pulls image corres version

- `docker images` ⇒ shows the available images

- `docker run _appName` ⇒ create a container corres app.
    
    - whenever a container is created it gets an id and a name  
        `docker run —name redis123 redis:4.0`
    

- `docker run -d _appName` ⇒ create a container corres app in detached mode.
    
    - If you press ^C in terminal it will stop the container if not in detached mode.
    

- `docker ps` ⇒ show all the running images/containers and the port

- `docker stop _containerId` ⇒ stop the container

- `docker start _containerId` ⇒ start the container with the given id which was stopped earlier

- `docker ps -a` ⇒ shows all the container that are running/not running(closed)

- **Running Multiple containers of same app :**
    
    - When such a thing happens, the container port remains the same but the system port that the container bindds to changes. Hence we can support multiple containers of same app.  
        We specify the binding in the following way ⇒ -p<sysPort>:<containerPort>
    
    - `docker run -p6000:6739 redis`
    
    - `docker run -p6001:6739 redis:4.0`
    

- `docker rm` `_containerName_`

- `docker rmin` `_imageName_`

  

## Debugging Containers

`docker logs _containerId`

`docker logs _containerName`

`docker exec -it _containerId /bin/bash` ⇒ shows the file system of the container. Use Linux commands to navigate.

> [!important] Use env in exec mode to get the env variables.
> 
>   
> Use exit to exit the exec mode.

  

## Docker Workflow

- You pull an image of mongoDB from docker to build our app.

- You create the JS app and push it to git.

- Jenkins CI builds the JS application and creates artifacts.

- These artifacts are then converted too images which are pushed to a repository

- Dev Server pulls the newly created image and all its dependency images and runs it.

![[95127fc4-3c8e-4661-bfb3-f474cda4c231.png]]

  

## Docker Networks

Docker has multiple networks which arre like environments where a set of containers which interact with each other can run.

When we want multiple containers to interact, we need to run them in the same network.

Any outside application will connect to the network via the container ports.

`docker network create mongo-network` ⇒ Creates a neew network

  

## Docker Compose

facilitates easy startup of services. In command line you need a lot of commands instead use Docker Compose

```XML
## create docker network
docker network create mongo-network


## start mongodb

docker run -d \
-p 27017:27017
-e MONGO INITDB_ROOT_USERNAME=admin\
-e MONGO_INITDB_ROOT_PASSWORD=password \
--net mongo-network\
--name mongodb \
mongo

## start mongo-express

docker run -d \
-p 8081:8081\
-e ME CONFIG_MONGODB ADMINUSERNAME=admin\
-e ME CONFIG_MONGODB ADMINPASSWORD=password \
-e ME CONFIG MONGODB SERVER-mongodb \
--net mongo-network \
--name mongo-express \
mongo-express
```

```YAML
version: '3'
services:
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password

  mongo-express:
    image: mongo-express
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
```

  

`docker compose -f __pathToYamlFile up -d` ⇒ runs the env described in compose.

  

## Docker File - Creating Your Own Container

```JSON
FROM node //image name => base image defined
ENV MONGO_DB_USERNAME=admin \
		MONGO_DB_PASSWORD=password
RUN mkdir =p /home/app //RUN => used to run linux commands
COPY . /home/app //copy all the files in current directory to the newly created directory
CMD ["node","/home/app/server.js"] // executes "node server.js" => probably works as arg1 arg2
```

Always save docker file as “Dockerfile”

`docker build -t __imageName __pathToCode` ⇒ build custom image

This is basicallly what Jenkins does. It pulls the code from the git and builds it. Then creates a dockerfile corres to that. Creates the image and pushes it to the private repo.

  

## Pushing into private repository

See the tutorial of the service where you are creating.

The images in private repos have a << registryDomain/imageName >> naming convention.

```JSON
version: '3'
services:
	app: 
		image: __regDomain/app:1.0
		ports:
			- 3000:3000
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password

  mongo-express:
    image: mongo-express
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
```

This will deploy everything in the dev server.

  

## Docker Volumes

Used to facilitate data persistence for docker containers. When docker containers are stopped all the data in them is lost.

In this system, a folder in the host file system is mounted onto the virtual docker system. Any changes on the container is replicated on the system.

`docker run -v _hostDir:_containerDir`⇒ create volume on _hostDir

`docker run -v containerDir`⇒ create volume anonymously

`docker run -v _name:containerDir`⇒ create volume named _name

use volume attribute for each image in docker compose setup

```JSON
version: '3'
services:
  # my-app:
  # image: ${docker-registry}/my-app:1.0
  # ports:
  # - 3000:3000
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db
  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8080:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
    depends_on:
      - "mongodb"
volumes:
  mongo-data:
    driver: local
```