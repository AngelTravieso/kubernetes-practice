# 17-intro-cloud-k8s


## Google Kubernetes Engine (GKE)

# Prerequisitos y Login en tu cuenta de Google Cloud

```bash
./google-cloud-sdk/install.sh
gcloud auth login
```

# Activar API de Kubernetes Engine y Compute Engine

```bash
gcloud services enable container.googleapis.com
gcloud services enable compute.googleapis.com
```

# Crear un cluster en GKE

```bash
gcloud container clusters create k8s-course-demo --zone us-central1-a --num-nodes 2
```

# Install plugin kubectl

```bash
gcloud components install gke-gcloud-auth-plugin
```

# (Optional) Configurar contexto de k8s en k8bectl

```bash
gcloud container clústers get-credentials k8s-course-demo --zone us-central1-a
```

# Clean up Cluster

```bash
gcloud container clusters delete k8s-course-demo --zone us-central1-a
```

--------------------------------


## Azure Kubernetes Service (AKS)

# Recuerda crear tu cuenta segun lo aprendido en el curso de Azure

https://platzi.com/home/clases/11884-az-900/74238-creando-tu-cuenta-de-azure/

# Prerequisitos y Login en tu cuenta de Azure

```bash
az --version
az aks install-cli
az login

```

# (Optional) Configuración Provider Register

```bash
az provider list --output table
az provider register --namespace microsoft.insights
az provider register --namespace Microsoft.ContainerService
```

# Crear un cluster en AKS

```bash
az group create --name k8scourse-aks-demo --location eastus

az aks create --resource-group k8scourse-aks-demo --name k8sCourseAKSDemo --node-count 3 --enable-addons monitoring --generate-ssh-keys --node-vm-size Standard_D2s_v3
```

# Extraer las credenciales del cluster

```bash
az aks get-credentials --resource-group k8scourse-aks-demo --name k8sCourseAKSDemo
```

# Validar la conexión al cluster

```bash
kubectl get nodes
```

# Crear un namespace

```bash
kubectl create namespace platzi
```

# Clean up Cluster

```bash
az aks delete --resource-group k8scourse-aks-demo --name k8sCourseDemoAKS --yes --no-wait
```