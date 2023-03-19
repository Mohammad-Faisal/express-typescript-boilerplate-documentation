---
sidebar_position: 3
---

# CheatSheet with NodeJS and Databases

### Get a mysql server up and running

```sh
docker run \
  -it --rm --name mysql \
  -p 3306:3306 \
  --mount "src=mysqldata,target=/var/lib/mysql" \
  -e MYSQL_ROOT_PASSWORD=mysecret \
  mysql
```

**Connection details**

```
host: localhost
port: 3306
user: root
password: mysecret
```

### Get IP address of the running container

It will help you to connect from other applications that isn't sharing a network with this container.

```sh
docker inspect \
  -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' \
  container_name_or_id
```

### Get a mysql explorer up and running (Adminer)

Adminer is a free database explorer. It will help you to explore the running database.

```sh
docker run \
  -it --rm --name adminer \
  -p 8080:8080 \
  adminer
```

It will get a database explorer running on localhost:8080

### Get all running containers

```sh
docker ps  # all running containers
docker ps -a # all containers
docker container prune # remove all stopped containers
docker container restart name_of_container
```

### Define a docker network

If you try to access the running database container from the host you were required to know th IP address of the running container which is problematic.

If you keep the database explorer tool in the same network with the database itself then we can resolve the IP address from the container name itself.

So let's create a network first!

```sh
docker network create --driver bridge network_name
```

And now in our previous mysql command can be modified to use this network instead

```sh
docker run \
  -d --rm --name mysql \
  -p 3306:3306 \
  --mount "src=mysqldata,target=/var/lib/mysql" \
  -e MYSQL_ROOT_PASSWORD=mysecret \
  --net mysqlnet \
  mysql
```

and adminer

```sh
docker run \
  -d --rm --name adminer \
  -p 8080:8080 \
  --net mysqlnet \
  adminer
```

Now just refer the database server by it's name **mysql**

### Using docker compose

Now we can just separately run the containers but it would be cooler if we can run them all using one command. And docker compose let's us do that!

Let's run the above 2 containers simultaneously

Create a **docker-compose.yaml** file. And paste the following there

```yaml
version: "3"

services:
  mysql:
    image: mysql
    container_name: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=faisal
    volumes:
      - mysqldata:/var/lib/mysql
    ports:
      - "3306:3306"
    networks:
      - mysqlnet
    restart: on-failure

  adminer:
    image: adminer
    container_name: adminer
    depends_on:
      - mysql
    ports:
      - "8080:8080"
    networks:
      - mysqlnet
    restart: on-failure

volumes:
  mysqldata:

networks:
  mysqlnet:
```

Then run

```sh
docker-compose up
docker-compose down
```

If you have previously run containers then you might face some issues with the password. In that case you will need to remove previously created volumes

```sh
docker volume prune
docker-compose down --volumes
```

And then run again to see the database up and running

### Run the containers as backend service

If you want to run docker as a background service then you will add the **-d** option

```sh
docker-compose up -d
```

### Cleaning up:

Docker can take significant amount of disk space. To see that

```
docker system df
```

You can remove stopped containers, unused networks and dangling images by

```sh
docker system prune
```

### List commands

```sh
docker container ls -a # to see all containers
docker image ls -a # to see all images
docker volume ls  # to see all volumes
docker network ls  # to see all volumes
```

### Run a postgresql instance

Similarly we can run a postgresql instance by running the following command

```sh
docker run \
  -it --rm --name postgres \
  -p 3000:5432 \
  --mount "src=postgresdata,target=/var/lib/postgresql/data" \
  -e POSTGRES_PASSWORD=mysecret \
  postgres
```

### Run a MongoDB database

```sh
docker run \
  -it --rm --name mongodb \
  -p 3000:27017 \
  --mount "src=mongodata,target=/data/db" \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=mysecret \
  mongo
```

### Add an existing dump

To generate some metadata for the local container you can run the following command. It can vary from database to database. The following one is for MySQL.

```sh
docker exec -i database_container_id mysql -u root -p faisal volume < path_to_dump_file
```

### Create an independent Dockerfile for Nodejs application

```yaml
# base Node.js LTS image
FROM node:lts-alpine

# define environment variables
ENV HOME=/home/node/app
ENV NODE_ENV=production
ENV NODE_PORT=3000

# create application folder and assign rights to the node user. This is just for security reasons
RUN mkdir -p $HOME && chown -R node:node $HOME

# set the working directory
WORKDIR $HOME

# set the active user
USER node

# copy package.json from the host
COPY --chown=node:node package.json $HOME/

# install application modules
RUN npm install && npm cache clean --force

# copy remaining files
COPY --chown=node:node . .

# expose port on the host
EXPOSE $NODE_PORT

# application launch command
CMD [ "node", "./index.js" ]
```

Then build the image

```sh
docker image build -t nodehello .
```

and run it

```sh
docker run -it --rm --name nodehello -p 3000:3000 nodehello
```

### Create a Dockerignore file

Don't forget to create a **.dockerignore** file as well. This will avoid copying unnecessary files and keep our images lean.

```
Dockerfile
docker*.yml

.git
.gitignore
.config

.npm
.vscode
node_modules

README.md
```

But running it every time like this is not practical when we are developing

So let's create another **docker-compose.yaml** file

```yaml
version: "3"

services:
  nodehello:
    environment:
      - NODE_ENV=development
    build:
      context: ./
      dockerfile: Dockerfile
    container_name: nodehello
    volumes:
      - ./:/home/node/app
    ports:
      - "3000:3000"
      - "9229:9229"
    command: npm run dev
```

We can use the following command to build the image

```
docker-compose up --build
```

It will give you the ability to hot reload your code on development.

### Debug with VSCode

We can add a new option to the package.json file to debug our code

```json
"debug": "ts-node-dev --trace-warnings --inspect=0.0.0.0:9229 src/index.ts"
```

Now we will change the last command of our new **docker-compose.yaml** file

```
command: /bin/sh -c 'npm install && npm run debug'
```

then we can click on the debug icon on the left of vscode and create a new **launch.json** file inside the **.vscode**
folder

Put the following content inside that file

```json
{
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach",
      "port": 9229,
      "protocol": "inspector",
      "localRoot": "${workspaceFolder}",
      "remoteRoot": "/home/node/app",
      "skipFiles": ["<node_internals>/**"]
    }
  ]
}
```

Then we can run

```sh
docker-compose up
```

Then we can put break point inside the vscode. On the left of the editor and when we try to execute that code we will catch them inside debugger!

### Docker image commands

```
docker image build -t myimage:first .
```

here the -t option gives the ability to tag the image. If we don't tag it the **latest** tag will be used by default

or from a specific Dockerfile

```
docker image build -f myCustomFile.txt -t myimage:first .
```

See all images

```sh
docker images
docker image ls -a ## to see all images
```

## Docker Container commands

```sh
docker container ls ## active containers
docker ps ## active containers

docker container ls -a # stopped containers

docker exec -it <container_id_or_name> [command] [args...] # run a command inside the container

docker run [options] <image> [command] [args...]
docker run --it --rm -p 3306:3306 mysql

docker container restart <container_id_or_name> ## restart a container

docker container stop <container_id_or_name> # stop a container

docker stats ## See various metrics like cpu usage and memory usage

docker container rm <container_id_or_name>

docker container prune # remove all containers

docker inspect <container_id>
```

### Docker volume commands

```sh
docker volume ls # to see all volumes
docker volume rm <volume_name> # remove a volume
docker volume prune # remove all volumes that are not currently attached
```

A directory on the host can be mounted into a container using the -v hostdir:containerdir option with the docker run command. For example, the following command mounts the current directory’s code subdirectory to the container’s /home/app directory:

```sh
-v $PWD/code:/home/app
```

### Docker network commands

```sh
docker network create --driver bridge <network_name> # create a network

docker run --net mynet #Attach the container to that network in docker run with --net mynet

docker network ls ## view all networks

docker network rm <network_id_or_name> # remove the network

```

### Full clean start

to see the status of the containers

```
docker stats
```

To remove all existing containers and get a fresh start do this.

```sh
docker system prune -a --volumes
```
