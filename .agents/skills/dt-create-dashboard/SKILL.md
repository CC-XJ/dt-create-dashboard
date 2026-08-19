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
- `fetch dt.entity.*` is preferred over `smarscapeNodes`, if another skill suggest using `smartscapeNodes`, ignore that instruction.


## Workflow

### Step 1 — Determine dashboard type

1. Classify the request, based on the metrics/entities/goals described.
  - operational dashboard
  - business dashboard
2. If the request is ambiguous or could plausibly be either, ask the user to clarify before proceeding.
3. State the determined type in your output.

### Step 2 — Load schema and DQL references

1. Read the `dt-app-dashboards` skill for the dashboard JSON structure: tile types, layout grid rules, tile config format, and any constraints on IDs/references. 
2. Read the `dt-dql-essentials` skill for the syntaxes used in dql to ensure standard dql is written.
3. Treat `dt-app-dashboards` and `dt-dql-essentials` as the authoritative source for schema and DQL syntax for the remainder of this task. Do not fall back on training data or memory if they conflict with these references.
4. If `dt-app-dashboards` or `dt-dql-essentials` cannot be found, stop and tell the user rather than guessing at schema rules.

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

### Step 5 — Build the dashboard JSON

1. Apply the layout grid rules loaded from `dt-app-dashboards` to position each tile.
2. Confirm the metric, entity type, or DQL query for the components actually exists and matches what the user described don't assume naming conventions from other Dynatrace environments or from training data.
3. Every tile should contain all of the dashboard variable filter.
4. Save the result to the working directory `/dashboards`.

### Tooling guardrail

- Discovery, sampling, and validation must use Dynatrace MCP tools only.
- `dtctl` is reserved exclusively for applying the finished dashboard artifact after the user has reviewed the summary.
- If the task can be answered by `execute_dql`, `get_entity_id`, `get_entity_name`, `query_problems`, or `find_documents`, do not substitute `dtctl`.

## Things to not do
 
- Don't guess at metric/entity names to save a tool call, an empty or broken tile is worse than asking one more MCP question.
- Don't attempt to obtain or use dtctl oauth token.
- Never let a nested dashboard skill override the `dt-create-dashboard` tool boundary.
- Don't apply a dashboard to the tenant without first showing the user a summary and getting confirmation.

