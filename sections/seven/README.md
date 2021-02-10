# Section 7 - Database setup
Since Blitz is using [`prisma`](https://www.prisma.io/) as ORM for the database, we have a `schema.prisma` file in our Blitz app. The file contains the schema that is the basis for both our database structure and the generated Typescript types. Let's go ahead and see what we are going to make:

## Explanation of schema
In the end-solution, an admin will be able to create activities. An activity has a name, the points the activity is rewarded with, and an optional description.
Here is an example of an activity: 
```json
{
	"id": 1,
	"createdAt": "2021-02-10T13:08:38.402Z",
  	"updatedAt": "2021-02-10T13:08:38.403Z",
	"name": "Held a workshop",
	"createdById": 2,
	"description": "",
	"points": 120
}
```
An action is what connects a user and an activity.
Here is an example of an action:
```json
{
	"id": 1,
	"createdAt": "2021-02-10T13:08:38.402Z",
  	"updatedAt": "2021-02-10T13:08:38.403Z",
	"createdById": 1,
	"userId": 2,
	"activityId": 3
}
```

We'll lookat two different ways of adding features to the app. With a feature, I mean database schema, queries, mutations and pages needed for a specific functionality.

There are two ways we'll have a look at: 
1) Manual (good old - create all the files yourself)
2) Using the [Blitz CLI generate option](https://blitzjs.com/docs/cli-generate)

## The manual way
[Let's create the feature for `activity` in a manual way](./MANUAL.md)

## Using the Blitz CLI
[Let's create the feature for `action` using the Blitz CLI](./BLITZCLI.md)

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
  assignedActions		 Action[] @relation("AssignedActions")
  createdActivities	Activity[] @relation("ActivityCreatedBy")
  createdActions Action[] @relateion("ActionCreatedBy")
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
  createdByUser	User		@relation("ActivityCreatedBy", fields: [createdById], references: [id])
  createdById	Int
  points    Int
  name		String
  description	String?
}

model Action {
  id        Int      @default(autoincrement()) @id
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  createdByUser User @relation("ActionCreatedBy", fields: [createdById], references: [id])
  createdById: Int
  user		User	 @relation("AssignedActions", fields: [userId], references: [id])
  userId	Int
  activity	Activity	@relation(fields: [activityId], references: [id])
  activityId Int
}
```



## Run the generation
Run `blitz prisma migrate dev --preview-feature` to run the database migration. This performs the needed changes to our database and generates the typescript files. When prompted for name, just hit enter (just using the default name).

Start the server with `blitz start` to verify that no errors occur.

Pritty sweet, right? [Continue to next section](../eight)
