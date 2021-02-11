# Section 9 - UI Improvements
Right now, we have to enter the id of the user and the activity we want to add. That's not ideal. Let's fix it now.

## Adding a user select component
1) Create file `./app/component/UserSelect.tsx`:
```tsx
import { User } from "db"
import getUsers from "app/users/queries/getUsers"
import { useQuery } from "blitz"
import React, { FC, PropsWithoutRef, Suspense } from "react"
import { Controller, useFormContext } from "react-hook-form"

export interface UserSelectFieldProps extends PropsWithoutRef<JSX.IntrinsicElements["input"]> {
	/** Field name. */
	name: string
	/** Field label. */
	label: string
	outerProps?: PropsWithoutRef<JSX.IntrinsicElements["div"]>
	users: Array<Pick<User, "email" | "name" | "id">>
}

export const UserSelectField = React.forwardRef<HTMLInputElement, UserSelectFieldProps>(
	({ label, outerProps, name, placeholder, users, ...props }, ref) => {
		const {
			formState: { isSubmitting },
			control,
			errors,
		} = useFormContext()
		const error = Array.isArray(errors[name])
			? errors[name].join(", ")
			: errors[name]?.message || errors[name]

		return (
			<div {...outerProps} className="mb-6 max-w-lg">
				<Controller
					control={control}
					name={name}
					render={({onChange, ...rest}, { invalid, isTouched, isDirty }) => (
						<label className="block w-full mb-1">
							{label}
							<select
								className="text-black block w-full p-1 pl-2 rounded-sm mt-2"
								disabled={isSubmitting}
								aria-invalid={invalid}
								placeholder={placeholder}
								onChange={(e) => {
									const val = e.target.value
									if (!isNaN(parseInt(val, 10))) {
										onChange(parseInt(val, 10))
									} else {
										onChange(undefined)
									}
								}}
								{...rest}>
								<option value="">Select</option>
								{users.map((u) => <option key={u.id} value={u.id}>{u.name || u.email}</option>)}
							</select>
						</label>
					)}
				/>

				{error && (
					<div role="alert" className="text-red-600">
						{error}
					</div>
				)}
			</div>
		)
	}
)

const UserSelectLoader: FC<Omit<UserSelectFieldProps, "users">> = (props) => {
	const [users] = useQuery(getUsers, undefined)
	return (
		<UserSelectField users={users} {...props} />
	)
}

const UserSelect: FC<Omit<UserSelectFieldProps, "users">> = (props) => (
	<Suspense fallback={<div>Loading...</div>}>
		<UserSelectLoader {...props} />
	</Suspense>
)

export default UserSelect
``` 

2) Create `app/users/queries/getUsers.ts`:
```ts
import { Ctx } from "blitz"
import db from "db"

export default async function getUsers(_ = null, ctx: Ctx) {
  ctx.session.authorize()

  const user = await db.user.findMany({
	  select: { name: true, email: true, id: true }
  })

  return user
}
```
## Adding a activity select component
Add file `./app/component/ActivitySelect.tsx`:
```tsx
import { Activity } from "db"
import { useQuery } from "blitz"
import React, { FC, PropsWithoutRef, Suspense } from "react"
import { Controller, useFormContext } from "react-hook-form"
import getActivities from "app/activities/queries/getActivities"

export interface ActivitySelectFieldProps extends PropsWithoutRef<JSX.IntrinsicElements["input"]> {
	/** Field name. */
	name: string
	/** Field label. */
	label: string
	outerProps?: PropsWithoutRef<JSX.IntrinsicElements["div"]>
	activities: Array<Activity>
}

export const ActivitySelectField = React.forwardRef<HTMLInputElement, ActivitySelectFieldProps>(
	({ label, outerProps, name, placeholder, activities, ...props }, ref) => {
		const {
			formState: { isSubmitting },
			control,
			errors,
		} = useFormContext()
		const error = Array.isArray(errors[name])
			? errors[name].join(", ")
			: errors[name]?.message || errors[name]

		return (
			<div {...outerProps} className="mb-6 max-w-lg">
				<Controller
					control={control}
					name={name}
					render={({onChange, ...rest}, { invalid, isTouched, isDirty }) => (
						<label className="block w-full mb-1">
							{label}
							<select
								className="text-black block w-full p-1 pl-2 rounded-sm mt-2"
								disabled={isSubmitting}
								aria-invalid={invalid}
								placeholder={placeholder}
								onChange={(e) => {
									const val = e.target.value
									if (!isNaN(parseInt(val, 10))) {
										onChange(parseInt(val, 10))
									} else {
										onChange(undefined)
									}
								}}
								{...rest}>
								<option value="">Select</option>
								{activities.map((u) => <option key={u.id} value={u.id}>{u.name}</option>)}
							</select>
						</label>
					)}
				/>

				{error && (
					<div role="alert" className="text-red-600">
						{error}
					</div>
				)}
			</div>
		)
	}
)

const ActivitySelectLoader: FC<Omit<ActivitySelectFieldProps, "activities">> = (props) => {
	const [activities] = useQuery(getActivities, {})
	return (
		<ActivitySelectField activities={activities} {...props} />
	)
}

const ActivitySelect: FC<Omit<ActivitySelectFieldProps, "activities">> = (props) => (
	<Suspense fallback={<div>Loading...</div>}>
		<ActivitySelectLoader {...props} />
	</Suspense>
)

export default ActivitySelect
```

Update the form `./app/actions/components/ActionForm.tsx`:
```tsx
import React, { FC } from "react"
import { LabeledTextField } from "app/components/LabeledTextField"
import { Form } from "app/components/Form"
import { ActionInputType, ActionInput } from "../validations"
import UserSelect from "app/components/UserSelect"
import ActivitySelect from "app/components/ActivitySelect"

type ActionFormProps = {
	initialValues: Partial<ActionInputType>
	onSubmit: (values: ActionInputType) => any
	submitText?: string
}

export const ActionForm: FC<ActionFormProps> = (props) => {
	return (
		<Form submitText={props.submitText || "Create"} schema={ActionInput} {...props}>
			<UserSelect name="userId" label="User" placeholder="User" />
			<ActivitySelect name="activityId" label="Activity" placeholder="Activity" />
			<LabeledTextField name="comment" label="Comment" placeholder="Comment" type="text" />
		</Form>
	)
}
export default ActionForm
```

> Getting lint errors after adding these components? See below

I took a quick fix and updated `.eslintrc.js` to get rid of the errors:
```js
module.exports = {
  env: {
    es2020: true,
  },
  extends: ['react-app', 'plugin:jsx-a11y/recommended'],
  plugins: ['jsx-a11y'],
  rules: {
    "import/no-anonymous-default-export": "error",
    'import/no-webpack-loader-syntax': 'off',
    'react/react-in-jsx-scope': 'off', // React is always in scope with Blitz
    'jsx-a11y/anchor-is-valid': 'off', //Doesn't play well with Blitz/Next <Link> usage
	'jsx-a11y/no-onchange': 'off'
  },
}
```

Nice! Let's check out [next section](../ten)
