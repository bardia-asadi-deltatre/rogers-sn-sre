# AXIS - ROCKET_LATENCY_HIGH

> Full findings live here:
> [[_shared/AXIS - ROCKET_LATENCY_HIGH]]

---

## How We Got Here

Investigated via Datadog APM after the monitor kept firing 1–2x/day. Started with the monitor view, then drilled into the APM service page for `rogers-diva-rocket-api`, narrowed to the `GET /api/page?path=/` endpoint which consistently showed the worst latency.

Pulled traces from two separate incidents (May 25 ~22:30 and May 26 ~23:30). In both cases, the flame graph showed the same pattern: TCP connection time to ISL dominating, ISL's own processing time negligible. Also observed two failure modes — slow successes where TCP eventually connected after 30s+, and fast failures where the 1.4s connect timeout fired and propagated as 500/502.

The 502s come from the ALB timing out and cutting the connection before rocket-api finishes — a separate but related issue.

Both incidents appear to have the same root cause. Handed off to dev team to investigate connection pooling / keep-alive on the ISL HTTP client.
