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
import { Suspense } from "react"
import Layout from "app/layouts/Layout"
import { Link, useQuery, BlitzPage } from "blitz"
import getActivities from "app/activities/queries/getActivities"

const NewActivity = () => (
	<Link href="/activities/new">
		<a className="bg-gradient-to-r from-purple-800 to-green-500 hover:from-pink-500 hover:to-green-500 text-white font-bold py-2 px-4 rounded focus:ring transform transition hover:scale-105 duration-300 ease-in-out">New activity</a>
	</Link>
)

export const ActivityList = () => {  
  const [activities] = useQuery(getActivities, {
    orderBy: { id: "asc" }
  })

  return (
      <ul className="list-outside list-disc">
		{activities.length ? (
			<>
				 {activities.map((activity) => (
					<li key={activity.id} className="mb-2 text-lg font-bold underline">
						<Link href={`/activities/${activity.id}`}>
							<a>{activity.name}</a>
						</Link>
					</li>
				))}
			</>
		) : (
			<li>
				<div className="mb-4">There are no activities yet.</div>
				<NewActivity />
			</li>
		)}
      </ul>
  )
}

const ActivitiesPage: BlitzPage = () => (
	<>
		 <div className="flex justify-between mb-10 items-center">
		 	<h1 className="text-6xl">Activities</h1>
			 <NewActivity />
		 </div>
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

Nice! Now we can go to [http://localhost:3000/activities](http://localhost:3000/activities) and see the page. But we're not there yet! Let's continue to create a form for create a activity.

## Make a new activty
To create a new activity, we want the following:
* A page /activities/new
* A form (reusable, so it also can be used for editing)
* A mutation (a way to handle the "server" things, like making sure the user is logged in, talk to the database etc)
* A way to validate our data, in this case a `zod` schema

1) Let's start with the page. Create file `./app/activities/pages/activities/new.tsx`:
```tsx
import Layout from "app/layouts/Layout"
import { Link, useRouter, useMutation, BlitzPage } from "blitz"
import createActivity from "app/activities/mutations/createActivity"
import ActivityForm from "app/activities/components/ActivityForm"
import { FORM_ERROR } from "app/components/Form"

const NewActivityPage: BlitzPage = () => {
  const router = useRouter()
  const [createActivityMutation] = useMutation(createActivity)
  return (
    <>
      <h1 className="text-6xl mb-10">New activity</h1>
      <ActivityForm
        initialValues={{ name: "", points: 0, description: "" }}
        onSubmit={async (values) => {
          try {
            const activity = await createActivityMutation(values)
            router.push(`/activities/${activity.id}`)
          } catch (error) {
            return {
              [FORM_ERROR]: error.message || error.toString(),
            }
          }
        }}
      />

      <p className="mt-10">
        <Link href="/activities">
          <a>Back to all activities</a>
        </Link>
      </p>
    </>
  )
}

NewActivityPage.getLayout = (page) => <Layout title={"Create New Test"}>{page}</Layout>

export default NewActivityPage
```
2) Then create the ActivityForm `./app/activities/components/ActivityForm.tsx`: 
```tsx
import React, { FC } from "react"
import { LabeledTextField } from "app/components/LabeledTextField"
import { Form } from "app/components/Form"
import { ActivityInput, ActivityInputType } from "../validations"

type ActivityFormProps = {
	onSubmit: (value: ActivityInputType) => any
	initialValues: ActivityInputType
	submitText?: string
}

export const ActivityForm: FC<ActivityFormProps> = (props) => {
	return (
		<Form
			submitText={props.submitText || "Create"}
			schema={ActivityInput}
			{...props}
		>
			<LabeledTextField name="name" label="Activity name" placeholder="Name" />
			<LabeledTextField name="points" label="Points" placeholder="Points" type="number" />
		</Form>
	)
}

export default ActivityForm
```
3) Let's do the mutation. Create a file `./app/activities/mutations/createActivity.ts`:
```ts
import { Ctx } from "blitz"
import db from "db"
import { ActivityInput, ActivityInputType } from "../validations"

export default async function createActivity(data: ActivityInputType, ctx: Ctx) {
  ctx.session.authorize()
  const parsedData = ActivityInput.parse(data)

  const activity = await db.activity.create({
    data: {
      ...parsedData,
      createdBy: {
        connect: {
          id: ctx.session.userId,
        },
      },
    },
  })

  return activity
}
```
4) Validation. Create file `./app/activities/validations.ts`::
```ts
import * as z from "zod"

export const ActivityInput = z.object({
  points: z.number().min(0).max(9999),
  name: z.string().min(2).max(100),
  description: z.string().max(500).optional(),
})
export type ActivityInputType = z.infer<typeof ActivityInput>
```

## View / Edit activity
1) Create file `./app/activities/pages/activities/[activityId].tsx`. The name of the file, `[activityId]` is similar to route params in React-Router.
```tsx
import { FC, Suspense } from "react"
import Layout from "app/layouts/Layout"
import { Link, useRouter, useQuery, useParam, BlitzPage, useMutation } from "blitz"
import deleteActivity from "app/activities/mutations/deleteActivity"
import getActivity from "app/activities/queries/getActivity"
import ActivityForm from "app/activities/components/ActivityForm"
import updateActivity from "app/activities/mutations/updateActivity"

export const Activity: FC<{ id: number }> = ({ id }) => {
  const router = useRouter()
  const [activity, { refetch }] = useQuery(getActivity, id)
  const [deleteMutation] = useMutation(deleteActivity)
  const [updateMutation] = useMutation(updateActivity)

  if (!activity) return null
  return (
    <div>
      <h1 className="text-6xl mb-10">{activity.name}</h1>

      <div className="mb-10">
        <ActivityForm
          initialValues={{
            name: activity.name,
            description: activity.description || "",
            points: activity.points,
          }}
          onSubmit={async (values) => {
            await updateMutation({ data: values, id })
            await refetch()
          }}
          submitText="Update"
        />
      </div>
      <button
        type="button"
        className="bg-gradient-to-r from-red-800 to-red-500 hover:from-red-500 hover:to-red-500 text-white font-bold py-2 px-4 rounded focus:ring transform transition hover:scale-105 duration-300 ease-in-out"
        onClick={async () => {
          if (window.confirm(`Delete activity named "${activity.name}"?`)) {
            await deleteMutation(id)
            router.push("/activities")
          }
        }}
      >
        Delete
      </button>
    </div>
  )
}

const ShowEditActivity: BlitzPage = () => {
  const activityId = useParam("activityId", "number")
  if (!activityId) return null
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <Activity id={activityId} />
      </Suspense>
      <p className="mt-10">
        <Link href="/activities">
          <a>Back to activites</a>
        </Link>
      </p>
    </div>
  )
}

ShowEditActivity.getLayout = (page) => <Layout title="Show/Edit activity">{page}</Layout>

export default ShowEditActivity
```

2) Let's start with the easiest, the `delete` mutation. Create a file `./app/activities/mutations/deleteActivity.ts`:
```ts
import { Ctx } from "blitz"
import db from "db"

export default async function deleteActivity(id: number, ctx: Ctx) {
	ctx.session.authorize()

	await db.activity.delete({
		where: { id }
	})
}
```

3) Next mutation is `update`. Create file `./app/activities/mutations/updateActivity.ts`:
```ts
import { Ctx } from "blitz"
import db from "db"
import { ActivityInput, ActivityInputType } from "../validations"

export default async function updateActivity(
	{ data, id }: { data: ActivityInputType; id: number },
	ctx: Ctx
) {
	ctx.session.authorize()
	const parsedData = ActivityInput.parse(data)

	const activity = await db.activity.update({
		where: { id: id },
		data: parsedData
	})

	return activity
}
```

4) We need a way to get the activity from the server as well! Create the file `./app/activities/queries/getActivity.ts`:
```ts
import { Ctx } from "blitz"
import db from "db"

export default async function getActivity(
	id: number,
	ctx: Ctx
) {
	ctx.session.authorize()

	const activities = await db.activity.findFirst({
		where: { id }
	})

	return activities
}
```
Ok that was quite a lot😅 But hopefully you start to see a pattern. Let's now try to do it the simple way - using the CLI. 

[Back to section 7](./README.md)
