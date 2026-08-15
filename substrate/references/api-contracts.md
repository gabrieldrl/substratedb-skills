# Public API contracts

## Contents

- Contract rules
- Shared Strata surface
- User client
- Service client
- Workspace client
- React packages
- Results and errors
- Version discipline

## Contract rules

Treat installed package exports and generated handles as authoritative. This reference records
public method families, not permission to use every method from every authority. Generated
features narrow the runtime surface to the application's compiled contract.

Use only documented package exports, generated handles, and CLI commands. Do not rely on
undocumented APIs.

## Shared Strata surface

User and service clients, connected workspace branch clients, and workflow-injected clients use
the same Strata language:

```ts
client.read({ from, select?, with? }).where(...).orderBy(...).limit(...).run();
client.write({ from }).<generatedOperation>(...).run();
```

Read-only client plans expose `.run()` and `.subscribe()`; React read plans also expose `.use()`.
Generated or runtime-imported sources may include tables, views, State Store, Object Store, Live
Map, Presence, Streams, Machines, Flags, Cohorts, and Notifications.

Generated source capabilities determine valid stages and operations. Unsupported operations must
be absent rather than failing late.

## User client

Constructor:

```ts
createUserClient({ url, branchId, publishableKey });
```

Root families:

- `read`, `write`
- `auth`
- `history`
- `analytics`
- `appUsage`
- `cohorts`
- `events`
- `flags`
- `machines`
- `mcps`
- `notifications`
- `workflows`
- `workflowRuns`
- `ai`

Auth method families include local session/user reads, remote user refresh, sign-up, sign-in,
sign-out, password and verification flows, magic link, OTP, step-up, session listing/revocation,
and account deletion as enabled by the deployed auth contract.

User History includes list, diff, preview, revert, undo, and redo for the current actor's
reversible changes. User workflow APIs include typed run/call and permitted run get/subscribe.
User feature access remains subject to compiled permissions.

## Service client

Constructor:

```ts
createServiceClient({ url, branchId, secretKey });
```

It exposes the shared application families except app-user session ownership, with trusted
service variants where declared. Service-only capabilities include:

- global or actor-selected History and branch reset;
- flag overrides;
- workflow catalog, limits, workflow-data read, branch-wide runs, snapshots, and subscriptions;
- trusted branch authority across feature APIs;
- branch operations under `service.branch`.

`service.branch` groups branch administration such as:

- schema and schema History;
- permission inspection;
- auth administration;
- analytics and entity resolution;
- flag, notification, workflow, AI, Machine, and MCP administration;
- App Usage reporting;
- State Store, Object Store, Streams, and Live Map administration where exposed;
- logs, activity, timeline, observability, and billing/usage reporting;
- secret and webhook management.

Use the installed type for exact inputs and result schemas. Do not assume operator reporting is a
Strata source.

## Workspace client

Constructor:

```ts
createWorkspaceClient({ url, workspaceId, secretKey });
```

Workspace root families:

- `get`, `getManagementOverview`, `update`, `delete`
- `members.list`, `members.update`, `members.remove`
- `invites.list`, `invites.create`, `invites.revoke`
- `billing.entitlement`, `billing.usage`, `billing.checkout`, `billing.portal`
- billing payment details, spend limits, credit quote/top-up/status where exposed
- `keys.list`, `keys.create`, `keys.revoke`, `keys.refresh`
- `projects.list`, `projects.create`
- `scopes`, `hasScope`

Project handle families:

```ts
const project = workspace.project(projectId);
```

- `project.get`, `project.update`, `project.delete`
- `project.branches.list`, `create`, `get`, `update`, `delete`, `health`, `job`
- `project.branch(branchId).connect`
- `project.branch(branchId).clientSecret`
- `project.branch(branchId).rotateClientSecret`
- `project.push.plan`, `project.push.apply`

`connect()` returns a short-lived, scope-constrained branch client. Its `schema` exposes runtime
handles under `table`, `view`, `objectStore`, `stateStore`, `liveMap`, `stream`, `presence`,
`machine`, `flag`, `cohort`, and `notification`. Use those handles with Strata.

The permanent workspace secret belongs only in trusted operator code. A browser operator app
must use a separately minted short-lived credential.

## React packages

`@substratedb/react`:

- `SubstrateProvider`
- `useSubstrate()`
- generated `read({ from, select?, with? }).<stages>.use()`

The `.use()` result exposes status, complete materialized data, error, and change metadata where
declared. Updates may arrive from local replica events, optimistic state, or remote realtime
deltas without changing the query API.

`@substratedb/workspace-react` supports Pro and Enterprise operator applications. Public families
include `WorkspaceProvider`, workspace/project hooks, connected branch-client hooks, cached
branch service queries, and cache invalidation. Use short-lived workspace credentials and
preserve workspace scopes.

## Results and errors

Strata `.run()` rejects when execution fails. Catch it when the UI or process can recover.

Workspace and branch administration methods return discriminated results:

```ts
const result = await workspace.projects.list();
if (result.error) {
  handle(result.error.code, result.error.message);
} else {
  useProjects(result.data.projects);
}
```

Runtime commands such as Machine operations throw on failure. Feature evaluations may instead
return a top-level `{ error }`; follow the installed return type. Do not ignore errors or expose
sensitive details.

## Version discipline

Before implementing:

1. Read installed package versions.
2. Inspect the generated package exports.
3. Inspect TypeScript definitions for the exact method.
4. Prefer project-local examples using the same version.
5. If this reference differs, follow installed types and flag the skill as stale.

Update this reference with SDK releases and validate representative examples against the packed
public artifacts before publishing a new skill version.
