# Prisma + PostgreSQL Quickstart (TypeScript + ESM)

## 1. Create Project + Setup

``` bash
mkdir hello-prisma
cd hello-prisma
npm init -y
npm install typescript tsx @types/node --save-dev
npx tsc --init
```

## 2. Install Dependencies

``` bash
npm install prisma @types/pg --save-dev
npm install @prisma/client @prisma/adapter-pg pg dotenv
```

## 3. Configure ESM

### tsconfig.json

``` json
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler",
    "target": "ES2023",
    "strict": true,
    "esModuleInterop": true,
    "ignoreDeprecations": "6.0"
  }
}
```

### package.json

``` json
{
  "type": "module"
}
```

## 4. Initialize Prisma

``` bash
npx prisma
npx prisma init --datasource-provider postgresql --output ../generated/prisma
```

### prisma.config.ts

``` ts
import "dotenv/config";
import { defineConfig, env } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
  },
  datasource: {
    url: env("DATABASE_URL"),
  },
});
```

### prisma/schema.prisma

``` prisma
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}
```

### .env

``` env
DATABASE_URL="postgresql://username:password@localhost:5432/mydb?schema=public"
```

## 5. Define Models

``` prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int     @id @default(autoincrement())
  title     String
  content   String?
  published Boolean @default(false)

  author    User @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

## 6. Migrate & Generate

``` bash
npx prisma migrate dev --name init
npx prisma generate
```

## 7. Prisma Client Setup

``` ts
import "dotenv/config";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "../generated/prisma/client";

const connectionString = `${process.env.DATABASE_URL}`;

const adapter = new PrismaPg({ connectionString });

const prisma = new PrismaClient({ adapter });

export { prisma };
```

## 8. First Query

``` ts
import { prisma } from "./lib/prisma";

async function main() {
  const user = await prisma.user.create({
    data: {
      name: "Alice",
      email: "alice@prisma.io",
      posts: {
        create: {
          title: "Hello World",
          content: "This is my first post!",
          published: true,
        },
      },
    },
    include: { posts: true },
  });

  console.log("Created user:", user);

  const allUsers = await prisma.user.findMany({
    include: { posts: true },
  });

  console.log("All users:", JSON.stringify(allUsers, null, 2));
}

main()
  .then(async () => {
    await prisma.$disconnect();
  })
  .catch(async (e) => {
    console.error(e);
    await prisma.$disconnect();
    process.exit(1);
  });
```

## 9. Run

``` bash
npx tsx script.ts
```

## 10. Prisma Studio

``` bash
npx prisma studio
```
