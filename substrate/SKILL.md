---
name: substrate
description: Build, review, debug, and operate applications with SubstrateDB. Use when working with @substratedb/client, @substratedb/react, @substratedb/workspace-client, Strata reads and writes, schema and permissions, workflows and events, flags, cohorts, usage, realtime, storage, generated handles, the Substrate CLI, or when creating a first Substrate application.
---

# SubstrateDB

Build against Substrate's generated, typed application contract. Treat the installed
`@substratedb/*` TypeScript definitions as authoritative. If this skill conflicts with the
installed package version, follow the installed definitions and report the mismatch.

## Start with the project

Inspect before proposing or editing:

- `package.json` and the installed `@substratedb/*` versions
- `substrate.config.ts`
- `.env` and `.env.local` key names without printing secret values
- `substrate/*.substrate.ts`
- `substrate/workflows/`
- `substrate/_substrate/generated/`
- existing imports from `@substratedb/generated/*`

Do not invent a parallel setup when the project already has one. Preserve unrelated work and
never hand-edit generated files.

## Choose authority before APIs

Select the smallest honest authority:

- Use `createUserClient` in browsers and user-facing runtimes. Enforce entity permissions.
- Use `createServiceClient` only in trusted servers, Workers, jobs, scripts, MCP
  implementations, and service workflows.
- Use `createWorkspaceClient` for workspace, project, branch, billing, key, and operator work.
  Treat a connected workspace branch client as `kind: "workspace"`, never as a service client.
- Treat React as a binding over user or scoped workspace authority, not as a fourth client.

Read [client-authorities.md](references/client-authorities.md) before constructing a client,
moving code across a trust boundary, or using workspace/operator APIs. Read
[api-contracts.md](references/api-contracts.md) for exact public method families and result
conventions.

## Think in Substrate

Express typed intent and let Substrate own transport, validation, persistence, permissions,
local replication, optimistic projection, reconciliation, realtime delivery, history, and
durable workflow execution.

Choose the narrowest primitive that matches the state and lifecycle:

- Table: durable relational application entities.
- View: derived, read-only relational results.
- State Store: durable JSON keyed by a stable application key.
- Object Store: blobs with typed paths and metadata.
- Live Map: high-frequency shared coordination state.
- Presence: transient participant state.
- Stream: durable ordered chunks or streamed output.
- Strata program: one atomic sequence of typed data operations.
- Workflow: durable, retryable, multi-step or event-driven application logic.
- Workflow data: private, scoped, retained working state for one workflow.
- Flag: a rollout or variant decision, never authorization.
- Cohort: named entity segmentation, never an ACL.
- Event: one canonical application fact that may drive workflows and analytics.
- App Usage: metering and quota enforcement composed by application logic.

Read [mental-model.md](references/mental-model.md) and
[choosing-features.md](references/choosing-features.md) for design work. Read
[schema-and-definitions.md](references/schema-and-definitions.md) before changing application
schema. Keep feature modules independent; compose cross-feature product policy in application
workflows and events.

## Use the generated contract

- Import generated source, relation, entity, event, workflow, and store handles.
- Start every Strata plan with explicit `read({ from })` or `write({ from })`.
- Use typed column predicates and `with` for relations.
- Use `.run()` for one-off execution, `.subscribe()` outside React, and `.use()` in React.
- Render the complete reactive result. Do not manually patch a Substrate cache.
- Await a write when later procedural work depends on the authoritative result.
- Compose dependent reads and writes into one Strata program when they must be atomic.
- Put arbitrary branching, provider calls, waits, or retries in normal server code or a
  workflow, not in a Strata callback.

Read [strata-and-react.md](references/strata-and-react.md) before changing data access,
realtime behavior, optimistic UI, or React integration.

## Define security once

- Define entity and resource rules in `permissions.substrate.ts`.
- Treat every omitted operation as denied.
- Derive identity from verified runtime context, never request-body actor fields.
- Check both existing `row` and proposed `value` when an update could move ownership or scope.
- Test denial cases with a user client; do not use service authority to hide a bad rule.
- Keep permanent branch and workspace secrets out of browsers, URLs, logs, and public env.
- Use flags for delivery and permissions for security even when both guard the same feature.

Read [permissions-and-security.md](references/permissions-and-security.md) for auth,
credentials, permissions, encrypted fields, and operator scope rules.

## Use workflows for durable application logic

Use a workflow when work must survive the initiating request, retry durably, coordinate typed
steps, react to committed events, call AI/providers, emit progress, or retain scoped working
state. Do not wrap ordinary CRUD in a workflow without a durability or orchestration reason.

Inside steps:

- Validate strict typed input and output.
- Make effects safe to retry.
- Use `workflow.substrate` for normal application state.
- Use `workflow.data` only for the workflow's private internal tables.
- Keep internal workflow data out of user-facing product models.
- Emit bounded progress and non-sensitive structured logs.
- Throw to fail a node; logging an error does not fail it.
- Declare explicit entity scope when authority or isolation belongs to an entity.

Read [workflows.md](references/workflows.md) completely before authoring or reviewing a
workflow.

## Create a first application

When asked to create or teach a first Substrate application, read
[cli.md](references/cli.md), [project-lifecycle.md](references/project-lifecycle.md), and
[first-app.md](references/first-app.md) completely.

Guide the user from authentication through a running, verified application. Run safe commands
directly. Pause for browser login, workspace selection, branch selection, or other user-only
interaction, then resume.

Use `substrate init` as the owner of package installation, scaffolding, generation, branch
selection, optional `.env.local` writing, and Next provider wiring. Do not duplicate those steps
unless init reports a specific failure.

Finish with:

1. A small real feature using generated handles.
2. A successful artifact sync.
3. `substrate generate --check`.
4. A production application build.
5. Observed browser behavior and realtime behavior when browser access is available.

Report static, sync, browser, realtime, staging, and production evidence separately. Never treat
generated files or a green TypeScript check as proof that the connected runtime works.

## Review quality

Read [anti-patterns.md](references/anti-patterns.md) for implementation or code-review tasks.
Reject or repair code that bypasses generated contracts, weakens authority boundaries, leaks
secrets, duplicates optimistic state, misuses workflow storage, couples independent modules, or
keeps a legacy path beside its replacement.

## Validate changes

Use checks proportional to the change:

1. Regenerate after definition changes.
2. Run `npx substrate generate --check`.
3. Run the project's typecheck, lint, tests, and production build.
4. Run `npx substrate dev` or `npx substrate migrate dev` when connected runtime artifacts must
   change.
5. Exercise user success and permission denial paths.
6. Exercise workflow retries, scope, progress, and terminal failure where applicable.
7. Exercise optimistic, reload, second-tab realtime, offline, rollback, and reconnect behavior
   where applicable.
8. State which environments were and were not tested.
