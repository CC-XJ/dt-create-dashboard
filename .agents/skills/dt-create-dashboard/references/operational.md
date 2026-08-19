# Operational Dashboard Workflows

This file includes the important metrics and components in an operational dashboard.

## Dashboard Flow

### Dashboard Variables as filter
- Host Group, Host Name, Host ID, Service running on Host are mandatory variables to be included.
- All the variables must be `multiple: true`.
- Under `tags`, identify top 3 categories suitable to be dashboard filters.
    - These identified categories should substitute the key field `Tag_Categories` and the placeholder `{variable_category}` in the example `variables.json`.
    - Each category identified must become it's own variable.
- Relationship between Variables should be as followed
    - `Host_Group` -> Identified Categories -> `Host_Name`, `Host_ID` -> `Running_Service`

### Clean the Dashboard Variable data
- If `tags` data contains the category in the data (eg. Application:API), you should clean it using Dynatrace Pattern Language (DPL)
```
DQL
| parse tags, """LD: category << ':' LD: application"""
```

### Variable Usage
- Variables as `multiple: true` must be queried with `filter in (attribute, array($Variable_Name))`, to ensure safe filtering.

### 1. Health Overview

- `Number of Active Problem(s)`
- `Problem List`
- `Host Health`
- `Services Health`
- `Database Health`

### 2. Infrastructure Performance Overview

- `Host CPU Consumption`
- `Host Memory Consumption`
- `Host Disk Consumption`

### 3. Services Performance Overview

- `Response time`
- `Success Rate`

### 4. Database Performance Overview

- `Response time`
- `Success Rate`
