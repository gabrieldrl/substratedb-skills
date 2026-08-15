# Substrate mental model

## Contract first

Treat Substrate as the runtime for a compiled application contract:

1. Declare resources in `substrate/*.substrate.ts`.
2. Run `substrate generate` or `substrate dev`.
3. Import handles from `@substratedb/generated/*`.
4. Use those handles through user, service, workflow, workspace, React, or transport surfaces.

Do not duplicate the schema, permission rules, query language, workflow IDs, or client cache in
application code. A generated-contract mismatch should fail; do not fall back to stringly typed or
legacy APIs.

## Choose authority deliberately

- **User client:** browser or end-user code; permission predicates remain authoritative.
- **Service client:** trusted server code; never expose its secret or move it into the browser.
- **Workflow client:** `workflow.substrate`, injected as the declared `user` or `service` mode.
- **Workspace branch client:** workspace and branch administration, not normal application traffic.

Permissions answer whether an actor may perform an operation. Flags, cohorts, metering, and UI
visibility do not replace permissions.

## Place data and logic

- Put durable, user-visible domain truth in normal schema resources.
- Use tables for relational records, views for derived relational reads, State Store for keyed JSON,
  and Object Store for files plus queryable metadata.
- Use a Strata program when generated data operations must commit atomically.
- Use trusted server code for short, request-bound computation.
- Use a workflow when work must retry durably, outlive the request, react to events, call providers,
  or expose progress and History.
- Use workflow tables only for private, scoped working state. Write product state through
  `workflow.substrate` into normal resources.

Treat committed application data and events as the causal boundary. Trigger downstream work from
canonical events; do not invent a second state transition in analytics, logs, or a client cache.

## Compose features in the application

Keep feature modules independent. Compose product policy in application source:

- data event -> workflow -> provider call -> table update;
- activity event -> cohort membership -> flag evaluation;
- machine event -> workflow -> App Usage consumption;
- object event -> workflow -> classification -> product record.

Do not make Machines depend on App Usage, Flags grant authorization, or Analytics become operational
state merely because one application combines those features.

## Ask before building

1. What is the source of truth?
2. Is the actor a user, trusted service, or workspace operator?
3. Must this commit atomically?
4. Must it survive the request or retry?
5. Are repeated external effects idempotent?
6. Must the UI query this as normal product data?
7. Is this authorization, delivery, segmentation, or metering?
8. Is the rule application composition or reusable feature behavior?
9. Which connected-runtime behavior needs testing beyond compilation?
