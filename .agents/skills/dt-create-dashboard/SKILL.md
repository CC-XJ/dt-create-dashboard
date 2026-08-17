---
name: dt-create-dashboard
description: Builds and deploys a new Dynatrace dashboard from a plain-language request that includes a monitoring goal and scope. Use when creating a new dashboard, validating live tenant metrics/entities, and deploying via dtctl. Do not use for analyzing an existing dashboard, or DQL-only questions.
---

## Prerequisites

- Ensure `dtctl` is installed and reachable using `dtctl version`.
- Ensure `dtctl` is authenticated with platform token to the target Dynatrace tenant using `dtctl auth status`.
- Dynatrace MCP Tools are available in the current session.

## Workflow

### Step 1 — Discover the Dynatrace MCP tools

1. Identify Dynatrace MCP Server tools available to be used.
2. If no Dynatrace MCP Server tools are found at all, stop and tell the user.

### Step 2 — Load the dashboard schema reference

Read the `dt-app-dashboards` skill for the dashboard JSON structure: tile types, layout grid rules, tile config format, and any constraints on IDs/references. If `dt-app-dashboards` cannot be found, stop and tell the user rather than guessing at schema rules from training data or memory.

### Step 3 — Find real data via MCP

1. Use the Dynatrace MCP Server tools to confirm the metric, entity type, or DQL query actually exists and matches what the user described don't assume naming conventions from other Dynatrace environments or from training data.
2. Prefer querying real entity/metric IDs over inventing plausible-looking ones.
3. If something the user asked for doesn't seem to exist in this environment, say so explicitly rather than silently substituting something close.
4. For entity-backed dashboard tiles, prefer `fetch dt.entity.*` or the dashboard's existing `dt.*` entity data objects over `smartscapeNodes` so new dashboard work stays consistent with the operational dashboard references.

### Step 4 — Build the dashboard JSON

1. Construct the dashboard JSON following the `dt-app-dashboards` schema, using the real metric/entity references found in Step 3.
2. Apply the layout grid rules loaded from `dt-app-dashboards` to position each tile.
3. If optional components (e.g. Network Analysis, User Satisfaction) were explicitly requested, merge them into the core dashboard now — do not append them as-is:
   - **Tile IDs**: renumber optional-component tile IDs so they don't collide with existing core tile IDs (e.g. offset optional IDs by 1000+).
   - **Layout Y-offset**: shift every optional tile's `y` value so it starts below the lowest point of the core layout (`max(y + h)` across all core tiles), not at `y: 0`.
   - **Variables**: if a variable key already exists (e.g. `Host_Name`), reuse it do not add a duplicate definition.
4. Resolve every placeholder token (e.g. `{{APP_NAME}}`) against data confirmed in Step 3 or explicitly provided by the user. Never leave a `{{...}}` token in JSON that will be passed to `dtctl apply`.
5. Save the result to the working directory `/dashboards`.

### Step 5 — Validate and confirm before applying

1. Check the constructed JSON is well-formed and contains no leftover `{{placeholder}}` tokens or example/default values copied verbatim from reference assets.
2. Present a short summary to the user before deploying: dashboard name, sections included, tile count, and the key metrics/entities used.
3. Get explicit confirmation from the user before proceeding to Step 6. This is a write action against a live tenant do not auto-apply.

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

## Things to not do
 
- Don't skip Step 1 and assume last session's MCP tool names still apply.
- Don't guess at metric/entity names to save a tool call, an empty or broken tile is worse than asking one more MCP question.
- Don't attempt to obtain or use dtctl oauth token.
- Don't use dtctl for any other purpose than to create/apply the dashboard to the environment.
- Don't auto-retry failed `dtctl apply` calls with modified JSON.
- Don't copy `AppName`, entity IDs, tenant IDs, or other environment-specific literals from reference assets without re-resolving them for the current tenant, reference assets contain illustrative values only, not real defaults.
- Don't apply a dashboard to the tenant without first showing the user the Step 5 summary and getting confirmation.
