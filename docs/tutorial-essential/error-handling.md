---
sidebar_position: 1
---

# Error Handling

### Handle errors like a pro using all the best practices

Handling errors are one of the most important aspects of any production grade application. Anyone can code for the success cases. Only true professionals take care of the error cases.

Today we will learn just that. Let's dive in.

First, we have to understand that not all errors are the same. Let's see how many types of errors can occur in an application.

- User Generated Error
- Hardware failure
- Runtime Error
- Database Error

We will see how we can easily handle these different types of errors.

### Get a basic express application

Run the following command to get a basic express application built with typescript.

```sh
git clone https://github.com/Mohammad-Faisal/express-typescript-skeleton.git
```

### Handle not found URL errors

How do you detect if a hit URL is not active in your express application? You have an URL like `/users,` but someone is hitting `/user.` We need to inform them that the URL they are trying to access does not exist.

That's easy to do in ExpressJS. After you define all the routes, add the following code to catch all unmatched routes and send back a proper error response.

```js
app.use("*", (req: Request, res: Response) => {
  const err = Error(`Requested path ${req.path} not found`);
  res.status(404).send({
    success: false,
    message: "Requested path ${req.path} not found",
    stack: err.stack,
  });
});
```

Here we are using "\*" as a wildcard to catch all routes that didn't go through our application.

### Handle all errors with a special middleware

Now we have a special middleware in Express that handles all the errors for us. We have to include it at the end of all the routes and pass down all the errors from the top level so that this middleware can handle them for us.

The most important thing to do is keep this middleware after all other middleware and route definitions because otherwise, some errors will slip away.

Let's add it to our index file.

```js
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  const statusCode = 500;
  res.status(statusCode).send({
    success: false,
    message: err.message,
    stack: err.stack,
  });
});
```

Have a look at the middleware signature. Unline other middlewares, This special middleware has an extra parameter named `err`, which is of the `Error` type. This comes as the first parameter.

And modify our previous code to pass down the error like the following.

```js
app.use("*", (req: Request, res: Response, next: NextFunction) => {
  const err = Error(`Requested path ${req.path} not found`);
  next(err);
});
```

Now, if we hit a random URL, something like `http://localhost:3001/posta`, then we will get a proper error response with the stack.

```json
{
  "success": false,
  "message": "Requested path ${req.path} not found",
  "stack": "Error: Requested path / not found\n    at /Users/mohammadfaisal/Documents/learning/express-typescript-skeleton/src/index.ts:23:15\n"
}
```

### Custom error object

Let's have a closer look at the NodeJS provided default error object.

```js
interface Error {
  name: string;
  message: string;
  stack?: string;
}
```

So when you are throwing an error like the following.

```js
throw new Error("Some message");
```

Then you are only getting the name and the optional `stack` properties with it. This stack provides us with info on where exactly the error was produced. We don't want to include it in production. We will see how to do that later.

But we may want to add some more information to the error object itself.

Also, we may want to differentiate between various error objects.

Let's design a basic Custom error class for our application.

```js
export class ApiError extends Error {
  statusCode: number;
  constructor(statusCode: number, message: string) {
    super(message);

    this.statusCode = statusCode;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

Notice the following line.

```js
Error.captureStackTrace(this, this.constructor);
```

It helps to capture the stack trace of the error from anywhere in the application.

In this simple class, we can append the `statusCode` as well.
Let's modify our previous code like the following.

```js
app.use("*", (req: Request, res: Response, next: NextFunction) => {
  const err = new ApiError(404, `Requested path ${req.path} not found`);
  next(err);
});
```

And take advantage of the new `statusCode` property in the error handler middleware as well

```js
app.use((err: ApiError, req: Request, res: Response, next: NextFunction) => {
  const statusCode = err.statusCode || 500; // <- Look here

  res.status(statusCode).send({
    success: false,
    message: err.message,
    stack: err.stack,
  });
});
```

Having a custom-defined Error class makes your API predictable for end users to use. Most newbies miss this part.

### Let's handle application errors

Now let's throw a custom error from inside our routes as well.

```js
app.get(
  "/protected",
  async (req: Request, res: Response, next: NextFunction) => {
    try {
      throw new ApiError(401, "You are not authorized to access this!"); // <- fake error
    } catch (err) {
      next(err);
    }
  }
);
```

This is an artificially created situation where we need to throw an error. The real life, we may have many situations where we need to use this kind of **try/catch** block to catch errors.

If we hit the following URL `http://localhost:3001/protected`, we will get the following response.

```json
{
  "success": false,
  "message": "You are not authorized to access this!",
  "stack": "Some details"
}
```

So our error response is working correctly!

### Let's improve on this!

So we now can handle our custom errors from anywhere in the application. But it requires a try catch block everywhere and requires calling the `next` function with the error object.

This is not ideal. It will make our code look bad in no time.

Let's create a custom wrapper function that will capture all the errors and call the next function from a central place.

Let's create a wrapper utility for this purpose!

```js
import { Request, Response, NextFunction } from "express";

export const asyncWrapper =
  (fn: any) => (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch((err) => next(err));
  };
```

And use it inside our router.

```js
import { asyncWrapper } from "./utils/asyncWrapper";

app.get(
  "/protected",
  asyncWrapper(async (req: Request, res: Response) => {
    throw new ApiError(401, "You are not authorized to access this!");
  })
);
```

Run the code and see that we have the same results. This helps us to get rid of all try/catch blocks and call the next function everywhere!

### Example of a custom error

We can fine-tune our errors to our needs. Let's create a new error class for the not found routes.

```js
export class NotFoundError extends ApiError {
  constructor(path: string) {
    super(404, `The requested path ${path} not found!`);
  }
}
```

And simplify our bad route handler.

```js
app.use((req: Request, res: Response, next: NextFunction) =>
  next(new NotFoundError(req.path))
);
```

How clean is that?

Now let's install a small little package to avoid writing the status codes ourselves.

```sh
yarn add http-status-codes
```

And add the status code in a meaningful way.

```js
export class NotFoundError extends ApiError {
  constructor(path: string) {
    super(StatusCodes.NOT_FOUND, `The requested path ${path} not found!`);
  }
}
```

And inside our route like this.

```js
app.get(
  "/protected",
  asyncWrapper(async (req: Request, res: Response) => {
    throw new ApiError(
      StatusCodes.UNAUTHORIZED,
      "You are not authorized to access this!"
    );
  })
);
```

It just makes our code a bit better.

### Handle programmer errors.

The best way to deal with programmer errors is to restart gracefully. Place the following line of code at the end of your application. It will be invoked in case something is not caught in the error middleware.

```js
process.on("uncaughtException", (err: Error) => {
  console.log(err.name, err.message);
  console.log("UNCAUGHT EXCEPTION! 💥 Shutting down...");

  process.exit(1);
});
```

### Handle unhandled promise rejections.

We can log the reason for the promise rejection. These errors never make it to our express error handler. For Example, if we want to access a database with the wrong password.

```js
process.on("unhandledRejection", (reason: Error, promise: Promise<any>) => {
  console.log(reason.name, reason.message);
  console.log("UNHANDLED REJECTION! 💥 Shutting down...");
  process.exit(1);
  throw reason;
});
```

### Further improvement

Let's create a new ErrorHandler class to handle the errors in a central place.

```js
import { Request, Response, NextFunction } from "express";
import { ApiError } from "./ApiError";

export default class ErrorHandler {
  static handle = () => {
    return async (
      err: ApiError,
      req: Request,
      res: Response,
      next: NextFunction
    ) => {
      const statusCode = err.statusCode || 500;
      res.status(statusCode).send({
        success: false,
        message: err.message,
        rawErrors: err.rawErrors ?? [],
        stack: err.stack,
      });
    };
  };
}
```

This is just a simple error handler middleware. You can add your custom logic here.
And use it inside our index file.

```js
app.use(ErrorHandler.handle());
```

That's how we can separate the concerns by respecting the single responsibility principle of SOLID.

I hope you learned something new today. Have a wonderful rest of your day!

### Github Repo:

https://github.com/Mohammad-Faisal/nodejs-expressjs-error-handling
