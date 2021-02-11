# Section 10 - Simple user rights
At this point, all logged in users can create an `action`. That means they can give themselves points. Let's stop that:

1) Open file `./app/actions/mutations/createAction.ts` and add the line `ctx.session.authorize("admin")`:
```ts
import { Ctx, NotFoundError } from "blitz"
import db from "db"
import { ActionInput, ActionInputType } from "../validations"

type CreateActionInput = {
  data: ActionInputType
}

export default async function createAction({ data }: CreateActionInput, ctx: Ctx) {
  ctx.session.authorize("admin")

  const { comment, activityId, userId } = ActionInput.parse(data)

  // Make sure the given activity and user exist
  const [user, activity] = await Promise.all([
    db.user.findUnique({ where: { id: data.userId } }),
    db.activity.findUnique({ where: { id: data.activityId } }),
  ])

  if (!user || !activity) throw new NotFoundError("User or activity not found")

  const action = await db.action.create({
    data: {
      activity: {
        connect: { id: activityId },
      },
      user: {
        connect: { id: userId },
      },
      createdByUser: {
        connect: { id: ctx.session.userId },
      },
      comment,
    },
  })

  return action
}
```
2) Now start the console (`blitz console`) and update your user to have the admin role:
`await db.user.update({where: {id: <YOURUSERID>}, data: {role: "admin"}})`

Good job! Now log out and back in again. You should be able to create actions 🎉
