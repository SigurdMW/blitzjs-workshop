# Section 3 - Add TailwindCss
## Using Blitz Recipe
1) Run `blitz install tailwind`

*NB!* Might break due to this issue: https://github.com/blitz-js/blitz/issues/1879

## Setting up Tailwind manually
1) Run `yarn add tailwindcss`
2) Run `yarn add -D autoprefixer postcss`
3) Create file `tailwind.config.js` in project root:
```js
module.exports = {
	purge: ["{app,pages}/**/*.{js,jsx,ts,tsx}"],
	darkMode: false, // or 'media' or 'class'
	theme: {
		extend: {},
	},
	variants: {
		extend: {},
	},
	plugins: [],
}
```
4) Create file `postcss.config.js` in project root:
```js
module.exports = {
	plugins: {
		tailwindcss: {},
		autoprefixer: {},
	},
}
```
5) Create `style.css` in `./app`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

.hero {
	width: 100%;
	color: #333;
}

.title {
	margin: 0;
	width: 100%;
	padding-top: 80px;
	line-height: 1.15;
	font-size: 48px;
}

.title,
.description {
	text-align: center;
}
```
6) Import `style.css` in `./app/pages/_app.tsx`:
```ts
import "../style.css"
```
7) Verify that nothing breaks by starting server with `blitz start`

## Setup the app layout
1) Update the `./app/layouts/Layout` to have the following content:
```tsx
import { ReactNode } from "react"
import { Head } from "blitz"

type LayoutProps = {
	title?: string
	children: ReactNode
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
						<a href="#">
							<span className="text-xl pl-2">LOGO</span>
						</a>
					</div>
					<div className="flex flex-1 md:w-1/3 justify-center md:justify-start text-white px-2"></div>
					<div className="flex w-full pt-2 content-center justify-between md:w-1/3 md:justify-end">
						<ul className="list-reset flex justify-between flex-1 md:flex-none items-center">
							<li className="flex-1 md:flex-none md:mr-3">
								<a className="inline-block py-2 px-4 text-white no-underline" href="#">Active</a>
							</li>
							<li className="flex-1 md:flex-none md:mr-3">
								<a className="inline-block no-underline text-white hover:text-gray-200 hover:text-underline py-2 px-4" href="#">Link</a>
							</li>
							<li className="flex-1 md:flex-none md:mr-3">
								<div className="relative inline-block">
									<button className="drop-button text-white focus:outline-none">
										Hi, User <svg className="h-3 fill-current inline" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20">
											<path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z" /></svg>
									</button>
									<div id="myDropdown" className="dropdownlist absolute bg-gray-800 text-white right-0 mt-3 p-3 overflow-auto z-30 invisible">
										<input type="text" className="drop-search p-2 text-gray-600" placeholder="Search.." id="myInput" />
										<a href="#" className="p-2 hover:bg-gray-800 text-white text-sm no-underline hover:no-underline block">Profile</a>
										<a href="#" className="p-2 hover:bg-gray-800 text-white text-sm no-underline hover:no-underline block">Settings</a>
										<div className="border border-gray-800"></div>
										<a href="#" className="p-2 hover:bg-gray-800 text-white text-sm no-underline hover:no-underline block">Log Out</a>
									</div>
								</div>
							</li>
						</ul>
					</div>
				</div>
			</nav>
			<div className="flex flex-col md:flex-row mt-24">
				<div className="container mx-auto">
					{children}
				</div>
			</div>
		</>
	)
}

export default Layout
```
2) Update the file `./app/pages/_document.tsx` to contain the following:
```tsx
import { Document, Html, DocumentHead, Main, BlitzScript /*DocumentContext*/ } from 'blitz'

class MyDocument extends Document {
	// Only uncomment if you need to customize this behaviour
	// static async getInitialProps(ctx: DocumentContext) {
	//   const initialProps = await Document.getInitialProps(ctx)
	//   return {...initialProps}
	// }

	render() {
		return (
			<Html lang="en">
				<DocumentHead />
				<body className="bg-gray-800 font-sans leading-normal tracking-normal mt-12 text-white">
					<Main />
					<BlitzScript />
				</body>
			</Html>
		)
	}
}

export default MyDocument

```
3) Update the `./app/pages/index.tsx` to contain: 
```tsx
import { Link, BlitzPage, useMutation } from "blitz"
import Layout from "app/layouts/Layout"
import logout from "app/auth/mutations/logout"
import { useCurrentUser } from "app/hooks/useCurrentUser"
import { Suspense } from "react"

/*
 * This file is just for a pleasant getting started page for your new app.
 * You can delete everything in here and start from scratch if you like.
 */

const UserInfo = () => {
	const currentUser = useCurrentUser()
	const [logoutMutation] = useMutation(logout)

	if (currentUser) {
		return (
			<>
				<button
					className="button small"
					onClick={async () => {
						await logoutMutation()
					}}
				>
					Logout
        </button>
				<div>
					User id: <code>{currentUser.id}</code>
					<br />
					User role: <code>{currentUser.role}</code>
				</div>
			</>
		)
	} else {
		return (
			<>
				<Link href="/signup">
					<a className="button small">
						<strong>Sign Up</strong>
					</a>
				</Link>
				<Link href="/login">
					<a className="button small">
						<strong>Login</strong>
					</a>
				</Link>
			</>
		)
	}
}

const Home: BlitzPage = () => {
	return (
		<>
			<h1 className="text-6xl">dotjs Leaderboard</h1>
			<Suspense fallback="Loading...">
				<UserInfo />
			</Suspense>
		</>
	)
}

Home.getLayout = (page) => <Layout title="Home">{page}</Layout>

export default Home
```
Good job getting here! Move on to [Section Four](../four)
