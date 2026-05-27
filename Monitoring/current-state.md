# Rogers Sportsnet+ — Monitoring Current State
_Last updated: May 2026 — update this as progress is made_

## What We're Working With

Three AWS accounts, one Datadog org. See `../Inventory/aws-compute-inventory.md` for the full compute picture.

## Manager's Alarm Checklist (Excel)

Shared file maintained collaboratively:
`/Users/bardia.asadi/Library/CloudStorage/OneDrive-SharedLibraries-DeltatreS.p.A/SRE - Documents/Alarm_Checklist RogersSN.xlsx`

Full audit of that sheet vs actual DD monitors: `excel-checklist-audit.md` (this folder).

**Key findings from the audit:**
- IDs `231409103` and `231409709` appear across VOLT/FORGE rows as "Present" — these monitors **do not exist** in DD. Those rows are false positives. Needs correction.
- DivaBO section in Excel is entirely blank but monitors DO exist — Excel needs updating with links.
- 3 host monitors covering Axis Core, ESB, FORGE/VOLT k8s nodes are **permanently muted** (`232952255`, `231406974`, `232952890`) — nobody is getting alerted.

## Finding All Sportsnet+ Monitors in Datadog

The tag strategy is inconsistent across teams. Use this query to find everything:

```
(tag:"customer:rogers" AND (tag:"app_owner:VI" OR tag:"app_owner:DivaBO")) OR tag:"customer:rogers-diva" OR tag:"project:rogers-diva" OR tag:"project:axis-rogers-diva" OR tag:"axis.project_code:rogers-diva" OR tag:"project:axis-rogers"
```

**Why this is necessary:** monitors were created by multiple teams with no agreed tagging standard. Some use `project:rogers-diva`, others `project:rogers` + `app_owner:VI`, others `axis.project_code:rogers-diva`, others have no project tag at all. A single tag filter misses 30–50% of monitors.

**Confirmed counts (May 2026):**
- UI shows 138 total with `customer:rogers OR project:*rogers*`
- ~92 belong to Sportsnet+ (after excluding ~37 DISCOVERY monitors, 1 LBA, a few dead/test ones)
- The query above returns ~92 Sportsnet+ monitors

**Script to export all:**
```bash
DD_API_KEY=$(security find-generic-password -a "$USER" -s datadog-api-key -w)
DD_APP_KEY=$(security find-generic-password -a "$USER" -s datadog-app-key -w)

QUERY='(tag:"customer:rogers" AND (tag:"app_owner:VI" OR tag:"app_owner:DivaBO")) OR tag:"customer:rogers-diva" OR tag:"project:rogers-diva" OR tag:"project:axis-rogers-diva" OR tag:"axis.project_code:rogers-diva" OR tag:"project:axis-rogers"'
ENCODED_QUERY=$(python3 -c "import urllib.parse, sys; print(urllib.parse.quote(sys.argv[1]))" "$QUERY")
PER_PAGE=100
PAGE=0

curl -s "https://api.datadoghq.com/api/v1/monitor/search?query=${ENCODED_QUERY}&per_page=${PER_PAGE}&page=${PAGE}" \
  -H "DD-API-KEY: ${DD_API_KEY}" \
  -H "DD-APPLICATION-KEY: ${DD_APP_KEY}" \
  -H "Accept: application/json" | jq > rogers-sportsnet-monitors.json
```

## Existing Monitors — What's Actually Covered

### AXIS-BE
- ECS CPU + memory (broad, covers all tagged ECS services) ✅
- ECS running tasks < desired / no running tasks ✅
- ECS unhealthy host % > 20% ✅
- APM error rates: WCF services, .NET services, ISL OData, Rocket, Slingshot, Shain ✅
- APM latency: WCF, .NET, Rocket (Apdex), Slingshot (Apdex), Shain, Geolocation, ISL OData ✅
- Redis ElastiCache: CPU + free memory ✅
- ALB 5xx anomaly (production-webfacing) ✅
- ALB internal 5xx ✅
- Redis: evictions, swap, cache miss anomaly ✅
- Synthetics: Presentation Manager, Geolocation API, Neustar Geo API, Rocket CDN, Preference API (Kremer), TVE MVPD ✅
- SSL: watch.sportsnet.ca, *.citytvplus.ca, production.d3-rgr-diva.com ✅
- RDS MSSQL free storage ✅
- Host CPU/disk/memory scoped to account 388700888837 — **MUTED**, not alerting ⚠️

### VOLT / FORGE / Digital
- Entitlements: synthetic HC, APM error rate + latency ✅
- Guishell: synthetic HC + APM health probes ✅
- Forge Distribution API: synthetic HC + APM health probes ✅
- Forge Backoffice: synthetic HC + APM health probes ✅
- K8s Redis (forge): CPU + memory ✅
- K8s container CPU + memory scoped to account 388700888837 ✅
- K8s deployment replicas available < desired ✅
- VOLT Entitlement circuit breaker (log alert) ✅

### DivaBO
- EC2 CPU + free disk (divabo EC2 hosts) ✅
- EC2 MongoDB memory (divabo-mongo hosts) ✅
- K8s host CPU (divabo cluster) ✅
- K8s container CPU + memory (divabo cluster) ✅
- K8s pod memory usage ✅
- K8s pod CPU usage ✅
- Pod restart spike ✅
- Domain validation SSL ✅

### VI
- K8s pod CPU + memory (all VI services) ✅
- K8s pod restart spike ✅
- K8s node CPU (vi cluster) ✅
- EC2 CPU + memory for vi-core-* hosts (covers MongoDB + RabbitMQ EC2 nodes) ✅
- EC2 free disk ✅
- Synthetics: 1st/2nd/3rd set + Streaming Dashboard ✅
- Entitlement warnings (log alert) ✅
- RabbitMQ synthetic ✅
- Conviva video playback failure % — **No Data** ⚠️

## Broken Monitors (No Data — not alerting)

| ID | Name | Action |
|----|------|--------|
| 143183006 | WIP: Rogers MongoDB CPU % (Atlas) | Fix Atlas integration |
| 143177956 | WIP: Rogers MongoDB IOPS % (Atlas) | Fix Atlas integration |
| 70575973 | axis-rogers-isl-odata-api-mongodb error rate | Permanently muted + No Data — review/delete |
| 116397629 | Rogers ISL has unhealthy containers | Permanently muted + No Data — review/delete |
| 116985392 | Rogers Rocket has unhealthy containers | Permanently muted + No Data — review/delete |
| 149030137 | GEOMANAGER_ERROR_RATE_HIGH | No Data — check if service still exists |
| 245280344 | VI Video Playback Failure % (Conviva) | No Data — check Conviva integration |

## Coverage Gaps — Genuine Missing Monitors

### High priority
- **Kafka** — completely unmonitored
- **OpenSearch** — completely unmonitored
- **VOLT Redis + Redis Sentinel** — completely unmonitored
- **VOLT MongoDB** — completely unmonitored
- **VOLT RabbitMQ** — completely unmonitored
- **Host monitors for account 878223349253** (ISL OData EC2 fleet, 50+ hosts) — no infra monitoring at all
- **Muted host monitors** (`232952255`, `231406974`, `232952890`) — unmute or replace

### Medium priority
- AXIS-BE: Search API, Oggi, Notifications — no monitors at all (ECS CPU broad monitor should cover CPU/memory if tagged correctly — verify)
- AXIS-BE: MongoDB Atlas — monitors exist but broken (No Data)
- AXIS-BE: Core DB RDS — no disk monitor
- DivaBO: RabbitMQ-specific monitoring (only EC2 host covered)
- VI: RabbitMQ-specific monitoring (queue depth, message rate — only EC2 host covered)
- No SLOs defined for any product

### Low priority / confirm with team
- Octopus Deploy — no specific monitors (broad host monitor may cover if agent installed)
- Azure Service Bus — confirm if still in use or deprecated for RabbitMQ
- AXIS-BE: Octopus Deploy EC2

### 3rd Party — all missing
- MediaConvert, MediaLive, MediaConnect, MediaTailor (AWS)
- Castlabs
- VPF Spikes

## Decisions / Progress Log

### Tagging (see `tagging-strategy.md` for full context and rollout plan)
- [ ] Bulk-add `proj:rogers-sn` metadata tag to all ~92 existing DD monitors (API script, no behavioral change)
- [ ] Tag AWS resources in account 878223349253 with `proj:rogers-sn` (start here — lowest risk)
- [ ] Tag AWS resources in accounts 321254714924 and 388700888837
- [ ] Validate: `proj:rogers-sn` returns all expected monitors in DD search

### Existing monitor fixes
- [ ] Verify DD monitor IDs `231409103` and `231409709` in DD UI — they appear in Excel but don't exist in exports
- [ ] Unmute or replace host monitors `232952255`, `231406974`, `232952890`
- [ ] Fix MongoDB Atlas monitors (No Data) — check Atlas DD integration
- [ ] Update Excel with DivaBO monitor links (monitors exist, Excel is blank)
- [ ] Confirm DD agent presence on standalone EC2 hosts (ISL OData fleet, mediaservices)
- [ ] Add host-level monitors for account 878223349253 (ISL OData fleet)
- [ ] Add Kafka monitoring
- [ ] Add OpenSearch monitoring
- [ ] Add VOLT Redis / RabbitMQ / MongoDB monitoring
- [ ] Confirm: Azure Service Bus deprecated? Octopus Deploy still in use?
- [ ] Tier 1 monitor JSON created: `tier1-monitors.json` (ready to import, needs service names filled in for APM monitors)

## Files in This Folder
- `tagging-strategy.md` — canonical tag decision (`proj:rogers-sn`), rationale, and rollout plan
- `roadmap.md` — full 3-tier observability plan
- `tier1-monitors.json` — importable DD monitor definitions for Tier 1
- `excel-checklist-audit.md` — audit of manager's Excel checklist vs actual DD monitors
- `current-state.md` — this file (update as you go)
