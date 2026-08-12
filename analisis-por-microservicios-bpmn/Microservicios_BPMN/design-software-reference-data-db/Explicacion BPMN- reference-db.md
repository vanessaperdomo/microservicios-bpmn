# BPMN – Proceso del Administrador

## Descripción

El siguiente BPMN representa el proceso realizado por el **Administrador** dentro del microservicio **Reference Data Service**. Este proceso corresponde a las operaciones administrativas sobre los datos de referencia del sistema, como la creación, actualización, desactivación y consulta de la jerarquía institucional, catálogos y parámetros.

Este BPMN no representa el funcionamiento interno del microservicio, sino el flujo de trabajo que sigue un usuario administrador al interactuar con el sistema.

---

# Objetivo

Modelar el flujo de actividades que realiza un administrador para gestionar los datos de referencia del sistema de manera segura y controlada.

---

# Actor

- Administrador

Es el único actor que tiene permisos para realizar operaciones de escritura sobre los datos del microservicio.

---

# Descripción del proceso

## 1. Inicio

El proceso comienza cuando el administrador accede al sistema para realizar una operación administrativa.

---

## 2. Iniciar sesión

El administrador debe autenticarse mediante el sistema de autenticación (IAM Service).

Durante este paso se valida:

- Usuario
- Contraseña
- Token JWT
- Permisos asignados

---

## 3. Validar autenticación

El sistema verifica que el usuario tenga permisos para administrar los datos de referencia.

### Si la autenticación falla

El sistema muestra un mensaje de error y finaliza el proceso.

### Si la autenticación es correcta

El administrador puede continuar con la operación.

---

## 4. Seleccionar operación

El administrador selecciona la acción que desea realizar.

Las operaciones disponibles son:

- Registrar información
- Actualizar información
- Desactivar registros
- Consultar información

Estas operaciones pueden aplicarse sobre cualquiera de las entidades del microservicio, por ejemplo:

- Macroregiones
- Microrregiones
- Departamentos
- Municipios
- Centros de formación
- Unidades institucionales
- Catálogos
- Parámetros

---

## 5. Validar información

Antes de guardar la información, el sistema realiza diferentes validaciones.

Entre ellas:

- Campos obligatorios
- Longitud de los datos
- Códigos únicos
- Integridad referencial
- Restricciones de negocio

Ejemplos:

- El código DANE no puede repetirse.
- El código del centro de formación debe ser único.
- Un municipio debe pertenecer a un departamento existente.

---

## 6. Decisión

El sistema determina si la información ingresada es válida.

### Si la información no es válida

Se muestran los errores encontrados para que el administrador pueda corregirlos.

El proceso termina sin realizar cambios en la base de datos.

### Si la información es válida

La operación continúa.

---

## 7. Guardar cambios

El sistema registra los cambios en PostgreSQL.

Dependiendo de la operación realizada, se ejecuta:

- INSERT
- UPDATE
- Soft Delete

---

## 8. Publicación de eventos

Si la modificación corresponde a un catálogo del sistema, el microservicio publica el evento:

```
reference.catalog.updated
```

Este evento es enviado mediante RabbitMQ para informar a los demás microservicios que los datos del catálogo fueron modificados.

---

## 9. Invalidación de caché

Los microservicios consumidores reciben el evento y eliminan la información almacenada en Redis para garantizar que las próximas consultas obtengan los datos actualizados.

---

## 10. Confirmación

Finalmente, el sistema informa al administrador que la operación fue realizada correctamente.

Con esto finaliza el proceso.

---

# Resultado

Al finalizar el proceso pueden ocurrir dos escenarios:

## Operación exitosa

- Información almacenada correctamente.
- Base de datos actualizada.
- Evento publicado (si aplica).
- Caché invalidada.
- Confirmación enviada al administrador.

## Operación fallida

- No se modifica la base de datos.
- Se muestran los errores encontrados.
- El administrador debe corregir la información antes de intentar nuevamente.

---

# Justificación del BPMN

Se decidió modelar únicamente el proceso del **Administrador** porque es el único actor que ejecuta un flujo de negocio dentro del microservicio.

Las operaciones de consulta (GET) realizadas por otros servicios no requieren un BPMN, ya que consisten únicamente en leer información desde la base de datos y devolver una respuesta, sin involucrar decisiones de negocio ni múltiples actividades.

Por esta razón, el proceso administrativo es el único que representa un flujo de trabajo susceptible de ser modelado mediante BPMN.

---

# Resumen del flujo

Inicio

↓

Autenticación

↓

Validación de permisos

↓

Selección de operación

↓

Validación de información

↓

¿Información válida?

├── No → Mostrar errores → Fin

└── Sí

↓

Guardar cambios

↓

¿Se modificó un catálogo?

├── Sí → Publicar evento → Invalidar caché

└── No

↓

Confirmar operación

↓

Fin