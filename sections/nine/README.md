# Section 9 - UI Improvements
Right now, we have to enter the id of the user and the activity we want to add. That's not ideal. Let's fix it now.

## Adding a user select component
Create file `./app/component/UserSelect.tsx`:
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
