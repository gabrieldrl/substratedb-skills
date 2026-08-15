# Strata and React

## Contents

- Generated sources
- Reads and relations
- Writes and atomic programs
- React and realtime
- Optimistic and offline behavior
- History

## Generated sources

Import application sources from the generated package. Do not construct source names, fields,
relations, or operators from strings.

```ts
import { Orgs, Tasks } from "@substratedb/generated/schema";

const tasks = Tasks();
const orgs = Orgs();
```

Every read and write requires `from`:

```ts
substrate.read({ from: tasks });
substrate.write({ from: tasks });
```

Never omit `from`, even when a source appears inferable.

## Reads and relations

Run a typed read once:

```ts
const rows = await substrate
  .read({
    from: tasks,
    select: [tasks.id(), tasks.title(), tasks.completed()] as const,
  })
  .where(tasks.org().eq(orgId))
  .where(tasks.completed().eq(false))
  .orderBy(tasks.createdAt().desc())
  .limit(25)
  .run();
```

Use `with` for relations rather than ad hoc follow-up loops or include-style APIs:

```ts
const rows = await substrate
  .read({
    from: tasks,
    select: [tasks.id(), tasks.title()] as const,
    with: [
      tasks.orgRelation({
        select: [orgs.id(), orgs.name()] as const,
      }),
    ] as const,
  })
  .where(tasks.id().eq(taskId))
  .run();
```

Use generated columns such as `tasks.org()` and `tasks.createdAt()`. Object Store fields keep
their qualified names, for example `files["metadata.category"]()` and `files["path.orgId"]()`.

## Writes and atomic programs

Use only operations exposed by the generated source:

```ts
const created = await substrate
  .write({ from: tasks })
  .create({ title: "Ship", completed: false, ownerUser: userId, org: orgId })
  .run();

await substrate.write({ from: tasks }).update(created.id, { completed: true }).run();
```

Compose dependent operations into a single atomic program:

```ts
const result = await substrate
  .write({ from: tasks })
  .create({ title: "Before", completed: false, ownerUser: userId, org: orgId }, taskId)
  .write({ from: tasks })
  .update(taskId, { title: "After" })
  .read({ from: tasks })
  .where(tasks.id().eq(taskId))
  .run();
```

Later stages observe earlier writes. Do not split dependent writes across `Promise.all`. Do not
place arbitrary callbacks inside a Strata program; use application code or a workflow.

## React and realtime

Use the same read with `.use()` in a client component:

```tsx
"use client";

import { useSubstrate } from "@substratedb/react";
import { Tasks } from "@substratedb/generated/schema";

export function TaskList({ orgId }: { orgId: string }) {
  const substrate = useSubstrate();
  const tasks = Tasks();
  const query = substrate
    .read({ from: tasks })
    .where(tasks.org().eq(orgId))
    .orderBy(tasks.createdAt().desc())
    .use();

  if (query.status === "loading") return <p>Loading...</p>;
  if (query.status === "error") return <p>{query.error?.message}</p>;
  return query.data?.map((task) => <p key={task.id}>{task.title}</p>);
}
```

Render `query.data`, which is the complete current materialized result. Read `lastChange` only
for a genuine change-specific effect. Do not append deltas into component state.

Outside React, use `.subscribe()` on a read plan. The initial callback receives the complete
result; later notifications update that same materialization.

## Optimistic and offline behavior

Replica-backed browser writes project into matching local reads immediately. Substrate owns the
persistent cache, mutation outbox, cross-tab coordination, replay, reconciliation, and rollback.

```ts
async function toggle(id: string, completed: boolean) {
  try {
    await substrate.write({ from: tasks }).update(id, { completed }).run();
  } catch (error) {
    showError(error);
  }
}
```

Keep rendering the `.use()` result before, during, and after the write. Do not maintain a second
optimistic copy or manually undo a rejected change. For replica-backed writes, `.run()` confirms
durable queueing; matching reads later reflect the authoritative commit or rollback.

Test at least immediate projection, rejection rollback, reload persistence, second-tab updates,
disconnect, offline write, reconnect, and convergence when the feature depends on them.

## History

History is a root client surface. Use generated resource handles, never resource-name strings:

```ts
const changes = await substrate.history.list({
  resource: Tasks(),
  reversible: true,
  limit: 25,
});
```

User history is permanently scoped to that user's reversible changes. A service client can read
global history or select `userId`; only a service client can reset the branch to an earlier
version. Preview and diff destructive history operations before applying them.
