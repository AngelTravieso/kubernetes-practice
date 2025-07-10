# 12-daemonsets-statefulsets

# Controladores de Carga de Trabajo en Kubernetes: DaemonSet y StatefulSet

Kubernetes ofrece varios controladores para gestionar el ciclo de vida de los **Pods**, cada uno diseñado para diferentes tipos de aplicaciones y necesidades. En este `README`, exploraremos dos controladores clave: **DaemonSet** y **StatefulSet**, sus diferencias, casos de uso y la importancia de los **Headless Services** en el contexto de los StatefulSets.

---

## 1. DaemonSet

Un **DaemonSet** garantiza que una copia específica de un Pod se ejecute en **todos (o algunos)** nodos de un clúster. Son ideales para servicios a nivel de nodo que necesitan estar siempre presentes.

### ¿Qué hace?

* **Un Pod por Nodo:** Asegura que una instancia del Pod esté en cada nodo calificado.
* **Implementación Automática:** Cuando se añade un nuevo nodo al clúster, un Pod del DaemonSet se programa automáticamente en él.
* **Recolección de Basura:** Cuando un nodo se elimina, el Pod del DaemonSet que se ejecutaba en él también se termina y elimina.
* **Sin Escalamiento Horizontal:** No se enfoca en el número de réplicas de la aplicación, sino en su presencia por nodo.
* **Sin Identidad Estable:** Los Pods no tienen una identidad estable ni un orden garantizado de creación/terminación.

### Casos de Uso Típicos:

* **Recopiladores de logs:** Agentes como Fluentd o Filebeat que recogen logs de cada nodo.
* **Agentes de monitoreo:** Herramientas como Prometheus Node Exporter o Datadog Agent que recolectan métricas del nodo.
* **Proxies de red:** Componentes como `kube-proxy` o Calico que manejan el enrutamiento de red.
* **Sistemas de almacenamiento en clúster:** Demonios que se ejecutan en cada nodo para proveer almacenamiento distribuido (ej. Ceph, GlusterFS).

### Ejemplo (Conceptual):

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: mi-agente-de-logs
spec:
  selector:
    matchLabels:
      app: agente-logs
  template:
    metadata:
      labels:
        app: agente-logs
    spec:
      containers:
      - name: agente
        image: tu-imagen-agente-logs:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
        - name: varlog
          hostPath:
            path: /var/log

## StatefulSet

Un **StatefulSet** se utiliza para gestionar **aplicaciones con estado**, como bases de datos o sistemas de colas. Proporciona garantías sobre la identidad y el orden de los Pods.

### ¿Qué hace?

* **Identidad Estable y Única por Pod:** Cada Pod obtiene un nombre ordinal predecible (ej., `app-0`, `app-1`), una identidad de red estable y un volumen persistente estable. Esta identidad se mantiene incluso si el Pod se reinicia o se reprograma.
* **Despliegue y Escalamiento Ordenado:**
    * **Creación:** Los Pods se crean en orden secuencial (del `0` al `N-1`), esperando que el anterior esté listo.
    * **Terminación:** Los Pods se eliminan en orden inverso (del `N-1` al `0`).
* **Almacenamiento Persistente Estable:** Cada Pod está vinculado a su propio PersistentVolume (PV) a través de un PersistentVolumeClaim (PVC) generado por `volumeClaimTemplates`.

### Diferencia Clave con un Deployment (con volúmenes):

Aunque un Deployment puede usar PersistentVolumes, sus Pods son fungibles y carecen de una identidad estable. Un StatefulSet va más allá:

* **Identidad de Red:** Los Pods de un StatefulSet tienen nombres de host únicos y predecibles (ej., `app-0.servicio-headless`), lo que permite que otras aplicaciones o los propios Pods se comuniquen con instancias específicas.
* **Mapeo de Almacenamiento:** Un StatefulSet garantiza que un Pod con una identidad específica (ej., `app-0`) siempre se monte al mismo volumen persistente, independientemente de dónde se reprograme.
* **Orden:** El orden garantizado de creación y terminación es crucial para la coherencia de aplicaciones distribuidas (ej., un nodo primario de una BD debe estar listo antes que los secundarios puedan unirse).

### Casos de Uso Típicos:

* **Bases de datos distribuidas:** MongoDB, PostgreSQL, Cassandra, MySQL, etc., donde cada instancia necesita su propio almacenamiento y una identidad estable para la replicación.
* **Sistemas de colas de mensajes:** Kafka, RabbitMQ, que requieren mantener el estado de los mensajes y la identidad de los brokers.
* **Aplicaciones que requieren un orden de inicio/apagado estricto.**

### Ejemplo (Conceptual):

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mi-db
spec:
  serviceName: "mi-db-headless" # Se vincula a un Headless Service para identidad de red estable
  replicas: 3
  selector:
    matchLabels:
      app: mi-db
  template:
    metadata:
      labels:
        app: mi-db
    spec:
      containers:
      - name: db
        image: tu-imagen-db:latest
        ports:
        - containerPort: 3306 # Ejemplo para MySQL
          name: db-port
        volumeMounts:
        - name: db-data
          mountPath: /var/lib/data
  volumeClaimTemplates: # Crea automáticamente un PVC único para cada Pod
  - metadata:
      name: db-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard" # Asegúrate de que este StorageClass esté disponible en tu clúster
      resources:
        requests:
          storage: 5Gi

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