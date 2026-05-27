[[Datadog Tag]]
# Rogers Sportsnet+ — Tagging Strategy
_Decided: May 2026_

## The Problem

Tags across AWS resources and Datadog monitors are inconsistent because each team applied their own convention at creation time. There is no single tag that reliably identifies "this belongs to Rogers Sportsnet+." The result is a fragile multi-clause query just to find all our monitors:

```
(tag:"customer:rogers" AND (tag:"app_owner:VI" OR tag:"app_owner:DivaBO")) OR tag:"customer:rogers-diva" OR tag:"project:rogers-diva" OR tag:"project:rogers-diva-axis" OR tag:"axis.project_code:rogers-diva" OR tag:"project:rogers"
```

A single tag filter misses 30–50% of monitors. This makes audits, dashboards, and automation unreliable.

## Options Considered

### Option A — New canonical tag `proj:rogers-sn` (original plan)
Purely additive: add alongside existing tags. Nothing breaks. Downside: two sets of tags to maintain until old ones are cleaned up.

### Option B — Normalize everything to `project:rogers-diva`
Retag the 7 `project:axis-rogers` monitors and the DivaBO/VI monitors (currently tagged `customer:rogers` + `app_owner:*`) to `project:rogers-diva`. This reuses the tag already on the majority of monitors, avoids introducing a new namespace, and results in a single clean filter with no new tag to communicate to the team. Downside: anyone filtering on `project:axis-rogers` or `app_owner:DivaBO/VI` would need to update their saved searches.

**Currently leaning toward Option B** — lower overhead, reuses existing convention, one conversation with the manager to align.

## The Decision

**Add a new canonical tag — `proj:rogers-sn` — to all Rogers Sportsnet+ resources.**

Do not remove or change existing tags. Adding alongside existing tags is safe: nothing that currently works will break, because all existing monitors, dashboards, and queries still filter on the old tags. This is purely additive.

Once the tag is on everything, the find-everything query becomes just:
```
proj:rogers-sn
```

Old tags can be cleaned up lazily over time, or left in place — there's no pressure to.

## What to Tag

### Datadog — two levels

**Monitor metadata tags** (labels on the monitor object, don't affect alert behaviour):
- Add `proj:rogers-sn` to all ~92 existing monitors via the DD API
- Safe to do any time, no freeze needed, no behavioral change

**Monitor query tags** (inside the `avg:metric{tags}` query — these control what gets evaluated):
- Leave existing queries alone for now
- For all *new* monitors, include `proj:rogers-sn` in the query scope

### AWS — resource tags

Add `proj:rogers-sn` to resources across all three accounts. The Datadog agent picks up host tags from AWS automatically; tags flow into metrics after the next sync.

Suggested order (lowest risk first):
1. **878223349253** (ISL OData EC2 fleet, ESB, DC, RDGW, staging MongoDB) — no infra monitoring exists here anyway, so nothing to break
2. **321254714924** (divabo, VI EKS, MongoDB, RabbitMQ)
3. **388700888837** (forge prd/stg/dev, VOLT entitlement nodes, forge MongoDB)

Resource types to tag in each account: EC2 instances, ECS clusters + services, EKS node groups, RDS instances, ElastiCache clusters, load balancers.

## Going Forward

All new resources and monitors created for this project should include `proj:rogers-sn` from the start. This is the single source of truth for project membership.

## Implementation Tasks

### If going with Option B (normalize to `project:rogers-diva`)
- [ ] Align with manager — let them know saved searches on `project:axis-rogers` and `app_owner:DivaBO/VI` will need updating
- [ ] Retag the 7 `project:axis-rogers` monitors to `project:rogers-diva` (IDs: 70575973, 77889712, 116397629, 116985392, 143177956, 143183006, 265595548)
- [ ] Retag DivaBO/VI monitors from `customer:rogers` + `app_owner:*` to `project:rogers-diva`
- [ ] Validate: confirm `project:rogers-diva` returns all ~92 expected monitors
- [ ] Tag AWS resources in account 878223349253
- [ ] Tag AWS resources in account 321254714924
- [ ] Tag AWS resources in account 388700888837

### If going with Option A (new `proj:rogers-sn` tag)
- [ ] Bulk-add `proj:rogers-sn` metadata tag to all ~92 existing DD monitors (API script, can be done anytime)
- [ ] Tag AWS resources across all three accounts
- [ ] Add `proj:rogers-sn` to query scope of new monitors going forward
- [ ] Validate: confirm `proj:rogers-sn` returns all expected monitors in DD monitor search
