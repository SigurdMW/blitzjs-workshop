# Leaderboard setup
We are going to setup the leaderboard as a feature, that means it will live at /leaderboard and have it's own queries and mutations.

## Setting up the feature
1) Create file and folders `./app/leaderboard/pages/leaderboard/index.tsx`:
```tsx
import { Suspense } from "react"
import Layout from "app/layouts/Layout"
import { useQuery, BlitzPage } from "blitz"
import getLeaderboard from "app/leaderboard/queries/getLeaderboard"

export const Leaderboard = () => {
	const [leaderboard] = useQuery(getLeaderboard, {})

	return (
		<table className="min-w-full leading-normal">
			<thead>
				<tr>
					<th className="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider">
						Place
					</th>
					<th className="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider">
						User
					</th>
					<th className="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider">
						Points
					</th>
				</tr>
			</thead>
			<tbody>
				{leaderboard.map((row, index) => (
					<tr key={row.id}>
						<td className="px-5 py-5 border-b border-gray-200 bg-white text-sm">
							<p className="text-gray-900 whitespace-no-wrap">{index + 1}</p>
						</td>
						<td className="px-5 py-5 border-b border-gray-200 bg-white text-sm">
							<div className="flex items-center">
								<p className="text-gray-900 whitespace-no-wrap">{row.name || row.email}</p>
							</div>
						</td>
						<td className="px-5 py-5 border-b border-gray-200 bg-white text-sm">
							<span className="relative inline-block px-3 py-1 font-semibold text-green-900 leading-tight">
								<span aria-hidden="true" className="absolute inset-0 bg-green-200 opacity-50 rounded-full"></span>
								<span className="relative">{row.points}</span>
							</span>
						</td>
					</tr>
				))}
			</tbody>
		</table>
	)
}

const LeaderboardPage: BlitzPage = () => (
	<>
		<h1 className="text-6xl  mb-10">Leaderboard</h1>
		<Suspense fallback={<div>Loading...</div>}>
			<Leaderboard />
		</Suspense>
	</>
)

LeaderboardPage.getLayout = (page) => <Layout title="Activities">{page}</Layout>

export default LeaderboardPage
```

2) Create file and folder `./app/leaderboard/queries/getLeaderboard.ts`:
```ts
import { Ctx } from "blitz"
import db, { User } from "db"

interface UserWithPoints extends User {
	points: number
}

interface Leaderboard {
	[userId: number]: UserWithPoints
}

export default async function getLeaderboard(data: {}, ctx: Ctx) {
	ctx.session.authorize()

	const actions = await db.action.findMany({
		include: { activity: true, user: true },
	})

	const userScore = actions.reduce<Leaderboard>((leaderboard, item) => {
		if (leaderboard.hasOwnProperty(item.user.id)) {
			const existingPoints = leaderboard[item.user.id].points
			leaderboard[item.user.id].points = existingPoints + item.activity.points
		} else {
			leaderboard[item.user.id] = {
				...item.user,
				points: item.activity.points,
			}
		}
		return leaderboard
	}, {})

	const leaderboard: UserWithPoints[] = Object.values(userScore)
	return leaderboard.sort((a, b) => b.points - a.points)
}
```

## Leaderboard in header
Let's add the link to the leaderboard in the header.
Update `./app/layouts/Layout.tsx`:
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
				<a className="inline-block py-2 px-4 text-white no-underline">Sign Up</a>
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
					<Link href="/leaderboard">
						<a className="inline-block no-underline text-white hover:text-gray-200 hover:text-underline py-2 px-4">
							Leaderboard
						</a>
					</Link>
				</li>
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

## Frontpage
Update `./app/pages/index.tsx` to show different links when the user is logged in:
```tsx
import { Link, BlitzPage } from "blitz"
import Layout from "app/layouts/Layout"
import React, { Suspense } from "react"
import { useCurrentUser } from "app/hooks/useCurrentUser"

const UnauthLinks = () => (
	<ul className="list-outside list-disc">
		<li className="mb-2 text-lg font-bold underline">
			<Link href="/signup">
				<a className="inline-block no-underline hover:text-gray-100 hover:text-underline py-1 text-base">
					Sign up
				</a>
			</Link>
		</li>
		<li className="mb-2 text-lg font-bold underline">
			<Link href="/login">
				<a className="inline-block no-underline hover:text-gray-100 hover:text-underline py-1 text-base">
					Login
				</a>
			</Link>
		</li>
	</ul>
)

const HeaderLinks = () => {
	const currentUser = useCurrentUser()

	if (currentUser) {
		return (
			<ul className="list-outside list-disc">
				<li className="mb-2 text-lg font-bold underline">
					<Link href="/leaderboard">
						<a className="inline-block no-underline text-white hover:text-gray-200 hover:text-underline py-2 px-4">
							Leaderboard
						</a>
					</Link>
				</li>
				<li className="mb-2 text-lg font-bold underline">
					<Link href="/activities">
						<a className="inline-block no-underline text-white hover:text-gray-200 hover:text-underline py-2 px-4">
							Activities
						</a>
					</Link>
				</li>
				<li className="mb-2 text-lg font-bold underline">
					<Link href="/actions">
						<a className="inline-block no-underline text-white hover:text-gray-200 hover:text-underline py-2 px-4">
							Actions
						</a>
					</Link>
				</li>
			</ul>
		)
	}
	return <UnauthLinks />
}

const Home: BlitzPage = () => {
	return (
		<>
			<h1 className="text-6xl mb-10">dotjs Leaderboard</h1>
			<Suspense fallback={<UnauthLinks />}>
				<HeaderLinks />
			</Suspense>

		</>
	)
}

Home.getLayout = (page) => <Layout title="Home">{page}</Layout>

export default Home
```

Allright! We're looking pretty good. [Back to section 8](./README.md)
