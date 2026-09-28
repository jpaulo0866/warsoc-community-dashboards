# DNS Threat & Anomaly Monitoring

Real-time visibility into internal DNS requests, anomalous query volumes, top queried domains, and suspicious record types.

## Overview

This dashboard monitors DNS resolution traffic across your infrastructure to detect reconnaissance, command-and-control (C2) beacons, data exfiltration over DNS (tunneling), and anomalous internal lookups.

## Widgets

1. **Total DNS Queries (`kpi_sparkline`)**
   - Displays the current query count, percentage comparison with the prior period, and a daily sparkline trend.
   - SQL Target: `%catalog.dns_logs` (`log_ts`)

2. **DNS Queries by Record Type (`donut`)**
   - Distribution of queries across record types (`A`, `AAAA`, `CNAME`, `TXT`, `MX`, `PTR`).
   - Sudden spikes in `TXT` or `NULL` records often signify DNS tunneling or exfiltration.

3. **Daily DNS Volume by Transport Protocol (`stacked_bar_timeline`)**
   - Timeline breakdown comparing standard UDP vs TCP DNS resolutions.
   - High-volume TCP traffic can highlight zone transfers or large payload exfiltration.

4. **Top Queried Domains & Client Sources (`data_table`)**
   - Granular telemetry table featuring queried host, registrable domain, client source IP (`src_ip`), resolver destination IP (`dest_ip`), and query frequency.

## Prerequisites & Data Sources

- **Service**: Azure DNS Logs / DNS Telemetry
- **Canonical Table**: `%catalog.dns_logs`
- **Time Bounds**: Bound automatically via `:start_ts` and `:end_ts`
