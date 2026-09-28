# WarSOC Community Dashboards

A curated repository of community-contributed security operations, threat hunting, and compliance dashboard templates for the **WarSOC** platform.

Dashboards in this repository can be automatically synced into WarSOC and cloned into any analyst's personal workspace with one click.

---

## Navigation Index

| Dashboard | Domain | Target Telemetry | Folder |
| :--- | :--- | :--- | :--- |
| **DNS Threat & Anomaly Monitoring** | Network Security / DNS | `dns_logs` | [View Dashboard](dashboards/dns-threat-monitoring/) |
| **Vulnerability Posture & Risk Overview** | Vulnerability Management | `vulnerability_findings` | [View Dashboard](dashboards/vulnerability-posture-overview/) |
| **Network Perimeter & WAF Defense** | Edge / Application Defense | `waf_logs` | [View Dashboard](dashboards/network-perimeter-traffic/) |
| **Web Proxy & Egress Traffic Intelligence** | Egress / Web Proxy | `proxy_logs` | [View Dashboard](dashboards/web-proxy-traffic-intelligence/) |

---

## Connecting with WarSOC

WarSOC syncs community dashboards server-side through GitHub's API. Configure your WarSOC backend environment:

```dotenv
# backend/.env
COMMUNITY_BOARDS_GITHUB_OWNER=jpaulo0866
COMMUNITY_BOARDS_GITHUB_REPO=warsoc-community-dashboards
COMMUNITY_BOARDS_GITHUB_PATH=dashboards
```

### Syncing Dashboards
1. Log in to the WarSOC portal as a `PlatformAdmin`.
2. Navigate to **My Dashboards** (`/dashboards`).
3. Scroll to the **Community Dashboards** section and click **Sync with GitHub**.
4. WarSOC automatically fetches all dashboard definitions, verifies their SQL queries against `sqlguard`, and surfaces them in the gallery.
5. Any user can click **Copy to my dashboards** on any community card to import and customize the dashboard.

---

## Contributing a New Dashboard

We welcome new dashboard contributions! Each dashboard must be self-contained in its own directory with documentation and a schema-compliant JSON file.

### 1. Directory Structure

Place each dashboard in a dedicated directory under `dashboards/`:

```
dashboards/
└── <dashboard-slug>/
    ├── README.md               # Overview, use cases, and widget explanations
    └── <dashboard-slug>.json   # Dashboard definition file
```

Example: `dashboards/dns-threat-monitoring/dns-threat-monitoring.json`.

---

### 2. Dashboard JSON Format (`schemaVersion: 1`)

Each dashboard JSON must follow the standard export format:

```json
{
  "schemaVersion": 1,
  "name": "Dashboard Display Name",
  "description": "Short explanation of the dashboard's purpose.",
  "widgets": [
    {
      "widgetType": "donut",
      "title": "Widget Title",
      "queries": {
        "sql": "SELECT severity AS label, count(*) AS value FROM %catalog.vulnerability_findings WHERE last_seen BETWEEN :start_ts AND :end_ts GROUP BY severity"
      }
    }
  ]
}
```

---

### 3. SQL Guardrails & Security Requirements

All widget queries are validated server-side by WarSOC's `sqlguard` engine before being imported or executed. To ensure your contribution passes validation:

- **Operations**: Only single read-only `SELECT` (or `WITH ... SELECT`) statements are permitted. DDL/DML operations (`DROP`, `DELETE`, `INSERT`, `UPDATE`, `TRUNCATE`, `ALTER`) are strictly forbidden.
- **No Injections / Comments**: SQL comments (`--`, `/* */`, `#`) and multi-statement delimiters (`;`) are rejected fail-closed.
- **Table References**: Must use canonical catalog references: `%catalog.<table_alias>`.
- **Mandatory Time Bounds**: Every query must be bounded using `:start_ts` and `:end_ts` parameters to prevent unbounded full-table scans.
  - Standard time filter: `WHERE <time_col> BETWEEN :start_ts AND :end_ts`
  - For comparison slots (`previous_sql`): use `:prev_start_ts` and `:prev_end_ts`.

---

### 4. Supported Widget Types & Contracts

| Widget Type | Required Query Slots | Output Contract / Columns |
| :--- | :--- | :--- |
| **`kpi_sparkline`** | `current_sql`<br>`previous_sql`<br>`sparkline_sql` | • `current_sql`: `metric_value`<br>• `previous_sql`: `metric_value`<br>• `sparkline_sql`: `bucket_ts`, `metric_value` |
| **`donut`** | `sql` | • `label` (string / category)<br>• `value` (numeric) |
| **`stacked_bar_timeline`** | `sql` | • `bucket_ts` (timestamp via `date_trunc`)<br>• `category` (string)<br>• `metric_value` (numeric) |
| **`geo_map`** | `sql` | • `lat` (latitude)<br>• `lon` (longitude)<br>• `count` (numeric) |
| **`data_table`** | `sql` | Tabular query columns from approved tables. |

---

### 5. Available Canonical Telemetry Tables

| Table Alias | Time Column | Description |
| :--- | :--- | :--- |
| `dns_logs` | `log_ts` | Internal DNS query and resolution logs. |
| `proxy_logs` | `log_date` | Web proxy HTTP/HTTPS traffic logs. |
| `network_flow_logs` | `log_time` | VNet / subnet network flow decisions (allow/deny). |
| `waf_logs` | `time_stamp` | Web Application Firewall rule match events. |
| `load_balancer_access_logs` | `time_stamp` | Application Gateway and load balancer access logs. |
| `vulnerability_findings` | `last_seen` | Canonical vulnerabilities normalized from Tenable, Qualys, Uptycs, and Onapsis. |
| `warsoc_compliance` | `log_date` | Policy and compliance scan outcomes. |

---

### 6. Submission Checklist

Before submitting a Pull Request:
- [ ] Created a dedicated subfolder: `dashboards/<slug>/`.
- [ ] Added `<slug>.json` with `schemaVersion: 1`.
- [ ] Added `README.md` detailing the operational use cases and widget descriptions.
- [ ] Ensured all queries use `%catalog.<alias>` and include mandatory `:start_ts` and `:end_ts` bounds.
- [ ] Updated the [Navigation Index](#navigation-index) in this README.