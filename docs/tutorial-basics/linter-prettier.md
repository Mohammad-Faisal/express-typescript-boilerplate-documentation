---
sidebar_position: 2
---

# Setup Eslint Prettier and Husky

### Setup code quality tools in a painless way in typescript

Having Typescript in a NodeJS project is a great way to ensure type safety, and it also improves the developer experience a lot. But we can take this even further by adding Linter and Formatter.

Today we will see how we can add linter and prettier to a NodeJS project using Typescript.

We are continuing this project from the previous article. The repository can be found [here](https://github.com/Mohammad-Faisal/nodejs-typescript-skeleton)

### Step 1: Install required dependencies

Let's first install the required dependencies.

```sh
yarn add -D eslint \
@typescript-eslint/eslint-plugin \
@typescript-eslint/parser
```

Now Eslint doesn't have typescript support by default, so we add two additional packages as a developer dependency.

### Step 2: Create a config file

Now create a new eslint config file.

```sh
touch .eslintrc
```

And paste the following code there.

```json
{
  "parser": "@typescript-eslint/parser",
  "parserOptions": { "ecmaVersion": "latest", "sourceType": "module" },
  "extends": ["plugin:@typescript-eslint/recommended"],
  "env": {
    "node": true // Enabling Node.js global variables
  },
  "rules": {}
}
```

Let's add two scripts in our **package.json** file to lint and format easily.

```json
"scripts": {
    "lint": "eslint src/**/*.ts",
    "format": "eslint src/**/*.ts --fix"
}
```

Now you can run the following command to lint the whole project.

```sh
yarn lint
```

Also, don't forget to create a `.eslintignore` file to omit some folders from being linted. As that can take up some time.

```sh
.eslintignore
```

and put the following code there

```
node_modules
dist
```

### Step 3: Add Prettier

Let's now add `prettier` to the project. First, install the dependency.

```sh
yarn add -D prettier
```

then create a `.prettierrc` file in the root folder and add the following configuration there

```json
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "printWidth": 120,
  "tabWidth": 2
}
```

Now, this configuration is just my personal preference. You can use your configuration if you want.

Also, you can add a new script for running prettier on all files at once

```json
"scripts": {
    "pretty": "prettier --write \"src/**/*.ts\""
}
```

and run prettier for all files like the following

```sh
yarn pretty
```

### Step 4: Add Husky

Now all these are just awesome. But I always forget to run these commands. To ensure that no bad code is going to be pushed to the source control, we can use a great tool called **husky**

It will run before you try to make a commit and can run some health checks. To understand better, let's first install it.

```sh
yarn add -D husky
```

Then add a new block in the `package.json` file

```json
"husky": {
    "hooks": {
      "pre-commit": "yarn lint"
    }
}
```

So every time you try to commit your changes, this pre-commit hook will run, and your code will be fixed automatically. How cool is that!

### Github repo:

https://github.com/Mohammad-Faisal/nodejs-typescript-skeleton
