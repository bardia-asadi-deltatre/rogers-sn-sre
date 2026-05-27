# Rogers Sportsnet+ — SRE Knowledge Base
_Read this first at the start of every session_

## What This Is

Rogers Sportsnet+ (rogers.com/sportsnet) — streaming platform. Deltatre runs the infra/SRE.
Three AWS accounts, one Datadog org.

| Account | Alias | What's in it |
|---------|-------|-------------|
| 321254714924 | aws-rogers-diva | EKS (divabo, vi), self-hosted MongoDB, RabbitMQ, mediaservices |
| 388700888837 | aws-rogers-axis-torino | EKS (forge prd/stg/dev), forge MongoDB, entitlement nodes |
| 878223349253 | aws-rogers-discovery-axis | ISL OData EC2 fleet, ESB, DC, RDGW, staging MongoDB |

## Folder Map

```
RogersSN/
  README.md               ← you are here
  Inventory/
    aws-compute-inventory.md    ← all EC2, EKS, RDS across 3 accounts
  Monitoring/
    current-state.md            ← what's monitored, what's broken, what's missing
    roadmap.md                  ← 3-tier monitoring plan
    excel-checklist-audit.md    ← audit of manager's checklist vs actual DD monitors
    tier1-monitors.json         ← importable Datadog monitor definitions (Tier 1)
  Architecture/           ← (to be built) service maps, traffic flow, dependency diagrams
  Runbooks/               ← (to be built) how to do recurring ops tasks
  Learning/               ← general AWS/tech learning notes (not Rogers-specific ops); use for concepts, reference material, and background knowledge that supports the work but doesn't belong in a product/infra folder
```

## Key External Files

- **Manager's alarm checklist (Excel):**
  `/Users/bardia.asadi/Library/CloudStorage/OneDrive-SharedLibraries-DeltatreS.p.A/SRE - Documents/Alarm_Checklist RogersSN.xlsx`

## Note Conventions

- **Always include an H1 title** at the top of every note. Obsidian shows the filename, but GitHub doesn't — without an H1 the note looks unfinished when shared.

## Folder Structure

Feel free to suggest reorganising folders anywhere in this root when it makes sense. For `Learning/`, keep it flat — Obsidian links are the structuring tool there, not subfolders.

**Special folders:**
- `_scripts/` — standalone scripts referenced from notes; sorts to top
- `_shared/` — notes shared with the team; this is what gets linked externally (see below)
- `Ωassets/` — images and attachments; create only when needed
- `_support/` — sub-notes that have no standalone value (e.g. fragments of a larger topic broken into pieces); use sparingly

## Sharing Notes with the Team

When a note is ready to share:
1. Put the full note in `_shared/` (this is what you link externally via GitHub)
2. Replace the original note in its home folder with a one-line stub + Obsidian wikilink pointing to `_shared/`
3. Send the GitHub URL to the `_shared/` file — GitHub renders Markdown natively, no Obsidian needed on their end

**File naming:** two conventions, depending on type:
- **Note-style files** (findings, decisions, explanations): `Main Topic - Subtopic`, e.g. `ECS - Container Limits and Reservations`, `Datadog - APDEX — current finding and decision`
- **Structural/functional files** (state snapshots, roadmaps, checklists): short kebab-case, e.g. `current-state.md`, `roadmap.md`

## Current Focus

- Monitoring coverage — in progress (see `Monitoring/current-state.md` for full status)
- Architecture mapping — not started
- Runbooks — not started
