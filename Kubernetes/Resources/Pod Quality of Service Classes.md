====================================================================================================
                                **Quality of Service Classes**
====================================================================================================

Quality Of Services Classes : 

Kubernetes assigns every Pod a QoS class based on the resource requests and limits of its component Containers.

QoS classes are used by Kubernetes to decide which Pods to evict from a Node experiencing Node Pressure.

QoS Classes as below,

🟢 Guaranteed --> Stickest resoruce requests, and Memory limits to containers.
🟢 Burstable --> Only requests set, OR requests ≠ limits.
🟢 BestEffort -->  Neither requests nor limits set.

====================================================================================================

### What Kubernetes Allows at Runtime

- Pod is **scheduled** based on the 250m CPU / 128Mi memory request — this part works normally.
- At runtime, the container is **free to use as much CPU and memory as the node has available** — there is **no ceiling enforced by Kubernetes itself**.
- If the node comes under **memory pressure**, the Kubelet will **evict Burstable pods before Guaranteed pods** but **after BestEffort pods**.

**Eviction priority (lowest to highest protection):**
BestEffort  →  Burstable  →  Guaranteed
(evicted first)              (evicted last)

====================================================================================================
                            **Some behavior is independent of QoS class**
====================================================================================================

Any Container exceeding a resource limit will be killed and restarted by the kubelet without affecting other Containers in that Pod.

If a Container exceeds its resource request and the node it runs on faces resource pressure, the Pod it is in becomes a candidate for eviction. If this occurs, all Containers in the Pod will be terminated. Kubernetes may create a replacement Pod, usually on a different node.

The resource request of a Pod is equal to the sum of the resource requests of its component Containers, and the resource limit of a Pod is equal to the sum of the resource limits of its component Containers.

The kube-scheduler does not consider QoS class when selecting which Pods to preempt. Preemption can occur when a cluster does not have enough resources to run all the Pods you defined.

The QoS class is determined when the Pod is created and remains unchanged for the lifetime of the Pod. If you later attempt an in-place resize that would result in a different QoS class, the resize is rejected by admission.

====================================================================================================

### CFS (Linux Completely Fair Scheduler) Perspective

This is where it gets deeper. When no CPU limit is set, **no CFS quota is configured** for the container's cgroup.

#### With Limits Set:
Kubernetes translates CPU limits into CFS bandwidth control parameters:

cpu.cfs_quota_us  = <limit in millicores> / 1000 * cfs_period_us
cpu.cfs_period_us = 100000 (100ms default)


For example, a `500m` limit means:

cpu.cfs_quota_us = 50000  → container gets 50ms of CPU every 100ms


#### Without Limits (Your Case):

cpu.cfs_quota_us  = -1   ← means UNLIMITED, no throttling ever
cpu.cfs_period_us = 100000
cpu.shares        = <derived from request (250m)>

The `cpu.shares` value is set from the **request**:

cpu.shares = (250m / 1000) * 1024 = 256 shares

> **`cpu.shares` doesn't cap usage — it only controls relative priority when the CPU is contested.**

### Two Distinct Scenarios for CFS

#### Scenario 1 — Node is idle / has spare CPU

Container with 250m request + no limit
         ↓
CFS sees cpu.cfs_quota_us = -1
         ↓
Container can burst to 100%, 200%, 400% CPU — whatever is free
         ↓
No throttling occurs at all

✅ Great for bursty workloads — they get to use free cycles.

#### Scenario 2 — Node CPU is Contested

Multiple containers competing for CPU
         ↓
CFS uses cpu.shares to divide CPU proportionally
         ↓
Container with 256 shares gets proportional slice
vs other containers with their own shares
         ↓
Still no hard cap — just fair weighted sharing

⚠️ Your container won't be throttled, but it also won't starve others — CFS keeps it fair via shares.

### Memory — No Limits Case

For memory, there's **no "sharing" concept** like CPU shares. It's simpler but more dangerous:

No memory limit set
       ↓
cgroup memory.limit_in_bytes = -1 (unlimited)  
       ↓
Container can grow until node runs out of memory
       ↓
Linux OOM Killer kicks in (not Kubernetes)
       ↓
OOM Killer picks a process to kill based on oom_score
(may kill your container OR another process on the node)

====================================================================================================