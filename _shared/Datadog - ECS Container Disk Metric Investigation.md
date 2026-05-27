# Datadog - ECS Container Disk Metric Investigation

## Context

We investigated whether Datadog disk metrics for the following ECS service can be used as evidence of ECS EC2 host / container-instance disk monitoring:

```
service:rogers-diva-isl-odata-api
clustername:production-isl-odata
```

The Datadog query used:

```
avg:system.disk.in_use{project:*rogers*,env:production,service:rogers-diva-isl-odata-api} by {host}
```

At first glance this looked like host disk telemetry given the metric name `system.disk.in_use`, but the `host` values did not map to EC2 instance IDs or normal EC2 hostnames.

---

## What We Found

The `host` value in Datadog matches the ECS/Docker container runtime identity, not the EC2 instance:

```
EC2 hostname:        EC2AMAZ-KQNILAP
Docker container ID: 9e7cb900f84f87c5cb8282f1eb43769e54b64e0e87d586d964b462bd4a5a344c
Container hostname:  9e7cb900f84f
Datadog host:        9e7cb900f84f
```

**Datadog host == first 12 characters of the Docker/ECS container runtime ID.**

Adding `by {host,device}` to the query confirms `device:c` with ~0.10% usage — far too small to represent a real EC2 volume. This is the container's own `C:` drive (Windows container layer), not the host disk.

---

## Why This Is Happening

The application image has the Datadog .NET Tracer (v1.28.8) baked in at `C:\Program Files\Datadog\.NET Tracer`. The task definition has `DD_RUNTIME_METRICS_ENABLED=true`, which causes the tracer to emit runtime and system metrics — including disk — via DogStatsD on port 8125. Port 8125 is mapped to the host in the task definition, so these metrics flow directly to Datadog.

There is no Datadog agent sidecar or daemon involved. The tracer was added for APM/tracing purposes; the system metrics are an unintentional side effect.

---

## Why This Metric Should Not Be Used for ECS Host Disk Monitoring

This metric does not answer:

```
Is the ECS EC2 host disk filling up?
Is the EC2 root volume filling up?
Can the ECS node still place/run tasks safely?
```

It answers:

```
What does disk usage look like from inside this application container?
```

That is not useful for the original requirement of ECS instance disk monitoring.

---

## Why It Needs to Stop

We are ingesting disk metrics that are operationally meaningless at container-level granularity. Additionally, this creates high cardinality in Datadog — one host-like identity per container runtime ID, with a new timeseries on every deployment. This is both misleading and wasteful from an ingestion/cost perspective.

---

## Why We Can't Reuse It for Host Disk Metrics

The .NET tracer collects disk metrics by calling .NET's `DriveInfo.GetDrives()` API, which enumerates logical drive letters. A bind-mounted host volume would appear as a folder on `C:`, not as a separate drive letter, so it would not be picked up. This is not a configurable disk check — it is a passive system call inside the tracer. Only a real Datadog agent with a proper disk check can be pointed at specific mount paths.

---

## Recommendations

1. Set `DD_RUNTIME_METRICS_ENABLED=false` in the `production-isl-odata` task definition to stop emitting these metrics. If runtime metrics are needed in future, a proper Datadog agent sidecar should be added with explicit configuration.
2. Do not include `system.disk.in_use{service:rogers-diva-isl-odata-api}` in any ECS host disk coverage story.
3. For actual ECS EC2 host disk monitoring, install the CloudWatch Agent on the EC2 instances via the ASG launch template User Data script, then ingest those metrics into Datadog via the AWS integration.
