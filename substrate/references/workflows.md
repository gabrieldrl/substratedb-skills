# Workflows

## Contents

- Choose a workflow
- Author and register
- Graphs and retries
- Scope and authority
- Internal workflow data
- Events, progress, logs, and History
- Review checklist

## Choose a workflow

Use a workflow for work that must survive the request, retry durably, coordinate typed steps, call
external providers, react to committed events, or expose progress and History.

Use a Strata program for one atomic chain of generated data operations. Use trusted server code for
short request-bound computation that needs neither durable execution nor replay.

## Author and register

Define typed steps, compose a static graph, then register the workflow key in
`workflows.substrate.ts`. That key owns workflow identity; never hand-author an ID or registry.

```ts
import { table, string } from "@substratedb/core";
import { defineStep, defineWorkflow, defineWorkflows, workflow } from "@substratedb/workflows";
import { z } from "zod";
import { Tasks } from "@substratedb/generated/schema";
import { Attempts } from "@substratedb/generated/workflow-data/completeTask";

const input = z.strictObject({ taskId: z.string() });
const output = z.strictObject({ completed: z.boolean() });

const complete = defineStep({
  input,
  output,
  retry: { maxAttempts: 3, backoff: "exponential" },
  async run({ taskId }) {
    await workflow.data.write({ from: Attempts() }).create({ id: taskId, taskId }).run();
    await workflow.substrate.write({ from: Tasks() }).update(taskId, { completed: true }).run();
    return { completed: true };
  },
});

export const completeTask = defineWorkflow({
  client: "service",
  input,
  output,
  ttl: "7d",
  data: { tables: { attempts: table({ taskId: string() }) } },
  graph: (g) => g.step(complete),
});

export default defineWorkflows({ completeTask });
```

Import workflow-data handles only from the compiler-owned path matching the registry key. Run
`substrate generate` after changing workflow data.

## Graphs and retries

Graphs are static DAGs. Use sequential `.step(...)`, `.parallel(...)`, and exhaustive
discriminated `.when(...)` branches. Dynamic loops and arbitrary runtime callbacks are unsupported.

Step output is Zod-validated before durable checkpointing. One attempt is the default;
`retry.maxAttempts` includes the initial attempt. A step can run more than once, so make every
external effect idempotent or guard it with a durable idempotency key.

Keep inputs and outputs small. Use internal workflow tables for bounded accumulating state. Every
run has a fixed one-hour execution deadline, including waits and retries; TTL controls retention,
not execution time.

## Scope and authority

Declare client mode honestly:

- `client: "user"` injects a user client and requires application-user authority.
- `client: "service"` injects a trusted service client.

Declare entity scope when state, permissions, inspection, or idempotency belongs to one entity:

```ts
export const evaluate = defineWorkflow({
  client: "service",
  scope: Org(),
  input,
  output,
  graph: (g) => g.step(classify).step(persist),
});

await service.workflows.run(workflows.evaluate, {
  input: { taskId },
  scope: Org(orgId),
});
```

Never infer scope from input. Direct runs, nested calls, MCPs, schedules, and event bindings use the
same `{ input, scope }` envelope. Unscoped workflows reject an accidental scope.

## Internal workflow data

Declare literal `table(...)` definitions under `data.tables`. Only tables are supported in v4.
Generated handles live under `@substratedb/generated/workflow-data/<workflow-key>`.

Inside a step, use the pre-scoped client:

```ts
await workflow.data.write({ from: Attempts() }).create({ id: taskId, taskId }).run();
```

Workflow tables are:

- isolated by workflow identity and, when declared, entity scope;
- hidden from normal schema APIs, table browsers, generic Strata, and external writes;
- readable externally only through authorized workflow inspection;
- retained with workflow data and History, subject to TTL and plan limits.

Scoped workflows with data must declare a TTL. Use internal tables for deduplication, accumulation,
and intermediate state. Write user-visible records through `workflow.substrate` into normal schema
resources.

There is no public raw workflow database, input bag, prior-output bag, raw scope key,
`workflow.database`, or `workflow.step` API.

## Events, progress, logs, and History

Bind generated events to registered workflows and map payload to explicit input and scope:

```ts
e.on(events.data.Tasks().created).run(workflows.evaluate, ({ payload }) => ({
  input: { taskId: payload.current.id },
  scope: Org(payload.current.org),
}));
```

- Use `await workflow.progress.emit(value)` for ordered, schema-validated progress.
- Use `workflow.log.debug|warn|error(message, context?)` for bounded, non-sensitive metadata. Do not
  await logs; logging is synchronous and best-effort.
- Throw an `Error` to fail a step and apply its retry policy; `workflow.log.error` only logs.
- Use Workflow History—not logs—for inputs, outputs, attempts, snapshots, and data diffs.

Never log secrets, tokens, full inputs/outputs, decrypted fields, or workflow-table snapshots.

## Review checklist

- Is durable orchestration necessary?
- Are client mode and entity scope correct at every invocation?
- Are workflow, progress, and step schemas strict and typed?
- Are retries bounded and repeated effects safe?
- Is product state written through `workflow.substrate`?
- Is internal data private, scoped, bounded, and retained appropriately?
- Are logs bounded and non-sensitive?
- Do tests cover success, retry, timeout, denial, and terminal failure?
