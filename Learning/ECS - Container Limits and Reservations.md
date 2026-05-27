
[[Task Level]]
[[Container Level]]

## ECS resource model, simplified

| Level         | Setting                |                                        Means |                                         Placement? |                                                   Runtime enforcement? | Kubernetes-ish equivalent              |
| ------------- | ---------------------- | -------------------------------------------: | -------------------------------------------------: | ---------------------------------------------------------------------: | -------------------------------------- |
| **Task**      | Task CPU               |            Total CPU size for the whole task | Important in Fargate; optional/less central on EC2 |    Linux yes-ish; **Windows task-level CPU not enforced the same way** | Pod-level resource envelope, roughly   |
| **Task**      | Task memory            | Total memory size for all containers in task | Important in Fargate; optional/less central on EC2 | Linux yes-ish; **Windows task-level memory not enforced the same way** | Pod-level memory envelope, roughly     |
| **Container** | CPU units              | CPU share / requested CPU for this container |      Yes, affects whether ECS thinks host has room |                     Not a hard CPU cap in the same clean way as memory | K8s CPU request-ish, not exactly limit |
| **Container** | Memory [[reservation]] |           Soft memory reserved for placement |                                                Yes |                                               Soft, not kill threshold | K8s memory request                     |
| **Container** | Memory [[hard limit]]  |                 Max memory container can use |                              Yes / also considered |                               Yes, container can be killed if exceeded | K8s memory limit                       |
| **Container** | Host port              |                         Port on the EC2 host |                                Yes, very important |                                                 Yes, port must be free | `hostPort` in Kubernetes               |

## Your three statements

### 1. "Task level means all containers combined should be within that limit?"

Yes. Conceptually:

```text
Task CPU/memory = envelope for the whole task
containers inside task share that envelope
```

Very true for Fargate. Less clean for ECS-on-EC2 Windows.

### 2. "Windows cannot follow that properly?"

Correct enough.

For **Windows ECS on EC2**, task-level CPU/memory is not the main reliable enforcement boundary. Container-level settings matter more.

So for Windows EC2 tasks, don't overthink task-level CPU/memory if they are blank/null.

### 3. "No CPU limit at container level, only reservation?"

Basically yes.

ECS container CPU is mostly about **scheduling/share**, not a clean hard max like memory.

Memory has a clearer hard limit:

```text
memory = hard limit
memoryReservation = soft reservation
```

CPU is more like:

```text
cpu = how much CPU this container is allocated/weighted/reserved for scheduling
```

## For your ISL case

| Item   |                    Host |       OData task/container |
| ------ | ----------------------: | -------------------------: |
| CPU    |    `m7a.large` = 2 vCPU |  `2048` CPU units = 2 vCPU |
| Memory |                   8 GiB |     hard limit about 7 GiB |
| Port   | host has one port `443` | task needs host port `443` |

So the practical result is simple:

```text
One OData task basically consumes one whole EC2 host.
```

Because of:

```text
2 vCPU task on 2 vCPU host
~7 GiB memory limit on 8 GiB host
fixed host port 443
```

That's why:

```text
84 OData tasks ≈ 84 usable Windows EC2 hosts
```
