# Rogers Sportsnet+ — Monitoring Roadmap

## Current Dashboard: What's Wrong

The existing dashboard is a **passive resource monitor** — it shows 19 widgets (several duplicated), all at the infrastructure layer, with no monitors/alerts wired up. Key gaps:

- **No alerts fire from it** — thresholds are drawn as marker lines but no DD monitors back them
- **No application signals** despite APM being enabled (zero error rate, latency, or throughput widgets)
- **Duplicated widgets** — VM Memory (×2), POD Memory Remaining (×2), POD CPU Usage (×2)
- **MongoDB gets 1 metric** (load avg) — no connections, replication lag, op counters, or index misses
- **No Kubernetes health signals** — no OOMKills, no pending pods, no node-not-ready events
- **No SLOs** — no reliability target for any product (sdf, divabo, l2v, vi)
- **Video-specific metrics absent** — no ingest health, encoding pipeline, or stream delivery signals

---

## Tier 1 — Critical + Easy (Set up this week)

These all use metrics already flowing into DD. The thresholds are either marked on existing widgets or self-evident. Each one should be a **DD Monitor with a PagerDuty/Slack notification**.

### Infra monitors (convert dashboard thresholds → real alerts)

| # | Monitor | Why critical | Effort |
|---|---------|-------------|--------|
| 1 | **CPU Credit Balance < 100** | T-type EC2 instances burst on credit. Exhaustion = CPU hard-capped, silent degradation | 5 min |
| 2 | **Pod Restart Count spike** | Crash loops are the #1 early indicator of a bad deployment or OOM | 5 min |
| 3 | **VM Disk Usage > 85% (warn) / 92% (crit)** | Full disks kill processes silently, especially MongoDB and log writers | 5 min |
| 4 | **VM Memory > 80% (warn) / 90% (crit)** | Threshold already marked on dashboard; just needs a monitor | 5 min |
| 5 | **VM CPU > 80% (warn) / 90% (crit) sustained 15m** | Use `avg` over 15m to avoid single-spike noise | 5 min |
| 6 | **MongoDB System Load > 0.8 (warn) / 1.0 (crit)** | Load > 1.0 on a single-core equivalent = queue forming. Already marked on dashboard | 5 min |
| 7 | **K8s CPU Throttling rate > 25%** | Throttling = pods are bursting past their CPU limit; affects video processing latency hard | 10 min |

### APM monitors (easy wins since traces are already flowing)

| # | Monitor | Why critical | Effort |
|---|---------|-------------|--------|
| 8 | **Service error rate > 1% (warn) / 5% (crit)** | First signal for any broken deployment or upstream dependency failure | 10 min |
| 9 | **Service p99 latency > 500ms (warn) / 2s (crit)** | Video ingest/delivery pipelines are latency-sensitive | 10 min |
| 10 | **APM downstream dependency errors** | Catches DB or external API failures before they cascade | 10 min |

**Setup notes for Tier 1:**
- Apply tags `env:production`, `customer:rogers` to all monitors
- Route warnings → Slack `#rogers-vi-alerts`, criticals → PagerDuty
- Set `evaluation_window` to 10–15m for infra metrics to suppress flap noise
- For APM monitors: create one per product (`sdf`, `divabo`, `l2v`, `vi`) using the `service:` tag

---

## Tier 2 — Critical + Medium Effort (Next 2 weeks)

### Kubernetes health

- **OOMKilled container detection** — query `kubernetes.containers.state.terminated` with `reason:OOMKilled`. Currently invisible; OOMKills explain many pod restarts.
- **Pods not ready / pending > 5m** — `kubernetes.pods.running` vs. `kubernetes.pods.desired`. A stuck rollout shows here before users notice.
- **Node Not Ready** — EC2 node failure in EKS. Use `kubernetes.node.status` or a service check.
- **EKS node group capacity** — track available node count vs. min; catches autoscaler failures before scheduling pressure hits.

### MongoDB observability (currently only 1 metric)

- **Active connections** — `mongodb.connections.current` vs. `mongodb.connections.available`. Alert at 80% of max connections.
- **Replication lag** — `mongodb.replset.optime_lag` — critical for any replica set reads; >30s lag is a problem.
- **Op counters** — `mongodb.opcounters.*` — baseline and alert on deviation (anomaly monitor after 1 week of baseline).
- **Index miss ratio** — `mongodb.indexcounters.missratio` — signals missing or inefficient indexes, directly impacts query latency.
- **Page faults** — `mongodb.extra_info.page_faults` — memory pressure indicator more accurate than system.mem for MongoDB.

### APM deeper

- **Composite monitor: high error rate + high latency** — fires only when both are true (reduces noise, higher confidence signal).
- **Downstream dependency map** — use APM service map to identify which external services (CDN, encoding APIs) are in the call chain; add monitors for each.
- **DB query latency via APM** — MongoDB spans in traces give you slow query p99 without needing a separate MongoDB profiler.

---

## Tier 3 — Full Observability (1 month out)

### SLOs

Define reliability targets per product. Suggested starting points:

| Product | Availability SLO | Latency SLO |
|---------|-----------------|-------------|
| sdf | 99.5% | p95 < 500ms |
| divabo | 99.5% | p95 < 1s |
| l2v | 99.0% | p95 < 2s |
| vi | 99.9% | p95 < 200ms |

Create error-budget monitors that alert at 50% and 90% burn rate (use DD SLO burn rate alerts).

### Video-specific signals

- **Stream ingest health** — if you're ingesting RTMP/HLS/SRT, track ingest drop rate, bitrate variance, and reconnect count
- **Encoding pipeline** — job queue depth, encoding error rate, time-to-first-frame
- **CDN origin hit rate** — high origin hit rate means CDN is not caching, increasing cost and latency
- **VoltDB health** — if `volt*` pods are VoltDB, expose JMX metrics to DD: transaction throughput, partition imbalance, latency percentiles

### Anomaly detection

After 2 weeks of baseline data on the Tier 1/2 metrics, convert key monitors to **anomaly monitors** (DD `anomalies()` function) rather than static thresholds. Best candidates:
- VM Network In/Out (traffic pattern changes)
- MongoDB op counters
- APM request rate per service

### Log-based monitors

Even with APM, structured log monitors catch things traces don't:
- Auth failures / permission errors
- Specific error codes from video codec or CDN APIs
- Configuration reload errors

---

## Quick-reference: Tag strategy

All monitors should consistently use these tags from the existing template variables:

```
env:production        # or env:prd
customer:rogers
product:<sdf|divabo|l2v|vi>
```

This lets you scope alerts, dashboards, and SLOs by product and environment without rebuilding queries.
