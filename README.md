# Repositorio de Configuración de Infraestructura

Este repositorio contiene el código para la gestión de la configuración de servidores y los manifiestos de despliegue en Kubernetes.

El propósito es estandarizar y automatizar la configuración del entorno de ejecución de la aplicación de la materia.

---

## Stack Tecnológico

*   **Gestión de Configuración**: Ansible
*   **Orquestación de Contenedores**: Kubernetes (k3s)

---

## Estructura del Repositorio

*   `ansible/`: Contiene los playbooks y roles de Ansible utilizados para configurar los servidores base. Esto incluye la instalación de software y la preparación de los nodos de Kubernetes.
    *   `roles/`: Tareas de configuración reutilizables.
    *   `inventory/`: Listas de servidores agrupados por entorno.
    *   `setup-server.yml`: El playbook principal que aplica la configuración.

*   `kubernetes/`: Contiene los manifiestos YAML de Kubernetes para el despliegue de las aplicaciones. Los manifiestos están separados por entorno (`staging`, `production`).

---

## Procedimientos Operativos

### 1. Configuración de un Nuevo Servidor

Para configurar un servidor previamente aprovisionado, se deben seguir los siguientes pasos:

1.  Añadir la dirección IP del servidor al archivo de inventario correspondiente (`ansible/inventory/`).
2.  Ejecutar el playbook de Ansible.

```bash
# Ejemplo de ejecución para el entorno de staging
ansible-playbook ansible/setup-server.yml -i ansible/inventory/staging.hosts --ask-become-pass
```
Este comando instala las dependencias necesarias, como Docker y k3s, basándose en los roles definidos.

### 2. Despliegue de Aplicaciones en Kubernetes

El despliegue o actualización de una aplicación se realiza aplicando los manifiestos YAML del entorno correspondiente.

```bash
# Ejemplo de despliegue en el entorno de STAGING
kubectl apply -f kubernetes/staging/

# Ejemplo de despliegue en el entorno de PRODUCCIÓN
kubectl apply -f kubernetes/production/
```
**Nota:** El comando `kubectl apply -f <directorio>/` aplicará todos los archivos `.yaml` dentro de ese directorio.

La actualización de las etiquetas de las imágenes de contenedor (`image:tag`) en los archivos `deployment.yaml` es una tarea que debe ser gestionada por el pipeline de CI/CD antes de ejecutar el comando `kubectl apply`.

---

