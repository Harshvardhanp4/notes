# Prisma + PostgreSQL Quickstart (TypeScript + ESM)

## 1. Create a New Project

```bash
mkdir hello-prisma
cd hello-prisma
Initialize a TypeScript project
npm init -y
npm install typescript tsx @types/node --save-dev
npx tsc --init
2. Install Required Dependencies
npm install prisma @types/pg --save-dev
npm install @prisma/client @prisma/adapter-pg pg dotenv
Package Explanation
prisma → CLI for commands like init, migrate, generate
@prisma/client → Prisma Client to query DB
@prisma/adapter-pg → Adapter to connect Prisma with PostgreSQL
pg → PostgreSQL driver
@types/pg → TypeScript types for pg
dotenv → Load environment variables
3. Configure ESM Support
tsconfig.json
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
package.json
{
  "type": "module"
}
4. Initialize Prisma ORM
npx prisma
npx prisma init --datasource-provider postgresql --output ../generated/prisma
This command:
Creates prisma/schema.prisma
Creates .env
Creates prisma.config.ts
prisma.config.ts
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
prisma/schema.prisma (initial)
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}
.env
DATABASE_URL="postgresql://username:password@localhost:5432/mydb?schema=public"

Replace with:

username → your DB username
password → your DB password
localhost:5432 → host & port
mydb → database name
5. Define Your Data Model

Update prisma/schema.prisma:

generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}

datasource db {
  provider = "postgresql"
}

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
6. Run Migration & Generate Client
npx prisma migrate dev --name init
npx prisma generate
7. Instantiate Prisma Client
lib/prisma.ts
import "dotenv/config";
import { PrismaPg } from "@prisma/adapter-pg";
import { PrismaClient } from "../generated/prisma/client";

const connectionString = `${process.env.DATABASE_URL}`;

const adapter = new PrismaPg({
  connectionString,
});

const prisma = new PrismaClient({
  adapter,
});

export { prisma };
8. Write Your First Query
script.ts
import { prisma } from "./lib/prisma";

async function main() {
  // Create a user with a post
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
    include: {
      posts: true,
    },
  });

  console.log("Created user:", user);

  // Fetch all users
  const allUsers = await prisma.user.findMany({
    include: {
      posts: true,
    },
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
Run the Script
npx tsx script.ts
Prisma Studio (GUI for DB)
npx prisma studio
