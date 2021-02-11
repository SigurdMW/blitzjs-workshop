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
  comment	String @default("")
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
		<ul className="list-outside list-disc">
			<>
				{actions.length ? (
					<>
						{actions.map((action) => (
							<li key={action.id} className="mb-2 text-lg font-bold underline">
								<Link href={`/actions/${action.id}`}>
									<a>{action.user.name || action.user.email} got points!</a>
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

2) Update `./app/actions/queries/getActions.ts` so that we include the user and remove the pagination:
```ts
import { Ctx } from "blitz"
import db, { Prisma } from "db"

type GetActionsInput = Pick<Prisma.FindManyActionArgs, "where" | "orderBy" | "skip" | "take">

export default async function getActions(
	{ where, orderBy }: GetActionsInput,
	ctx: Ctx
) {
	ctx.session.authorize()

	const actions = await db.action.findMany({
		where,
		orderBy,
		include: { user: true }
	})

	return {
		actions
	}
}
```

3) Update `./app/actions/components/ActionForm.tsx`:
```tsx
import React, { FC } from "react"
import { LabeledTextField } from "app/components/LabeledTextField"
import { Form } from "app/components/Form"
import { ActionInputType, ActionInput } from "../validations"

type ActionFormProps = {
	initialValues: Partial<ActionInputType>
	onSubmit: (values: ActionInputType) => any
	submitText?: string
}

export const ActionForm: FC<ActionFormProps> = (props) => {
	return (
		<Form submitText={props.submitText || "Create"} schema={ActionInput} {...props}>
			<LabeledTextField name="userId" label="User" placeholder="User" type="number" />
			<LabeledTextField name="activityId" label="Activity" placeholder="Activity" type="number" />
			<LabeledTextField name="comment" label="Comment" placeholder="Comment" type="text" />
		</Form>
	)
}
export default ActionForm
```
4) Create file `./app/actions/validations.ts`:
```ts
import * as z from "zod"

export const ActionInput = z.object({
	userId: z.number().min(0),
	activityId: z.number().min(0),
	comment: z.string().optional()
})

export type ActionInputType = z.infer<typeof ActionInput>
```

5) Update `./app/actions/pages/actions/new.tsx`:
```tsx
import Layout from "app/layouts/Layout"
import { Link, useRouter, useMutation, BlitzPage } from "blitz"
import createAction from "app/actions/mutations/createAction"
import ActionForm from "app/actions/components/ActionForm"

const NewActionPage: BlitzPage = () => {
	const router = useRouter()
	const [createActionMutation] = useMutation(createAction)

	return (
		<>
			<h1 className="text-6xl mb-10">New action</h1>

			<ActionForm
				initialValues={{}}
				onSubmit={async (values) => {
					try {
						const action = await createActionMutation({ data: values })
						router.push(`/actions/${action.id}`)
					} catch (error) {
						alert("Error creating action " + JSON.stringify(error, null, 2))
					}
				}}
			/>

			<p>
				<Link href="/actions">
					<a>Back to actions</a>
				</Link>
			</p>
		</>
	)
}

NewActionPage.getLayout = (page) => <Layout title={"Create New Action"}>{page}</Layout>

export default NewActionPage
```

6) Update `./app/actions/mutations/createAction.ts`:
```ts
import { Ctx, NotFoundError } from "blitz"
import db from "db"
import { ActionInput, ActionInputType } from "../validations"

type CreateActionInput = {
	data: ActionInputType
}

export default async function createAction({ data }: CreateActionInput, ctx: Ctx) {
	ctx.session.authorize()

	const { comment, activityId, userId } = ActionInput.parse(data)

	// Make sure the given activity and user exist
	const [user, activity] = await Promise.all([
		db.user.findUnique({ where: { id: data.userId }}),
		db.activity.findUnique({ where: { id: data.activityId }})
	])

	if (!user || !activity) throw new NotFoundError("User or activity not found")

	const action = await db.action.create({
		data: {
			activity: {
				connect: { id: activityId }
			},
			user: {
				connect: { id: userId }
			},
			createdByUser: {
				connect: { id: ctx.session.userId }
			},
			comment
		}
	})

	return action
}
```

7) Let's delete the created folder `./app/actions/pages/[actionId]` and the file `edit.tsx`. We will allow a action to be deleted, but not edited.
8) Update file `./app/actions/pages/[actionId].tsx`:
```tsx
import { Suspense } from "react"
import Layout from "app/layouts/Layout"
import { Link, useRouter, useQuery, useParam, BlitzPage, useMutation } from "blitz"
import getAction from "app/actions/queries/getAction"
import deleteAction from "app/actions/mutations/deleteAction"

export const Action = () => {
	const router = useRouter()
	const actionId = useParam("actionId", "number")
	const [action] = useQuery(getAction, { where: { id: actionId } })
	const [deleteActionMutation] = useMutation(deleteAction)

	const displayName = action.user.name || action.user.email
	return (
		<div>
			<h1 className="text-6xl mb-10">{displayName} got points!</h1>
			<p className="mb-2">{displayName} got {action.activity.points} points {action.comment && <span>for {action.comment}</span>}</p>
			<div className="mb-10">
				<button
					type="button"
					className="bg-gradient-to-r from-red-800 to-red-500 hover:from-red-500 hover:to-red-500 text-white font-bold py-2 px-4 rounded focus:ring transform transition hover:scale-105 duration-300 ease-in-out"
					onClick={async () => {
						if (window.confirm(`Delete action with id ${action.id}`)) {
							await deleteActionMutation({ where: { id: action.id } })
							router.push("/actions")
						}
					}}
				>
					Delete
				</button>
			</div>
		</div>
	)
}

const ShowActionPage: BlitzPage = () => {
	return (
		<div>
			<Suspense fallback={<div>Loading...</div>}>
				<Action />
			</Suspense>
			<p className="mt-10">
				<Link href="/actions">
					<a>Back to actions</a>
				</Link>
			</p>
		</div>
	)
}

ShowActionPage.getLayout = (page) => <Layout title={"Action"}>{page}</Layout>

export default ShowActionPage
```

9) Create file `app/actions/queries/getAction.ts`:
```ts
import { Ctx, NotFoundError } from "blitz"
import db, { Prisma } from "db"

type GetActionInput = Pick<Prisma.FindFirstActionArgs, "where">

export default async function getAction({ where }: GetActionInput, ctx: Ctx) {
  ctx.session.authorize()

  const action = await db.action.findFirst({ where, include: { user: true, activity: true } })

  if (!action) throw new NotFoundError()

  return action
}
```
[All done! Back to section 7](./README.md)
