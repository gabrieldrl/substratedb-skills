# Client authorities

## Contents

- Authority matrix
- User client
- Service client
- Workspace client
- React authority
- Environment boundaries

## Authority matrix

| Client    | Intended location                   | Credential                                        | Authority                   |
| --------- | ----------------------------------- | ------------------------------------------------- | --------------------------- |
| User      | Browser or user-facing runtime      | Publishable key plus app-user session             | Entity permissions enforced |
| Service   | Trusted server, Worker, job, script | Permanent branch secret                           | Trusted branch authority    |
| Workspace | Workspace and branch administration | Workspace secret or short-lived scoped credential | Workspace scopes enforced   |

## User client

```ts
import {
  createUserClient,
  resolveSubstrateUserEnv,
  substrateClientEnvFromProcess,
} from "@substratedb/client";
import "@substratedb/generated/schema";

const substrate = createUserClient(resolveSubstrateUserEnv(substrateClientEnvFromProcess()));
```

The explicit constructor contract is:

```ts
createUserClient({ url, branchId, publishableKey });
```

Use it for browser and app-user operations. It provides app auth, branch-scoped anonymous
identity, generated Strata, user-safe feature APIs, user workflow invocation, and user-scoped
History. Permission predicates apply to reads, writes, relations, realtime, user workflows, and
feature commands.

Never treat an invalid session as anonymous. Never accept a body-supplied user ID as verified
identity.

## Service client

```ts
import { createServiceClient, resolveSubstrateServiceEnv } from "@substratedb/client";
import "@substratedb/generated/schema";

const service = createServiceClient(resolveSubstrateServiceEnv(process.env));
```

The explicit constructor contract is:

```ts
createServiceClient({ url, branchId, secretKey });
```

Use it in trusted servers, Workers, jobs, scripts, migrations, MCP implementations, and
workflows declared with `client: "service"`. It shares application data and feature surfaces,
adds global or actor-aware History, and exposes operator-style branch APIs under
`service.branch`.

Do not place it in a React provider. Do not expose `SUBSTRATE_SECRET_KEY` to a browser. Do not
use service authority merely to make a permission error disappear.

## Workspace client

```ts
import { createWorkspaceClient, resolveSubstrateWorkspaceEnv } from "@substratedb/workspace-client";

const workspace = createWorkspaceClient(resolveSubstrateWorkspaceEnv(process.env));
```

The explicit constructor contract is:

```ts
createWorkspaceClient({ url, workspaceId, secretKey });
```

Use it for workspace metadata, members, invites, billing, workspace keys, projects, branches,
branch connection, branch secrets, and push operations.

```ts
const connected = await workspace.project(projectId).branch(branchId).connect();
if (connected.error) throw connected.error;
const branch = connected.data;
// branch.kind === "workspace"
```

`connect()` mints a short-lived branch credential constrained by the workspace key's scopes. It
does not return or imply permanent service authority. The connected client exposes runtime
schema handles and permitted Strata or branch operations.

Obtain a permanent branch secret only through separately authorized trusted code:

```ts
const secret = await workspace.project(projectId).branch(branchId).clientSecret();
if (secret.error) throw secret.error;
provisionToServer(secret.data.secretKey);
```

Provision the secret only to an application server.

## React authority

React is not a fourth authority.

```tsx
<SubstrateProvider client={userClient}>{children}</SubstrateProvider>
```

`SubstrateProvider` accepts a user client or a connected `kind: "workspace"` branch client. It
must not retain a permanent service client.

Operator applications use `WorkspaceProvider` with a separately minted short-lived workspace
credential. When connected to a branch, it can provide the scoped branch client to
`SubstrateProvider` so `useSubstrate()` retains the same authority.

## Environment boundaries

Browser bundle values:

- `NEXT_PUBLIC_SUBSTRATE_PUBLIC_URL`
- `NEXT_PUBLIC_SUBSTRATE_BRANCH_ID`
- `NEXT_PUBLIC_SUBSTRATE_PUBLISHABLE_KEY`
- equivalent `VITE_SUBSTRATE_*` values

`substrate init` writes the canonical `SUBSTRATE_*` values plus these browser-safe mirrors.

Trusted application-server values:

- `SUBSTRATE_PUBLIC_URL`
- `SUBSTRATE_BRANCH_ID`
- `SUBSTRATE_SECRET_KEY`

Trusted operator values:

- `SUBSTRATE_WORKSPACE_ID`
- `SUBSTRATE_WORKSPACE_SECRET_KEY`

Changing targets should change env and bindings, not application code. Never create a public
variant of a secret key.
