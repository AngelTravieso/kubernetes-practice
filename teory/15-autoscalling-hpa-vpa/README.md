# 15-autoscaling-hpa-vpa

## Deploy APP

```bash
kubectl apply -f app.yaml
```

## Deploy HPA

```bash
kubectl apply -f hpa.yaml
```

### Enable Required Metric-Server Addon (execute if not enabled)
```bash
# List available addons
minikube addons list

# Enable metrics-server for resource monitoring
minikube addons enable metrics-server
```

# Generate load

```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.001; do wget -q -O- http://my-app-service; done"
```

# Monitor HPA y Kubernetes dashboard

```bash
kubectl get hpa
minikube dashboard
watch -n 0.5 kubectl top pods
```

## Deploy VPA

# Enlaces para instalar los CRD de VPA

<!-- https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/installation.md -->

```bash (Optional: In case the secrets gets not properly created)
bash ./pkg/admission-controller/gencerts.sh
```

# Aplicar VPA

```bash
kubectl apply -f my-vpa.yaml
kubectl describe vpa my-vpa
```

--------------------------------------------------------------------------------------------------

# Scaling Demo

## App deployment

### Deploy Backend Services
```bash
# Apply backend deployments and configurations
kubectl apply -f k8s/backend/

# Verify Horizontal Pod Autoscaler (HPA) configuration
kubectl get hpa
```

### Deploy Frontend Services
```bash
# Apply frontend deployments and configurations
kubectl apply -f k8s/frontend/
```

### Enable Required Metric-Server Addon
```bash
# List available addons
minikube addons list

# Enable metrics-server for resource monitoring
minikube addons enable metrics-server
```

### Monitor Resource Usage
```bash
# View resource usage of pods (CPU and Memory)
kubectl top pods
```

## Notes
- The metrics-server is required for HPA to function properly
- Use `kubectl get hpa` to monitor scaling status
- Monitor pod resource usage with `kubectl top pods`