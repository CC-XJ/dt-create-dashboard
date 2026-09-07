# Operational Dashboard Workflows

This file includes the important metrics and components in an operational dashboard.

## Dashboard Constraints
- The dashboard size must not exceed `w:24, h:30` 

## Dashboard Variable
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
- For problem related tiles, `Host_ID` or `Running_Service` or `Running_Database` must be added as a filter, matching it with `affected_entity_ids`.
    - Example: 
        ```DQL
        | expand affected_entity_ids
        | filter in(affected_entity_ids, array($Host_ID)) or in(affected_entity_ids, array($Running_Service)) or in(affected_entity_ids, array($Running_Database))
        ```
- For service related tiles, `Running_Service` must be added as a filter.
- For database related tiles, `Running_Database` must be added as a filter.
- For other tiles, `Host_ID` must be added as a filter.
- To filter using variables in the tiles. Do not attempt to use toSmartscapeId("variable") on the variables. This will result in an error. Instead, you should parse the tile's variable to string to perform comparison with the variable.
    - Example: `| filter in(toString(id), array($Host_ID))`


## Dashboard Flow

### 1. Health Overview
- This section size must take only `w:24, h:8`

1. `Number of Active Problem(s)`
    - Number of Active Problem(s) must only include total ACTIVE problem count.
    - Visualization: `singleValue`
    - Color should be applied to the background of the tile.
    - Color: 
        - `Red: active problem > 0`
        - `Green: active problem == 0`
2. `Problem List`
    - Problem list should only include ACTIVE problems in the environment.
    - All row values should be red.
3. `Host Health`
    - Host health is defined by whether a monitored host has any ACTIVE problems.
    - Visualization: `honeycomb`
    - Color:
        - `Red: host active problem > 0`
        - `Green: host active problem == 0`
4. `Services Health`
    - Service health is defined by whether a monitored service has any ACTIVE problems.
    - Visualization: `honeycomb`
    - Color:
        - `Red: service active problem > 0`
        - `Green: service active problem == 0`
5. `Database Health`
    - Database can be found under `fetch dt.entity.service | filter serviceType == "DATABASE_SERVICE"`
    - Database health is defined by whether a monitored database has any ACTIVE problems.
    - Visualization: `honeycomb`
    - Color:
        - `Red: database active problem > 0`
        - `Green: database active problem == 0`

### 2. Infrastructure Performance Overview

- `Host CPU Consumption`
    - Average host CPU consumption by dt.entity.host.
- `Host Memory Consumption`
    - Average host memory consumption by dt.entity.host.
- `Host Disk Consumption`
    - Average host disk consumption by dt.entity.host.
- `Table of Hosts with the CPU, Memory, and Disk Consumption`

### 3. Services Performance Overview

- `Response time`
- `Success Rate`
- `Table of Services with Response time and Success Rate`

### 4. Database Performance Overview
- Do not use `dt.smartscape.service` here, always use `dt.entity.service`.

- `Response time`
- `Success Rate`
- `Table of Database with Response time and Success Rate`
