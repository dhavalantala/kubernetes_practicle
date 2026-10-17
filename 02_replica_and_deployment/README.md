# ReplicationController, ReplicaSet and Deployment

Notes on how Kubernetes keeps a set number of pods running.

## ReplicationController (RC)

The old way to keep a fixed number of pod copies (replicas) running. If a pod dies, the RC creates a new one.

**rc.yaml**

```yaml
# ReplicationController
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
  labels:
    env: demo
spec:
  replicas: 20
  template:
    metadata:
      name: nginx
      labels:
        env: demo
    spec:
      containers:
        - image: nginx
          name: nginx-rc
```

```bash
kubectl apply -f rc.yaml
kubectl get rc
kubectl get pods
kubectl delete rc nginx-rc
```

## ReplicaSet (RS)

The newer version of RC. The difference is the `selector`, which tells the ReplicaSet which pods it manages (matched by labels). It uses `apps/v1`.

**rs.yaml**

```yaml
# ReplicaSet
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    env: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      env: demo
  template:
    metadata:
      name: nginx
      labels:
        env: demo
    spec:
      containers:
        - image: nginx
          name: nginx
```

```bash
kubectl apply -f rs.yaml
kubectl get rs
kubectl get pods
kubectl delete rs nginx-rs
```

The `labels` in `template` must match `selector.matchLabels`, otherwise the ReplicaSet will not be created.

## Deployment

Manages ReplicaSets for you. It is what we use in real work, because it also supports updates and rollbacks.

**deployment.yaml**

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    env: demo
spec:
  replicas: 3
  selector:
    matchLabels:
      env: demo
  template:
    metadata:
      name: nginx
      labels:
        env: demo
    spec:
      containers:
        - image: nginx
          name: nginx
```

```bash
kubectl apply -f deployment.yaml
kubectl get deployment
kubectl get rs
kubectl get pods
kubectl delete deployment nginx-deployment
```

Creating a Deployment also creates a ReplicaSet, and the ReplicaSet creates the pods.

```
Deployment -> ReplicaSet -> Pods
```

## Scale replicas

```bash
kubectl scale rs nginx-rs --replicas=5
kubectl scale deployment nginx-deployment --replicas=5
```

## Compare

| | ReplicationController | ReplicaSet | Deployment |
|---|---|---|---|
| apiVersion | `v1` | `apps/v1` | `apps/v1` |
| Keeps replicas running | Yes | Yes | Yes |
| `selector` | Not needed | Required | Required |
| Rolling update / rollback | No | No | Yes |
| Use today | No (old) | Rarely, directly | Yes |

## Commands used

| Command | What it does |
|---------|--------------|
| `kubectl get rc` | List ReplicationControllers |
| `kubectl get rs` | List ReplicaSets |
| `kubectl get deployment` | List Deployments |
| `kubectl scale` | Change the number of replicas |
| `kubectl delete rc/rs/deployment <name>` | Delete it and its pods |
