# Sustentación Técnica: Justificación del No Uso de BPMN en el Audit Service

## Introducción

El **Business Process Model and Notation (BPMN)** es un estándar internacional desarrollado para modelar **procesos de negocio**, permitiendo representar de manera gráfica el flujo de actividades, decisiones, participantes y eventos que intervienen en la ejecución de un proceso organizacional.

Su principal objetivo es facilitar la comprensión, análisis y automatización de procesos que involucran reglas de negocio, interacción entre diferentes actores y toma de decisiones.

## Análisis del Audit Service

El **Audit Service** es un microservicio cuya responsabilidad es registrar de forma persistente los eventos de auditoría generados por los demás microservicios del sistema. Su funcionamiento está orientado exclusivamente al procesamiento técnico de eventos y no a la ejecución de procesos de negocio.

De acuerdo con la documentación del proyecto, este microservicio presenta las siguientes características:

- No expone una API REST para interacción con usuarios u otros sistemas.
- No recibe solicitudes directas provenientes de clientes.
- No implementa reglas de negocio.
- No coordina procesos entre diferentes microservicios.
- No genera ni publica nuevos eventos.
- Consume eventos provenientes del broker de mensajería y los almacena en la base de datos de auditoría.

## Flujo de funcionamiento

El comportamiento del microservicio puede resumirse en las siguientes etapas:

1. Recepción de un evento desde el broker de mensajería.
2. Validación de la estructura del evento recibido.
3. Verificación de idempotencia mediante el identificador único (`event_id`).
4. Registro del evento en la tabla `audit_record`.
5. Finalización del procesamiento.

Este flujo corresponde a un proceso completamente lineal y automatizado, sin interacción de usuarios ni ejecución de lógica de negocio.

## Razones para no utilizar BPMN

La utilización de BPMN no resulta necesaria para este microservicio debido a que no cumple con las características propias de un proceso de negocio. En particular, el flujo no presenta:

- Participación de múltiples actores o áreas organizacionales.
- Tareas ejecutadas por usuarios.
- Reglas de negocio complejas.
- Procesos de aprobación o autorización.
- Subprocesos o procesos colaborativos.
- Decisiones de negocio que modifiquen el flujo del proceso.
- Interacción entre diferentes participantes dentro del mismo proceso.

En consecuencia, elaborar un diagrama BPMN únicamente representaría un flujo técnico compuesto por la recepción, validación y almacenamiento de un evento, sin aportar información adicional para la comprensión del sistema.

## Diagramas más adecuados

Considerando la naturaleza del Audit Service, resulta más apropiado documentar su funcionamiento mediante los siguientes diagramas:

- Diagrama de Arquitectura.
- Diagrama de Componentes.
- Diagrama de Secuencia.
- Diagrama de Despliegue.

Estos diagramas describen con mayor precisión la interacción entre el broker de mensajería, el Audit Service y la base de datos, permitiendo comprender el comportamiento técnico del microservicio sin recurrir a la modelación de procesos de negocio.

## Conclusión

Después de analizar la arquitectura y la documentación del **Audit Service**, se concluye que **no es necesario utilizar BPMN**, debido a que este microservicio no implementa un proceso de negocio, sino un proceso técnico de infraestructura encargado del almacenamiento de eventos de auditoría.

Su responsabilidad se limita al consumo de eventos, la validación de su información, la verificación de idempotencia y la persistencia de los registros en la base de datos. Por esta razón, la documentación del servicio se representa de manera más adecuada mediante diagramas de arquitectura, componentes o secuencia, los cuales describen con mayor precisión la interacción entre los elementos que conforman la solución.