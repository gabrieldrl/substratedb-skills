# Substrate anti-patterns

## Authority and security

Do not:

- expose branch/workspace secrets in browser code, public env, URLs, or logs;
- create a service client in client code or disguise a workspace branch session as one;
- trust actor, owner, organization, or role fields from an untrusted body as identity;
- bypass a user permission denial with service authority;
- use flags, cohorts, UI checks, or route guards as authorization;
- authorize an update from `row` while letting `value` move ownership or scope;
- copy decrypted data into unencrypted fields, events, analytics, or logs.

## Strata and generated contracts

Do not:

- omit `from` or replace typed handles with strings, raw SQL, or raw query objects;
- use an include-style relation API instead of typed `with`;
- edit or duplicate generated source, entity, event, workflow, or store contracts;
- hand-author source or workflow IDs;
- split dependent atomic writes across calls or `Promise.all`;
- put provider calls or arbitrary callbacks inside a Strata plan;
- keep a legacy query path beside its replacement.

## React and realtime

Do not:

- copy `.use().data` into component state;
- append deltas or patch and roll back a second cache manually;
- subscribe in an effect when the generated read supports `.use()`;
- render a delta as if it were the complete materialized result;
- claim realtime proof from a build or one-tab test.

## Data and workflows

Do not:

- use one State Store document as a relational database;
- use Presence for durable truth or workflow data for product records;
- put large blobs in table JSON instead of Object Store;
- use analytics or logs as operational truth;
- query, index, relate, order, group, or aggregate an encrypted column;
- wrap CRUD in a workflow without a durability or orchestration need;
- make retryable effects non-idempotent;
- infer workflow scope from an input field or access workflow data outside its steps;
- log an error instead of throwing to fail a workflow node;
- emit the same fact through separate event definitions;
- emit an event before its authoritative change commits.

## Features and lifecycle

Do not:

- use a flag as an ACL or a cohort as membership/ownership;
- couple reusable modules to application-specific quota or policy logic;
- record usage before an operation is accepted unless billing semantics require it;
- use Strata for workspace, project, or branch management;
- run the unrelated unscoped `substrate` npm package;
- assume global checkout rewrites project env or controls `substrate dev`;
- use a project ID as `SUBSTRATE_BRANCH_ID`;
- replace `substrate init` package installation with a divergent manual set;
- edit generated output to satisfy `generate --check`;
- fork app code by environment instead of changing env/bindings.

## Code quality and evidence

Do not:

- bypass generated types with `any`, unchecked casts, or non-null assertions;
- recreate clients during render;
- swallow rejected Strata execution or report success after catching an error;
- mix server-only and client-only code in one module;
- retain compatibility shims after an intentional replacement;
- claim browser, realtime, staging, or production proof from static checks.

Prefer generated types, explicit authority, deterministic inputs, bounded outputs,
idempotent effects, and tests at the behavior boundary.
