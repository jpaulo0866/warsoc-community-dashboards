# Web Proxy & Egress Traffic Intelligence

Web proxy traffic monitoring tracking allowed and blocked egress traffic, high-volume HTTP transactions, and egress destinations.

## Overview

This dashboard analyzes outbound enterprise traffic through web proxies. It surfaces unauthorized egress attempts, top bandwidth-consuming domains, and the global distribution of external server connections.

## Widgets

1. **Total Proxied Requests (`kpi_sparkline`)**
   - Volume of web proxy requests compared against the prior time window, with a day-over-day sparkline.
   - SQL Target: `%catalog.proxy_logs` (`log_date`)

2. **Proxy Decision Breakdown (`donut`)**
   - High-level ratio of `allowed` vs `blocked` proxy actions.

3. **Global Egress Destination Traffic (`geo_map`)**
   - Geo-coordinates map of external server destinations reached by internal clients.

4. **High-Volume Egress Requests (`data_table`)**
   - Detailed transactions table featuring source client IP (`src_ip`), destination IP (`dest_ip`), URL domain, proxy action, payload size in bytes, and HTTP response status.

## Prerequisites & Data Sources

- **Service**: Web Proxy Telemetry (`proxy`)
- **Canonical Table**: `%catalog.proxy_logs`
- **Time Bounds**: Bound automatically via `:start_ts` and `:end_ts`
