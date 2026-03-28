==========================================================================================================
                                    CPU measurement units in Kubernetes
==========================================================================================================

Kubernetes introduces its own unit system to be cloud-agnostic and precise across any hardware.

![CPU units in Kubernetes](image.png)

Kubernetes expresses CPU in two equivalent ways — you can mix them freely within a cluster:

- 0.5,1, and 2 : 1 means one full logical CPU
- 1000m - millicore notation. m --> milli, so 1000m = 1 CPU, 500m = 0.5 CPU, 10m = 0.01 CPU.

# Requests vs Limits :

Every container spec has two separate CPU fields:

- requests : the CPU the scheduler guarantees. The pod is only placed on a node with at least this much free capacity.

- limits : the CPU ceiling. If the container tries to use more, the kernel throttles it (via CFS quota)

## Practical Scenarios :

My configuration is something like below:

requests:
  cpu: 100m
limits:
  cpu: 500m

This is considering the Burstable pod.

# Two Different Phases Are Happening

1. Scheduling Phase (Kubernetes decision):

Kubernetes scheduler looks only at requests:

the pod only required 0.1 CPU.

2. Runtime Phase (Linux enforcement):

Now the Completely Fair Scheduler

Our Container is allowed:

| Level         | CPU behavior         |
| ------------- | -------------------- |
| Guaranteed    | 100m (minimum share) |
| Burst allowed | up to 500m           |
| Hard ceiling  | 500m                 |

# How It Actually Operates

Case 1: Low CPU contention (idle node):

If node has free CPU, than our container can use upto 500m continously.

We will get full bursts without throttling. Fast performance, No restriction felt

Case 2: Moderate contention:

Now multiple pods are competing, Scheduler ensures that we will get at least ~100m share.

We can still get bursting upto 500m, But the Performance fluctuates, Some latency Variation.

Case 3: High contention (CPU pressure):

Now node is saturated, Kernel enforces fairness:

- We will guaranteed onyl 100m
- Bursting becomes limited
- If we try >500m → throttling kicks in

# What our app experiences

| Situation  | Behavior                     |
| ---------- | ---------------------------- |
| Idle node  | Fast (uses ~500m)            |
| Busy node  | Slower (~100–300m effective) |
| Over limit | Throttled at 500m            |

🔥 Real Production Scenario:

Lets assume we are having the 10 pods running inside the node.

Each requests: 100m → total = 1 CPU
Each limit: 500m → potential = 5 CPU

👉 Node has only 2 CPUs.

Now all pods spike:

💥 CPU contention
💥 Throttling everywhere
💥 Latency spikes

🎯 Practical Interpretation

## What Kubernetes guarantees:

You will NOT be starved below 100m

## What Linux allows:

You MAY go up to 500m (if available)

## What actually happens:

Your performance = cluster load dependent.


===================================================================================================================
                          Example to Understand the behaviour of CPU Allocation to Pods
===================================================================================================================

## Deployment with requests: 100m and limits: 500m

apiVersion: apps/v1
kind: Deployment
metadata:
  name: cpu-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: cpu-demo
  template:
    metadata:
      labels:
        app: cpu-demo
    spec:
      containers:
      - name: cpu-demo-container
        image: nginx
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "500m"

🧩 Step 1: Scheduling

Kubernetes scheduler sees: Each pod needs only 100m CPU

👉 So on a node with 2 CPUs: It can schedule ~20 pods (2000m / 100m).

But actual max usage could be: 20 pods × 500m = 10 CPU demand

👉 This is intentional overcommit.

⚙️ Step 2: Runtime Behavior

Handled by 👉 Completely Fair Scheduler

🟢 Scenario A: Node is idle

- Your pod can use up to 500m
- No throttling

kubectl top pod

cpu-demo-xxx   300m
cpu-demo-yyy   450m

✔ Fast response
✔ Smooth performance

🟡 Scenario B: Moderate load

Now more pods are active.

You are guaranteed: 100m
You may get: 200–400m

👉 CPU fluctuates

🔴 Scenario C: High load (important)

All pods try to use CPU heavily. 

- Kernel enforces fairness
- You drop close to 100m effective CPU

👉 And if trying >500m → throttled

===================================================================================================================

🧪 Let’s Simulate Real Load (Practical Test)

Update container to generate CPU load:

containers:
- name: cpu-demo-container
  image: busybox
  command: ["sh", "-c", "while true; do :; done"]
  resources:
    requests:
      cpu: "100m"
    limits:
      cpu: "500m"

📊 Observe Behavior

1️⃣ Check CPU usage

kubectl top pod

2️⃣ Check throttling (advanced)

Inside node: cat /sys/fs/cgroup/cpu/cpu.stat

Look for: nr_throttled

3️⃣ Describe pod

kubectl describe pod <pod-name>

#### You may see: QoS Class: Burstable

🎯 Real Production Interpretation

"I need at least 10% CPU,
but I may use up to 50% if available"

Important Information :

- Scheduler trusts requests
- Kernel enforces limits
- Reality depends on contention

## Your pod behaves like a “minimum guaranteed user” with “opportunistic bursting”.