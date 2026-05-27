# AXIS - ROCKET_LATENCY_HIGH

## Summary

The `ROCKET_LATENCY_HIGH` monitor fires 1–2x/day during traffic spikes on the `rogers-diva-rocket-api` service. Based on the APM traces investigated so far, the current evidence points toward a connection management issue between rocket-api and the ISL dependency, rather than slow ISL request processing itself.

This note summarizes the observations from the traces reviewed on May 25 and May 26.

---

## What the monitor shows

- Alert threshold: avg latency > 800ms over last 5m
- Typical spike observed: p99 reaches 30s–60s+, then self-resolves within ~5 minutes
- Main endpoint observed in traces: `GET /api/page?path=/`
- The incidents reviewed so far (May 25 ~22:30 and May 26 ~23:30) showed a very similar pattern

---

## Observations from APM traces

For the traces reviewed, each request to `GET /api/page?path=/` appears to trigger multiple parallel calls to ISL (`production-isl.d3-rgr-diva.com:443`).

One thing that stood out consistently is that a significant portion of execution time appears to be spent establishing TCP connections rather than processing responses.

Examples from the flame graphs reviewed:

- `rogers-diva-rocket-api-tcp`: ~20–39% of exec time
- `rogers-diva-rocket-api-http-client`: ~50–63% of exec time
- `rogers-diva-isl-odata-api`: generally very small in comparison (< 0.1% in the traces reviewed)

From the traces inspected so far, ISL itself appears to process requests relatively quickly once the connection is established (typically a few ms).

This may suggest that the larger bottleneck during spikes is the repeated setup of new TCP connections under load, especially when many ISL calls happen in parallel for a single page request.

---

## Behaviours observed during spikes

| Behaviour | Observation | Possible user impact |
| --- | --- | --- |
| Slow success | Requests eventually complete after long wait times | Very slow page load |
| Fast failure (timeout) | Some requests hit the ~1.4s connect timeout | Error page / partial failure |
| 502 Bad Gateway | In some cases ALB appears to timeout waiting for rocket-api | Error page |

---

## Possible next step / area to investigate

One area that may be worth reviewing is whether the HTTP client used by rocket-api is reusing connections efficiently (keep-alive / connection pooling).

If connections are not being reused, that could explain why TCP setup overhead becomes amplified during traffic spikes.

Depending on the HTTP library/configuration in use (axios, node-fetch, got, etc.), enabling or tuning connection reuse may help reduce the overhead significantly.

Separately, it may also be worth reviewing:

- ALB timeout settings relative to worst-case request duration during spikes
- Whether retry/fallback behaviour around ISL connection timeouts could be improved

---

## Things that do not currently appear to be primary contributors

Based on the traces and ECS observations reviewed so far:

- ISL request processing latency itself does not appear to be the dominant bottleneck
- ECS/Fargate scaling delay does not currently appear to align with the spike timing observed
- Cluster warm-up/readiness did not appear to be the main issue during the incidents reviewed

---

## Monitor note

The current threshold (800ms avg over 5m) does not appear unreasonable given that some spikes reach tens of seconds in p99 latency.

That said, since the issue tends to self-resolve relatively quickly, it may be worth considering a slightly wider evaluation window to reduce alert noise until the underlying behaviour is addressed.
