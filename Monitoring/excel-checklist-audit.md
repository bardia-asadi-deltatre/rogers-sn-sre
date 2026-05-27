# Manager's Excel Checklist — Final Audit
_Against confirmed DD monitor inventory (May 2026)_

---

## ⚠️ Two Ghost Monitor IDs in the Excel

IDs **231409103** and **231409709** appear across multiple VOLT/FORGE rows as "Present" — but these monitors **do not exist** in any of our exports (tested across 138+ monitors with multiple queries). Either deleted or never created. Affects: VOLT Entitlement k8s, VOLT NGINX, FORGE Guishell k8s, Forge Distribution API k8s, Forge WCM, VOD Uploader, Forge2Diva, Cloudinary, FORGE k8s Node, VOLT k8s Node Main.

---

## AXIS-BE

| Service | Type | Status | Notes |
|---------|------|--------|-------|
| Presentation Manager | Web App | ⚠️ Partial | Synthetic HC ✅ (`156594940`). Infra (CPU/Disk/Memory/Network) ❌ |
| Search API | Web App | ❌ Missing | Nothing |
| Oggi (EPG) | Web App | ❌ Missing | Nothing |
| Notifications | Web App | ❌ Missing | Nothing |
| Kafka | AWS Service | ❌ Missing | Nothing |
| OpenSearch | AWS Service | ❌ Missing | Nothing |
| Redis (ElastiCache) | Redis | ⚠️ Partial | CPU ✅ Memory ✅ — Disk/Network ❌ |
| ISL Integration | ECS-EC2 | ⚠️ Partial | ECS CPU+Memory ✅ (broad). No APM, no Disk/Network |
| ISL-Odata | ECS-EC2 | ✅ Good | ECS CPU+Memory ✅, Error Rate ✅, Latency ✅. Disk/Network ❌ |
| Slingshot | ECS-Fargate | ✅ Good | ECS CPU+Memory ✅, Error Rate ✅, Latency ✅, Apdex ✅ |
| Nightwatch | ECS-Fargate | ⚠️ Partial | ECS CPU+Memory ✅ only (broad .net monitor) |
| Shain | ECS-Fargate | ✅ Good | ECS CPU+Memory ✅, Error Rate ✅, Latency ✅ |
| Rocket API | ECS-Fargate | ✅ Good | ECS CPU+Memory ✅, Error Rate ✅, Latency ✅, Apdex ✅, CDN Synthetic ✅ |
| Populist | ECS-Fargate | ⚠️ Partial | ECS CPU+Memory ✅ only (broad .net monitor) |
| Geolocation | ECS-Fargate | ✅ Good | ECS CPU+Memory ✅, Error Rate ✅, Latency ✅, Synthetic HC ✅ |
| Reception, MRSS, Sync, GeoManager | ECS-Fargate | ⚠️ Partial | ECS CPU+Memory ✅ only. GeoManager APM has **No Data** |
| MongoDB (Atlas) | Mongo Atlas | ⚠️ Broken | WIP monitors exist but **No Data** (`143183006`, `143177956`) |
| Core DB RDS | RDS | ❌ Missing | Nothing. MSSQL RDS free storage exists (`265595925`) — check if this covers it |
| Axis Core / ESB | EC2 | ⚠️ Muted | Host CPU/Disk/Memory exist but **MUTED** (`232952255`, `231406974`, `232952890`). APM Error Rate/Latency ✅ |
| Octopus Deploy | EC2 | ❌ Missing | No monitors. Broad host monitor may cover it if DD agent installed |

---

## VOLT

| Service | Type | Status | Notes |
|---------|------|--------|-------|
| Entitlement | k8s Pod | ✅ Good | Synthetic HC ✅, Error Rate ✅, Latency ✅. k8s CPU/Memory — ghost IDs, unconfirmed |
| Redis | Redis | ❌ Missing | Nothing |
| Redis Sentinel | Redis | ❌ Missing | Nothing |
| NGINX ingress | k8s Pod | ❓ Unverified | Only ghost IDs (231409103/231409709) |
| RabbitMQ | Service Bus | ❌ Missing | Nothing |
| MongoDB | Mongo | ❌ Missing | Nothing |
| VOLT Diva Delta Feed | k8s Pod | ❌ Missing | Nothing |
| k8s Node Main | k8s Node | ❓ Unverified | Ghost IDs only |
| k8s Node Entitlement | k8s Node | ⚠️ Muted | Uses broad host monitors, **MUTED** |
| Azure Service Bus | Service Bus | ❌ Missing | Nothing (may be deprecated for RabbitMQ — confirm with team) |

---

## FORGE

| Service | Type | Status | Notes |
|---------|------|--------|-------|
| Guishell | k8s Pod | ⚠️ Partial | Synthetic HC ✅ (`153470818`), APM health probes ✅. k8s infra — ghost IDs |
| Distribution API | k8s Pod | ⚠️ Partial | Synthetic HC ✅ (`153471586`), APM health probes ✅. k8s infra — ghost IDs |
| Forge WCM / VOD Uploader / Forge2Diva / Cloudinary | k8s Pod | ❓ Unverified | Ghost IDs only |
| MongoDB | k8s Node | ⚠️ Muted | Broad host monitors, **MUTED** |
| k8s Node | k8s Node | ⚠️ Muted | Broad host monitors, **MUTED** |
| Redis | k8s Pod | ✅ | CPU ✅ Memory ✅ (`231112386`, `231113176`) |

---

## DivaBO

> Excel updated May 2026 — CPU and Memory monitors linked for all k8s pods. Original sheet had all DivaBO as "Missing" because the CPU monitor was tagged `app_owner:VI` instead of `app_owner:DivaBO`, making it invisible to anyone searching by team.

> **Note:** Disk monitoring not applicable for pods — they are stateless/ephemeral. CPU and Memory are sufficient.

### k8s Pods — CPU ✅ Memory ✅ (Excel linked)

| Service | Type | Status | Notes |
|---------|------|--------|-------|
| Video Feeds API | k8s Pod | ✅ | CPU ✅ Memory ✅ |
| Gateway | k8s Pod | ✅ | CPU ✅ Memory ✅ |
| Diva Sport API | k8s Pod | ✅ | CPU ✅ Memory ✅ |
| LSS Worker | k8s Pod | ✅ | CPU ✅ Memory ✅ |
| Scheduler | k8s Pod | ✅ | CPU ✅ Memory ✅ |
| Shell | k8s Pod | ✅ | CPU ✅ Memory ✅ |
| VOD API | k8s Pod | ✅ | CPU ✅ Memory ✅ |
| VOD Worker | k8s Pod | ✅ | CPU ✅ Memory ✅ |
| LSS API | k8s Pod | ✅ | CPU ✅ Memory ✅ |
| Video Feeds Worker | k8s Pod | ✅ | CPU ✅ Memory ✅ |

> ⚠️ **Open issue — CPU vs Memory group count mismatch:** Memory monitor shows 10 groups, CPU monitor shows only 7. Missing from CPU: `videofeeds-api`, `divasport-api`, `vod-worker`. Tried changing the CPU query from `cluster_name` to `kube_cluster_name` (to match memory query) but still only 7 groups — so the tag difference is not the cause. **Speculation:** those 3 pods likely have no CPU limits configured; Datadog drops groups where the denominator (`kubernetes.cpu.limits`) returns no data. **Action needed:** confirm with DivaBO team whether those 3 pods intentionally have no CPU limits set.

### Other DivaBO Services

| Service | Type | Status | Notes |
|---------|------|--------|-------|
| Diva Data Connector | k8s Pod | ❌ Missing | Not covered by the pod monitors — none of the 10 groups map to it |
| k8s Node | k8s Node | ✅ | Host CPU ✅ (`166908008`) |
| NGINX ingress | k8s Pod | ❌ Missing | Nothing |
| MongoDB | Mongo | ⚠️ Partial | EC2 Memory monitor exists (`149457784`). No Atlas/app-level |
| RabbitMQ | Service Bus | ❌ Not Required | Marked N in Excel |

---

## VI

| Service | Type | Status | Notes |
|---------|------|--------|-------|
| All k8s Pods (Streaming Dashboard, Scheduler, L2V, EPG, etc.) | k8s Pod | ✅ | CPU ✅ Memory ✅ Pod Restart ✅. Disk not applicable — pods are stateless/ephemeral |
| Synthetics (1st/2nd/3rd Set, Streaming Dashboard) | Synthetic | ✅ | All confirmed |
| k8s Node | k8s Node | ⚠️ Partial | CPU ✅ (`166908912`). Disk/Memory at node level ❌ |
| MongoDB (EC2 hosts) | EC2 | ✅ | EC2 CPU ✅ Memory ✅ scoped to `*vi-core-*` |
| RabbitMQ | Service Bus | ⚠️ Partial | Same EC2 host monitors cover the nodes — no RabbitMQ-specific (queue depth, message rate) |

---

## 3rd Party

| Service | Status |
|---------|--------|
| Castlabs | ❌ Missing |
| MediaConvert | ❌ Missing |
| MediaLive | ❌ Missing |
| MediaConnect | ❌ Missing |
| MediaTailor | ❌ Missing |
| VPF Spikes | ❌ Missing |

---

## Summary

| | Count |
|--|--|
| ✅ Confirmed present & good | ~25 services |
| ⚠️ Partial / muted / broken | ~20 services |
| ❓ Ghost IDs — need manual DD check | ~10 services |
| ❌ Genuinely missing | ~20 services |

**Immediate actions:**
1. Verify IDs `231409103` and `231409709` exist in DD — if not, those rows are false "Present"
2. Unmute or delete the 3 muted host monitors (`232952255`, `231406974`, `232952890`) — they're covering Axis Core, ESB, FORGE/VOLT k8s nodes
3. Fix the 2 WIP MongoDB Atlas monitors (No Data)
4. ~~Update Excel with DivaBO monitors that exist but aren't linked~~ ✅ Done — CPU & Memory linked for all 10 DivaBO pods
5. Investigate why 3 DivaBO pods (videofeeds-api, divasport-api, vod-worker) are missing from CPU monitor — likely no CPU limits set
5. Everything under 3rd Party and VOLT Redis/MongoDB is a genuine gap
6. Some confirmed monitors may be routing notifications to individual recipients (e.g. infra team members) rather than teams-rogers-diva-alert-p1 / teams-rogers-diva-alert-p2. Need to audit notification targets across all monitors and standardise routing to the shared channels.
