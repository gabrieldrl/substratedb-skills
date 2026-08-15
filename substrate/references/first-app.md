# First Substrate application

Build a connected Next.js counter that proves initialization, generated contracts, permissions,
optimistic writes, persistence, and realtime updates. Read [cli.md](cli.md) for command semantics
and recovery.

## 1. Authenticate

```bash
node --version
npx @substratedb/cli@latest login
```

Use Node.js 20 or newer. Wait for browser login; never ask the user to paste a token or callback
URL.

If the user authorizes a new backend project, create it now. Otherwise use an existing project.

```bash
npx @substratedb/cli@latest project create "My Substrate App" my-substrate-app
```

## 2. Create and initialize Next.js

Use the standard Next.js defaults:

```bash
npx create-next-app@latest my-substrate-app --yes
cd my-substrate-app
npx @substratedb/cli@latest init
```

During init, select the intended workspace, project, and branch. For a new tutorial app, recommend
allowing `.env.local` and Next provider wiring. Never force those choices in an existing app.

Confirm that init created `substrate/`, `substrate.config.ts`,
`substrate/_substrate/generated/`, and either `app/substrate-provider.tsx` or
`src/app/substrate-provider.tsx`. Check only env key names; never print values. Ensure
`NEXT_PUBLIC_SUBSTRATE_SECRET_KEY` does not exist.

## 3. Define the counter

Replace `substrate/schema.substrate.ts`:

```ts
import { defineSchema, number, stateStore } from "@substratedb/core";

export default defineSchema({
  counter: stateStore({
    value: number(),
    maxBytes: 1024,
  }),
});
```

Replace `substrate/permissions.substrate.ts`:

```ts
import { definePermissions, s } from "@substratedb/permissions";

export default definePermissions((p) => {
  const appUser = s.or(s.user.anonymous(), s.user.signedIn());

  return p.define(
    p.stateStore("counter").allow({
      read: () => appUser,
      write: () => appUser,
      delete: () => appUser,
    }),
  );
});
```

Anonymous writes are acceptable only for this isolated tutorial. Production rules must scope
access to verified users or entities and test denial paths.

## 4. Sync and use the generated handle

Start the watcher and wait for its first successful sync:

```bash
npx substrate dev
```

Then replace `app/page.tsx` or `src/app/page.tsx`:

```tsx
"use client";

import { Counter } from "@substratedb/generated/stores";
import { useSubstrate } from "@substratedb/react";

export default function Home() {
  const substrate = useSubstrate();
  const counter = Counter();
  const query = substrate.read({ from: counter }).where(counter.key().eq("global")).use();
  const value = query.data?.[0]?.value ?? 0;

  async function increment() {
    await substrate.write({ from: counter }).incr("global", 1).run();
  }

  return (
    <main>
      <h1>My first Substrate app</h1>
      <output>{value}</output>
      <button type="button" onClick={increment} disabled={query.status === "loading"}>
        Increment
      </button>
      {query.status === "error" ? <p role="alert">{query.error?.message}</p> : null}
    </main>
  );
}
```

Treat generated exports as authoritative. If a handle differs, inspect
`@substratedb/generated/stores`; do not edit generated files.

## 5. Run and verify

Keep `npx substrate dev` running. In a second terminal:

```bash
npm run dev
```

Then run:

```bash
npx substrate status
npx substrate generate --check
npm run build
```

Verify in the browser:

1. Increment once and observe the immediate update.
2. Reload and confirm persistence.
3. Open a second tab, increment, and confirm both tabs converge.
4. Confirm browser output contains no permanent secret.

Report build, connected sync, browser behavior, and two-tab realtime evidence separately. Stop
temporary processes unless the user asks to keep them running.
