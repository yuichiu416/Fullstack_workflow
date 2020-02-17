## Running a Container
`docker container run -d -p 8080:80 --name web nginx`

Let's break this command down into its separate parts.

`docker container run `- is the command telling docker you want to start up a new container using the following options.
`-d` - this flag means starting up the container in `detached` mode. Containers started in detached mode exit when the root process used to run the container exits.
`-p 8080:80 `- this is letting docker know that you want expose a port on your local machine and that any traffic on that port should route to the container IP.
The internal host IP is on the left: `8080`
The ip for the container is on the right: `80`
With this configuration you can go to `http://localhost:8080` and see your container running!
The port number is the exposed port number in the docker file

`--name web` - the name flag allows you to directly name a container
`nginx` - the final part of this command is the image we want to use for running this container.

## Common Docker Container Commands

Here are the commands you'll find yourself running most often. We recommend also bookmarking this [Docker Cheatsheet](https://www.docker.com/sites/default/files/Docker_CheatSheet_08.09.2016_0.pdf).

`docker --help `- the most useful command! It will list out all the options available to you.

`docker run [OPTIONS] IMAGE[:TAGNUMBER] [COMMAND]` - Check out the Docker run documentation for a list of options and flags.

`docker container ls` - lists all your running containers

`docker container ls -a `- lists all your containers (running or stopped)

`docker container stats` - with no provided container name to get live performance data

`docker container inspect <CONTAINERNAME> -` Will return json with the metadata about that specific container

`docker container top <CONTAINERNAME>`- Display the running processes of a container.

`docker container rm <CONTAINERNAME> `- remove one or more stopped containers

`docker container rm -f <CONTAINERNAME> `- stop and remove a running container

`docker container run -it <IMAGENAME> bash `- For running interactive processes (like a shell in this instance).

`docker container exec` - Run a command in a running container
ex. `docker container exec -it DogsRGood bash`


### Run one container with the mysql image
`docker container run --name mysql -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql`

### Start up your Ubuntu Shell
`docker container run -it --name ubuntu ubuntu bash`


### Create the network
`docker network create funtim`

### Run our two Containers`
`docker container run -d --net funtime --net-alias party elasticsearch:2`
`docker container run -d --net funtime --net-alias party elasticsearch:2`


### Make sure everything if configured
`docker container run --net funtime alpine nslookup part`

Now let's query them:
`docker container run --net funtime centos curl -s party:920`


### multi-line command
```docker
docker run -d --name nginx\
 --mount type=bind,source=/Users/rosekoron/rad,target=/rad \
 nginx
```

```docker
docker run -d --name DogsRGood -p 8080:80^
 --mount type=bind,source="D:/rad",target=/rad ^
 nginx
 ```

## Volumes

### Create first image:
`docker container run -d --name postgres -v psql-data:/var/lib/postgresql/data postgres:9.6.2`

### Enter the Postgres container
`docker container exec -it 84dc039d4555 psql -U postgres`

###  Create a table
```SQL
CREATE TABLE cats
    (
    id SERIAL PRIMARY KEY,
    name VARCHAR (255) NOT NULL
    );

    -- cat seeding
    INSERT INTO
    cats (name)
    VALUES
    ('Jet');
```

###  Get rid of any unused volumes
`docker volume prune`

# Images and the Dockerfile

### Building a Dockerfile
`docker build . -t <NEWIMAGENAME>`

## Dockerfile Example
```dockerfile
 FROM node:argon
    # Create app directory
    RUN mkdir -p /usr/src/app \

    # These two stanzas are part of the same created RUN layer
    # you can chain commands like this using '\' and '&&'
        && echo "This is part of the same layer" \
        && echo "and so is this one!"

    # change our current working directory
    WORKDIR /usr/src/app

    # Install app dependencies
    COPY package.json /usr/src/app/
    RUN npm install

    # Bundle app source
    COPY . /usr/src/app

    # expose a port
    EXPOSE 8080

    #  This is the command that will be run
    # each time a container is built using this image
    CMD [ "npm", "start" ]
```

# Dockerfile Cheat Sheet
## Common Dockerfile Commands

### FROM
Almost every Dockerfile begins with the `FROM` instruction that will set the Base Image for all subsequent instructions. The only argument that can come before `FROM` is [ARG](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact) which can be used to declare a variable outside of the build stage, that can be later be used inside the build stage.

### ENV
This command is the preferred way to inject keys and values into image building. All the variables set using this command can be used in the subsequent instructions in the build stage.

### RUN
The RUN instruction will execute each phrase of commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile. Additionally the `RUN` command can run shell scripts or whatever you could use within a container.

### WORKDIR
The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile. It is best practice to use `WORKDIR` to run commands that rely on being in a certain location in the file tree. The `WORKDIR` instruction can be also be used multiple times in a Dockerfile. If a relative path is provided, it will be relative to the path of the previous `WORKDIR` instruction.

### EXPOSE
Specify to the image which ports are going to be exposed within that image.

### CMD
This is the final command that will run every time you launch a new container from this image or restart a stopped container of this image.

### COPY
Will make a copy of the files in the first given location to the second given location.

## Pushing to Docker Hub
`docker login`
`docker image tag <originalImage> <USERNAME>/<originalImage>`
`docker image push <IMAGENAME>`

# Dockerfiles Galore
### Flow
1. Create the Dockerfile and define the steps that build up your images
2. Issue the `docker build` command which will build a Docker image from your Dockerfile
3. Use this image to start containers with the `docker container run` command
4. Reiterate and Rebuild As Needed

# Docker Compose
```dockerfile
# If no version is specified then version 1.0 is assumed. 
# Recommend version 2 at the minimum
version: '3.1'  

services:  # Will start up containers. Is the same as using docker container run.
  servicename: # A Friendly name (postgres, node, etc.). This is also DNS name inside your network
    image: # the image this service will use
    command: # Optional, will replace the default CMD specified by the image
    environment: # Optional, same as -e in docker container run
    volumes: # Optional, same as -v in docker container run
  psql: # servicename2

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```

# Docker Compose CLI
1. `docker-compose --help` - Because who doesn't need a little help now and again?
2. `docker-compose up` - Which will setup your volumes, networks, and start the specified containers
3. `docker-compose down` - Which will stop and remove all containers and networks.
4. `docker-compose down -v` - Which will stop and remove all volumes, containers and networks.

## Dockerfile for Backend

```dockerfile
FROM node:12.10.0-alpine
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY package.json /usr/src/app
RUN npm install
COPY . /usr/src/app
EXPOSE 5000
CMD ["npm", "start"]
```

### Building your image
`docker build -t <your username>/node-web-app .`

### Run the image
`docker run -p 49160:8080 -d <your username>/node-web-app`

## Dockerfile for Frontend

```dockerfile
FROM node:12.10.0-alpine
RUN mkdir client
WORKDIR /client
COPY package.json /client/
RUN npm install
COPY . /client/
EXPOSE 3000
CMD ["npm", "start"]
```

## Docker-compose.yml
```yml
version: '3.0' # specify docker-compose version
 
# Define the services/ containers to be run
services:
  client: # name of the first service
    build: client # specify the directory of the Dockerfile
    ports:
    - "3000:3000" # specify port mapping

  express: # name of the second service
    build: . # specify the directory of the Dockerfile
    ports:
    - "5000:5000" #specify ports mapping
    links:
    - database # link this service to the database service

  database: # name of the third service
    image: mongo # specify image to build container from
    ports:
    - "27017:27017" # specify port forwarding
```