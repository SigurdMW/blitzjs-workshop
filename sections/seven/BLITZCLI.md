# Using the CLI to create features in Blitz
We're using the CLI to generate all the files needed for out `action` feature.
1) Run `blitz generate all action`. This will generate the form, pages, queries, and mutations we need to add basic CRUD in our feature.
2) Even though the CLI is pretty sweet, it can't know what our database should look like. Open `./db/schema.prisma` and add the following in the `Action` model:
```prisma
model Action {
  id            Int      @default(autoincrement()) @id
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  createdByUser User     @relation("ActionCreatedBy", fields: [createdById], references: [id])
  createdById:  Int
  user		      User	   @relation("AssignedActions", fields: [userId], references: [id])
  userId	      Int
  activity	    Activity @relation(fields: [activityId], references: [id])
  activityId    Int
}
```
And add this on the `User` model:
```prisma
model User {
  id                Int       @default(autoincrement()) @id
  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt
  name              String?
  email             String    @unique
  hashedPassword    String?
  role              String    @default("user")
  sessions          Session[]
  createdActivities	Activity[]
  assignedActions		Action[] @relation("AssignedActions")
  createdActions    Action[] @relateion("ActionCreatedBy")
}
```
3) Run `blitz prisma migrate dev --preview-feature` to migrate the database!

[All done! Back to section 7](./README.md)
