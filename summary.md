* [What is container](#what-is-container)
* [Commands](#commands)
# What is image
Image is the actually artifact that packages the physical files/configurations/images that will be stored on our local machine or in docker hub. We can shift this image to other machines in order to start and run them. When we ask docker to run a image, this will be a **container**. 

## image tag
Image can be in specific version. This is indicated as *tag*. There is a special tag that is called *latest* which indicates the most updated image version. When we use docker pull and does not specify the tag, it will pull the version that considered as lastest.

# What is container
As mentioned above, **container** is actually the instance of image that was executed. This is actually the running environment of the image. It has ports binded to it, virtual file system, So we could have image X
in pur machine, and run it twice, and therefore get **two containers** from this image.
Dokcer container is built by layers of images. Each image provides functionallty/infrastructure to the next layer. For example, most containers has in the first layer the image of `alpine` which is base linux image. Uselly, our app will be in the last layer. When we download new container X, and we already have container Y, and lets say that they both have same first 2 layers, it means that when we download container X, the first 2 layers will not be downloaded because we already use them in container Y.

# What is Host
Host is the machine that actually runs the docker. This is the machine where the containers live on.

# Image And Container Relation
We can think about the relation between image and container like the relation betweehn class and object instance, where image is like a class, and container is like object instance. The class defines the blueprint of how object should look like, which variables and configurations. The instanitation of the class will create actual object instance that is acting as the class defined.

# Difference between docker and VM
VM implements application layer and the kernel layer. On the other hand, docker implements only the application layer (the kernel layer is of the host). This difference leads
to the technical diffs:
1. Docker is smaller then VM (because there is no kernel layer code)
2. Docker is more faster in terms of starting (there is no need to boot kernel)
3. Docker is compitable on on host that implements the kernel it needs (there are workarounds)

# Commands

## docker run
This command creating new container from image and starting it. We uselly configure the container (like ports, name, etc...) in this command.
This is actually equivalent to docker pull ->  docker create  ->  docker start. 
In order to pull image and then create and start a container from it, we will run:

    docker run <imageName:version>

For example, if we want container of postgres of version 9.6 we will run:

    docker run postgres:9.6

After executing the command, the docker will be run in detouch mode, which means that the docker will be running in the terminal session, and if we will press *ctrl+c* it will terminate the conatiner. 
If we want to run the docker in the background we need to add *-d* parameter to the command. For example:

    docker run -d redis

This will container of the redis image in the background. The output of this command is the id of the container.


## docker pull
This command will pull image from local machine. If it can't find it in local, it will try to pull from docker repo that is defined as the hub.

    docker pull redis

## docker images
In order to list the images that we have on current machine, we will run:

    docker images

## docker create
This command will create container from image (the container will not be started, just created). 

    docker create redis
This command will create container from the image redis. This act is like instantiating object from class (but still without using it). In this stage we will set parameters that will configure the environment of the container, such as port binding, name, environment varables etc... (if in the future we will want to update the configurations of the container, we will use `docker update` and then stop and start the container for the changes to take effect).<br>
The output of this command is the created container id. If we will list all containers that exist on machine, we will see that the container that we just created is in *Created* Status.

## docker start
If we want to start a not-running container, we will run:

    docker start <container-id>
Notice, when we starting container, we cannot set the environment of the container (for example: port binding). The environment modification
is happend only when we **creating** the container in the beginning (or with `docker update` command if the container is already created).<br>
Notice! by default, using this command will start the container in background (detach mode).

## docker stop
If we want to stop container, we will run

    docker stop <conatiner-id>

Notice! When you stop container and then start it, it still will save the files and other persist issues.

## docker rm
If we want to delete container:

    docker rm <container-id/name>

## docker rmi
If we want to delete image:

    docker rmi <image-id/name>

## docker ps
In order to list all the running containers, we will run:

    docker ps

If we want to list all containers, running and stopped, we will run:

    docker ps -a

## Debugging Commands

### docker logs
This command shows the stdout and sterr of a container

    docker logs <container-id/name>

### docker exec
This command allow to run terminal commands inside the virtual environment of the docker.

    docker exec -it <container-id/name> <path to terminal exe>

for example:

    docker exec -it amazing_feynman  /bin/bash
This will get as prompt of bash, of the system of the container with the name *amazing_feynman*. We then can print environment variables values, look on file system, files, etc...

# Docker Run Configurations
When creating container (using `docker run` or `docker create`), we uselly want to configure the container.
* `--name <name>` for configure container name.
* `-e <ENVIRONMENT_VAR>=<vale>` for configure environment variable.
* `--net <network-anme>` for specify docker network for the container.

# Port Binding
When creating a container, we can bind ports of the host, to ports of the container. For example, we can bind port 5000 of the host, to port 6000 of the container, which means that every request for host:5000 will directed to the container:6000. <br>
In some host, we canot use the same port for two different services. On the other hand, two different containers actually can uses same ports internally, because container is isolated environment that does not know about pther containers that running in same host. <br>
For example, redis image is running redis service which internally listens to port 6379. If we will run two containers of redis, they both will listen to port 6379. In order to be able to send request to the container, we must bind one of the host ports, to the container port. We can do this by specifying the parameter *-pHOST_PORT:CONTAINER_PORT*. For example: 

    docker run -p5000:6379 -d redis
    docker run -p6000:6379 -d redis

In the first example we binded host port 5000 to the container port 6379. So when we access locahost:5000, we will be redirected to container:6379. <br>
In the second example we binded host port 6000 to the container port 6379. So when we access locahost:6000, we will be redirected to seconds_container:6379. <br>
If we running `docker ps` we can see: <br>
```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                    NAMES        
005363ed7e6f   redis          "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes   0.0.0.0:6000->6379/tcp   angry_wescoff
3e1795fd1858   redis          "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes   0.0.0.0:5000->6379/tcp   ecstatic_mccarthy
```

We can see in the *Ports* column the port binding. Notice! we canot bind the same HOST port to two different containers.

# Docker Network
We can create isolated network, and then assign containers to "live" in this network. When two containers live in same network,  they can access each other by their names. For example: <br>
Let's say we executed created one container named c1, and then created another container named c2, and they both in the same network. If c1 wants to access c2 in port 6000, c1 could just call `c2:6000` (calling to the name) and the request will be directed to c2 (in port 6000). <br>

## List Docker Networks
```docker network ls``` will list all existing networks.

## Creating Network
`docker network create <network-name>`

## Creating Container In Specific Network
If we want to create and running container in specific network: <br>

    docker run --net <network-name> <image-name>


# Docker Compose
When there are multiple containers that working together in a system, we have to start them all. The docker compose is a way to make the execution of multiple containers easier.<br>
In order to use docker compose, we write new yaml file with the all the conatiners configuration. Notice! docker compose create new docker network for all the containers that was configured in it. Then we use `docker-compose` command to up the containers<br>

## Example 
Say we have these two containers which "live" in same `mongo-network` docker network:

    docker network create mongo-network 

    docker run -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb --net mongo-network mongo    

    docker run -d -p 8081:8081 -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin -e ME_CONFIG_MONGODB_ADMINPASSWORD=password --net mongo-network --name mongo-express -e ME_CONFIG_MONGODB_SERVER=mongodb mongo-express  

The equivalent docker compose yaml file that will configure those containers is:

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

Then, for make these container running we will execute:

    docker-compose -f docker-compose.yaml up

This will run the containers, the names of the containers will ge prefix and suffix, and also a new docker network will be created for those containers.

## Depending Containers
There is option to create logic if any container is depend and has to be started after other container.


# Creating Docker Image
Building a docker image is based on file that is called `Dockerfile` (must be named exactlly like this!).

## Dockerfile
This is special file that we will configure out new image. Example:

    FROM node:13-alpine

    ENV MONGO_DB_USERNAME=admin \
        MONGO_DB_PWD=password

    RUN mkdir -p /home/app

    COPY ./app /home/app

    # set default dir so that next commands executes in /home/app dir
    WORKDIR /home/app

    # will execute npm install in /home/app because of WORKDIR
    RUN npm install

    # no need for /home/app/server.js because of WORKDIR
    CMD ["node", "server.js"]

### From
This section tells docker on which image we want to build our image (as mentioned above, docker image built by few images layers). In this example, there is need for node, so there is image that already has node installed on it.

### ENV
This section set environment variables for the containers that will be created from this image.

### RUN
This section define linux command that will be executed **inside** the contaiers that will be created from this image.

### COPY
This section responsible to copy files **from host into path inside the container**. In the example above, the `./app` directory that is located in current directory of the Dockerfile in the **host**, will be copied to the container to the path `/home/app`.

### CMD
This section defins list of words. The words is actually a command that is the entrypoint of the container. So in this example, the entrypoint of the containers that we be created from this image is `node server.js`.

## Building Docker Image From Dockfile
After we createed `Dockerfile`, we can create an image. The command is:

    docker build -t <imageName:tag> </path/directory/containsDockerfile>

# Private Docker Repo
Docker repo is a server repository that saves images. We can create our own repo in order to share the images that we are creating for the org.

## Image Naming in Docker
The naming of image defines also the repo from where the image. The naming is as follows:

    repoDomain/imageName:tag

So, when we use `docker pull repoDomain/imageName:tag`, then `imageName:tag` will be pulled from the repo `repoDomain`.
When we executes `docker pull imageName:tag` without repo prefix, docker pulls from the default docker repo domain (which is `docker.io/library/`).

## Docker login
In order to push images to specific docker repo, we must obtain access by login with credentials for this server.
After the login, we can proceed to pushing.

## Docker push
For pushing docker to repoDomain, we first have to tag (name) the image as follows:

    docker tag <my-image> repoDomain/<my-image>:version

This will copy `my-image` image and create new copied image with the name `repoDomain/<my-image>:version`. In that way, the repo is embedded in the image name. Then we need to execute:

    docker push repoDomain/<my-image>:version

## Private Docker Repo In Docker Compose
When we need one of our containers from docker repo in our docker compose, in the image field, we have to insert 
    image: repoDomain/image-name

# Persisting Data With Volumes
When our application is stateful and uses database, we need the docker to be able to persist data. When we run image again, the data of the virtual file system of the container is "new", so we need a way to use previous data.

## Docker Volume
Docker volume is a way that we map between directory from the virtual filesystem of the container, into physical directory in the host. In order to configure the volume, we use `-v` parameter. In that way, if the container path `path/in/container/data` is mapped to `/path/in/host/data`, so every adding/modification of files in the host, inside `/path/in/host/data`, will be replicate to the container in the path `/path/in/container/data`. And vica versa, if we modify/add files in the container in `/path/in/container/data`, it will be replicate to the host in `/path/in/host/data`.

### 3 Volume Types

#### Host Volume
You specify mapping between specific path of the host, to specific path on the container. Example:

    docker run -v /path/in/host/data:/path/in/container/data <image-name>

#### Anonymous Volume 
You specify only the which path inside the container will be mapped to host voluem, but, you don't specify in which path of the host. Docker will do it automatically (actually it will create seperate folder in the host for each container). Example:

    docker run -v /path/in/container/data <image-name>

Notice! as mentioned, in this way, each container has seperate folder. So if you want that different containers will share same folder in host, in order to share files, dont use this way. Use named volume(see next type).

#### Named Volume (preferable type)
You first create volume in host and call it a name. Docker will create a folder in host.
Then, if you want that container will be mapped to this volume, you can specify this. Example:

    docker volune create <volume_name>
    docker run -v <volume_name>:/path/in/container <image-name>
    docker run -v <volume_name>:/path/in/container2 <image-name2>

In the this example, we first created volume with name *volume_name* and then created two containers that mapped some folder in them to the host volume.

### Volumes With Docker Compose
Best practice to use *Named Volume* inside docker compose. Example:

    version: '3'
    services:
        mongodb:
            image: mongo
            ports:
            - 27017:27017
            environment:
            - MONGO_INITDB_ROOT_USERNAME=admin
            - MONGO_INITDB_ROOT_PASSWORD=password
            volumes:
            - mongo-data:/data/db
    volumes:
        mongo-data:
            driver: local

In this example we defined *volumes* section, and one volume named *mongo-data*. The driver attribute indicates to docker to create on local machine. It will manage the actually host specific path. Then, in the service of *mongodb*, 
in *volumes* section we mapped the named volume *mongo-data* to the container path `/data/db` (this is the path where the mongo saves his db data).

**Notice! It is important to check for each database system, what is the path that used for saving the data on.**

### Where Docker Saves Data In Host
In each OS the path is different. If you cd to that path, you will see the volumes data folders. Those folders has the same data as there is inside the container mapped folder.
Let's say that we have con