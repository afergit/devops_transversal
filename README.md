# Evaluación y Mejora Continua del Pipeline DevOps (Actividad 3.6)

Este proyecto implementa y valida un pipeline optimizado en **GitHub Actions** para el despliegue automático en **Amazon EKS**.

Se enfoca en la **paralelización de compilaciones** y el desacoplamiento de etapas para acelerar la entrega de software, siguiendo los principios de la mejora continua de DevOps.

---

## 📈 Mejoras Aplicadas en esta Versión

1.  **Compilaciones en Paralelo (Jobs Concurrentes):** 
    Reescribimos el workflow [.github/workflows/deploy-eks.yml](.github/workflows/deploy-eks.yml) dividiéndolo en dos Jobs:
    *   `build-and-push`: Utiliza una matriz de estrategia para construir y subir las tres imágenes (`tienda-db`, `tienda-backend`, `tienda-frontend`) **en paralelo en 3 runners independientes**.
    *   `deploy-to-eks`: Realiza la descarga y actualización de las imágenes en el clúster de EKS únicamente después de que el Job de compilación finaliza de forma exitosa (`needs: build-and-push`).
2.  **Eliminación del Bug de Red en EKS:** 
    Se removió la anotación conflictiva `service.beta.kubernetes.io/aws-load-balancer-type: "external"` en el frontend para asegurar la provisión nativa del Classic Load Balancer.

---

## 🚀 Cómo Ejecutar la Validación

1.  Asegúrate de estar en la carpeta de la sesión 3.6 en tu terminal local.
2.  Ejecuta estos comandos para inicializar Git y subir el código al mismo repositorio `tienda-perritos-eks`:
    ```bash
    # 1. Navegar a la carpeta del proyecto 3.6
    cd "C:\Users\skate\Desktop\USIL\DUOC UC\CURSOS\INTRODUCCION  A HERRAMIENTAS DEVOPS\SESION_03\3.6\3.6.3 APP tienda-perritos-EKS_GITHUB (1)"

    # 2. Inicializar Git y realizar el commit
    git init
    git add .
    git commit -m "feat: optimización de pipeline con compilaciones en paralelo"

    # 3. Renombrar la rama a main
    git branch -M main

    # 4. Vincular al mismo repositorio de GitHub (Reemplaza con tu URL de GitHub)
    git remote add origin https://github.com/afergit/tienda-perritos-eks.git

    # 5. Sobrescribir y subir el código para disparar el pipeline optimizado
    git push -f -u origin main
    ```
3.  Entra a la pestaña **Actions** en tu repositorio en GitHub y observa el nuevo diseño visual del pipeline: verás que el primer paso se divide en tres flujos paralelos y luego pasa a la etapa de despliegue.

---

## 📝 Reporte de Mejora Continua
Puedes consultar el análisis técnico detallado, los tiempos de ejecución y las métricas en el archivo [reporte_mejora_continua.md](reporte_mejora_continua.md).
