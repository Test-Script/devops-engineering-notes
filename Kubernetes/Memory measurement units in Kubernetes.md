===================================================================================================================
                                            🚀 Practical Memory Test
===================================================================================================================

🧪 Step 1: Deployment YAML

apiVersion: apps/v1
kind: Deployment
metadata:
  name: memory-test
spec:
  replicas: 2
  selector:
    matchLabels:
      app: memory-test
apiVersion: apps/v1
kind: Deployment
metadata:
  name: memory-test
spec:
  replicas: 2
  selector:
    matchLabels:
      app: memory-test
  template:
    metadata:
      labels:
        app: memory-test
    spec:
      containers:
      - name: memory-container
        image: polinux/stress
        command: ["stress"]
        args:
          - "--vm"
          - "1"
          - "--vm-bytes"
          - "300M"
          - "--vm-hang"
          - "1"
        resources:
          requests:
            memory: "128Mi"
          limits:
            memory: "256Mi"

## kubectl apply -f memory-test.yaml

🔍 What we will see now

Step 1: Pods start

kubectl get pods

Step 2: Then memory exceeds limit

After a few seconds: CrashLoopBackOff

Step 3: Confirm OOM

kubectl describe pod <pod-name>

Now we should see:

Reason: OOMKilled
Exit Code: 137

👉 This confirms behavior of
👉 Linux OOM Killer

🎯 What This Demonstrates (Your Learning Goal)

Now we will observe:

Container starts → allocates memory → exceeds 256Mi → killed instantly

No gradual slowdown like CPU.