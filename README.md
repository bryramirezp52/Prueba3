# Despliegue y Configuración de un Clúster de Kubernetes en AWS

## Informe Tecnologías de Virtualización - Prueba Parcial 3

**Integrante:** Bryan Ramírez Palacios
**Sección:** DIY7111_004V
**Fecha:** 20 de Junio de 2025

---

## 1. Introducción

Este documento detalla el despliegue, configuración y administración de un clúster de Kubernetes. Se aborda la preparación de la infraestructura, la implementación de una aplicación web multicontenedor (WordPress con MySQL) y se responden preguntas teóricas clave sobre Kubernetes.

## 2. Despliegue de Infraestructura Práctica

Se desplegó un clúster de dos nodos (maestro y trabajador) desde cero utilizando instancias EC2 de AWS con Ubuntu Server 22.04 LTS. Cada nodo se configuró con 4GB de RAM y 2vCPUs. Se establecieron IP elásticas y un Security Group permisivo para facilitar el desarrollo.

Los pasos clave para la configuración de los nodos incluyeron:
* **Preparación del Sistema Operativo:** Actualización de paquetes, desactivación de swap, carga de módulos del kernel (`overlay`, `br_netfilter`) y ajuste de parámetros de red (`sysctl`) para compatibilidad con Kubernetes.
* **Instalación y Configuración de Containerd:** Se instaló y configuró `containerd` como el runtime de contenedores, asegurando que usara `systemd` para la gestión de cgroups.
* **Inicialización del Nodo Maestro:** Se utilizó `kubeadm init` para iniciar el clúster, especificando el `pod-network-cidr` (172.24.0.0/16) y el `control-plane-endpoint`. Se configuró `kubectl` para interactuar con el clúster.
* **Instalación y Configuración del CNI Calico:** Se descargaron y aplicaron los manifiestos de Calico, ajustando el CIDR para la red de pods, asegurando que los pods del sistema estuvieran en estado `Running` y el nodo maestro en `Ready`.
* **Unión del Nodo Trabajador:** El nodo trabajador se unió exitosamente al clúster utilizando el comando `kubeadm join` generado en el maestro.

## 3. Despliegue de Aplicación Práctica

Se desplegó una aplicación WordPress con una base de datos MySQL en el clúster de Kubernetes. Esto implicó la creación de directorios para datos persistentes en el nodo trabajador y el uso de manifiestos YAML para los siguientes recursos:
* **PersistentVolume (PV) y PersistentVolumeClaim (PVC):** Para garantizar el almacenamiento persistente de la base de datos MySQL.
* **Secret:** Para almacenar de forma segura la contraseña de MySQL.
* **Deployment para MySQL:** Gestionando el contenedor de la base de datos.
* **Service para MySQL:** Exponiendo la base de datos dentro del clúster.
* **Deployment para WordPress:** Gestionando el contenedor de la aplicación.
* **Service tipo NodePort para WordPress:** Exponiendo la aplicación al exterior.

La aplicación se desplegó en un `namespace` dedicado (`wordpress-ns`).

## 4. Desafíos y Lecciones Aprendidas

El proceso presentó desafíos significativos que enriquecieron la experiencia de depuración:

* **Repositorios Obsoletos de Kubernetes (404 Not Found):** Durante la instalación de los componentes de Kubernetes, los repositorios de paquetes originales estaban desactualizados. La solución fue **investigar y adoptar el nuevo repositorio oficial (pkgs.k8s.io)**, lo que subrayó la necesidad de validar constantemente las fuentes de documentación en el dinámico ecosistema de código abierto.
* **Incompatibilidad en la Instalación de Calico:** Una incompatibilidad entre manifiestos de Calico generó un estado inconsistente en el clúster. La resolución requirió una **limpieza manual de recursos fallidos** y la adopción de un método de instalación alternativo y más robusto, utilizando el manifiesto consolidado oficial del proyecto.
* **"Error establishing a database connection" en WordPress:** El reto más significativo fue un error persistente al conectar WordPress a MySQL. Un diagnóstico sistemático reveló que la causa raíz era un **estado persistente corrupto en el volumen de la base de datos MySQL**. Este volumen, inicializado incorrectamente en un despliegue previo, impedía que MySQL creara el usuario necesario para WordPress en los reinicios. La **solución definitiva fue un reinicio completo del estado**, eliminando los recursos de Kubernetes (Pods, Services, PVC) y el contenido físico del volumen (`hostPath`) en el nodo trabajador, forzando una inicialización limpia con la configuración ya corregida.

## 5. Desarrollo Teórico (Resumen)

El informe también incluyó explicaciones sobre:
* **kubelet:** Agente en cada nodo que asegura que los Pods estén corriendo y saludables.
* **kubeadm:** Herramienta de bootstrapping para crear y gestionar el ciclo de vida de un clúster.
* **kubectl:** Interfaz de línea de comandos para interactuar y gestionar los recursos del clúster.
* **`apt-mark hold`:** Importancia de bloquear las versiones de los paquetes de Kubernetes en producción para evitar actualizaciones no planificadas que puedan causar inestabilidad debido a la política de compatibilidad de versiones (version skew).
* **Nodo `NotReady` sin CNI:** Un nodo aparece como `NotReady` después de `kubeadm init` porque carece de un Container Network Interface (CNI) como Calico. Sin el CNI, los Pods no pueden obtener una IP ni comunicarse, impidiendo que el nodo se considere operativo.

## 6. Conclusiones Finales

Esta evaluación no solo demostró la capacidad de seguir procedimientos de instalación, sino, y más importante, la **habilidad para diagnosticar, investigar y resolver problemas complejos** de estado, configuración y dependencias en un entorno de orquestación. El resultado es un clúster Kubernetes plenamente funcional, sirviendo una aplicación web persistente y accesible, lo que valida la adquisición de competencias fundamentales en la administración de sistemas orquestados.

---
