System Manages Memoery in small chunks called pages (Typically 4KiB). Huge Pages are much larger memory pages like 2Mib or 1Gib that the OS can allocate as a single block.

- hugepages-2Mi: 80Mi

this is resource limit set on a container,

hugepages-2Mi : Use huge pages of size 2 MiB each

80Mi : The container's total huge page memory cannot exceed 80 MiB

Formula :

80 MiB total limit ÷ 2 MiB per page = 40 huge pages

So the container is allowed to allocate at most 40 huge pages of 2 MiB size.

✅ Allocating 39 pages = 78 MiB → Allowed
✅ Allocating 40 pages = 80 MiB → Allowed
❌ Allocating 41 pages = 82 MiB → Fails / Rejected


Why mention "default page size is 4 KiB"?

The note about the 4 KiB default is just giving you context — it's the normal page size on most Linux systems. Huge Pages exist specifically because 4 KiB pages can cause overhead for memory-intensive workloads (like databases or high-performance computing), since the CPU's memory management unit (MMU) has to track thousands of tiny pages.
By using 2 MiB huge pages, you reduce that overhead significantly.

In a Pod spec, it looks like this:

resources:
  limits:
    hugepages-2Mi: 80Mi
    memory: 1Gi
  requests:
    hugepages-2Mi: 80Mi
    memory: 1Gi

Note: Huge page requests and limits must be equal in Kubernetes — you can't set a request lower than the limit for huge pages.

============================================================================================

🏦 Scenario: Running PostgreSQL in Kubernetes

You're deploying PostgreSQL inside a Kubernetes Pod. PostgreSQL manages a large chunk of memory called shared_buffers — this is where it caches frequently accessed data to avoid hitting the disk every time.

shared_buffers = 80MB

## Without Huge Pages (Default 4 KiB pages)

PostgreSQL needs 80 MiB of shared memory.

80 MiB ÷ 4 KiB = 20,480 tiny pages

The Linux kernel now has to track 20,480 individual page entries in the CPU's Translation Lookaside Buffer (TLB). The TLB is a small hardware cache that maps virtual → physical memory addresses.
The Problem:

- TLB can only hold a limited number of entries
- With 20,480 pages, it constantly runs out of space
- This causes TLB misses → CPU has to walk the page table manually every time
- Under heavy query load, this becomes a serious performance bottleneck

With Huge Pages (2 MiB pages)

Same 80 MiB of shared memory, but now:

80 MiB ÷ 2 MiB = only 40 huge pages

The kernel now tracks just 40 entries in the TLB instead of 20,480.

The Result:

- Far fewer TLB misses
- Faster memory access for every query
- PostgreSQL under heavy load performs significantly better

============================================================================================


The Kubernetes Configuration

apiVersion: v1
kind: Pod
metadata:
  name: postgres-pod
spec:
  containers:
  - name: postgres
    image: postgres:15
    env:
    - name: POSTGRES_SHARED_BUFFERS
      value: "80MB"
    resources:
      requests:
        memory: 1Gi
        hugepages-2Mi: 80Mi      # Request exactly 40 huge pages
      limits:
        memory: 1Gi
        hugepages-2Mi: 80Mi      # Hard cap: cannot exceed 80 MiB of huge pages
```

---

## What Happens at Runtime

| Action | Result |
|---|---|
| Pod starts, PostgreSQL initializes shared_buffers | Kubernetes **reserves 40 × 2MiB** huge pages from the Node |
| PostgreSQL tries to use 78 MiB of huge page memory | ✅ Allowed — within the 80 MiB limit |
| PostgreSQL tries to allocate a 41st huge page (82 MiB) | ❌ **Fails** — limit enforced by kernel |
| Another Pod on same Node tries to steal those pages | ❌ **Blocked** — pages are pre-reserved, not shared |

---

## The Big Picture
```
Node has 200 MiB of Huge Pages pre-allocated by admin
│
├── postgres-pod     → hugepages-2Mi: 80Mi  (40 pages) ✅ reserved
├── redis-pod        → hugepages-2Mi: 60Mi  (30 pages) ✅ reserved
├── app-pod          → hugepages-2Mi: 60Mi  (30 pages) ✅ reserved
│
└── Total used: 200Mi — Node is now FULL for huge pages
    any new pod requesting huge pages → ❌ won't be scheduled here


## Take Takeaway: 

hugepages-2Mi: 80Mi told Kubernetes:

Reserve exactly 40 huge pages of 2MiB for this PostgreSQL container — no more, no less — so it gets guaranteed, fast, pre-allocated memory that no other process can steal.

================================================================================================================================================

Question : Who decide the page size?

The Linux Node (kernel) decides the page size — NOT PostgreSQL, NOT Kubernetes.

┌─────────────────────────────────────────────────────┐
│                   LINUX NODE                        │
│                                                     │
│  Admin pre-configures huge page SIZE at boot time   │
│  e.g.  echo 100 > /sys/kernel/mm/hugepages/         │
│              hugepages-2048kB/nr_hugepages           │
│                                                     │
│  This FIXES the page size as 2MiB on this node      │
└─────────────────┬───────────────────────────────────┘
                  │ Node advertises available huge pages
                  ▼
┌─────────────────────────────────────────────────────┐
│                  KUBERNETES                         │
│                                                     │
│  Reads what the Node has available                  │
│  Schedules your Pod ONLY on a node that has         │
│  enough hugepages-2Mi available                     │
│                                                     │
│  hugepages-2Mi: 80Mi  ← you're just saying          │
│  "I need 80Mi worth of whatever 2Mi pages           │
│   the node has pre-allocated"                       │
└─────────────────┬───────────────────────────────────┘
                  │ Pod lands on node, container starts
                  ▼
┌─────────────────────────────────────────────────────┐
│                  POSTGRESQL                         │
│                                                     │
│  Requests shared memory from the kernel             │
│  Kernel serves it using the pre-allocated           │
│  2MiB huge pages                                    │
│                                                     │
│  PostgreSQL doesn't pick the size —                 │
│  it just gets fast memory automatically             │
└─────────────────────────────────────────────────────┘

Your YAML is Just a REQUEST, Not a Declaration

hugepages-2Mi: 80Mi

"Schedule me on a node that already has 2MiB huge pages available, and reserve 80MiB worth of them for me."

You are not configuring the page size here. You are selecting which pre-existing huge page pool on the node you want to consume from.

Real-World Scenario :

Node Setup (Admin does this once)

# On the Linux Node — done by the sysadmin or node startup script

# Tell kernel to pre-allocate 100 huge pages of 2MiB each
echo 100 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

# Verify
cat /proc/meminfo | grep Huge
# AnonHugePages:         0 kB
# HugePages_Total:     100       ← 100 pages available
# HugePages_Free:      100
# Hugepagesize:       2048 kB    ← each page is 2MiB

Kubernetes then auto-discovers this and advertises it as a schedulable resource on that node.

What If You Put the Wrong Size?

# Node only has 2Mi huge pages pre-allocated
# But you request 1Gi huge pages

hugepages-1Gi: 80Gi   ← ❌ Pod stays in Pending forever
                           No node has 1Gi huge pages available

Kubernetes will never schedule the Pod because no node matches.

Bottom line: By writing hugepages-2Mi in your YAML, you are not creating 2MiB pages — you are asking to be placed on a node where the kernel has already set up 2MiB huge pages.

=========================================================================================================

## Final Mental Modal:

1. Page Size & reservation for Memory will be decide by the Linux Admin on the Node based on the request.

2. Postgresql configuration has to be done by database engineer to refer the decided Huge pages size.

3. Now we will have to mention the requests & limits for pod manifest to refer the huge-pages limit that is set by linux admin & Database Engineer.

4. Upon deployment of postgres container, the kubelet auto-discover the node where the decided huge-page size is defined & available memory for the decided pages.

===========================================================================================================