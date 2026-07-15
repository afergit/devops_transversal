# Examen Final Transversal: Automatización y Orquestación de Plataforma Multicapa en AWS (EKS)

Este repositorio contiene el proyecto práctico final de la asignatura **Introducción a Herramientas DevOps (ISY1101)**. Consiste en la contenedorización, automatización de la integración y entrega continua (CI/CD) y la orquestación productiva de la plataforma **"Tienda de Perros"** para la empresa Innovatech Chile.

---

## 📋 Arquitectura General del Sistema

La solución está diseñada bajo una arquitectura desacoplada de tres capas ejecutándose en contenedores independientes y orquestados:

1.  **Capa de Presentación (Frontend):** Interfaz web estática optimizada servida a través de un servidor Nginx minimalista (Puerto 80).
2.  **Capa Lógica (Backend API):** Servicio REST desarrollado en Node.js con Express que provee los endpoints de negocio (Puerto 3001).
3.  **Capa de Persistencia (Base de Datos):** Motor de base de datos relacional MySQL (Puerto 3306) con inicialización automática de esquema y semillas.

---

## 🛠️ Contenedorización y Desarrollo Local

### Buenas Prácticas de Contenedores:
*   **Dockerfile Multietapa (Multi-Stage):** Implementado en el Frontend para separar la etapa de compilación de la etapa de ejecución, reduciendo el peso de la imagen final a solo ~30MB y aislando dependencias de desarrollo.
*   **Imágenes base minimalistas:** Uso de imágenes `alpine` y `slim` para endurecimiento de contenedores (reducción de vulnerabilidades y optimización de descargas).
*   **Archivos `.dockerignore`:** Configurados en cada microservicio para omitir del contexto carpetas innecesarias como `node_modules` y `.git`.
*   **Desarrollo Local:** Orquestado de forma automática mediante **`docker-compose.yml`** en la raíz del proyecto para emular el entorno completo localmente en una red aislada.

---

## 🚀 Pipeline de CI/CD (GitHub Actions)

El despliegue está automatizado de forma completa mediante GitHub Actions ([.github/workflows/deploy-eks.yml](.github/workflows/deploy-eks.yml)):

*   **Compilación Paralela (Matrix Strategy):** Las tareas de build y push para `frontend`, `backend` y `db` se ejecutan simultáneamente en 3 runners independientes, acelerando la integración en un **48%**.
*   **Versionado y Trazabilidad:** Las imágenes se publican de forma segura en **Amazon ECR** etiquetadas de forma dinámica con el **Git SHA corto** del commit de origen.
*   **Manejo de Secretos:** Configuración cifrada de credenciales de AWS Academy en los Repository Secrets de GitHub bajo el principio de mínimo privilegio.

---

## ☁️ Infraestructura Productiva en AWS EKS

La plataforma corre en producción sobre un clúster de **Amazon EKS (Kubernetes)**:
*   **Networking:** Despliegue sobre una VPC personalizada con alta disponibilidad física distribuida en dos Zonas de Disponibilidad (`us-east-1a` y `us-east-1b`) y subredes etiquetadas para EKS.
*   **Cómputo FinOps:** Grupo de Nodos trabajadores (`tienda-nodes`) utilizando instancias de tipo **Spot** (`t3.medium`) con autoescalado elástico.
*   **Orquestación y Elasticidad (HPA):** Autoescalado automático configurado mediante Horizontal Pod Autoscaler al superar el 70% de CPU en el backend.
*   **Resiliencia (Auto-Healing):** Monitoreo constante del estado de los contenedores mediante sondas de salud que activan reinicios automáticos en caso de fallas de proceso.
