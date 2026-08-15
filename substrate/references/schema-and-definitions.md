# Schema and definitions

## Model application data

Define application schema in `substrate/schema.substrate.ts` with the installed
`@substratedb/core` exports. Do not guess a helper signature: inspect the package types and
project-local examples when the installed version differs.

Use tables for durable relational entities. Substrate supplies row IDs; declare domain fields,
references, reverse relations, timestamps, and indexes explicitly.

```ts
import {
  boolean,
  date,
  defineSchema,
  enum_,
  index,
  many,
  now,
  ref,
  string,
  table,
} from "@substratedb/core";

export default defineSchema({
  users: table({
    email: string(),
    memberships: many("memberships").via("user"),
  }),
  orgs: table({
    name: string(),
    memberships: many("memberships").via("org"),
    tasks: many("tasks").via("org"),
  }),
  memberships: table({
    user: ref("users", { onDelete: "cascade" }),
    org: ref("orgs", { onDelete: "cascade" }),
    role: enum_("memberRole", ["admin", "member"]),
  }).indexes([index("memberships_user_org_idx", ["user", "org"])]),
  tasks: table({
    org: ref("orgs", { onDelete: "cascade" }),
    title: string(),
    completed: boolean().default(false),
    createdAt: date().default(now()),
    updatedAt: date().default(now()).onUpdate(now()),
  }).indexes([index("tasks_org_idx", ["org"])]),
});
```

Use a normal application table for profiles and memberships; a Substrate workspace is an
operator boundary, not an end-user tenant. Use a branch as the deployment environment, not as an
organization.

## Keep contracts generated

- Add reverse `many(...).via(...)` relations only when callers need them.
- Index actual filter, ordering, uniqueness, and relation access paths.
- Use a view for shared read-only relational derivation, not writable state.
- Use `.encrypted()` only on supported table columns and accept its query restrictions.
- Choose State Store, Object Store, Live Map, Presence, or Stream by lifecycle; see
  [choosing-features.md](choosing-features.md).
- Keep workflow tables under the workflow's `data.tables`; never add them to application schema.

After editing any `.substrate.ts` definition, run the connected sync and regenerate handles:

```bash
npx substrate dev
npx substrate generate --check
```

Import schema handles from `@substratedb/generated/schema`. Never duplicate row types, construct
source names as strings, or edit generated output.
