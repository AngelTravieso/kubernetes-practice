# 21-others-use-cases

## Use Cases

### 1. Deploy a the DeepSeek LLM in local K8s

```bash
cd deepseek-local-k8s

kubectl apply -f server-deployment.yaml

kubectl apply -f ui-deployment.yaml

```

# Validate the LLM is executing properly

```bash
kubectl port-forward svc/deepseek-server -n deepseek 35000:11434
```

```bash
curl -X POST "http://localhost:35000/v1/chat/completions" -H "Content-Type: application/json" --data '{"model": "deepseek-r1:1.5b", "messages": [{"role": "user", "content": "What is kubernetes?"}]}'
```

# Configure the UI to use the local LLM

```bash
minikube tunnel

# Edit de /etc/hosts file
sudo vi /etc/hosts

# Add the following line to the file
127.0.0.1       running.deepseek.local
```