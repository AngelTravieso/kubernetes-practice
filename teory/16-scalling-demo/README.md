# 16-scaling-demo

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