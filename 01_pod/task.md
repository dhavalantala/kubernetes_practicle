# Task 2: Create a new pod from YAML

* Create the YAML from the nginx pod created in task 1
* Update the pod name in the YAML
* Use that YAML to create a new pod with the name nginx-new

## Step 1: Generate the YAML from the existing pod

```bash
kubectl get pod nginx-pod -o yaml > nginx-new.yaml
```

This file has extra fields (status, uid, etc.). Remove them and keep only what is needed, or generate a clean file with a dry run:

```bash
kubectl run nginx-new --image=nginx:1.14.2 --dry-run=client -o yaml > nginx-new.yaml
```

## Step 2: Update the name in the YAML

**nginx-new.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-new
  labels:
    env: task
    type: frontend
spec:
  containers:
    - name: nginx-new
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

## Step 3: Create the pod

```bash
kubectl create -f nginx-new.yaml
kubectl get pods
```

```
NAME        READY   STATUS    RESTARTS   AGE
nginx-new   1/1     Running   0          10s
```

---

# Task 3: Fix the errors in a pod YAML

Apply the YAML below and fix the errors.

**redis.yaml (with error)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: test
  name: redis
spec:
  containers:
  - image: rediss
    name: redis
```

## Step 1: Apply the YAML

```bash
kubectl apply -f redis.yaml
```

```
pod/redis created
```

## Step 2: Check the pod

```bash
kubectl get pods
```

```
NAME    READY   STATUS             RESTARTS   AGE
redis   0/1     ErrImagePull       0          10s
```

After a few seconds:

```
NAME    READY   STATUS             RESTARTS   AGE
redis   0/1     ImagePullBackOff   0          30s
```

## Step 3: Find the reason

```bash
kubectl describe pod redis
```

Look at the `Events` section at the bottom:

```
Warning  Failed   Failed to pull image "rediss": pull access denied, repository does not exist or may require authorization
Warning  Failed   Error: ErrImagePull
Normal   BackOff  Back-off pulling image "rediss"
Warning  Failed   Error: ImagePullBackOff
```

## Step 4: Cause

The image name is wrong. `rediss` has an extra `s`, and no image with that name exists. The correct image is `redis`.

## Step 5: Fix

Change the image in the YAML to `redis:alpine`.

**redis.yaml (fixed)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis
  labels:
    app: test
spec:
  containers:
    - name: redis
      image: redis:alpine
      ports:
        - containerPort: 6379
```

Apply the fix:

```bash
kubectl apply -f redis.yaml
```

```
pod/redis configured
```

If apply gives an error, delete the pod and create it again:

```bash
kubectl delete pod redis
kubectl apply -f redis.yaml
```

## Step 6: Verify

```bash
kubectl get pods
```

```
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   0          15s
```

```bash
kubectl describe pod redis
kubectl logs redis
```

## Commands used while troubleshooting

| Command | What it does |
|---------|--------------|
| `kubectl apply -f redis.yaml` | Create or update the pod |
| `kubectl get pods` | Check the status |
| `kubectl describe pod redis` | See the events and the error reason |
| `kubectl logs redis` | See the container logs |
| `kubectl delete pod redis` | Delete the pod to recreate it |

## Common pod errors

| Status | Meaning |
|--------|---------|
| `ErrImagePull` | Kubernetes could not download the image |
| `ImagePullBackOff` | Kubernetes is waiting before trying to pull again |
| `CrashLoopBackOff` | The container starts and keeps crashing |
