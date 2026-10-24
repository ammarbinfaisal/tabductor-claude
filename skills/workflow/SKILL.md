---
name: workflow
description: Publish a Tabductor workflow from a task description, run it once, and report the result.
argument-hint: "<what the workflow should do>"
disable-model-invocation: true
---

Create and run a Tabductor workflow for this task: $ARGUMENTS

Follow the `tabductor` skill. Concretely:

1. Turn the task into a directing prompt for `workflow_publish`: the site, the steps, what to collect
   and the shape of the result. If the user wants structured output, add a `result_schema`
   (JSON Schema draft-07). If the task has inputs that change per run, name them `$like-this` in
   the prompt and tell the user what they are.
2. Call `workflow_publish`, then `workflow_trigger` with any `inputs` the user gave.
3. Poll `workflow_status` with the returned execution id, backing off from 10 s to 30 s between
   polls, until `finished` is true.
4. Report the `result`, or the `errors` if it failed, and give the workflow id so the user can run it
   again with `workflow_trigger` or put it on a schedule with `workflow_schedule`.
