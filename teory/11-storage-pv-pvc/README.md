# Almacenamiento Persistente en Kubernetes: PV y PVC

## Resumen
El almacenamiento persistente en Kubernetes es fundamental para preservar datos críticos en escenarios de producción y desarrollo. Cuando trabajamos con aplicaciones que requieren conservar información, es imprescindible implementar volúmenes persistentes correctamente configurados. Los conceptos de `PV` (Persistent Volume) y `PVC` (Persistent Volume Claim) son esenciales para garantizar que nuestras aplicaciones mantengan la integridad de los datos incluso cuando los pods se reinician o se eliminan.

## ¿Por qué es crucial implementar almacenamiento persistente en Kubernetes?
Cuando ejecutamos aplicaciones en Kubernetes sin configurar adecuadamente el almacenamiento persistente mediante `PV` y `PVC`, nos arriesgamos a perder todos nuestros datos. Esto es especialmente crítico en entornos de producción donde la pérdida de información puede tener consecuencias devastadoras.

Si bien es cierto que en entornos productivos no se recomienda gestionar bases de datos directamente en Kubernetes (prefiriendo servicios gestionados como AWS RDS), en entornos de desarrollo implementar bases de datos dentro del clúster puede generar ahorros significativos. Esto nos evita mantener servicios externos como RDS, RabbitMQ u otros servicios de almacenamiento que generan costos constantes, especialmente cuando no se utilizan continuamente durante el desarrollo.

## Diferencias entre aplicaciones stateless y stateful
Antes de profundizar en `PV` y `PVC`, es importante entender dos conceptos fundamentales en el diseño de aplicaciones:

* **Stateless (sin estado):** Cada petición es independiente y contiene toda la información necesaria para ser procesada. No requiere acceder a datos almacenados previamente para resolver la solicitud. Un ejemplo típico es una API REST donde cada petición incluye en el body todos los datos necesarios para generar una respuesta.

* **Stateful (con estado):** Estas aplicaciones requieren acceder a datos persistentes para procesar las solicitudes. Cuando se recibe una petición con información limitada, el backend debe consultar una base de datos u otro sistema de almacenamiento para obtener información adicional y generar una respuesta adecuada.

La principal diferencia entre ambos enfoques radica en la necesidad de acceder a datos persistentes, y es aquí donde los conceptos de `PV` y `PVC` cobran vital importancia en el contexto de Kubernetes.

## ¿Cómo funcionan los PV y PVC en Kubernetes?
Podemos entender los `PV` y `PVC` a través de una metáfora sencilla: un almacén de datos y una llave para acceder a él.

> **Persistent Volume (PV):** Representa el almacén de datos físico, el volumen donde se encuentra nuestra información.
>
> **Persistent Volume Claim (PVC):** Actúa como la llave que permite a los pods acceder al almacén de datos. Es una solicitud de almacenamiento que realiza un pod.

Esta arquitectura ofrece una capa de abstracción que permite a los desarrolladores no preocuparse por los detalles específicos del almacenamiento subyacente, centrándose únicamente en solicitar el espacio que necesitan mediante los `PVC`.

### Tipos de almacenamiento (Storage Classes)
Kubernetes ofrece diferentes clases de almacenamiento:

* **Storage Class Manual:** Utilizada principalmente en entornos locales, haciendo referencia a directorios dentro de la máquina local.
* **Servicios de AWS:** Como EFS (Elastic File System) o EBS (Elastic Block Storage), que proporcionan mayor eficiencia, alta disponibilidad y facilitan los respaldos y migraciones de datos.

La relación entre estos componentes sigue un flujo específico: un pod hace referencia a un `PVC`, y este `PVC` se vincula a un `PV` específico, todo definido en los archivos YAML correspondientes.

## ¿Cómo implementar PV y PVC en un entorno de desarrollo?
Vamos a recorrer un ejemplo práctico utilizando MiniKube como entorno local:

### Paso 1: Preparar los datos en el host
Primero, necesitamos crear un archivo en el host que será expuesto a nuestro pod:
```bash
sudo su
cd /mnt/data
echo "<h1>Hello from volum</h1>" > index.html
cat index.html
exit
```

### Paso 2: Definir el Persistent Volume (PV)
Creamos un archivo YAML para definir nuestro `PV`:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: myPV
  labels:
    app: nginx-storage
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  hostPath:
    path: /mnt/data
```
En esta definición:
- Especificamos una capacidad de `1GB`.
- Configuramos el modo de acceso como `ReadWriteOnce` (solo un pod puede montarlo).
- Definimos la clase de almacenamiento como `manual` (para entornos locales).
- Apuntamos al directorio `/mnt/data` donde creamos nuestro archivo HTML.

### Paso 3: Definir el Persistent Volume Claim (PVC)
Ahora creamos el `PVC` que reclamará el `PV` anterior:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myPVC
spec:
  selector:
    matchLabels:
      app: nginx-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```
Observa que el `PVC` utiliza un selector con `matchLabels` para vincularse específicamente al `PV` que creamos, coincidiendo con la etiqueta `"app: nginx-storage"`.

### Paso 4: Crear un pod que utilice el PVC
Finalmente, definimos un pod que utilice nuestro `PVC`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myPod
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: myvolume
  volumes:
    - name: myvolume
      persistentVolumeClaim:
        claimName: myPVC
```
Este pod:
- Utiliza la imagen de `nginx`.
- Monta el volumen en la ruta por defecto donde nginx busca los archivos HTML.
- Hace referencia al `PVC` "myPVC" que creamos anteriormente.

### Paso 5: Aplicar las configuraciones
Aplicamos las configuraciones en orden:
```bash
kubectl apply -f pv-pvc.yaml
kubectl apply -f pod.yaml
```

### Paso 6: Verificar la configuración
Comprobamos que tanto el `PV` como el `PVC` están correctamente establecidos:
```bash
kubectl get pv,pvc
kubectl describe pod myPod
```

### Paso 7: Validar el funcionamiento
Para verificar que todo funciona correctamente:
```bash
kubectl exec myPod -- ls -la /usr/share/nginx/html
kubectl port-forward myPod 8080:80
```
Ahora podemos acceder a `localhost:8080` en nuestro navegador y deberíamos ver el mensaje "Hello from volum".

## ¿Cuáles son los beneficios de utilizar PV y PVC?
La implementación de almacenamiento persistente mediante `PV` y `PVC` ofrece múltiples ventajas:
- **Persistencia de datos:** La información sobrevive incluso si los pods se reinician o eliminan.
- **Abstracción del almacenamiento:** Los desarrolladores no necesitan conocer los detalles de la implementación del almacenamiento.
- **Flexibilidad:** Permite cambiar el almacenamiento subyacente sin modificar la aplicación.
- **Ahorro de costos:** Especialmente en entornos de desarrollo, evita tener que mantener servicios externos costosos.

> En la era de la inteligencia artificial, la información se ha convertido en oro. Implementar correctamente `PV` y `PVC` garantiza que este valioso recurso esté protegido y disponible para tus aplicaciones en Kubernetes, evitando la pérdida de datos críticos para tu organización o tus clientes.


# 11-storage-pv-pvc

## First, create the directory for hostPath, remember to run this on the minikube node using sudo
```bash
minikube ssh
echo '<h1>Hello from Volume!</h1>' > /mnt/data/index.html
```

## Apply the PV and PVC
```bash
kubectl apply -f pv-pvc.yaml
```

## Create the Pod
```bash
kubectl apply -f pod.yaml
```

## Verify the setup
```bash
# Check PV and PVC status
kubectl get pv,pvc

# Check pod status
kubectl get pod my-pod

# Verify the mounted content
kubectl exec my-pod -- ls -la /usr/share/nginx/html

# Test the nginx server

kubectl port-forward my-pod 8080:80
```

Then you can visit http://localhost:8080 in your browser to see the content.

## Cleanup
```bash
kubectl delete pod my-pod
kubectl delete -f pv-pvc.yaml
```

que otros modos de acceso hay?
diferencia entre pv y pvc