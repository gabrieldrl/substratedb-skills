# Substrate CLI

## Invocation

Before init installs the CLI locally, run the scoped package:

```bash
npx @substratedb/cli@latest login
npx @substratedb/cli@latest init
```

The unscoped npm package `substrate` is unrelated. After init, use the local binary:

```bash
npx substrate status
npx substrate dev
```

Run project commands from the directory containing `substrate.config.ts`.

## Commands

| Command                                    | Purpose                                                                     |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| `login`                                    | Sign in globally through the Substrate console                              |
| `logout`                                   | Revoke the global session and clear it from Keychain                        |
| `status`                                   | Show the account session and current project's branch connection            |
| `init`                                     | Install, scaffold, generate, connect a branch, and optionally wire Next.js  |
| `checkout [branch]`                        | Select a global workspace/project/branch; alias `switch`                    |
| `project create <name> [slug] [workspace]` | Create a project and wait for Development                                   |
| `dev`                                      | Sync artifacts and declared secrets, generate, then watch                   |
| `migrate dev`                              | Perform one development sync without watching                               |
| `generate`                                 | Regenerate local `@substratedb/generated`                                   |
| `generate --check`                         | Fail if source, declared dependency, or installed generated output is stale |
| `push`                                     | Review and apply local artifacts to another branch                          |
| `secrets status`                           | List declared secret names and status                                       |
| `secrets sync`                             | Sync declared secret values from the configured env file                    |

## Account and project selection

Login opens the browser and stores an account session in the platform credential store. It does
not select a project or edit project env files.

```bash
npx @substratedb/cli@latest login
npx @substratedb/cli@latest checkout
npx @substratedb/cli@latest checkout Development
```

Checkout changes only the global CLI selection. Create a backend project when needed:

```bash
npx @substratedb/cli@latest project create "My App" my-app
```

The command selects a workspace, creates the project, waits up to two minutes for its Development
branch, and stores that selection globally.

## Init contract

Run init from an application root containing `package.json`:

```bash
npx @substratedb/cli@latest init
```

Init:

1. Creates `substrate/`, `substrate/workflows/`, all current `.substrate.ts` definitions, and
   `substrate.config.ts`.
2. Installs matching CLI-version releases of `@substratedb/client`, `@substratedb/react`,
   `@substratedb/core`, `@substratedb/workflows`, `@substratedb/permissions`, and
   `@substratedb/strata`; installs `@substratedb/cli` as a development dependency.
3. Generates `substrate/_substrate/generated/` and declares it as the local
   `@substratedb/generated` dependency.
4. Prompts for workspace, project, and branch. Interactive init starts login when necessary.
5. Optionally merges the branch connection into `.env.local` without changing unrelated lines.
6. If `app/layout.tsx` or `src/app/layout.tsx` exists, optionally creates
   `substrate-provider.tsx` beside it and wraps `{children}` with `SubstrateProviders`.

Generated provider code and injected layout sections carry `substrate init` comments. Init asks
before replacing changed definitions or an existing provider. Let the user answer those prompts;
do not force overwrites in an existing project.

The env write uses mode `0600` and never writes a public secret. It manages:

```text
SUBSTRATE_PUBLIC_URL
SUBSTRATE_BRANCH_ID
SUBSTRATE_PUBLISHABLE_KEY
SUBSTRATE_SECRET_KEY
NEXT_PUBLIC_SUBSTRATE_PUBLIC_URL
NEXT_PUBLIC_SUBSTRATE_BRANCH_ID
NEXT_PUBLIC_SUBSTRATE_PUBLISHABLE_KEY
```

## Development

```bash
npx substrate dev
```

`dev` reads `.env`, then overriding `.env.local`, from this project. It does not target the global
checkout selection. It compiles and syncs schema, permissions, workflows, events, and other
features; regenerates handles; syncs declared secrets; then watches changes.

For one-shot or CI work:

```bash
npx substrate migrate dev
npx substrate generate
npx substrate generate --check
```

Never edit `substrate/_substrate/generated/`. Commit it and use `generate --check` in CI.

## Push and secrets

```bash
npx substrate push
npx substrate secrets status
npx substrate secrets sync
```

Review the push target and destructive changes before confirmation. Require explicit authorization
for Production. Never print or commit secret values.

## Failure handling

- Not signed in: run `login`, then retry.
- No project: run `project create`, then retry init.
- Env write declined: rerun init and approve it, or add one canonical branch connection set.
- Invalid credentials: verify this project's env and intended branch without printing values.
- Stale generated output: run `generate`; never patch generated files.
- Dev sync rejected: report the connected sync failure separately from local build results.
