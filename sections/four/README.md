# Section Four - Database setup
Since Blitz is using [`prisma`](https://www.prisma.io/) as ORM for the database, we have a `schema.prisma` file in our Blitz app. This is the file that contains the schema that is the basis for both our database structure and the generated Typescript types. Let's go ahead and update it.

## Update Prisma Schema
Open file `./db/schema.prisma`. Replace the content of the file with the following:
```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

// --------------------------------------

model User {
  id             Int       @default(autoincrement()) @id
  createdAt      DateTime  @default(now())
  updatedAt      DateTime  @updatedAt
  name           String?
  email          String    @unique
  hashedPassword String?
  role           String    @default("user")
  sessions       Session[]
  actions		 Action[]
}

model Session {
  id                 Int       @default(autoincrement()) @id
  createdAt          DateTime  @default(now())
  updatedAt          DateTime  @updatedAt
  expiresAt          DateTime?
  handle             String    @unique
  user               User?     @relation(fields: [userId], references: [id])
  userId             Int?
  hashedSessionToken String?
  antiCSRFToken      String?
  publicData         String?
  privateData        String?
}

model Activity {
  id        Int      @default(autoincrement()) @id
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  points    Int
  name		String
}

model Action {
  id        Int      @default(autoincrement()) @id
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  user		User	 @relation(fields: [userId], references: [id])
  userId	Int
  activity	Activity	@relation(fields: [activityId], references: [id])
  activityId Int
}
```

## Explanation of schema
* `activity`- is an activity that can give points. May users can perform the same activity to get points.
* `action`  - an action is an activity performed by a user.
In the end-solution, an admin will be able to create activities and create actions for users (hopefully users that deserves points).

## Run the generation
Run `blitz prisma migrate dev --preview-feature` to run the database migration. This performs the needed changes to our database and generates the typescript files. When prompted for name, just hit enter (just using the default name).

Start the server with `blitz start` to verify that no errors occur.