---
sidebar_position: 2
---

# Documentation with Swagger

Today we will learn how we can add swagger to an ExpressJS application. If you are interested in the code, you can look at the following repo.

https://github.com/Mohammad-Faisal/professional-express-sequelize-docker-boilerplate

### Step 1: Installation

We will be using [swagger-ui-express](https://www.npmjs.com/package/swagger-ui-express) for this purpose.
Let's first install that.

```sh
yarn add swagger-ui-express
```

That's it!

### Step 2: Add swagger configuration

Create a file named `swagger.ts` at the root of your application.

```js
export const swaggerDocument = {
  openapi: "3.0.1",
  info: {
    version: "1.0.0",
    title: "APIs Document",
    description: "your description here",
    termsOfService: "",
    contact: {
      name: "Mohammad Faisal",
      email: "mohammadfaisal@gmail.com",
      url: "https://www.mohammadfaisal.dev/",
    },
    license: {
      name: "Apache 2.0",
      url: "https://www.apache.org/licenses/LICENSE-2.0.html",
    },
  },
};
```

### Step 3:Inject the document

Now we need to inject this swagger document into the project.
Go inside your `index.ts` file and import `swagger` and `swaggerDocument`

```js
import swaggerUi from "swagger-ui-express";
import { swaggerDocument } from "../swagger";
```

And then initialize a route

```js
// other middlewares like body-parser and cors
app.use("/api-docs", swaggerUi.serve, swaggerUi.setup(swaggerDocument));
```

### Writing documentation

There are two ways to write the swagger specifications. One is `JSON` and another is `Typescript`
I will be using `Typescript` because it's easy to use.

Let's first create a documentation file for the GET `/users` path. Create a new file named `getUsers.ts` inside a folder named `open-api`.

The folder and the file name don't matter!

We are expecting a response from the following array of users.

```json
[
  {
    "id": 1,
    "name": "test@gmail.com",
    "email": "Mohammad Faisal",
    "password": "somerandomthing"
  }
]
```

Let's define that!

```js
export const getUsers = {
  tags: ["Users"],
  descripti0on: "Returns all users from the system",
  operationId: "getUsers",
  security: [
    {
      bearerAuth: [],
    },
  ],
  responses: {
    200: {
      description: "A list of users.",
      content: {
        "application/json": {
          schema: {
            type: "array",
            items: {
              id: {
                type: "number",
                description: "User ID",
              },
              name: {
                type: "string",
                description: "User Name",
              },
              email: {
                type: "string",
                description: "User Email",
              },
              password: {
                type: "string",
                description: "User Password",
              },
            },
          },
        },
      },
    },
  },
};
```

The file is pretty much self-explanatory. We are adding some description and the response schema here.
For this path, we are expected to get a response of the following type.

Now let's import that into the `swagger.ts` file and define the path.

```js
import { getUsers } from "./src/open-api/get-users";
export const swaggerDocument = {
  //   ... other configs
  tags: [
    {
      name: "Users",
    },
  ],
  paths: {
    "/auth/users": {
      get: getUsers,
    },
  },
};
```

One last thing that we need to add is the server specification.

```js
import { getUsers } from "./src/open-api/get-users";
export const swaggerDocument = {
  // ... other configs
  servers: [
    {
      url: "http://localhost:4000/api/v1",
      description: "Local server",
    },
    {
      url: "https://your_production_url/api/v1",
      description: "Production Env",
    },
  ],
};
```

Here we are specifying two paths. One is for the local server another is for the production server whose URL we don't know yet!

### Running it

Now run your express server.

```sh
yarn dev
```

And go to the following path `http://localhost:4000/api-docs`

and you will be greeted with swagger documentation.

### Github repo:

https://github.com/Mohammad-Faisal/professional-express-sequelize-docker-boilerplate
