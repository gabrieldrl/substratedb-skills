# Project lifecycle

Read [cli.md](cli.md) for command details and [first-app.md](first-app.md) only when building the
tutorial.

## Initialized layout

Current init creates:

```text
my-app/
  substrate/
    schema.substrate.ts
    permissions.substrate.ts
    entities.substrate.ts
    workflows.substrate.ts
    events.substrate.ts
    usage.substrate.ts
    machines.substrate.ts
    mcps.substrate.ts
    flags.substrate.ts
    notifications.substrate.ts
    analytics.substrate.ts
    auth.substrate.ts
    secrets.substrate.ts
    webhooks.substrate.ts
    workflows/
    _substrate/
      generated/
        package.json
        ...generated modules
  substrate.config.ts
  .env.local                     # only when branch env writing is approved
  app/substrate-provider.tsx     # or src/app; only when Next wiring is approved
```

Init also comments the provider file, injected layout import, and provider wrapper so their origin
is visible.

## Ownership

Developers own `.substrate.ts` definitions, workflow implementations, `substrate.config.ts`, and
the editable Next provider. The compiler owns `substrate/_substrate/generated/`; never edit it.

Import resource definitions from their source files and application handles from
`@substratedb/generated/*`. Do not hardcode physical IDs or duplicate registries.

## Development loop

Run the app server and watcher in separate terminals:

```bash
npx substrate dev
npm run dev
```

`substrate dev` targets this directory's env, not global checkout. After a definition change,
allow the watcher to sync and regenerate. Without a watcher, use:

```bash
npx substrate migrate dev
npx substrate generate
```

Stop temporary processes when finished unless the user asks to keep them running.

## Branch promotion

Treat each branch as a complete contract and runtime target. Use `substrate push` to review and
apply local artifacts to another branch. Check source, target, conflicts, destructive schema
changes, and expected artifact keys. Require explicit Production authorization.

## Evidence boundary

Keep these results distinct:

- static checks and `generate --check`;
- framework production build;
- connected branch sync;
- observed browser operations;
- observed realtime convergence;
- local platform, staging, and Production execution.

Do not infer a stronger result from a weaker one. Cleanup proves resource hygiene, not success.
