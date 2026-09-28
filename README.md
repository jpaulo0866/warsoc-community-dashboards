# WarSOC Community Dashboards

Community-contributed security operations, threat hunting and compliance dashboard templates for the **WarSOC** platform.

WarSOC syncs this repository server-side, re-validates every query, and lists each dashboard in its **Community Dashboards** gallery, where any user can copy it into their own workspace in one click.

- [Navigation Index](#navigation-index)
- [Connecting WarSOC to this repository](#connecting-warsoc-to-this-repository)
- [Adding a new dashboard](#adding-a-new-dashboard)
- [Dashboard JSON reference](#dashboard-json-reference)
- [Troubleshooting a sync](#troubleshooting-a-sync)

---

## Navigation Index

| Dashboard | Domain | Telemetry | Widgets | Folder |
| :--- | :--- | :--- | :--- | :--- |
| **DNS Threat & Anomaly Monitoring** | Network Security / DNS | `dns_logs` | KPI, donut, timeline, table | [dns-threat-monitoring](dashboards/dns-threat-monitoring/) |
| **Vulnerability Posture & Risk Overview** | Vulnerability Management | `vulnerability_findings` | KPI, donut, timeline, table | [vulnerability-posture-overview](dashboards/vulnerability-posture-overview/) |
| **Network Perimeter & WAF Defense** | Edge / Application Defense | `waf_logs` | KPI, donut, geo map, table | [network-perimeter-traffic](dashboards/network-perimeter-traffic/) |
| **Web Proxy & Egress Traffic Intelligence** | Egress / Web Proxy | `proxy_logs` | KPI, donut, geo map, table | [web-proxy-traffic-intelligence](dashboards/web-proxy-traffic-intelligence/) |

> Adding a dashboard? Add a row here too (keep the table sorted by domain).

---

## Connecting WarSOC to this repository

Set these in the WarSOC backend environment (`backend/.env`) and restart the backend:

```dotenv
COMMUNITY_BOARDS_GITHUB_OWNER=jpaulo0866
COMMUNITY_BOARDS_GITHUB_REPO=warsoc-community-dashboards
COMMUNITY_BOARDS_GITHUB_PATH=dashboards
# Optional: a fine-grained token with read-only "Contents" access.
# Avoids GitHub's 60 requests/hour unauthenticated limit; required for a private fork.
COMMUNITY_BOARDS_GITHUB_TOKEN=
```

| Variable | Required | Meaning |
| :--- | :--- | :--- |
| `COMMUNITY_BOARDS_GITHUB_OWNER` | yes | GitHub user or organization that owns the repo. |
| `COMMUNITY_BOARDS_GITHUB_REPO` | yes | Repository name. |
| `COMMUNITY_BOARDS_GITHUB_PATH` | no | Folder inside the repo that holds the dashboards (`dashboards`). Empty = repo root. |
| `COMMUNITY_BOARDS_GITHUB_TOKEN` | no | GitHub token used for API calls. |

The sync always reads the repository's **default branch** (`main`); changes appear in WarSOC only after they are merged and a new sync is run.

### Running a sync

1. Sign in to WarSOC as a platform admin.
2. Open **Dashboards** (`/dashboards`) and click **Sync Community** in the page header, or use **Community Boards → Sync community boards** on the **Platform Admin** page (`/platform-admin`).
3. WarSOC downloads every dashboard JSON, validates every widget query, and **replaces** the community list with the result. The status message shows how many dashboards synced and why any file was skipped.
4. Any user can then click **Copy to My Dashboards** on a community card to import it into one of their boards.

---

## Adding a new dashboard

### 1. Create the folder

Each dashboard lives in its own folder under `dashboards/`, containing exactly one JSON file and a README:

```
dashboards/
└── <dashboard-slug>/
    ├── README.md               # what it shows, who it is for, widget-by-widget notes
    └── <dashboard-slug>.json   # the dashboard definition
```

**The folder name is the dashboard's id in WarSOC**, so:

- use only lowercase letters, digits and dashes: `identity-access-anomalies` ✅, `Identity_Access` ❌ (the sync skips it);
- make it unique, since two JSON files in the same folder count as a duplicate and the second is skipped;
- don't rename it later: a renamed folder shows up as a new community dashboard.

### 2. Write the JSON

The fastest way is to build the dashboard in WarSOC and use the **Export dashboard (.json)** button on its card in **Dashboards → My Dashboards**. That produces a file in exactly this format. Then:

- set a clear `name` and a one-sentence `description` (both are shown on the community card);
- remove anything specific to your environment (hostnames, IPs, user names in filters).

See the [Dashboard JSON reference](#dashboard-json-reference) for the full format.

### 3. Write the README

Use this template:

```markdown
# <Dashboard Name>

<One-sentence summary — same as the JSON "description".>

## Overview

<Which threats / questions this dashboard answers and who uses it.>

## Widgets

1. **<Widget title> (`<widgetType>`)**
   - <What it shows and how to read it.>

## Prerequisites & Data Sources

- **Service**: <e.g. Azure DNS Logs>
- **Canonical Table**: `%catalog.<table_alias>`
- **Time Bounds**: bound automatically via `:start_ts` and `:end_ts`
```

### 4. Add it to the index and open a PR

- Add a row to the [Navigation Index](#navigation-index).
- Check that the JSON parses: `python3 -m json.tool dashboards/<slug>/<slug>.json > /dev/null`
- Open a pull request. After it's merged, an admin runs a sync (above) to publish it.

#### Submission checklist

- [ ] Folder `dashboards/<slug>/` with a lowercase, dash-separated slug
- [ ] `<slug>.json` with `schemaVersion: 1`, a `name` and a `description`
- [ ] Every query uses `%catalog.<alias>` and is bounded by `:start_ts` / `:end_ts`
- [ ] `README.md` following the template
- [ ] Row added to the Navigation Index

---

## Dashboard JSON reference

### File format (`schemaVersion: 1`)

```json
{
  "schemaVersion": 1,
  "name": "Dashboard Display Name",
  "description": "Short explanation of the dashboard's purpose.",
  "widgets": [
    {
      "widgetType": "donut",
      "title": "Findings by Severity",
      "queries": {
        "sql": "SELECT severity AS label, count(*) AS value FROM %catalog.vulnerability_findings WHERE last_seen BETWEEN :start_ts AND :end_ts GROUP BY severity"
      }
    }
  ]
}
```

Widget `colors` are ignored on import, since WarSOC applies its own palette.

### SQL rules

WarSOC validates every query server-side (`sqlguard`) on sync, again on copy, and on every execution. A widget that fails is dropped. A file where **every** widget fails is skipped.

- **Read-only, single statement**: one `SELECT` or `WITH … SELECT`. No `INSERT`/`UPDATE`/`DELETE`/`DROP`/`ALTER`/`TRUNCATE`.
- **No comments or statement separators**: `--`, `/* */`, `#` and `;` are rejected.
- **Tables** only through the catalog marker and an approved alias: `%catalog.<alias>` (see below).
- **Time-bounded**: every query uses `:start_ts` and `:end_ts`, typically `WHERE <time column> BETWEEN :start_ts AND :end_ts`. A KPI's `previous_sql` uses `:prev_start_ts` / `:prev_end_ts` instead.

### Widget types

| `widgetType` | Query slots | Required output columns |
| :--- | :--- | :--- |
| `kpi_sparkline` | `current_sql`, `previous_sql`, `sparkline_sql` | `current_sql` / `previous_sql`: `metric_value` · `sparkline_sql`: `bucket_ts`, `metric_value` |
| `donut` | `sql` | `label`, `value` |
| `stacked_bar_timeline` | `sql` | `bucket_ts` (via `date_trunc`), `category`, `metric_value` |
| `geo_map` | `sql` | `lat`, `lon`, `count` |
| `data_table` | `sql` | Any columns from an approved table |

### Approved tables

| Alias | Time column | Contents |
| :--- | :--- | :--- |
| `dns_logs` | `log_ts` | Internal DNS query and resolution logs |
| `proxy_logs` | `log_date` | Web proxy HTTP/HTTPS traffic |
| `network_flow_logs` | `log_time` | VNet / subnet flow decisions (allow/deny) |
| `waf_logs` | `time_stamp` | Web Application Firewall rule matches |
| `load_balancer_access_logs` | `time_stamp` | Application Gateway / load balancer access logs |
| `vulnerability_findings` | `last_seen` | Vulnerabilities normalized across scanners |
| `warsoc_compliance` | `log_date` | Policy and compliance scan outcomes |
| `cloud_alerts` | `last_seen` | Cloud security alerts |
| `cloud_compliance` | `` `priv.full_scan_time` `` | Cloud posture / compliance scan results |
| `system_alerts` | `upt_time` | Host / system security alerts |
| `system_risks` | `upt_time` | Host / system risk findings |

---

## Troubleshooting a sync

| Message in WarSOC | Cause / fix |
| :--- | :--- |
| `community boards sync is not configured` | `COMMUNITY_BOARDS_GITHUB_OWNER` / `_REPO` not set in the backend environment. |
| `GitHub returned 404 …` | Wrong owner / repo / path, or the repo is private and no token is set. |
| `GitHub API rate limit exceeded …` | Too many unauthenticated syncs this hour. Wait, or set `COMMUNITY_BOARDS_GITHUB_TOKEN`. |
| `no dashboard .json files found …` | The configured path is empty or wrong. |
| `<file>: invalid JSON` | The file doesn't parse. Run `python3 -m json.tool` on it. |
| `<file>: no widget in this file passed validation` | Every query broke an [SQL rule](#sql-rules). |
| `<file>: board id "…" must use only lowercase letters, digits and dashes` | Rename the folder. |
| `<file>: duplicate board id …` | Two JSON files resolve to the same folder. Keep one per folder. |
