# Monitoring - ECS Unhealthy Task Percentage - Query Fix

## What We Saw

On May 26, a P1 alert fired for `rogers-diva-rocket-api`:

> **P1 Recovered ROGERS-DIVA AXIS INFRAOPS Percentage of Unhealthy ECS Task

The alert lasted ~9 minutes (23:36 → 23:45 UTC-4). The monitor is designed to catch when a high percentage of ECS tasks behind the ALB are unhealthy.

## The Problem with the Original Monitor

The original query had two issues:

**1. Wrong denominator**
The query was dividing unhealthy hosts by *healthy* hosts, not by *total* hosts:

```
# Original (wrong)
sum(last_10m):(
  sum:aws.applicationelb.un_healthy_host_count{project:rogers-diva,env:prod*} by {service}
  / sum:aws.applicationelb.healthy_host_count{project:rogers-diva,env:prod*} by {service}
) * 100 > 20
```

This inflates the percentage. Example: 1 unhealthy + 4 healthy → reports 25% instead of the correct 20%.

**2. Misleading description text**
The alert message body said *"Unhealthy hosts detected > 0"* but the actual threshold is `> 20` (percent). Confusing for anyone triaging.



## The Fix

Corrected query uses `unhealthy / (unhealthy + healthy)` for a true percentage:

```
# Corrected
sum(last_10m):(
  sum:aws.applicationelb.un_healthy_host_count{project:rogers-diva,env:prod*} by {service}
  / (
    sum:aws.applicationelb.un_healthy_host_count{project:rogers-diva,env:prod*} by {service}
    + sum:aws.applicationelb.healthy_host_count{project:rogers-diva,env:prod*} by {service}
  )
) * 100 > 20
```

Validated in Metrics Explorer by splitting into:
- **a** = `sum:aws.applicationelb.un_healthy_host_count{...}`
- **b** = `sum:aws.applicationelb.healthy_host_count{...}`
- **Formula** = `(a / (a + b)) * 100`

The spike on May 26 around 22:30 UTC-4 showed ~22% with the corrected formula — above the 20% threshold, confirming the same incident would have been caught.

## Monitors

| | Monitor |
|---|---|
| Original (wrong denominator) | [176955631](https://app.datadoghq.com/monitors/176955631) |
| Duplicate with corrected query | [288009980](https://app.datadoghq.com/monitors/288009980) |

The duplicate was created to validate the new query in production without touching the live alert. Once confirmed stable, the original should be updated and the duplicate removed.

## Next Steps

- [ ] Monitor 288009980 in production for a few days
- [ ] Update the original monitor (176955631) with the corrected query and fix the description text
- [ ] Delete the duplicate (288009980)
