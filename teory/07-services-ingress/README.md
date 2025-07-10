# 07-services-ingress

Este repositorio contiene ejemplos y prácticas sobre cómo usar **Services** e **Ingress** en Kubernetes para exponer aplicaciones en un clúster. 

## Contenido

- [Descripción](#descripción)
- [Conceptos Clave](#conceptos-clave)
  - [Deployment](#deployment)
  - [Service](#service)
  - [Ingress](#ingress)
- [Configuración del Ingress Controller](#configuración-del-ingress-controller)
- [Despliegue de una Aplicación](#despliegue-de-una-aplicación)
- [Exposición del Deployment](#exposición-del-deployment)
- [Acceso al Servicio](#acceso-al-servicio)
- [Creación de Ingress](#creación-de-ingress)
- [Verificación](#verificación)

## Descripción

Un **Ingress** en Kubernetes es un objeto que permite exponer aplicaciones al exterior de un clúster, gestionando el acceso a través de una URL. Facilita el balanceo de carga, la terminación SSL y la administración de rutas para diferentes servicios.

## Conceptos Clave

### Deployment
Un **Deployment** en Kubernetes define la configuración deseada de un conjunto de Pods, incluyendo el número de réplicas y la imagen del contenedor. Se encarga de:

- Asegurar que el número deseado de Pods esté en ejecución.
- Facilitar actualizaciones mediante el método **rolling update**.
- Proporcionar capacidad de retroceso en caso de problema con una nueva versión.

### Service
Un **Service** es un objeto que expone uno o más Pods como un conjunto de servicios. Permite la comunicación entre las distintas partes de la aplicación, ofreciendo diferentes tipos:

- **ClusterIP**: Acceso interno solo dentro del clúster.
- **NodePort**: Acceso a través de un puerto específico en cada nodo.
- **LoadBalancer**: Provisión de una dirección IP externa para el balanceo de carga.
- **ExternalName**: Redireccionamiento a un servicio externo.

### Ingress
Un **Ingress** gestiona el acceso externo a los servicios en un clúster a través de HTTP. Proporciona:

- Rutas basadas en la URL para dirigir el tráfico a diferentes servicios.
- Terminación TLS/SSL para comunicaciones seguras.
- Configuraciones de balanceo de carga.

## Configuración del Ingress Controller

Habilita el **Ingress controller** en tu clúster de Minikube:

```bash  
minikube addons enable ingress  