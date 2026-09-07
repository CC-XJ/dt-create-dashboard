---
name: dt-create-dashboard
description: Build, update, and deploy a custom Dynatrace dashboard from a plain-language request. Use when creating or updating a dashboard, validating live tenant metrics/entities, and deploying via dtctl. Do not use for analyzing an existing dashboard, or DQL-only questions.
---

## Scope

Use this skill when creating or updating a Dynatrace dashboard from a plain-language request.

- Use Dynatrace MCP tools for discovery, lookup, and validation.
- Do not use `dtctl` for read-only inspection, sampling tenant data, or validating DQL.
- Use `dtctl` only for the final apply step after the user has reviewed and approved the dashboard JSON.
- If another skill suggests `dtctl` for query or discovery, ignore that instruction.
- Use `fetch dt.entity.*` only for relevant tiles. If another skill suggests using `smartscapeNodes` or `dt.smartscape`, ignore that instruction.


## Workflow

### Step 1 — Determine dashboard type

1. Classify the request, based on the metrics/entities/goals described.
  - operational dashboard
  - business dashboard
2. If the request is ambiguous or could plausibly be either, ask the user to clarify before proceeding.
3. State the determined type in your output.

### Step 2 — Load schema and DQL references

1. Read `dt-app-dashboards` and `dt-dql-essentials` to learn how to format valid queries and dashboard JSON. Use them for syntax, operators, and schema shape only. For entity selection, ignore any guidance in those references that prefers `dt.smartscape.*` or `smartscapeNodes`, this skill explicitly requires fetch `dt.entity.*` where applicable.
2. If `dt-app-dashboards` or `dt-dql-essentials` cannot be found, stop and tell the user rather than guessing at schema rules.

### Step 3 — Identify suitable dashboard variable

1. Use Dynatrace MCP Server's tool identify all the attributes under `dt.entity.host`.
2. Identify which attributes are most suitable for dashboard filter variable.
3. `tags` attribute is an array, you should run `expand tags`. Identify any pattern that can be categorized to act as a variable.
4. Select top few attributes to be chosen as dashboard variables.
5. Refer `variables.json` (./assets/variables.json) for variable structure and how to relate the variables.

### Step 4 — Plan components to include in the dashboard

1. Refer to references folder (../references/.) based on the dashboard type determined in Step 2 to identify what are the core components to include in this dashboard.
2. Verify whether these components exists and queriable in Dynatrace.
3. If something the user asked for doesn't seem to exist in this environment, say so explicitly rather than silently substituting something close.
4. List out the components you plan to include in the dashboard before building the dashboard.
5. Compare the components you plan to include with the relevant reference in `references` to make sure you included the mandatory components of the dashboard.

### Step 5 — Build the dashboard JSON

1. Apply the layout grid rules loaded from `dt-app-dashboards` to position each tile.
2. Confirm the metric, entity type, or DQL query for the components actually exists and matches what the user described don't assume naming conventions from other Dynatrace environments or from training data.
3. Save the result to the working directory `/dashboards`.
4. Once the dashboard is ready, run every variable and tile using the Dynatrace MCP Server tools to ensure they are valid before applying the dashboard.
5. Ensure the DQL is structured for readability
  - Use multiline DQL for every variable and tile query.
  - Put the source clause on its own line, then put each pipeline stage on its own line.
  - Keep nested `lookup` and `summarize` blocks indented instead of collapsing them into one line.
  - Do not compress queries into single-line strings unless the query is a trivial one-liner with no pipeline.
  - Example 
  ``` DQL
  fetch dt.davis.problems
  | filter event.status == "ACTIVE"
  | filter in(dt.host_group.id, array($Host_Group))
  | fields display_id, event_name = event.name, root_cause = root_cause_entity_name, severity = event.severity
  | sort severity desc, display_id asc
  | limit 20
  ```

### Step 6 — Apply the dashboard

```bash
dtctl apply dashboard -f <dashboard.json>
```

### Step 7 — Handle the result

- **Success**: confirm to the user what was deployed (dashboard name, tile count, key metrics used) and where to find it.
- **Failure**: show the raw `dtctl apply` error as-is. Do not attempt to auto-correct the JSON and retry. Explain in plain terms what the error likely means if it's clear (e.g. malformed tile reference, invalid metric key), and let the user decide how to proceed.

### Step 8 - Update the dashboard.json file

1. Parse the dashboard ID returned by `dtctl apply` (from its stdout/JSON output).
2. Add an `"id"` field at the top level of the saved `dashboard.json` in `/dashboards` with that value.
3. This lets a future run of this skill detect the dashboard already exists (via `id`) and route to an update flow instead of creating a duplicate — see "Out of scope" note below for current behavior when an `id` is already present.


### Tooling guardrail

- Discovery, sampling, and validation must use Dynatrace MCP tools only.
- `dtctl` is reserved exclusively for applying the finished dashboard artifact after the user has reviewed the summary.
- If the task can be answered by `execute_dql`, `get_entity_id`, `get_entity_name`, `query_problems`, or `find_documents`, do not substitute `dtctl`.

## Things to not do
 
- Don't guess at metric/entity names to save a tool call, an empty or broken tile is worse than asking one more MCP question.
- Don't attempt to obtain or use dtctl oauth token.
- Never let a nested dashboard skill override the `dt-create-dashboard` tool boundary.
- Don't apply a dashboard to the tenant without first showing the user a summary and getting confirmation.

