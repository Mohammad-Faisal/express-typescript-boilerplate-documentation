---
sidebar_position: 2
---

# Docker for Local Development

### How to use Docker for Local NodeJS development with Database

Today we will learn how we can create a full-fledged backend application using NodeJS and any Database (Postgres/MySQL etc.).

We will use docker-compose to get that application server and Database up and running at the same time, which can make our development experience so much better.

I assume you already know the basics of docker and have an idea of how to dockerize a basic NodeJS application. If not, you can take a look at the following article.

https://www.mohammadfaisal.dev/blog/express-typescript-docker

### Get the boilerplate

First, clone the [express-docker boilerplate repository](https://github.com/Mohammad-Faisal/express-typescript-docker) where we have a working Express application with Typescript and a basic setup with docker-compose.

We will integrate the Database on top of this repository.

```sh
git clone https://github.com/Mohammad-Faisal/express-typescript-docker.git
```

### But Why?

When you are developing locally, every time going into the PostgreSQL server with pgadmin or MySQL server can be boring. We can use the power of docker-compose to integrate the Database very easily, which will ensure that all on the team have the same experience when developing.

### Basics

Open the **docker-compose.yml** file where we already have a service up and running named **express-typescript-docker**. We will add another service for our Database on top of it.

We will also need a common network that these 2 services will share because our application server will need to communicate with the database server.

We will also need a volume for the database server. So in total we will need 3 things.

- A database service
- A[ docker volume](https://docs.docker.com/storage/volumes/) that will be used by the Database
- A shared [docker network](https://docs.docker.com/network/)

### Define the database service.

Open the **docker-compose.yml** file and add the following database service under the already existing express-typescript-docker service

```YAML
mysql-docker:
    image: mysql
    container_name: mysql-docker
    environment:
      - MYSQL_ROOT_PASSWORD=any_password
    volumes:
      - express-mysql-data:/var/lib/mysql
    ports:
      - '3306:3306'
    networks:
      - express-mysql-network
    restart: on-failure

volumes:
  express-mysql-data:

networks:
  express-MySQL-network:
```

Here there are several things that we can take a look at:

1. **environment** -> Defines the MySQL root password. If you don't want to use root password you can create a new user and assign a password as well. You can even create a new database through this as well.

```YAML
environment:
  MYSQL_DATABASE: "db_name"
  # So you don't have to use root, but you can if you like
  MYSQL_USER: "user"
  # You can use whatever password you like
  MYSQL_PASSWORD: "user_password"
  # Password for root access
  MYSQL_ROOT_PASSWORD: any_password
```

1. **volumes** -> We defined a volume at the end of the compose file and attached that volume to this docker image's
   _/var/lib/mysql_ Because the internal memory of a docker image is ephemeral so if you stop the container all the data between the sessions will be lost which is not expected.

A volume can help us to avoid this by persisting the data between sessions.

3. **networks** -> This helps us to connect the services together like a stitch

and on the express-typescript-docker service, add some new dependency

### Use an alternative database

If you want to use another database (like PostgreSQL), this is very easy to do now. All you have to do is swap the MySQL service with something like the following.

```YAML
postgresql-docker:
  image: postgres
  container_name: postgres-docker
  environment:
    - POSTGRES_PASSWORD=any_secret
  volumes:
    - postgresdata:/var/lib/postgresql/data
  ports:
    - "5432:5432"
  networks:
    - shared-network
  restart: on-failure
```

That's it! How easy was that? Are you interested in MongoDB? We can have that too!

```YAML
mongodb-docker:
  image: mongo
  container_name: mongo-docker
  environment:
    - MONGO_INITDB_ROOT_USERNAME=root
    - MONGO_INITDB_ROOT_PASSWORD=any_secret
  volumes:
    - mongodata:/data/db
  ports:
    - "27017:27017"
  networks:
    - shared-network
  restart: on-failure
```

I hope you understand the power of docker in local development now. No installation, No configuration, No OS-specific Pain point!

### Update the server

Now we will need to modify the configuration for a server like the following.

```YAML
depends_on:
  - mysql-docker
networks:
  - express-mysql-network
restart: on-failure
```

The **depends_on** part defines that we should wait for the Database to get up and running and then start the application server.

### Test it

So we have a good configuration now, but how to test that?
Let's run the container first.

```sh
docker-compose up
```

This will correctly get the Database and the application server up.

### Connect to Database using adminer.

Let's take advantage of docker a bit more. There is a good application called **adminer** that is a database tool that allows us to inspect many databases.

Let's add that to our **docker-compose.yml** file as well. Under the service add the following

```YAML
  adminer:
    image: adminer
    container_name: adminer-docker
    depends_on:
      - mysql-docker
    ports:
      - '8080:8080'
    networks:
      - express-mysql-network
    restart: on-failure
```

Let's run the docker-compose up again and go to **http://localhost:8080/**. There you will see a login page. Use the following credential there to log in to the Database.

```MD
server: MySQL-docker
username: root
password: any_password
database: test_db (optional)
```

And you should be able to log into the application.

### Connect to Database using Sequalize.

Well. So we can now connect to the Database. But how can we connect to the Database internally? We will use the most popular ORM named [Sequalize](https://sequelize.org/) to do that. Let's install it first

```sh
yarn add sequelize mysql2
yarn add -D @types/sequelize
```

To connect to the Database, you must create a Sequelize instance.

Let's create a database utils file and add the following code there.

```js
import { Sequelize } from "sequelize";

const sequelize = new Sequelize("test_db", "root", "any_password", {
  host: "mysql-docker",
  dialect: "mysql",
});

export const connectToDatabase = async () => {
  try {
    await sequelize.authenticate();
    console.log("Connection has been established successfully.");
  } catch (error) {
    console.error("Unable to connect to the database:", error);
  }
};
```

And update the **index.ts** file to connect to Database first and then start the server

```js
const startServer = async () => {
  try {
    await connectToDatabase();
    app.listen(PORT, (): void => {
      console.log(`Connected successfully on port ${PORT}`);
    });
  } catch (error: any) {
    console.error(`Error occured: ${error.message}`);
  }
};

startServer();
```

Now you can just run the following command to get everything up and running.

```sh
docker-compose up
```

And see the following result!

```JSON
express-typescript-docker    | Connection to Database has been established successfully.
express-typescript-docker    | Server running successfully on port 3000
```

Congratulations! Now you have a working NodeJS application up and running that works with Database, and we have a Database monitoring tool as a bonus!

### Keep in mind

Don't forget that this setup is only for development environments. For production, you might want to separate your Database from your server. You can use a dedicated host or take help from cloud services like AWS.

### Troubleshooting

If you have previously run containers, then you might face some issues with the password. In that case, you will need to remove previously created volumes.

```sh
docker volume prune
docker-compose down --volumes
```

### Cleaning up:

Docker can take a significant amount of disk space. To see that

```
docker system df
```

You can remove stopped containers, unused networks, and dangling images by

```sh
docker system prune
```

### Resources

- [Docker Network Documentation](https://docs.docker.com/network/)
- [Docker Volume Documentation](https://docs.docker.com/storage/volumes/)

## Github repo

https://github.com/Mohammad-Faisal/express-sequelize-docker-compose
