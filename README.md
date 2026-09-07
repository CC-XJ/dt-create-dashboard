# dt-create-dashboard

Custom AI skill for creating and updating Dynatrace operational and business dashboards.

## Required Skills

For creating an operational dashboard, use:

- `dt-create-dashboard`: Orchestrates dashboard planning, DQL validation, JSON generation, and deployment.
- `dt-app-dashboards`: Provides Dynatrace dashboard JSON, tile, layout, visualization, and variable schemas.
- `dt-dql-essentials`: Provides DQL syntax, query patterns, and validation guidance.

The primary skill for this context is `dt-create-dashboard`. It uses `dt-app-dashboards` and `dt-dql-essentials` when building the dashboard.

## Installation

Install the Dynatrace Agent Skills from the `dynatrace-for-ai` repository:

```bash
npx skills add dynatrace/dynatrace-for-ai
```

To install only the skills required for operational dashboards:

```bash
npx skills add dynatrace/dynatrace-for-ai --skill dt-app-dashboards
npx skills add dynatrace/dynatrace-for-ai --skill dt-dql-essentials
```

The custom `dt-create-dashboard` skill is included in this repository at:

```text
.agents/skills/dt-create-dashboard/SKILL.md
```

After installation, restart or reload your AI coding agent so it can discover the skills.

## Creating an Operational Dashboard

Ask the agent to create an operational dashboard and provide:

- The services, hosts, applications, or entities to monitor
- The operational goals and key questions
- The metrics, problems, logs, or traces to include
- Any required dashboard filters or variables
- The desired dashboard name

The agent will:

1. Classify the request as an operational dashboard.
2. Load the required Dynatrace dashboard and DQL skills.
3. Discover and validate relevant entities and queries.
4. Generate the dashboard JSON.
5. Save the dashboard definition in the `dashboards/` directory.
6. Validate the dashboard tiles and variables before deployment.
7. Deploy the dashboard with `dtctl` after review and approval.

## Dashboard Output

Generated dashboard definitions are stored in:

```text
dashboards/
```

Deployment requires an authenticated and configured `dtctl` environment.
