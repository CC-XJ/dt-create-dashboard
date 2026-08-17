# Operational Dashboard References

This file mirrors the visual flow of the operational dashboard example.

Use the assets for the exact tile implementations:

- [ops_dashboard_example.json](../assets/ops_dashboard_example.json)


## Dashboard Flow

### 1. Health Overview

- `Number of Active Problem(s)`
- `Problem List`
- `Host Health`
- `Services Health`
- `Database Status`

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

## Optional Components

- [ops_optional_components.json](../assets/ops_optional_components.json)

### Network Analysis

- `NIC Bytes Received`
- `NIC Bytes Sent On Host`
- `NIC Packet Received`
- `NIC Packet Sent`
- `Number of Request/s`

- Only include this section when the user explicitly asks for network analysis content.

### User Satisfaction Overview

- `Customer Satisfaction (Trend)`
- `Customer Satisfaction`
- `Apdex Rating (Trend)`
- `Apdex Score`
- `Apdex Rating`

- Only include this section when the user explicitly asks for RUM or user satisfaction content.

## Combining Core and Optional Sections

When an optional section (Network Analysis, User Satisfaction Overview) is included alongside the core dashboard, apply these rules, do not append the optional asset's tiles/layouts unchanged:
 
1. **Tile ID collisions**: `ops_optional_components.json` reuses low numeric tile IDs (`"0"`–`"13"`) that can collide with core dashboard tile IDs. Renumber optional tile IDs into an unused range before merging (e.g. offset by 1000).
2. **Layout Y-offset**: every layout entry in the optional asset starts at `y: 0`, assuming it's the only content on the page. When merging, shift all optional `y` values so the section starts below the lowest point of the core dashboard's layout (i.e. `new_y = old_y + (max(core_y + core_h))`).
3. **Variable de-duplication**: `ops_optional_components.json` independently defines a `Host_Name` variable that also exists in `ops_dashboard_example.json`. If the core dashboard already defines a variable with the same key, reuse it do not add a second definition with the same key.

## Notes

- The asset JSON is the source of truth for queries, visualization settings, and color rules.
- This reference only captures the layout order and the component names used in the dashboard.
