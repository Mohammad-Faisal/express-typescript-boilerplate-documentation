---
sidebar_position: 4
---

# Environment Handling

### Setup multiple environments for NodeJS application

For the production-level applications, we need support for multiple environments. For example, you will need to use different databases for different environments. There are many uses, and this is just one of them.

Today we will see how we can manage multiple environments in a NodeJS application.

### Traditional way

The most straightforward way to use environment variables is using our **package.json** file.
Let's create two commands just to demonstrate two environments.

```json
"scripts": {
    "start-dev": "PORT=3000 ts-node-dev --respawn src/index.ts",
    "start-prod": "PORT=4000  ts-node-dev --respawn src/index.ts"
  }
```

We have defined two commands. For development, we have set `start-dev` where the PORT variable should have a value of 3000. And for prod it will be 4000.

If we run our application

```sh
yarn start-dev
```

You will get the following message.

```JSON
Connected successfully on port 3000
```

and running **start-prod** will give you

```sh
Connected successfully on port 4000
```

So we've successfully made our application configurable.

### Let's take it one step further

So what if we need another variable that will change in different environments? We can just append another variable inside that string.

```json
"scripts": {
    "start-dev": "PORT=3000 SOMETHING_ELSE=dev ts-node-dev --respawn src/index.ts",
    "start-prod": "PORT=4000 SOMETHING_ELSE=dev ts-node-dev --respawn src/index.ts"
  }
```

Here `SOMETHING_ELSE` will be accessible via `process.env.SOMETHING_ELSE`

But in real life, we have many environments that need to be changed. How do we manage that?

### Let's create an environment file

TO manage many environment variables, we can use an environment file. Head over to your project and create a **.env** file and paste the following there.

```json
PORT=3000
SOMETHING_ELSE=dev
```

And remove the environments from the **package.json** commands.

```json
"scripts": {
    "start-dev": "ts-node-dev --respawn src/index.ts",
    "start-prod": "ts-node-dev --respawn src/index.ts",
  },
```

And let's run!

```sh
yarn start-dev
```

This will give the following output.

```sh
Connected successfully on port 5000
```

Now that's weird. Because we are supposed to see 3000 as the port, but it's giving us 5000.

The reason is although we have added the environment variables, NodeJS is not going to read them automatically.

### Let's read our environment variables

We have a very popular package to do that for us. The name is [dotenv](https://www.npmjs.com/package/dotenv)

Let's install that first.

```sh
yarn add dotenv
```

And import it inside our application like the following.

```js
import "dotenv/config";
```

or like this

```js
import dotenv from "dotenv";
const configuration: any = dotenv.config().parsed;
```

And that's enough. You don't need to do anything else to read the environment files. This package will make the variables available for our application via **process.env**

To test that, you can run.

```sh
yarn start-dev
```

And you will see the application running on PORT 3000 again.

### Multiple Environment

Let's make our application production ready now. Let's create two separate files for separate environments.

Create files according to environment like **.env.development** and **.env.production**

Now we need to load the files during the bootup. Windows environments sometimes face issues with loading the environments. To take care of that, let's install a package named [**cross-env** ](https://www.npmjs.com/package/cross-env)

```sh
yarn add -D cross-env
```

Then create separate scripts for those inside **package.json**

```json
"dev": "cross-env NODE_ENV=development ts-node-dev --respawn src/index.ts",
"prod": "cross-env NODE_ENV=production ts-node-dev --respawn src/index.ts",
```

Then modify the config file like this:

```js
import dotenv from "dotenv";
dotenv.config({ path: __dirname + `/../../.env.${process.env.NODE_ENV}` }); // change according to your need

const config = {
  port: process.env.APPLICATION_PORT,
  dbUrl: process.env.DB_URL,
  dbPassword: process.env.DB_PASSWORD,
};

export default config;
```

And we can now use the configuration everywhere in our application.

```js
import Config from "./utils/Configuration";

console.log(Config.port);
```

That's it! Now you can have as many environments as you like and handle them easily.

### Final words

Don't forget to add the **.env** files to your **.gitignore** files. Otherwise, all your secrets will be exposed.

Open your **.gitignore** file and add the following

```sh
.env.development
.env.production
```

That's it.

### Github Repo

https://github.com/Mohammad-Faisal/nodejs-environment-handling
