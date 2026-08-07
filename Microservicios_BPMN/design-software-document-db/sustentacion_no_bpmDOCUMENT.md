# Sustentación Técnica: Exclusión de BPM en `document-service`

Este documento detalla las razones técnicas y de arquitectura por las cuales **no es recomendable** implementar un motor de BPM (Business Process Management) dentro del microservicio `document-service`.

---

## 1. Contexto del Proyecto

El microservicio `document-service` tiene una responsabilidad puramente **técnica y transversal**:
*   Administrar plantillas HTML/Handlebars (`document_template`).
*   Registrar metadatos de documentos (`document` y `document_version`).
*   Gestionar la generación asíncrona de archivos (PDF/Excel/Word) a través de colas (`pdf-renderer-worker`).
*   Almacenar binarios en Object Storage (MinIO/S3).

---

## 2. Razones para NO Usar un BPM en este Microservicio

### A. Simplicidad Absoluta del Ciclo de Vida
Un motor de BPM destaca cuando existen flujos complejos con decisiones dinámicas, caminos condicionales y pasos humanos (aprobaciones, revisiones). El ciclo de vida de un documento en este servicio es lineal y técnico:
```mermaid
graph LR
    GENERATING --> AVAILABLE
    GENERATING --> GENERATION_FAILED
    AVAILABLE --> ARCHIVED/EXPIRED
```
Este flujo se resuelve de manera óptima con un simple campo de estado (`status`) en la base de datos relacional y lógica de código estándar, sin necesidad de orquestadores de procesos.

### B. Incompatibilidad de Responsabilidades (Dominio vs. Infraestructura)
*   **BPM (Enfoque funcional):** Está diseñado para modelar procesos de negocio (ej. "Aprobación de crédito", "Matrícula de estudiante").
*   **`document-service` (Enfoque técnico):** Es un servicio utilitario de bajo nivel. No posee lógica ni reglas de negocio; es agnóstico a lo que el PDF representa en el mundo real. Introducir un BPM aquí acoplaría lógica de procesos a una herramienta de infraestructura.

### C. Penalización de Rendimiento y Sobrecarga de Infraestructura
Los motores de BPM tradicionales (como Camunda o Flowable) introducen:
*   **Más de 20-30 tablas persistentes** en la base de datos para tracking de variables de proceso, tareas históricas y logs de auditoría.
*   **Mayor consumo de CPU y memoria RAM** para interpretar los diagramas BPMN/XML en tiempo de ejecución.
*   **Latencia adicional** en la persistencia de estados.
Para un servicio que debe responder de forma rápida a peticiones REST y procesar eventos masivos de generación de PDFs, esta sobrecarga es inaceptable.

### D. La Arquitectura Basada en Eventos (EDA) ya Resuelve la Asincronía
El diseño actual utiliza **RabbitMQ** y consumidores dedicados (`pdf-renderer-worker` y `document-lifecycle-worker`). La asincronía, los reintentos automáticos y el desacoplamiento ya están resueltos mediante coreografía de eventos. Añadir un BPM crearía una redundancia tecnológica innecesaria.

---

## 3. Conclusión y Recomendación

> [!IMPORTANT]
> **Decisión de Arquitectura:** Mantener `document-service` como un microservicio ligero orientado a datos y eventos sin motor de BPM.

*   **¿Dónde sí evaluar BPM?** En los servicios de negocio de nivel superior, como un `enrollment-service` (servicio de matrículas), donde sí existen flujos con pasos humanos, dependencias de tiempo y múltiples aprobaciones antes de que se solicite la creación física del PDF.
