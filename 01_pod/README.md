# Pods

Notes from my first practice with Kubernetes pods.

## What is a Pod?

A pod is the smallest unit in Kubernetes. It wraps one or more containers.

## Create a pod from the command line

```bash
kubectl run nginx-pod --image=nginx:latest
kubectl get pods
```

```
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          8s
```

`kubectl run` is good for quick tests. For real work, create pods from a YAML file with `kubectl create` or `kubectl apply`.

## Need help writing a pod?

```bash
kubectl explain pod
kubectl explain pod.spec
```

Or use the official documentation: https://kubernetes.io/docs/concepts/workloads/pods/

## Create a pod from YAML

**pod.yaml**

```yaml
# This is a sample pod yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    env: demo
    type: frontend
spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```

```bash
kubectl create -f pod.yaml     # create the pod
kubectl apply -f pod.yaml      # create or update the pod
```

## Delete a pod

```bash
kubectl delete pod nginx-pod
```

A pod's name cannot be changed on a running pod. To use a new name, change `name` in `metadata`, delete the old pod, and create it again.

## Edit a pod

```bash
kubectl edit pod nginx-pod
```

```
pod/nginx-pod edited
```

## Go inside a pod

```bash
kubectl exec -it nginx-pod -- sh
```

## Dry run

Shows what would be created without creating anything.

```bash
kubectl run nginx --image=nginx --dry-run=client
```

```
pod/nginx created (dry run)
```

Generate YAML or JSON from the command line:

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod_new.yaml
kubectl run nginx --image=nginx --dry-run=client -o json > pod_new.json
```

## Pod information

**Describe a pod**

```bash
kubectl describe pod nginx-pod
```

**Show labels**

```bash
kubectl get pod --show-labels
```

```
NAME        READY   STATUS    RESTARTS        AGE     LABELS
nginx-pod   1/1     Running   1 (5m43s ago)   7m51s   env=demo,type=frontend
```

**Extended information (IP, node)**

```bash
kubectl get pod -o wide
```

```
NAME        READY   STATUS    RESTARTS       AGE     IP           NODE                         NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   1 (6m6s ago)   8m14s   10.244.0.8   cka-cluster1-control-plane   <none>           <none>
```

**Node information**

```bash
kubectl get nodes -o wide
```

```
NAME                         STATUS   ROLES           AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                       KERNEL-VERSION            CONTAINER-RUNTIME
cka-cluster1-control-plane   Ready    control-plane   146m   v1.36.4   172.18.0.2    <none>        Debian GNU/Linux 13 (trixie)   7.0.14-linuxkit (arm64)   containerd://2.3.4
```

## Commands used

| Command | What it does |
|---------|--------------|
| `kubectl run` | Create a pod from the command line |
| `kubectl create -f` | Create from a YAML file |
| `kubectl apply -f` | Create or update from a YAML file |
| `kubectl get pods` | List pods |
| `kubectl describe pod` | Detailed info about a pod |
| `kubectl edit pod` | Edit a pod |
| `kubectl exec -it` | Open a shell inside a pod |
| `kubectl delete pod` | Delete a pod |
| `kubectl explain pod` | Show the pod fields and help |
