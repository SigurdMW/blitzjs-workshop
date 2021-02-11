# Section 4 - Auth and layout setup
Now that we have set up Tailwind, we want to adapt the layout of the application. We start by making the header display link to login, register new user and logout. In addition, we'll fix the forms so that they use Tailwind as well.

## Setting up the navbar
The navbar should display login and signup when the visitor is not authenticated. When the user authenticates, let's display the name and a logout link in the header. 
Let's make the required changes for this to happen:
1) Open `./app/Layouts/Layout.tsx` and add the following:
```tsx
import React, { ReactNode, Suspense } from "react"
import { Head, useMutation, Link } from "blitz"
import { useCurrentUser } from "app/hooks/useCurrentUser"
import logout from "app/auth/mutations/logout"

type LayoutProps = {
	title?: string
	children: ReactNode
}

const UnauthLinks = () => (
	<ul className="list-reset flex justify-between flex-1 md:flex-none items-center">
		<li className="flex-1 md:flex-none md:mr-3">
			<Link href="/signup">
				<a className="inline-block py-2 px-4 text-white no-underline">
					Sign Up
					</a>
			</Link>
		</li>
		<li className="flex-1 md:flex-none md:mr-3">
			<Link href="/login">
				<a className="inline-block no-underline text-white hover:text-gray-200 hover:text-underline py-2 px-4">
					Login
				</a>
			</Link>
		</li>
	</ul>
)

const HeaderLinks = () => {
	const currentUser = useCurrentUser()
	const [logoutMutation] = useMutation(logout)

	if (currentUser) {
		return (
			<ul className="list-reset flex justify-between flex-1 md:flex-none items-center">
				<li className="flex-1 md:flex-none md:mr-3">
					<span className="inline-block py-2 px-4 text-white no-underline">
						{currentUser.name || "Noname"}
					</span>
				</li>
				<li className="flex-1 md:flex-none md:mr-3">
					<button
						className="inline-block no-underline text-white hover:text-gray-200 hover:text-underline py-2 px-4"
						onClick={async () => {
							await logoutMutation()
						}}
					>
						Logout
					</button>
				</li>
			</ul>
		)
	}
	return <UnauthLinks />
}

const Layout = ({ title, children }: LayoutProps) => {
	return (
		<>
			<Head>
				<title>{title || "dotjs-leaderboardd"}</title>
				<link rel="icon" href="/favicon.ico" />
			</Head>
			<nav className="bg-gray-800 pt-2 md:pt-1 pb-1 px-1 mt-0 h-auto fixed w-full z-20 top-0">
				<div className="flex flex-wrap items-center">
					<div className="flex flex-shrink md:w-1/3 justify-center md:justify-start text-white">
						<a href="/" className="flex w-12 justify-center">
							<span className="text-xl pl-2">
								<img src="/dotjs.svg" alt="dotjs logo" />
							</span>
						</a>
					</div>
					<div className="flex flex-1 md:w-1/3 justify-center md:justify-start text-white px-2"></div>
					<div className="flex w-full pt-2 content-center justify-between md:w-1/3 md:justify-end">
						<Suspense fallback={<UnauthLinks />}>
							<HeaderLinks />
						</Suspense>
					</div>
				</div>
			</nav>
			<div className="flex flex-col md:flex-row mt-24">
				<div className="container mx-auto">{children}</div>
			</div>
		</>
	)
}

export default Layout
```

2) Open `./app/auth/pages/login.tsx` and make sure it looks like this (there is a slight problem at this point - the header does not reflect the login state before we refresh the page):
```tsx
import React from "react"
import { BlitzPage } from "blitz"
import Layout from "app/layouts/Layout"
import { LoginForm } from "app/auth/components/LoginForm"

const LoginPage: BlitzPage = () => (
	<LoginForm onSuccess={() => {
		window.location.href = "/"
	}} />
)

LoginPage.getLayout = (page) => <Layout title="Log In">{page}</Layout>

export default LoginPage
``` 

3) Let's make our login form pritty! Open `./app/components/Form.tsx` and paste the following:
```tsx
import React, { useState, ReactNode, PropsWithoutRef } from "react"
import { FormProvider, useForm, UseFormOptions } from "react-hook-form"
import * as z from "zod"

type FormProps<S extends z.ZodType<any, any>> = {
	/** All your form fields */
	children: ReactNode
	/** Text to display in the submit button */
	submitText: string
	schema?: S
	onSubmit: (values: z.infer<S>) => Promise<void | OnSubmitResult>
	initialValues?: UseFormOptions<z.infer<S>>["defaultValues"]
} & Omit<PropsWithoutRef<JSX.IntrinsicElements["form"]>, "onSubmit">

type OnSubmitResult = {
	FORM_ERROR?: string
	[prop: string]: any
}

export const FORM_ERROR = "FORM_ERROR"

export function Form<S extends z.ZodType<any, any>>({
	children,
	submitText,
	schema,
	initialValues,
	onSubmit,
	...props
}: FormProps<S>) {
	const ctx = useForm<z.infer<S>>({
		mode: "onBlur",
		resolver: async (values) => {
			try {
				if (schema) {
					schema.parse(values)
				}
				return { values, errors: {} }
			} catch (error) {
				return { values: {}, errors: error.formErrors?.fieldErrors }
			}
		},
		defaultValues: initialValues,
	})
	const [formError, setFormError] = useState<string | null>(null)

	return (
		<FormProvider {...ctx}>
			<form
				onSubmit={ctx.handleSubmit(async (values) => {
					const result = (await onSubmit(values)) || {}
					for (const [key, value] of Object.entries(result)) {
						if (key === FORM_ERROR) {
							setFormError(value)
						} else {
							ctx.setError(key as any, {
								type: "submit",
								message: value,
							})
						}
					}
				})}
				className="form"
				{...props}
			>
				{/* Form fields supplied as children are rendered here */}
				{children}

				{formError && (
					<div role="alert" style={{ color: "red" }}>
						{formError}
					</div>
				)}

				<button
					type="submit"
					disabled={ctx.formState.isSubmitting}
					className="bg-gradient-to-r from-purple-800 to-green-500 hover:from-pink-500 hover:to-green-500 text-white font-bold py-2 px-4 rounded focus:ring transform transition hover:scale-105 duration-300 ease-in-out"
				>
					{submitText}
				</button>
			</form>
		</FormProvider>
	)
}

export default Form
```

4) Then open `./app/components/LabeledTextField.tsx` and paste:
```jsx
import React, { PropsWithoutRef } from "react"
import { Controller, useFormContext } from "react-hook-form"

export interface LabeledTextFieldProps extends PropsWithoutRef<JSX.IntrinsicElements["input"]> {
	/** Field name. */
	name: string
	/** Field label. */
	label: string
	/** Field type. Doesn't include radio buttons and checkboxes */
	type?: "text" | "password" | "email" | "number"
	outerProps?: PropsWithoutRef<JSX.IntrinsicElements["div"]>
}

export const LabeledTextField = React.forwardRef<HTMLInputElement, LabeledTextFieldProps>(
	({ label, outerProps, type, name, placeholder, ...props }, ref) => {
		const {
			register,
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
					render={(
						{ onChange, ...rest },
						{ invalid, isTouched, isDirty }
					) => (
						<label className="block w-full mb-1">
							{label}
							<input
								disabled={isSubmitting}
								type={type}
								aria-invalid={invalid}
								onChange={(v) => {
									const value = v.target.value
									if (type === "number") {
										onChange(parseInt(value, 10))
									} else {
										onChange(value)
									}
								}}
								placeholder={placeholder}
								className="w-full p-1 pl-2 rounded-sm mt-2 text-black"
								{...rest}
							/>
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

export default LabeledTextField
```
5) Open `./app/auth/pages/login.tsx` and paste:
```tsx
import React from "react"
import { BlitzPage } from "blitz"
import Layout from "app/layouts/Layout"
import { LoginForm } from "app/auth/components/LoginForm"

const LoginPage: BlitzPage = () => (
	<>
		<h1 className="text-6xl mb-10">Login</h1>
		<LoginForm onSuccess={() => {
			window.location.href = "/"
		}} />
	</>
)

LoginPage.getLayout = (page) => <Layout title="Log In">{page}</Layout>

export default LoginPage
```
6) And do simiar for `./app/auth/pages/signup.tsx`:
```tsx
import React from "react"
import { BlitzPage } from "blitz"
import Layout from "app/layouts/Layout"
import { SignupForm } from "app/auth/components/SignupForm"

const SignupPage: BlitzPage = () => (
	<>
		<h1 className="text-6xl mb-10">Create account</h1>
		<SignupForm onSuccess={() => {
			window.location.href = "/"
		}} />
	</>
)

SignupPage.getLayout = (page) => <Layout title="Sign Up">{page}</Layout>

export default SignupPage
```

7) Remove the `h1` from `./app/auth/components/SignupForm.tsx`:
```tsx
import React from "react"
import { useMutation } from "blitz"
import { LabeledTextField } from "app/components/LabeledTextField"
import { Form, FORM_ERROR } from "app/components/Form"
import signup from "app/auth/mutations/signup"
import { SignupInput } from "app/auth/validations"

type SignupFormProps = {
  onSuccess?: () => void
}

export const SignupForm = (props: SignupFormProps) => {
  const [signupMutation] = useMutation(signup)

  return (
      <Form
        submitText="Create Account"
        schema={SignupInput}
        initialValues={{ email: "", password: "" }}
        onSubmit={async (values) => {
          try {
            await signupMutation(values)
            props.onSuccess?.()
          } catch (error) {
            if (error.code === "P2002" && error.meta?.target?.includes("email")) {
              // This error comes from Prisma
              return { email: "This email is already being used" }
            } else {
              return { [FORM_ERROR]: error.toString() }
            }
          }
        }}
      >
        <LabeledTextField name="email" label="Email" placeholder="Email" />
        <LabeledTextField name="password" label="Password" placeholder="Password" type="password" />
      </Form>
  )
}

export default SignupForm
```
8) Remove the `h1` from `./app/auth/components/LoginForm.tsx`:
```tsx
import React from "react"
import { AuthenticationError, Link, useMutation } from "blitz"
import { LabeledTextField } from "app/components/LabeledTextField"
import { Form, FORM_ERROR } from "app/components/Form"
import login from "app/auth/mutations/login"
import { LoginInput } from "app/auth/validations"

type LoginFormProps = {
  onSuccess?: () => void
}

export const LoginForm = (props: LoginFormProps) => {
  const [loginMutation] = useMutation(login)

  return (
	<>
    <Form
        submitText="Login"
        schema={LoginInput}
        initialValues={{ email: "", password: "" }}
        onSubmit={async (values) => {
          try {
            await loginMutation(values)
            props.onSuccess?.()
          } catch (error) {
            if (error instanceof AuthenticationError) {
              return { [FORM_ERROR]: "Sorry, those credentials are invalid" }
            } else {
              return {
                [FORM_ERROR]:
                  "Sorry, we had an unexpected error. Please try again. - " + error.toString(),
              }
            }
          }
        }}
      >
        <LabeledTextField name="email" label="Email" placeholder="Email" />
        <LabeledTextField name="password" label="Password" placeholder="Password" type="password" />
      </Form>

      <div style={{ marginTop: "1rem" }}>
        Or <Link href="/signup">Sign Up</Link>
      </div>
    </>
  )
}

export default LoginForm
```
Woohhooo you got here! Please [continue to section 5](../five)
