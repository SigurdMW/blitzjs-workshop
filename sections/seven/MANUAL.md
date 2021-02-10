# Manually creating features in Blitz
## Update database schema
1) Open file `./db/schema.prisma`. Add this to the bottom of the file: 
```prisma
	model Activity {
		id          Int      @default(autoincrement()) @id
		createdAt   DateTime @default(now())
		updatedAt   DateTime @updatedAt
		createdBy	User	 @relation(fields: [createdById], references: [id])
		createdById	Int
		points      Int
		name		String
		description	String?
	}
```
and replace the `User` model in the same file with this:
```prisma
	model User {
		id             		Int       @default(autoincrement()) @id
		createdAt      		DateTime  @default(now())
		updatedAt     		DateTime  @updatedAt
		name           		String?
		email          		String    @unique
		hashedPassword 		String?
		role           		String    @default("user")
		sessions        	Session[]
		createdActivities	Activity[]
	}
```
2) Run `blitz prisma migrate dev --preview-feature` to change the database.

## Make feature folder in `app`
1) Create a folder called `activities` in `./app`. This is where we'll group all code related to our feature.
2) Create a folders so that you get this file: `./app/activities/pages/activities/index.tsx`
3) Add the following content in the file. The IDE will scream (we haven't created the `getActivities` yet - that's next step):
```tsx
iimport { Suspense } from "react"
import Layout from "app/layouts/Layout"
import { Link, useQuery, BlitzPage } from "blitz"
import getActivities from "app/activities/queries/getActivities"

export const ActivityList = () => {  
  const [activities] = useQuery(getActivities, {
    orderBy: { id: "asc" }
  })

  return (
      <ul>
		{activities.length ? (
			<>
				 {activities.map((activity) => (
					<li key={activity.id}>
						<Link href={`/activities/${activity.id}`}>
						<a>{activity.name}</a>
						</Link>
					</li>
				))}
			</>
		) : (
			<li>There are no activities yet.</li>
		)}
      </ul>
  )
}

const ActivitiesPage: BlitzPage = () => (
	<>
		 <h1 className="text-6xl mb-10">Activities</h1>
		<Suspense fallback={<div>Loading...</div>}>
			<ActivityList />
		</Suspense>
	</>
)

ActivitiesPage.getLayout = (page) => <Layout title="Activities">{page}</Layout>

export default ActivitiesPage
```
4) Create the file and folder `./app/activities/queries/getActivities.ts`:
```ts
import { Ctx } from "blitz"
import db, { Prisma } from "db"

type GetActivitiesInput = Pick<Prisma.FindManyActivityArgs, "where" | "orderBy" | "skip" | "take">

export default async function getActivities(
	{ where, orderBy }: GetActivitiesInput,
	ctx: Ctx
) {
	ctx.session.authorize()

	const activities = await db.activity.findMany({
		where,
		orderBy
	})

	return activities
}
```

[All done! Back to section 7](./README.md)
