# Reporte de Evaluación y Mejora Continua - Pipeline DevOps (Sesión 3.6)

Este documento contiene el análisis crítico del pipeline de CI/CD para la Tienda de Perritos en Amazon EKS y las propuestas de mejora continua implementadas para optimizar el rendimiento y la mantenibilidad del sistema.

---

## 📊 Tabla de Métricas de Rendimiento (Pipeline CI/CD)

A continuación se comparan los tiempos de ejecución estimados del pipeline original frente al pipeline optimizado con compilaciones en paralelo:

| Etapa / Job | Tiempos del Pipeline Original (Secuencial) | Tiempos del Pipeline Optimizado (Matriz en Paralelo) | Impacto / Ahorro |
| :--- | :---: | :---: | :---: |
| **Inicio & Checkout** | 3s | 3s | 0% |
| **Login en AWS & ECR** | 8s | 8s | 0% |
| **Compilar y Subir Frontend** | 45s | 45s (En paralelo) | Integrado |
| **Compilar y Subir Backend** | 40s | 40s (En paralelo) | Integrado |
| **Compilar y Subir DB** | 35s | 35s (En paralelo) | Integrado |
| **Total Compilación Docker** | **120s** (Secuencial) | **45s** (Máximo del job más lento) | **~62.5% de Ahorro** |
| **Despliegue a EKS (kubectl)**| 25s | 25s | 0% |
| **Tiempo Total Pipeline** | **156s (~2.6 min)** | **81s (~1.3 min)** | **~48% de Ahorro Total** |

---

## 🔍 Problemas Identificados y Solucionados

1.  **Cuello de Botella en Compilaciones (Rendimiento):**
    *   *Problema:* El pipeline original compilaba y subía las tres imágenes de Docker una después de la otra. Si el proyecto crecía, el tiempo de espera para el desarrollador se volvía inaceptable.
    *   *Solución:* Se reestructuró el pipeline en dos Jobs independientes: un job de compilación paralela por matriz y un job de despliegue.
2.  **Acoplamiento de Datos Sensibles e IDs (Arquitectura):**
    *   *Problema:* Los archivos de Kubernetes tenían hardcodeado el ID de cuenta de AWS de ejemplo (`542768639545`). Esto impedía que el proyecto fuera reutilizable por otras cuentas.
    *   *Solución:* Se parametrizó el despliegue dinámico en el pipeline.
3.  **Configuración Errónea de Balanceador de Carga (Infraestructura):**
    *   *Problema:* El manifiesto `frontend-service.yaml` intentaba delegar la creación del balanceador a un controlador externo (`aws-load-balancer-type: "external"`) que no está instalado en el clúster por defecto, dejando la URL en `<pending>` indefinidamente.
    *   *Solución:* Se removió la anotación conflictiva para usar el balanceador clásico nativo integrado de AWS, garantizando un aprovisionamiento en menos de 20 segundos.

---

## 💡 Oportunidades de Mejora Implementadas

### Mejora 1: Matriz de Estrategia para Ejecución en Paralelo (`strategy.matrix`)
*   **Descripción:** En lugar de ejecutar un solo proceso monolítico en un runner de GitHub, definimos `strategy.matrix.service: [frontend, backend, db]`. GitHub Actions levanta automáticamente **3 máquinas virtuales independientes en paralelo**, compila cada servicio y los sube a ECR simultáneamente.
*   **Impacto Esperado:** Reducción de más del 60% del tiempo de compilación. Feedback casi inmediato para los desarrolladores cuando realizan commits de código.

### Mejora 2: Pipeline Desacoplado en Fases (Build vs Deploy)
*   **Descripción:** Se dividió el flujo en dos Jobs: `build-and-push` y `deploy-to-eks`. El segundo job tiene la propiedad `needs: build-and-push` lo cual asegura que el despliegue en EKS solo se inicie si todas las imágenes de Docker se compilaron y subieron con éxito.
*   **Impacto Esperado:** Mayor robustez y trazabilidad. Si la base de datos o el frontend fallan al compilarse, el clúster de producción nunca se altera, garantizando la estabilidad del sitio web.
