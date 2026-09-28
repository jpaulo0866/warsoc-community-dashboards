# Network Perimeter & WAF Defense

Edge perimeter analysis tracking web application firewall evaluations, rule group matches, and client geo origins.

## Overview

This dashboard surfaces edge protection events from Web Application Firewalls (WAF) and Application Gateways. SOC analysts can quickly identify ongoing application-layer DDoS attacks, SQL injection attempts, cross-site scripting (XSS) patterns, and geographic origins of malicious client requests.

## Widgets

1. **Total WAF Evaluated Requests (`kpi_sparkline`)**
   - Ingestion throughput showing total inspected traffic requests and a daily trend sparkline.
   - SQL Target: `%catalog.waf_logs` (`time_stamp`)

2. **WAF Actions Distribution (`donut`)**
   - High-level ratio of firewall enforcement actions (`Allowed`, `Blocked`, `Matched`).

3. **WAF Client Geographic Distribution (`geo_map`)**
   - Interactive map pinpointing the source latitude and longitude (`lat`, `lon`) of incoming web traffic and security blocks.

4. **Prioritized WAF Threat Events (`data_table`)**
   - Deep log inspection table detailing the client IP, targeted hostname, requested URI, matched rule group, decision action, and rule message.

## Prerequisites & Data Sources

- **Service**: Azure WAF / Perimeter Gateway Logs (`waf`)
- **Canonical Table**: `%catalog.waf_logs`
- **Time Bounds**: Bound automatically via `:start_ts` and `:end_ts`
