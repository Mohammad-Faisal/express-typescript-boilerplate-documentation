---
sidebar_position: 4
---

# Sequelize with ExpressJS and PostgreSQL

Today we will learn how we can connect to a PostgreSQL database from an ExpressJS application and How we can setup Sequelize ORM inside the project.

First we need a database to connect with. You can take help from some cloud provider like AWS RDS or we can spin up a local database instance.

We have talked about how we can spin up a local database instance with the help of docker easily i[n a previous article](https://www.mohammadfaisal.dev/blog/express-database-docker-compose)

Our starting `docker-compose.yml` file will look like this

```yaml
version: "3"

services:
  database-layer:
    image: postgres
    container_name: database-layer
    environment:
      - POSTGRES_USER=dbuser
      - POSTGRES_PASSWORD=dbpassword
      - POSTGRES_DB=dbname

    volumes:
      - database-volume:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - shared-network
    restart: on-failure
  express-typescript-boilerplate:
    depends_on:
      - database-layer
    environment:
      - NODE_ENV=development
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - ./:/usr/src/app
    container_name: express-typescript-boilerplate
    expose:
      - "4000"
    ports:
      - "4000:4000"
    command: npm run dev
    networks:
      - shared-network
    restart: on-failure
  adminer:
    image: adminer
    container_name: adminer-docker
    depends_on:
      - database-layer
    ports:
      - "8080:8080"
    networks:
      - shared-network
    restart: on-failure

volumes:
  database-volume:

networks:
  shared-network:
```

And put this file inside your express-project and run the following command.

```sh
docker-compose up
```

It will spin up a local database instance of postgres and the credentials are

```js
username: dbuser
password: dbpassword
database name: dbname
host: database-layer
```

### Why Sequelize?

There are multiple ORM for us to choose from. TypeORM is a good alternative. But in terms of feature Sequelize is best. It's also widely popular. So we are choosing this.

But we are not going to use the base package. As that doesn't provide enough support for Typescript. So we will use a special package named [`sequelize-typescript`](https://www.npmjs.com/package/sequelize-typescript)

First install that

```sh
yarn add sequelize-typescript
```

As we are using PostgreSQL so we need to install the client for that as well.

```sh
yarn add pg
```

### Create a model for the database

Let's create a model for the `User` table in the database.

```js
import { Table, Model, Column, DataType } from 'sequelize-typescript';

@Table({
  timestamps: false,
  tableName: 'users',
})
export class User extends Model {
  @Column({
    type: DataType.STRING,
    allowNull: false,
  })
  name!: string;

  @Column({
    type: DataType.STRING,
    allowNull: false,
  })
  email!: string;

  @Column({
    type: DataType.STRING,
    allowNull: true,
    defaultValue: '',
  })
  password!: boolean;
}

```

### Configure the connection

Then create our connection for the database

```js
import { Sequelize } from "sequelize-typescript";
import config from "../config/Config";
import { User } from "../models/User";

const connection = new Sequelize({
  dialect: "postgres",
  host: config.dbHost,
  username: config.dbUser,
  password: config.dbPassword,
  database: config.dbName,
  logging: false,
  models: [User],
});

export default connection;
```

### Connect to database

Finally connect to database from the `index.ts` file

```js
import connection from './services/SequelizeClient';

const startServer = async () => {
  try {
    dbClient = await connection.sync(); // See here!
    server = app.listen(PORT, (): void => {
      console.log(`Connected successfully on port ${PORT}`);
    });
  } catch (error: any) {
    console.error(`Error occurred: ${error.message}`);
  }
};

startServer();
```

If everything goes well you should have the connection to database up and running.

### Create a repository

Let's isolate all our queries into the repository class

```js
import { User } from "../models/User";

export default class UserRepository {
  createUser = async (
    name: string,
    email: string,
    password: string
  ): Promise<User> => {
    const user = User.build({ name, email, password });
    return await user.save();
  };

  findByEmail = async (email: string): Promise<User | null> => {
    return await User.findOne({ where: { name: email } });
  };

  getAllUsers = async (): Promise<User[]> => {
    return await User.findAll();
  };
}
```

These are some examples that you can use quickly.

### Github Repository

https://github.com/Mohammad-Faisal/express-typescript-sequelize-docker-boilerplate

### Resources:

- https://sequelize.org/docs/v6/
- https://www.npmjs.com/package/sequelize-typescript
