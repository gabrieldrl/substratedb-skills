# Permissions and security

## Define entity rules

Define permissions in `substrate/permissions.substrate.ts`. Omitted operations are denied.
Use named predicates for reusable membership, ownership, and capability checks.

```ts
import { definePermissions, s } from "@substratedb/permissions";

export default definePermissions((p) => {
  const isMember = p.fn("isMember", (orgId) =>
    s.exists("memberships", (membership) =>
      s.and(s.eq(membership.user, s.user.id()), s.eq(membership.org, orgId)),
    ),
  );

  return p.define(
    p.table("tasks").allow({
      read: ({ row }) => s.and(s.user.signedIn(), isMember(row.org)),
      create: ({ value }) => s.and(s.user.signedIn(), isMember(value.org)),
      update: ({ row, value }) =>
        s.and(s.user.signedIn(), isMember(row.org), isMember(value.org), s.eq(row.org, value.org)),
      delete: ({ row }) => s.and(s.user.signedIn(), isMember(row.org)),
    }),
  );
});
```

Derive identity from verified runtime context. Never pass permissions to a client constructor or
treat request-body claims as identity.

## Protect updates and relations

Check both `row` and `value` when an update can change ownership, tenant, parent, or scope:

```ts
update: ({ row, value }) => s.and(isMember(row.org), isMember(value.org), s.eq(value.org, row.org));
```

Protect relation reads and writes independently. A filtered parent read does not authorize a
child operation.

## Keep authorities distinct

- User clients enforce application permissions.
- Service clients use trusted branch authority and stay server-side.
- Workspace branch clients enforce workspace scopes and remain `kind: "workspace"`.
- Workflow-injected clients carry the workflow's declared authority.

Never cast or wrap one credential kind as another. A short-lived workspace connection is not a
branch secret.

## Handle secrets and encrypted columns

Keep permanent secrets out of client bundles, public env, URLs, analytics, events, logs, and
committed files. Use Secret Store or server-only env for provider credentials.

Use `.encrypted()` only on supported table columns whose plaintext must not be stored in
Substrate-managed storage. Encrypted values cannot be filtered, ordered, grouped, aggregated, indexed,
unique, related, embedded, or assigned database-generated values.

Authorized reads return plaintext, and browser replicas may retain it. This is encryption at
rest, not end-to-end encryption or information-flow tracking. Encrypt every persisted
destination that needs protection.

## Test denials

Test:

- anonymous, non-member, member, and owner/admin access;
- cross-tenant reads, writes, relations, and ownership changes;
- realtime snapshots and later deltas;
- optimistic denial rollback;
- user workflow operations;
- invalid and expired sessions;
- workspace scope denial;
- absence of permanent secrets from browser output.
