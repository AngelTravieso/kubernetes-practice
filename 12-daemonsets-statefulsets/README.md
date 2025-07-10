# 12-daemonsets-statefulsets

# Create a DaemonSet

```bash
kubectl apply -f daemonset.yaml
```

# Verify the setup
```bash
kubectl get ds
kubectl get pods -l app=nginx-app
```

# Create a StatefulSet and its PersistentVolume

```bash
# Make sure your PV and PVC exist first
kubectl apply -f pv-pvc.yaml
# Then apply the StatefulSet
kubectl apply -f statefulset.yaml
```

# Verify the setup
```bash
kubectl get sts
kubectl get pods -l app=nginx-app
kubectl get pvc my-pvc
```

# Delete the StatefulSet
```bash
kubectl delete sts nginx-sts
```