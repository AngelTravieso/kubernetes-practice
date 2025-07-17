- Para crear la imagen docker en la VM de minikube:
eval $(minikube -p minikube docker-env)

- Para acceder al service desde el navegador:
minikube service webapp-deployment

falta que funcione con el NodePort