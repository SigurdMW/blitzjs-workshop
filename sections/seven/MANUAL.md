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
3) Add the following content in the file:
```tsx
import { Suspense } from "react"
import Layout from "app/layouts/Layout"
import { Link, useQuery, useRouter, BlitzPage } from "blitz"
import getTests from "app/tests/queries/getTests"

export const ActivityList = () => {
  const router = useRouter()
  const [{ activities }] = useQuery(getTests, {
    orderBy: { id: "asc" }
  })

  return (
      <ul>
        {activities.map((activity) => (
          <li key={activity.id}>
            <Link href={`/activities/${activity.id}`}>
              <a>{activity.name}</a>
            </Link>
          </li>
        ))}
      </ul>
  )
}

const ActivitiesPage: BlitzPage = () => (
	<Suspense fallback={<div>Loading...</div>}>
		<ActivityList />
	</Suspense>
)

ActivitiesPage.getLayout = (page) => <Layout title="Activities">{page}</Layout>

export default TestsPage
```




















```ts
import { Ctx } from "blitz"
import db, { Prisma } from "db"

type GetTestsInput = Pick<Prisma.FindManyTestArgs, "where" | "orderBy" | "skip" | "take">

export default async function getTests(
  { where, orderBy, skip = 0, take }: GetTestsInput,
  ctx: Ctx
) {
  ctx.session.authorize()

  const tests = await db.test.findMany({
    where,
    orderBy,
    take,
    skip,
  })

  const count = await db.test.count()
  const hasMore = typeof take === "number" ? skip + take < count : false
  const nextPage = hasMore ? { take, skip: skip + take! } : null

  return {
    tests,
    nextPage,
    hasMore,
    count,
  }
}
```

```tsx
import { Suspense } from "react"
import Layout from "app/layouts/Layout"
import { Link, usePaginatedQuery, useRouter, BlitzPage } from "blitz"
import getTests from "app/tests/queries/getTests"

const ITEMS_PER_PAGE = 100

export const TestsList = () => {
  const router = useRouter()
  const page = Number(router.query.page) || 0
  const [{ tests, hasMore }] = usePaginatedQuery(getTests, {
    orderBy: { id: "asc" },
    skip: ITEMS_PER_PAGE * page,
    take: ITEMS_PER_PAGE,
  })

  const goToPreviousPage = () => router.push({ query: { page: page - 1 } })
  const goToNextPage = () => router.push({ query: { page: page + 1 } })

  return (
    <div>
      <ul>
        {tests.map((test) => (
          <li key={test.id}>
            <Link href={`/tests/${test.id}`}>
              <a>{test.name}</a>
            </Link>
          </li>
        ))}
      </ul>

      <button disabled={page === 0} onClick={goToPreviousPage}>
        Previous
      </button>
      <button disabled={!hasMore} onClick={goToNextPage}>
        Next
      </button>
    </div>
  )
}

const TestsPage: BlitzPage = () => {
  return (
    <div>
      <p>
        <Link href="/tests/new">
          <a>Create Test</a>
        </Link>
      </p>

      <Suspense fallback={<div>Loading...</div>}>
        <TestsList />
      </Suspense>
    </div>
  )
}

TestsPage.getLayout = (page) => <Layout title={"Tests"}>{page}</Layout>

export default TestsPage
```

```tsx
import { Suspense } from "react"
import Layout from "app/layouts/Layout"
import { Link, useRouter, useQuery, useParam, BlitzPage, useMutation } from "blitz"
import getTest from "app/tests/queries/getTest"
import deleteTest from "app/tests/mutations/deleteTest"

export const Test = () => {
  const router = useRouter()
  const testId = useParam("testId", "number")
  const [test] = useQuery(getTest, { where: { id: testId } })
  const [deleteTestMutation] = useMutation(deleteTest)

  return (
    <div>
      <h1>Test {test.id}</h1>
      <pre>{JSON.stringify(test, null, 2)}</pre>

      <Link href={`/tests/${test.id}/edit`}>
        <a>Edit</a>
      </Link>

      <button
        type="button"
        onClick={async () => {
          if (window.confirm("This will be deleted")) {
            await deleteTestMutation({ where: { id: test.id } })
            router.push("/tests")
          }
        }}
      >
        Delete
      </button>
    </div>
  )
}

const ShowTestPage: BlitzPage = () => {
  return (
    <div>
      <p>
        <Link href="/tests">
          <a>Tests</a>
        </Link>
      </p>

      <Suspense fallback={<div>Loading...</div>}>
        <Test />
      </Suspense>
    </div>
  )
}

ShowTestPage.getLayout = (page) => <Layout title={"Test"}>{page}</Layout>

export default ShowTestPage
```

```ts
import { Ctx } from "blitz"
import db, { Prisma } from "db"

type CreateTestInput = Pick<Prisma.TestCreateArgs, "data">
export default async function createTest({ data }: CreateTestInput, ctx: Ctx) {
  ctx.session.authorize()

  const test = await db.test.create({ data })

  return test
}
```

```tsx
import Layout from "app/layouts/Layout"
import { Link, useRouter, useMutation, BlitzPage } from "blitz"
import createTest from "app/tests/mutations/createTest"
import TestForm from "app/tests/components/TestForm"

const NewTestPage: BlitzPage = () => {
  const router = useRouter()
  const [createTestMutation] = useMutation(createTest)

  return (
    <div>
      <h1>Create New Test</h1>

      <TestForm
        initialValues={{}}
        onSubmit={async () => {
          try {
            const test = await createTestMutation({ data: { name: "MyName" } })
            alert("Success!" + JSON.stringify(test))
            router.push(`/tests/${test.id}`)
          } catch (error) {
            alert("Error creating test " + JSON.stringify(error, null, 2))
          }
        }}
      />

      <p>
        <Link href="/tests">
          <a>Tests</a>
        </Link>
      </p>
    </div>
  )
}

NewTestPage.getLayout = (page) => <Layout title={"Create New Test"}>{page}</Layout>

export default NewTestPage
```

[All done! Back to section 7](./README.md)