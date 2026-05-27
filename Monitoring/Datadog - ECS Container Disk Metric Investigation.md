# Datadog - ECS Container Disk Metric Investigation

> Shared with team — full findings live here:
> [[_shared/Datadog - ECS Container Disk Metric Investigation]]


---

## My Working Notes — How We Got Here

**Step 1 — noticed suspicious values**

[[system.disk.in_use]] 
Queried `system.disk.in_use` by `{host,device}`. Device showed `c:` but values were ~0.10%, which is nowhere near a real EC2 host disk. Added `by {host,device}` to confirm:

```
avg:system.disk.in_use{project:*rogers*,env:production,service:rogers-diva-isl-odata-api} by {host,device}
```

**Step 2 — confirmed host = container ID**

The `host` values were short hex strings like `9e7cb900f84f` — not EC2 instance IDs. SSHed into an EC2 node and ran:

```powershell
docker ps
```

Confirmed the container hostname matched the first 12 chars of the Docker container ID, which matched the Datadog `host` tag exactly.

**Step 3 — no agent anywhere**

Checked the task definition — no sidecar container, no daemon, no agent service. Only the application container. Port 8125/udp was mapped to the host, which is DogStatsD.

**Step 4 — found the tracer in the image**

Exec'd into a running container:

```powershell
docker exec <container_id> powershell -Command "Get-ChildItem C:\ -Recurse -Filter 'Datadog.Trace*' -ErrorAction SilentlyContinue"
```

Found the .NET tracer at `C:\Program Files\Datadog\.NET Tracer` (v1.28.8, dated 2021).

**Step 5 — connected the dots**

Task definition has `DD_RUNTIME_METRICS_ENABLED=true`. That env var + tracer + port 8125 mapped = system metrics including disk flowing to Datadog unintentionally.

**Step 6 — ruled out the mount workaround**

Considered mounting host `C:\` as a bind mount so the tracer would see host disk. Ruled out because the tracer uses `DriveInfo.GetDrives()` which only sees drive letters, not subdirectories. A bind mount to `C:\hostdisk` would be invisible to it.
