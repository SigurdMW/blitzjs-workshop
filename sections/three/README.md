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
						 <a href="/" className="flex w-12 justify-center">
						      <span className="text-xl pl-2">
							<img src="/dotjs.png" alt="dotjs logo" />
						  </span>
					    	</a>
					</div>
					<div className="flex flex-1 md:w-1/3 justify-center md:justify-start text-white px-2"></div>
					<div className="flex w-full pt-2 content-center justify-between md:w-1/3 md:justify-end"></div>
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

const Home: BlitzPage = () => {
  return (
    <>
		<h1 className="text-6xl">dotjs Leaderboard</h1>
		<ul class="list-reset items-center text-sm pt-3">
			<li>
				<Link href="/signup">
					<a class="inline-block text-gray-600 no-underline hover:text-gray-100 hover:text-underline py-1 text-base">Sign up</a>
				</Link>
			</li>
			<li>
				<Link href="/login">
					<a class="inline-block text-gray-600 no-underline hover:text-gray-100 hover:text-underline py-1 text-base">Login</a>
				</Link>
			</li>
		</ul>
    </>
  )
}

Home.getLayout = (page) => <Layout title="Home">{page}</Layout>

export default Home
```
Good job getting here! Move on to [Section Four](../four)
