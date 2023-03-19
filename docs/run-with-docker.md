---
sidebar_position: 6
---

# Run with Docker

It's super simple. If you already have Docker installed and running on your machine you can just run

```sh
docker-compose up
```

It will give you 3 things

1. The Express server in development mode (With hot reloading support)
2. A PostgreSQL database server (If you prefer something else like MySQL just make a couple of change inside the `docker-compose.yaml` file) The credentials are

```sh
DB_HOST = database-layer;
DB_NAME = dbname;
DB_USER = dbuser;
DB_PASSWORD = dbpassword;
```

3. A Database investigation tool named `Adminer` (You can inspect any kind of database from the browser) You can access it from `http://localhost:8080`

If you want to change or update any code you can just make the change and from the console you will see that the server is getting updated.
