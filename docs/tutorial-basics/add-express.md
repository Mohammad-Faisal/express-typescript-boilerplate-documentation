---
sidebar_position: 3
---

# Install Express

### Bare minimum to get started with a backend API

ExpressJS is the most popular framework for NodeJS applications. It's very versatile and offers no restrictions. So you can do whatever you want and design your application however you like.

You can refer to the following article if you want a boilerplate without ExpressJS support.

https://www.mohammadfaisal.dev/blog/create-nodejs-typescript-boilerplate

### Get the NodeJS boilerplate

Let's first clone the [boilerplate repository](https://github.com/Mohammad-Faisal/nodejs-typescript-skeleton) where we have a working NodeJS application with Typescript, EsLint, and Prettier already set up.
We will integrate Express on top of this.

```sh
git clone https://github.com/Mohammad-Faisal/nodejs-typescript-skeleton.git
```

### Install the dependencies

Then go inside the project and install the dependencies.

```sh
cd nodejs-typescript-skeleton
yarn add Express
```

And also, add the type definitions for the express module.

```sh
yarn add -D @types/express
```

### Test it out!

Go inside the **src/index.ts** file, import the dependencies, and create a basic express application.

```js
import express, { Application, Request, Response } from "express";

const app: Application = express();
```

Then let's create a basic route that will accept a GET request and return a result.

```js
app.get("/", async (req: Request, res: Response): Promise<Response> => {
  return res.status(200).send({
    message: "Hello World!",
  });
});
```

Then start the server on port 3000;

```js
const PORT = 3000;

try {
  app.listen(PORT, (): void => {
    console.log(`Connected successfully on port ${PORT}`);
  });
} catch (error: any) {
  console.error(`Error occured: ${error.message}`);
}
```

And then run the application.

```sh
yarn dev
```

Go ahead and hit the following URL `http://localhost:3000/`, and you should be greeted with the following response.

```json
{ "message": "Hello World!" }
```

### Add Bodyparser

Now, this is the bare minimum to get started with Express. But in real life, we need a couple of things to get the server working properly.

to handle HTTP POST requests in Express, we need to install a middleware module [**body-parser**](https://www.npmjs.com/package/body-parser)

Let's install it first.

```sh
yarn add body-parser
yarn add -D @types/body-parser
```

Then use it inside the **index.ts** file.

```js
import bodyParser from "body-parser";

const app: Application = express();

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
```

And add another route to handle HTTP POST requests.

```js
app.post("/post", async (req: Request, res: Response): Promise<Response> => {
  console.log(req.body);
  return res.status(200).send({
    message: "Hello World from post!",
  });
});
```

You will notice that inside the route handler, we can get the body of the request by

```js
req.body;
```

It's possible because of the use of **body-parser**

### Conclusion

That's it. Now you have a basic express application that can expect HTTP requests.

### Github Repository

https://github.com/Mohammad-Faisal/express-typescript-skeleton
