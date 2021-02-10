# Using the CLI to create features in Blitz

## Generate and setup database schema
We're using the CLI to generate all the files needed for out `action` feature.
1) Run `blitz generate all action`. This will generate the form, pages, queries, and mutations we need to add basic CRUD in our feature.
2) Even though the CLI is pretty sweet, it can't know what our database should look like. Open `./db/schema.prisma` and add the following in the `Action` model:
```prisma
model Action {
  id            Int      @default(autoincrement()) @id
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  createdByUser User     @relation("ActionCreatedBy", fields: [createdById], references: [id])
  createdById   Int
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
  createdActions    Action[] @relation("ActionCreatedBy")
}
```
3) Run `blitz prisma migrate dev --preview-feature` to migrate the database!
4) Now start the server `blitz start`
5) Go to [http://localhost:3000/actions](http://localhost:3000/actions) and make sure you get the generated actions page. It doesn't look good, but let's fix that next!

## Styles and cleanup
The generated files are not using Tailwind like it should, so let's beautyfy it:
1) Open `./app/actions/pages/actions/index.tsx`. We don't care about pagination for now, so let's remove that and simplify:
```tsx
import { Suspense } from "react"
import Layout from "app/layouts/Layout"
import { Link, BlitzPage, useQuery } from "blitz"
import getActions from "app/actions/queries/getActions"

const NewAction = () => (
	<Link href="/actions/new">
		<a className="bg-gradient-to-r from-purple-800 to-green-500 hover:from-pink-500 hover:to-green-500 text-white font-bold py-2 px-4 rounded focus:ring transform transition hover:scale-105 duration-300 ease-in-out">
			New action
	  </a>
	</Link>
)

export const ActionsList = () => {
	const [{ actions }] = useQuery(getActions, {
		orderBy: { id: "asc" }
	})

	return (
		<ul>
			<>
				{actions.length ? (
					<>
						{actions.map((action) => (
							<li key={action.id}>
								<Link href={`/actions/${action.id}`}>
									<a>{action.id}</a>
								</Link>
							</li>
						))}
					</>
				) : (
						<li>
							<div className="mb-4">There are no actions yet.</div>
							<NewAction />
						</li>
					)}
			</>

		</ul>
	)
}

const ActionsPage: BlitzPage = () => {
	return (
		<div>
			<div className="flex justify-between mb-10 items-center">
				<h1 className="text-6xl">Actions</h1>
				<NewAction />
			</div>

			<Suspense fallback={<div>Loading...</div>}>
				<ActionsList />
			</Suspense>
		</div>
	)
}

ActionsPage.getLayout = (page) => <Layout title={"Actions"}>{page}</Layout>

export default ActionsPage
```
[All done! Back to section 7](./README.md)
