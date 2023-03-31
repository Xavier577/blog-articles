---
title: "How to setup prima to use two different databases in nestjs"
datePublished: Fri Mar 31 2023 22:55:03 GMT+0000 (Coordinated Universal Time)
cuid: clfx57ahj000309jl1qusbwdk
slug: how-to-setup-prima-to-use-two-different-databases-in-nestjs
tags: postgresql, mongodb, databases, nestjs, prisma

---

As the complexity of modern web applications grows, so does the need for scalable and robust database solutions. Prisma makes database access easier by generating a type-safe client for your database, allowing you to easily query and manipulate data. Furthermore, Prisma supports multiple databases, allowing you to work with different database systems, such as MySQL and PostgreSQL, within the same NestJS application at the same time.

In this article, we'll explore how to set up Prisma to use two different databases in NestJS (we would be connecting [postgres](https://www.postgresql.org/) and [mongodb](https://www.mongodb.com/)). You might be wondering why you will want to use two different databases in the same application but this is very common in many of the popular applications you use. In fact, most apps in production make use of a combination of several different databases for different purposes.

# Getting started

We'll be assuming we have postgres, mongodb and nodejs installed going forward.

### Creating our project director

```bash
nest new nestjs-prisma-postgres-mongo-example
```

### Setting up Prisma

> initiate prisma

```bash
npx prisma init
```

This would create the `prisma` directory with a `schema.prisma` file in it and a `.env` file with the `DATABASE_URL` variable in it.

```bash
(output)

✔ Your Prisma schema was created at prisma/schema.prisma
  You can now open it in your favorite editor.

warn You already have a .gitignore file. Don't forget to add `.env` in it to not commit any private information.

Next steps:
1. Set the DATABASE_URL in the .env file to point to your existing database. If your database has no tables yet, read https://pris.ly/d/getting-started
2. Set the provider of the datasource block in schema.prisma to match your database: postgresql, mysql, sqlite, sqlserver, mongodb or cockroachdb.
3. Run npx prisma db pull to turn your database schema into a Prisma schema.4. Run npx prisma generate to generate the Prisma Client. You can then start querying your database.

More information in our documentation:
https://pris.ly/d/getting-started
```

this would create the prisma directory in our project's root directory

```bash
.
├── README.md
├── nest-cli.json
├── package.json
├── prisma
│   └── schema.prisma
├── src
│   ├── app.controller.spec.ts
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── tsconfig.build.json
├── tsconfig.json
├── yarn-error.log
└── yarn.lock
```

> installing `prisma` as a devDependency

we'll install prisma locally to run commons with the prisma cli

```bash
yarn add -D prisma
```

> installing `@prisma/client`

```bash
yarn add @prisma/client
```

That's it's prisma is all set in our nestjs app.

### Setting up postgres and mongodb configurations in our prisma directory

In our prisma directory, we'll create two new folders `mongo` and `postgres` with their respective `schemsa.prisma` files in them (you can delete the default `schema.prisma` file, we won't be using it).

```bash
.
├── README.md
├── nest-cli.json
├── package.json
├── prisma
│   ├── mongo
│   │   └── schema.prisma
│   └── postgres
│       └── schema.prisma
├── src
│   ├── app.controller.spec.ts
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── tsconfig.build.json
├── tsconfig.json
├── yarn-error.log
└── yarn.lock
```

In `prisma/postgres/schema.prisma`

```plaintext

generator client {
    provider = "prisma-client-js"
    output   = "../../node_modules/@prisma/postgres/client"

}
 
datasource db {
    provider = "postgresql"
    url      = env("POSTGRES_DATABASE_URL")

}

model User {
    pk          Int      @id @default(autoincrement())
    id          String   @unique @default(uuid()) @db.Uuid
    email       String   @unique
    phoneNumber String?  @unique @map("phone_number")
    password    String?
    createdAt   DateTime @default(now()) @map("created_at")
    updatedAt   DateTime @updatedAt @map("update_at")
    @@unique([pk, id])
    @@map("users")
}
```

In our schema, we defined:

* A single table (`users`)
    
* Which variable to look for our database url in our `.env` file (which we set to `POSTGRES_DATABASE_URL`)
    
* And where to generate our prisma client (we set this to `node_modules/@prisma/postgres/client`).
    

In `prisma/mongo/prisma.schema`

```plaintext
generator client {
    provider = "prisma-client-js"
    output   = "../../node_modules/@prisma/mongo/client"
}

datasource db {
    provider = "mongodb"
    url      = env("MONGO_DATABASE_URL")
}  

model Image {
    id            String   @id @default(auto()) @map("_id") @db.ObjectId
    base64        String
    fileExtension String
    createdAt     DateTime @default(now())
    updatedAt     DateTime @updatedAt
    @@map("images")
}
```

In our schema, we defined:

* A mongo document (`images`)
    
* Which variable to look for our database url in our `.env` file (which we set to `MONGO_DATABASE_URL`)
    
* And where to generate our prisma client (we set this to `node_modules/@prisma/mongo/client`).
    

### Setting up scripts for prisma in our package.json

```json
{
  "name": "nestjs-prisma-postgres-mongo-example",
  "version": "0.0.1",
  "description": "",
  "author": "",
  "private": true,
  "license": "UNLICENSED",
  "scripts": {
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "postgres:migrate:dev": "prisma migrate dev --schema prisma/postgres/schema.prisma --name ",
    "postgres:migrate:dev:create": "prisma migrate dev --schema prisma/postgres/schema.prisma --create-only",
    "postgres:migrate:deploy": "npx prisma migrate deploy --schema prisma/postgres/schema.prisma --preview-feature",
    "prisma": "prisma",
    "prisma:postgres:dbpush": "npx prisma db push --schema prisma/postgres/schema.prisma",
    "prisma:mongo:dbpush": "npx prisma db push --schema prisma/mongo/schema.prisma",
    "prisma:generate:postgres_client": "prisma generate --schema prisma/postgres/schema.prisma",
    "prisma:generate:mongo_client": "prisma generate --schema prisma/mongo/schema.prisma",
    "prisma:generate:db_clients": "prisma generate --schema prisma/postgres/schema.prisma && prisma generate --schema prisma/mongo/schema.prisma"
  },
  "dependencies": {
    "@nestjs/common": "^9.0.0",
    "@nestjs/core": "^9.0.0",
    "@nestjs/platform-express": "^9.0.0",
    "reflect-metadata": "^0.1.13",
    "rxjs": "^7.2.0"
  },
  "devDependencies": {
    "@nestjs/cli": "^9.0.0",
    "@nestjs/schematics": "^9.0.0",
    "@nestjs/testing": "^9.0.0",
    "@types/express": "^4.17.13",
    "@types/jest": "29.2.4",
    "@types/node": "18.11.18",
    "@types/supertest": "^2.0.11",
    "@typescript-eslint/eslint-plugin": "^5.0.0",
    "@typescript-eslint/parser": "^5.0.0",
    "eslint": "^8.0.1",
    "eslint-config-prettier": "^8.3.0",
    "eslint-plugin-prettier": "^4.0.0",
    "jest": "29.3.1",
    "prettier": "^2.3.2",
    "source-map-support": "^0.5.20",
    "supertest": "^6.1.3",
    "ts-jest": "29.0.3",
    "ts-loader": "^9.2.3",
    "ts-node": "^10.0.0",
    "tsconfig-paths": "4.1.1",
    "typescript": "^4.7.4"
  },
  "jest": {
    "moduleFileExtensions": [
      "js",
      "json",
      "ts"
    ],
    "rootDir": "src",
    "testRegex": ".*\\.spec\\.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "collectCoverageFrom": [
      "**/*.(t|j)s"
    ],
    "coverageDirectory": "../coverage",
    "testEnvironment": "node"
  }
}
```

In `scripts` field of our `package.json` file we added:

* `postgres:migrate:dev`: this would be used to create and execute migrations on our postgres database.
    
* `postgres:migrate:dev:create`: this would create a postgres migration file which you can edit before executing.
    
* `prisma`: this is the prisma cli so we can run commands straight from prisma-cli.
    
* `postgres:migrate:deploy`: this applies pending migrations to staging, testing, or production environments to your postgres database.
    
* `prisma:postgres:dbpush`: this syncs the `prisma/postgres/shema.prisma` with the postgres database.
    
* `prisma:mongo:dbpush`: this syncs the `prisma/mongo/shema.prisma` with the mongo database.
    
* `prisma:generate:mongo_client`: this would generate the prima client for mongo.
    
* `prisma:generate:postgres_client`: this would generate the prisma client for postgres.
    
* `prisma:generate:db_clients`: this would generate the prisma client for postgres and mongo.
    

### Adding our connection urls to our `.env` file

Now we'll add variable to hold our database connection urls to match that of our respective databases `POSTGRES_DATABASE_URL` for postgres and `MONGO_DATABASE_URL` for mongo. (Do replace the respective database urls with your own)

```plaintext
POSTGRES_DATABASE_URL='postgres://postgres:password@localhost:5432/testapp'
MONGO_DATABASE_URL='mongodb://127.0.0.1:27017/testapp'
```

### Running Migrations and Generating Prisma clients

We created our scripts and our respective schemas, it's time to run migrations and generate the prisma clients for both `mongo` and `postgres`.

> Run migrations for postgres

```bash
yarn postgres:migrate:dev dbInit
```

```bash
(output)
yarn run v1.22.19
$ prisma migrate dev --schema prisma/postgres/schema.prisma --name  dbInit
Environment variables loaded from .env
Prisma schema loaded from prisma/postgres/schema.prisma
Datasource "db": PostgreSQL database "testapp", schema "public" at "localhost:5432"

Applying migration `20230331214820_db_init`

The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20230331214820_db_init/
    └─ migration.sql

Your database is now in sync with your schema.

✔ Generated Prisma Client (4.12.0 | library) to ./node_modules/@prisma/postgres/client in 63ms


Done in 3.31s.
```

this should create our migration file in `prisma/postgres/schema.prisma`.

```bash
.
├── README.md
├── nest-cli.json
├── package.json
├── prisma
│   ├── mongo
│   │   └── schema.prisma
│   └── postgres
│       ├── migrations
│       │   ├── 20230331214820_db_init
│       │   │   └── migration.sql
│       │   └── migration_lock.toml
│       └── schema.prisma
├── src
│   ├── app.controller.spec.ts
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── tsconfig.build.json
├── tsconfig.json
├── yarn-error.log
└── yarn.lock
```

> Sync prisma schema for mongo

```bash
yarn prisma:mongo:dbpush
```

```plaintext
(output)
yarn run v1.22.19
$ npx prisma db push --schema prisma/mongo/schema.prisma
Environment variables loaded from .env
Prisma schema loaded from prisma/mongo/schema.prisma
Datasource "db": MongoDB database "testapp" at "127.0.0.1:27017"
Applying the following changes:

[+] Collection `images`


🚀  Your database indexes are now in sync with your Prisma schema. Done in 91ms

✔ Generated Prisma Client (4.12.0 | library) to ./node_modules/@prisma/mongo/client in 63ms

Done in 2.33s.
```

> Generate prisma clients for both mongo and postgres

```bash
yarn prisma:generate:db_clients
```

```bash
(output)
yarn run v1.22.19
$ prisma generate --schema prisma/postgres/schema.prisma && prisma generate --schema prisma/mongo/schema.prisma
Environment variables loaded from .env
Prisma schema loaded from prisma/postgres/schema.prisma

✔ Generated Prisma Client (4.12.0 | library) to ./node_modules/@prisma/postgres/client in 104ms
You can now start using Prisma Client in your code. Reference: https://pris.ly/d/client
\```
import { PrismaClient } from './node_modules/@prisma/postgres/client'
const prisma = new PrismaClient()
\```
Environment variables loaded from .env
Prisma schema loaded from prisma/mongo/schema.prisma

✔ Generated Prisma Client (4.12.0 | library) to ./node_modules/@prisma/mongo/client in 102ms
You can now start using Prisma Client in your code. Reference: https://pris.ly/d/client
\```
import { PrismaClient } from './node_modules/@prisma/mongo/client'
const prisma = new PrismaClient()
\```
Done in 3.07s.
```

this would create our prisma clients in `@prisma/postgres/client` for `postgres` and `@prisma/mongo/client` for mongo. We would use our repective clients to interface with our repective database in our nestjs application.

### Creating a database module in our nestjs app

In our application we would create a database module. We would use nest cli to generate our `database.module`.

```bash
nest g module database
```

```bash
(output)
CREATE src/database/database.module.ts (85 bytes)
UPDATE src/app.module.ts (324 bytes)
```

```bash
.
├── README.md
├── nest-cli.json
├── package.json
├── prisma
│   ├── mongo
│   │   └── schema.prisma
│   └── postgres
│       ├── migrations
│       │   ├── 20230331214820_db_init
│       │   │   └── migration.sql
│       │   └── migration_lock.toml
│       └── schema.prisma
├── src
│   ├── app.controller.spec.ts
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   ├── database
│   │   └── database.module.ts
│   └── main.ts
├── test
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── tsconfig.build.json
├── tsconfig.json
├── yarn-error.log
└── yarn.lock
```

we would then create a `postgres-prisma.service` and `mongo-prisma.service` in `src/database` directory

> Generate `postgres-prisma.service.ts`

```bash
nest g service database/postgres-prisma --flat
```

```bash
(output)
CREATE src/database/postgres-prisma.service.spec.ts (517 bytes)
CREATE src/database/postgres-prisma.service.ts (98 bytes)
UPDATE src/database/database.module.ts (274 bytes)
```

> Generate `mongo-prisma.service.ts`

```bash
nest g service database/mongo-prisma --flat
```

```bash
(output)
CREATE src/database/mongo-prisma.service.spec.ts (496 bytes)
CREATE src/database/mongo-prisma.service.ts (95 bytes)
UPDATE src/database/database.module.ts (272 bytes)
```

```bash
.
├── README.md
├── nest-cli.json
├── package.json
├── prisma
│   ├── mongo
│   │   └── schema.prisma
│   └── postgres
│       ├── migrations
│       │   ├── 20230331214820_db_init
│       │   │   └── migration.sql
│       │   └── migration_lock.toml
│       └── schema.prisma
├── src
│   ├── app.controller.spec.ts
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   ├── database
│   │   ├── database.module.ts
│   │   ├── mongo-prisma.service.spec.ts
│   │   ├── mongo-prisma.service.ts
│   │   ├── postgres-prisma.service.spec.ts
│   │   └── postgres-prisma.service.ts
│   └── main.ts
├── test
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
├── tsconfig.build.json
├── tsconfig.json
├── yarn-error.log
└── yarn.lock
```

> `src/database/database.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { PostgresPrismaService } from './postgres-prisma.service';
import { MongoPrismaService } from './mongo-prisma.service';
  

@Module({
  providers: [PostgresPrismaService, MongoPrismaService],
})

export class DatabaseModule {}
```

> `src/app.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { DatabaseModule } from './database/database.module';

@Module({
  controllers: [AppController],
  providers: [AppService],
  imports: [DatabaseModule],
})

export class AppModule {}
```

### Editing our `PostgresPrismaService` and `MongoPrismaService`

In `src/database/postgres-prisma.service.ts` , We'll edit it's content to add the following.

```typescript
import {
  Injectable,
  OnModuleInit,
  OnModuleDestroy,
  Logger,
} from '@nestjs/common';
import { PrismaClient as PostgresPrismaClient } from '@prisma/postgres/client';


@Injectable()
export class PostgresPrismaService
  extends PostgresPrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  private readonly logger = new Logger(PostgresPrismaService.name);
  async onModuleInit() {
    let retries = 5;
    while (retries > 0) {
      try {
        await this.$connect();
        
        this.logger.log('Successfully connected to postgres database');
        
        break;
      } catch (err) {
        this.logger.error(err);
       
        this.logger.error(
          `there was an error connecting to database, retrying .... (${retries})`,
        );
        
        retries -= 1;
        
        await new Promise((res) => setTimeout(res, 3_000)); // wait for three seconds
      }
    }
  }
  
  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

This is the service we'll use to interract with our postgres database, in the `onModuleInit()` we have implemented retry logic to retry 5 times to connect to our database. We are doing this so if we setup our app to use docker for example we can wait for our database to start up and retry until we can connect.

And in our `src/database/mongo-prisma.service`, we'll add

```typescript
import {
  Injectable,
  OnModuleInit,
  OnModuleDestroy,
  Logger,
} from '@nestjs/common';
import { PrismaClient as MongoPrismaClient } from '@prisma/mongo/client';

  

@Injectable()
export class MongoPrismaService
  extends MongoPrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  private readonly logger = new Logger(MongoPrismaService.name);

  

  async onModuleInit() {
    let retries = 5;

    while (retries > 0) {

      try {
        await this.$connect();
        
        this.logger.log('Successfully connected to mongo database');
        
        break;

      } catch (err) {
        this.logger.error(err);

        this.logger.error(
          `there was an error connecting to database, retrying .... (${retries})`,
        );

        retries -= 1;
        
        await new Promise((res) => setTimeout(res, 3_000)); // wait for three seconds
      }
    }
  }

  

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

Which is basically the same code but for the uses the `MongoPrismaClient`.

### Running the server

It's time for us to run the nestjs server.

```bash
yarn start:dev
```

```bash
(output)
[11:34:58 PM] Starting compilation in watch mode...

[11:35:00 PM] Found 0 errors. Watching for file changes.

[Nest] 29940  - 03/31/2023, 11:35:00 PM     LOG [NestFactory] Starting Nest application...
[Nest] 29940  - 03/31/2023, 11:35:00 PM     LOG [InstanceLoader] DatabaseModule dependencies initialized +22ms
[Nest] 29940  - 03/31/2023, 11:35:00 PM     LOG [InstanceLoader] AppModule dependencies initialized +0ms
[Nest] 29940  - 03/31/2023, 11:35:01 PM     LOG [RoutesResolver] AppController {/}: +20ms
[Nest] 29940  - 03/31/2023, 11:35:01 PM     LOG [RouterExplorer] Mapped {/, GET} route +2ms
[Nest] 29940  - 03/31/2023, 11:35:01 PM     LOG [PostgresPrismaService] Successfully connected to postgres database
[Nest] 29940  - 03/31/2023, 11:35:01 PM     LOG [MongoPrismaService] Successfully connected to mongo database
[Nest] 29940  - 03/31/2023, 11:35:01 PM     LOG [NestApplication] Nest application successfully started +2ms
```

We can see that our app successfully connected to both our postgres and mongdb application repectively. And we'll check our repective databases to confirm that our migrations and changes reflect.

```bash
psql --dbname testapp
```

```bash
(output)

psql (15.2 (Ubuntu 15.2-1.pgdg20.04+1), server 12.14 (Ubuntu 12.14-1.pgdg20.04+1))
Type "help" for help.

testapp=# \d
                 List of relations
 Schema |        Name        |   Type   |  Owner   
--------+--------------------+----------+----------
 public | _prisma_migrations | table    | postgres
 public | users              | table    | postgres
 public | users_pk_seq       | sequence | postgres
(3 rows)
```

```bash
mongo mongodb://127.0.0.1:27017/testapp
```

```bash
(output)

MongoDB shell version v5.0.15
connecting to: mongodb://127.0.0.1:27017/testapp?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("3aaa43fa-14f9-4fe2-a115-3c123a96629f") }
MongoDB server version: 5.0.15
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
For installation instructions, see
https://docs.mongodb.com/mongodb-shell/install/
================
---
The server generated these startup warnings when booting: 
        2023-03-31T22:58:22.685+01:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
        2023-03-31T22:58:23.429+01:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
        2023-03-31T22:58:23.429+01:00: /sys/kernel/mm/transparent_hugepage/enabled is 'always'. We suggest setting it to 'never'
---
---
        Enable MongoDB's free cloud-based monitoring service, which will then receive and display
        metrics about your deployment (disk utilization, CPU, operation statistics, etc).

        The monitoring data will be available on a MongoDB website with a unique URL accessible to you
        and anyone you share the URL with. MongoDB may use this information to make product
        improvements and to suggest MongoDB products and deployment options to you.

        To enable free monitoring, run the following command: db.enableFreeMonitoring()
        To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
> db.testapp
testapp.testapp
```

# Conclusion

In summary, learning how to set up Prisma to use two different databases in NestJS may seem like a challenging task, but with a few steps, we were able to accomplish it. It's essential to understand the differences between the databases and how they function in your application. By configuring and implementing the Prisma Client properly like we did in this article, NestJS can work with multiple databases without any issues.

# References

* [example-repo](https://github.com/Xavier577/nestjs-prisma-postgres-mongo-example)
    
* [prisma-client](https://www.prisma.io/docs/concepts/components/prisma-client)
    
* [prisma-cli](https://www.prisma.io/docs/concepts/components/prisma-cli)
    
* [nestjs-prisma-recipe](https://docs.nestjs.com/recipes/prisma)
    
* [prisma-mongodb-connector](https://www.prisma.io/docs/concepts/database-connectors/mongodb)
    
* [prisma-postgresql-connector](https://www.prisma.io/docs/concepts/database-connectors/postgresql)