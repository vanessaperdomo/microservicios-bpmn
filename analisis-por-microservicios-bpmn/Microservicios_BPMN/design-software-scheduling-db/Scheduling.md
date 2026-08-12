# 📅 Microservicio Scheduling - Implementación con BPMN

## 📌 Descripción General

Este microservicio tiene como responsabilidad gestionar la **programación de horarios** dentro del sistema, permitiendo validar, verificar disponibilidad, asignar recursos y notificar resultados.

Se implementa utilizando el estándar **BPMN (Business Process Model and Notation)** ejecutado con Camunda, permitiendo modelar y orquestar el flujo de negocio de manera clara, mantenible y desacoplada del código.

---

## 🧠 ¿Por qué usar BPMN en este microservicio?

El microservicio de scheduling no es un simple CRUD, sino que implica:

- Validaciones de datos
- Verificación de disponibilidad
- Toma de decisiones (conflictos de horario)
- Interacción con otros servicios
- Notificaciones y eventos

BPMN permite:

- 📊 Modelar el flujo de negocio visualmente
- 🔄 Separar la lógica del flujo del código
- ⚙️ Orquestar múltiples pasos automáticamente
- 📈 Facilitar mantenimiento y cambios futuros
- 🧩 Integrar múltiples microservicios

---

## 🏗️ Arquitectura del proceso

El flujo implementado sigue la siguiente lógica:

1. Inicio del proceso
2. Validación de datos de entrada
3. Verificación de disponibilidad del recurso
4. Evaluación de conflicto
5. Ejecución de acciones según el resultado:
   - Si NO hay conflicto:
     - Reservar recurso
     - Crear horario
     - Publicar evento
     - Notificar asignación
   - Si hay conflicto:
     - Notificar conflicto

---

## 🔄 Flujo del proceso

```text
Inicio
 ↓
Validar datos
 ↓
Verificar disponibilidad
 ↓
¿Hay conflicto?
 ├── NO → Reservar → Crear → Publicar → Notificar → Fin OK
 └── SÍ → Notificar conflicto → Fin Error