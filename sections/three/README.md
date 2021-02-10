# Section 3 - Add TailwindCSS
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

a {
	text-decoration: underline;
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
							<img src="/dotjs.svg" alt="dotjs logo" />
						  </span>
					    	</a>
					</div>
					<div className="flex flex-1 md:w-1/3 justify-center md:justify-start text-white px-2"></div>
					<div className="flex w-full content-center justify-between md:w-1/3 md:justify-end"></div>
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
2) Add the awesome logo. Create a file named `./public/dotjs.svg` with the following content:
```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1920 1911" width="40" height="40" data-name="Layer 1"><defs><linearGradient gradientTransform="translate(1.967 2.719)" gradientUnits="userSpaceOnUse" y2="1870.31" x2="1877.42" y1="32.07" x1="39.18" id="a"><stop stop-color="#29abe2" offset="0"/><stop stop-color="#59b9ae" offset=".29"/><stop stop-color="#d0dd2a" offset=".95"/><stop stop-color="#d9e021" offset="1"/></linearGradient></defs><path fill="#fffff9" d="M175.597 401.816h1553.478v1075.728H175.597z"/><path d="M1838.967 18.049H81.557a57.15 57.15 0 00-57.15 57.15v1757.41a57.16 57.16 0 0057.15 57.16h1757.41a57.16 57.16 0 0057.16-57.16V75.199a57.16 57.16 0 00-57.16-57.15zm-1325.58 1376.67c-68.21 0-124.18-55.1-124.18-123.31 0-68.21 56-124.17 124.18-124.17s124.17 56 124.17 124.17-55.97 123.31-124.17 123.31zm523.12-448.62c0 244.91-140.79 444.05-313.06 444.05h-21.86v-240.81h21.86c50.72 0 94.44-130.3 94.44-203.28v-432.92h218.62zm494.82-212.13h-21.86c-51.6 0-95.32 43.72-95.32 94.44v248.31c0 173.14-140.79 313.06-313.07 313.06h-21.86v-218.61h21.86c50.72 0 94.45-42.85 94.45-94.44v-248.36c0-172.27 140.79-313.06 313.94-313.06h21.86z" fill="url(#a)"/></svg>
```

3) Update the file `./app/pages/_document.tsx` to contain the following:
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
4) Update the `./app/pages/index.tsx` to contain: 
```tsx
import { Link, BlitzPage } from "blitz"
import Layout from "app/layouts/Layout"

const Home: BlitzPage = () => {
  return (
    <>
	<h1 className="text-6xl">dotjs Leaderboard</h1>
	<ul className="list-reset items-center text-sm pt-3">
		<li>
			<Link href="/signup">
				<a className="inline-block text-gray-600 no-underline hover:text-gray-100 hover:text-underline py-1 text-base">Sign up</a>
			</Link>
		</li>
		<li>
			<Link href="/login">
				<a className="inline-block text-gray-600 no-underline hover:text-gray-100 hover:text-underline py-1 text-base">Login</a>
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
