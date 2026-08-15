# Choosing Substrate features

## Resource guide

| Need                                                                    | Use            | Avoid                                            |
| ----------------------------------------------------------------------- | -------------- | ------------------------------------------------ |
| Relational entities, ownership, joins, indexes, permissions, or history | Table          | Encoding records in one JSON value               |
| Reusable aggregate or derived relational read                           | View           | Persisting duplicate derived state               |
| Durable JSON addressed by one application key                           | State Store    | Using one value as a relational database         |
| Files with declared paths and typed metadata                            | Object Store   | Storing large bytes in table JSON                |
| Fast shared keyed values, often with short TTL                          | Live Map       | Treating coordination state as durable truth     |
| Ephemeral room participants, cursors, or typing state                   | Presence       | Using presence for ownership or business state   |
| Durable ordered chunks with open/append/close/abort lifecycle           | Stream         | Using a stream for independently queried records |
| Atomic dependent reads and writes                                       | Strata program | Splitting one invariant across requests          |
| Durable, retryable, multi-step orchestration                            | Workflow       | Using a workflow for one atomic data change      |

Use generated handles and operations. Unsupported operations should be absent from the generated
type surface; do not recreate them with raw HTTP or provider SDKs.

## Flags and cohorts

A flag chooses behavior or a variant for one target. A cohort is a named set of entity members.
Defining a cohort does **not** calculate membership: application code or a workflow must add and
remove members.

Use flags for staged delivery, experiments, plan presentation, and reversible overrides. Use
cohorts when several rules need the same named segment. Add TTL for temporary membership; re-adding
a member renews its lease.

Use stable entity targets and declared context. Never use flags or cohorts for authorization,
ownership, or organization membership.

```ts
import { defineFlags } from "@substratedb/core";
import { org } from "./entities.substrate";

export default defineFlags((f) => ({
  flags: {
    newDashboard: f
      .flag({ title: "New dashboard" })
      .target(org)
      .boolean({
        default: false,
        rules: [f.when(f.cohort("betaOrgs")).enabled()],
      })
      .overrides.target(),
  },
}));
```

Keep permission rules on the underlying reads, writes, and actions even when the UI hides them.

## Events and analytics

Define each application event once in `events.substrate.ts`.

- Use `substrate.events.emit(handle, payload)` for a canonical journal event that preserves
  correlation/causation, triggers bindings, and projects declared analytics subjects.
- Use the generated analytics capture call only for imported analytics facts that must not trigger
  workflows.
- Keep analytics and observability as reporting surfaces, never product source of truth.

Do not define duplicate events for analytics and workflows, or use Strata for operator analytics
and branch management.

## App Usage

Use App Usage for product meters and limits. Declare entity scopes with canonical entity handles,
then compose enforcement in application workflows or trusted actions.

- `checkLimit` checks prospective capacity without recording usage.
- `consume` records usage when allowed and returns an allowed/denied result.
- `require` consumes usage and throws when denied or on error.

Keep metadata bounded and non-sensitive. Make retries idempotent: a retried provider call or workflow
step must not charge the same real-world unit twice.

## Other generated features

Use Machines, MCPs, AI, Notifications, and Secret Store through their generated feature surfaces
when Substrate should own authority, events, metering, History, or lifecycle. Keep secrets in Secret
Store declarations or trusted runtime environment variables—never application rows, logs, or browser
code.

## Composition check

Good:

- data event -> service workflow -> AI call -> application table update;
- activity event -> workflow updates cohort -> flag rule reads cohort;
- machine lifecycle event -> workflow records App Usage;
- object event -> workflow classifies file -> table update.

Bad:

- Flags granting access without permission rules;
- cohort membership standing in for ownership;
- analytics rows serving as operational state;
- UI querying workflow-internal tables;
- reusable feature modules importing one another for application policy.
