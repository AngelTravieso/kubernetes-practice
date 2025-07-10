Ejercicio 2: Una Aplicación Escalable y Accesible (Deployment y Service)
Objetivo: Desplegar una aplicación con un Deployment para la escalabilidad y un Service para hacerla accesible desde fuera del clúster. Usaremos una imagen simple de un servidor web Flask.

Preparar la Imagen (Opcional, si no quieres crearla tú mismo):

Usaremos la imagen jocatalin/flask-hello-world. Esta es una imagen de Docker muy simple que expone un "Hello World" en el puerto 5000.

Crear un Deployment:

Crea un archivo YAML llamado hello-deployment.yaml.

Define un Deployment con 3 réplicas que use la imagen jocatalin/flask-hello-world.

El contenedor debe exponer el puerto 5000.

Pistas:

apiVersion: apps/v1

kind: Deployment

Usa replicas, selector, template y ports.

Crear un Service:

En el mismo archivo hello-deployment.yaml (o en uno nuevo hello-service.yaml), define un Service de tipo NodePort.

El Service debe seleccionar los Pods con la etiqueta adecuada (la misma que usaste en el Deployment).

Debe mapear el targetPort 5000 del Pod al port del Service (por ejemplo, 80) y un nodePort (por ejemplo, 30000-32767).

Pistas:

apiVersion: v1

kind: Service

type: NodePort

selector, ports (protocol, port, targetPort, nodePort).

Desplegar y Verificar:

Aplica los archivos: kubectl apply -f hello-deployment.yaml.

Verifica el Deployment: kubectl get deployments.

Verifica los Pods: kubectl get pods.

Verifica el Service: kubectl get svc. Anota el NodePort asignado.

Acceder a la Aplicación:

Obtén la IP de Minikube: minikube ip.

Accede a la aplicación usando curl <minikube-ip>:<node-port-asignado>. Deberías ver el mensaje "Hello World!".

Escalar la Aplicación:

Escala el Deployment a 5 réplicas: kubectl scale deployment <nombre-del-deployment> --replicas=5.

Verifica que se crearon los nuevos Pods.

Limpieza:

Elimina el Deployment y el Service: kubectl delete -f hello-deployment.yaml.