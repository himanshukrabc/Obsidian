## ORM 
Object Relational Mapping => basically maps objects to the database tables which simplifies the queries.

## Benefits
1. Syntax is simpler.
2. Abstraction - The same ORM can work on multiple databases.
3. Type safety , AutoCompletion
4. Automatic Migration -> You can change the schema and the ORM will migrate the data.

## Prisma ORM
- ORM for Nodejs
- Data Model => Define the schema in a single file, stating the relations.
- Automatic migrations.

```
npm init
npm i prisma typescript ts-node @types/node --save-dev
npx tsc --init
// change rootDir and outDir
npx prisma init
```

#### 1. Writing the Prisma File
```js
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

// Looking for ways to speed up your queries, or scale easily with your serverless or edge functions?
// Try Prisma Accelerate: https://pris.ly/cli/accelerate-init

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = "postgresql://userDB_owner:jpvwNHOu1A6k@ep-dawn-dream-a5l9kjdh.us-east-2.aws.neon.tech/userDB?sslmode=require"
}

model User{
  id Int @id @default(autoincrement())
  email String @unique
  firstName String
  firstLame String
  password String
}

model Todo{
  id Int @id @default(autoincrement())
  title String
  done Boolean @default(false)
  description String?
  userId Int
}
```

#### 2. Creating Tables from the Prisma file
- Whenever you create/alter a table you need to migrate it to the DB.
- Whenever you create column you need to provide default value.
```
npx prisma migrate dev --name MigrateName
```
- Prisma automatically creates a migrations folder which contains the SQL command that ran to change the tables.

#### 3. Getting the client
- Client gets the objects through which tables can be accessed
```
npx prisma generate
```

```ts
import { PrismaClient } from "@prisma/client";
const prisma = new PrismaClient();

prisma.user.create({
    
});
```

## Relations
```ts
model User {  
id Int @id @default(autoincrement())  
posts Post[]  
}  
  
model Post {  
id Int @id @default(autoincrement())  
author User @relation(fields: [authorId], references: [id])  
authorId Int // relation scalar field (used in the `@relation` attribute above)  
}
```