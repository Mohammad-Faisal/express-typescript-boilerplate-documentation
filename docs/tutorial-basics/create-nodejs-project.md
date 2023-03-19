---
sidebar_position: 1
---

# Create a NodeJS Project

### How to create NodeJS project from scratch with hot-reloading support

Creating a NodeJS project from scratch is often something we don't do that much. But how everything works on the backend is critical to know as a senior software engineer.

Let's learn by creating a boilerplate NodeJS project with Typescript. And also, know how we can add development mode with Typescript!

And we will do it in two ways.

### Step 1: Create a new project

First, create a new directory for your project.

```
mkdir nodejs-typescript-skeleton
cd nodejs-typescript-skeleton
```

### Step 2: Initialize an npm project

Then run the following command to initialize the project with yarn. You can use npm if you want. That is not that different.

```
yarn init -y
```

### Step 3: Create an entry to our project

Then create a new folder named `src`, and inside that, create a new file named **index.js**

Paste the following code directly into that.

```
const name = 'faisal'
console.log('this is working');
```

Now run the project from the console to see if it's working or not.

```sh
node src/index.js
```

It will print the message, which means our project is up and running.

### Step 4: Let's introduce Typescript

But we are not living in the past anymore, right? And we don't want javascript anymore. So let's convert the file into Typescript.

Rename our **index.js** file into **index.ts** . Then install Typescript locally (as a dev dependency).

```
yarn add -D typescript
```

Now we have Typescript, which is great, but it doesn't know how to work properly. For that, we need a configuration file.

Let's create a typescript configuration file in the root folder named **tsconfig.json**. And paste the following code there.

```json
{
  "extends": "@tsconfig/node16/tsconfig.json",
  "compilerOptions": {
    "outDir": "dist"
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

A few things to notice here,

1. We are extending a default configuration for `nodejs 16` in the first line. So you have to install that as well.

```sh
yarn add -D @tsconfig/node16
```

The same configuration is also[ available for other nodejs versions](https://github.com/tsconfig/bases) like 14 and 12.

2. we are configuring _outDir_ as `dist`, which means our bundled files will go into a new directory named `dist`
3. We only specify the `src` folder to be compiled. So all our typescript files will go there.
4. We are excluding the `node_modules` folder from the compilation to avoid issues and improve the build time.

Lastly, let's include the type definition for `NodeJS` by running the command.

```sh
yarn add -D @types/node
```

### Step 5: Let's compile

So now we have set up the Typescript. But our browser or NodeJS doesn't know anything about Typescript. They only understand Javascript right?

So what we need to do is convert our Typescript files into Javascript. The process is called Compiling.

Let's do that.

```
yarn tsc
```

Now go inside your `dist` folder to see the compiled files. You will see a Javascript version of the `index.ts` file there!

Then execute your code by running the following

```js
node dist/index.js
```

And boom! Your code is working just fine.

### Step 6: Executing a Typescript file directly

It's not practical to run the `tsc` command before running our application. We need a way to automate the process.

And there is a package to solve this problem! Let's add that!

```sh
yarn add -D ts-node
```

Now **ts-node** is a package that can be used to run any typescript file directly. So let's try first.

```sh
ts-node src/index.ts
```

It will directly execute the typescript nodejs file. No compilation is required! It will all happen automatically.

### Step 7: Development Mode

Now we can run a typescript file directly without compiling to javascript. Let's take it a bit further.
We need hot reloading support for the project during development.

Let's add a popular package to solve that.

```sh
yarn add -D nodemon
```

`nodemon` is a package that can watch the file changes and compile them when you save them so that you don't have to run the file again and again.

Let's try to run that.

```sh
yarn nodemon src/index.ts
```

Now go ahead and make a change inside your `index.ts` file to see the real-time changes.

### Bonus Step: Make development even more pleasant

There is another package that is specifically made for a typescript project. its **ts-node-dev** package.

Let's add that to the project and see what we can do.

```sh
yarn remove nodemon ts-node  // ->remove the previous 2 packages

yarn add -D ts-node-dev // -> add the new one!
```

Now let's run the following command and see if that works as expected.

```sh
yarn ts-node-dev --respawn src/index.ts
```

Note that the `--respawn` flag is given to ensure that the files are re-compiled when changed.

`ts-node-dev` is better than `nodemon,` and it's faster as well.

### Final Improvement: Add script

Now go to your **package.json** file and add the following script to make it easier.

```json
 "scripts": {
    "dev": "ts-node-dev --respawn src/index.ts",
    "build": "tsc"
  },
```

Then you can just run the following command to get it running in development mode.

```sh
yarn dev
```

And to build the project, just run the following command.

```sh
yarn build
```

Now you have a working nodejs skeleton project! You can use this any way you like.

### Github Repo:

https://github.com/Mohammad-Faisal/nodejs-typescript-skeleton
