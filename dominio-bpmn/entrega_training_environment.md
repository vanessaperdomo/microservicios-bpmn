# Entrega de Dominio Técnico: Training Environment

## 1. Entendimiento Inicial del Dominio

El dominio **Training Environment** es el núcleo encargado de la gestión de ambientes de formación física o virtual dentro de la institución. Su propósito principal es administrar los espacios (ambientes), controlar su disponibilidad, planificar mantenimientos y registrar las reservas para garantizar un uso ordenado y eficiente de los recursos.

**Principales Procesos del Dominio:**
1. **Gestión de Ambientes:** Creación y configuración de nuevos espacios de formación.
2. **Configuración de Disponibilidad:** Establecimiento de reglas y horarios en los que un ambiente puede ser utilizado.
3. **Gestión de Mantenimientos:** Bloqueo de ambientes por necesidades correctivas o preventivas.
4. **Gestión de Reservas:** Asignación de ambientes a eventos o sesiones sin que se presenten solapamientos de horario.

---

## 2. Diagrama BPMN por Dominio (Procesos de Training Environment)

A continuación, se presenta el modelo BPMN que representa el funcionamiento de extremo a extremo del dominio **Training Environment**. Este modelo documenta el proceso funcional, aunque la implementación técnica se basa en transacciones directas y eventos.

```mermaid
sequenceDiagram
    participant Usuario as "Usuario / Administrador"
    participant DominioTE as "Dominio: Training Environment"
    participant Dependencias as "Dependencias (IAM / Ref Data)"
    participant EventBus as "Bus de Eventos"

    Usuario ->> DominioTE: Selecciona Operación
    activate DominioTE
    
    alt Operación: Crear/Actualizar Ambiente
        DominioTE ->> Dependencias: Validar permisos (IAM) y Sedes (Ref Data)
        Dependencias -->> DominioTE: Datos OK
        DominioTE ->> DominioTE: Persistir cambios del ambiente
        DominioTE ->> EventBus: Emitir 'EnvironmentCreated/Updated'
        
    else Operación: Registrar Disponibilidad
        DominioTE ->> DominioTE: Validar reglas lógicas de horario
        DominioTE ->> DominioTE: Persistir disponibilidad
        DominioTE ->> EventBus: Emitir 'AvailabilityUpdated'
        
    else Operación: Programar Mantenimiento
        DominioTE ->> DominioTE: Validar conflictos con reservas existentes
        alt Hay Conflicto
            DominioTE -->> Usuario: Error: Ambiente ocupado
        else Sin Conflicto
            DominioTE ->> DominioTE: Bloquear ambiente
            DominioTE ->> EventBus: Emitir 'MaintenanceScheduled'
        end
        
    else Operación: Crear Reserva
        DominioTE ->> DominioTE: Validar disponibilidad, mantenimientos y solapamientos (DB constraints)
        alt Conflicto detectado
            DominioTE -->> Usuario: Rechazo: Regla de integridad violada
        else Sin Conflictos
            DominioTE ->> DominioTE: Confirmar Reserva
            DominioTE ->> EventBus: Emitir 'ReservationCreated'
        end
    end
    
    DominioTE -->> Usuario: Confirmación de operación exitosa
    deactivate DominioTE
```

---

## 3. Justificación Técnica de Decisiones de Arquitectura

A pesar de que el dominio tiene procesos de negocio definidos (como se observa en el BPMN anterior), a nivel técnico se han tomado las siguientes decisiones:

### 3.1. Ausencia de un Motor BPM Ejecutable
El microservicio **no implementa un motor BPM** (como Camunda). Los procesos del dominio (crear ambiente, reservar, etc.) son de naturaleza **transaccional y síncrona**. No existen flujos de larga duración (long-running processes) que requieran estado persistente de proceso, aprobaciones manuales en cadena o coreografía compleja. Implementar un BPM aquí añadiría complejidad innecesaria.

### 3.2. Diseño como Bounded Context Independiente
El dominio se aísla en su propio microservicio para que la lógica de prevención de conflictos de espacio (su core) esté centralizada. Otros servicios, como el de agendamiento académico (`scheduling-service`), no necesitan conocer cómo se gestiona el inventario de espacios.

### 3.3. Resolución de Reglas Críticas en Base de Datos
La regla principal es **evitar el solapamiento de reservas y mantenimientos**. Esto se resuelve mediante restricciones de integridad en la Base de Datos (o bloqueos a nivel de transacción) para garantizar consistencia absoluta, sin necesidad de validaciones frágiles en capa de aplicación.

### 3.4. Arquitectura Orientada a Eventos (Event-Driven)
Al confirmar cambios en su dominio (ver diagrama BPMN), el servicio publica eventos. Esto permite que otros dominios reaccionen de manera asíncrona, asegurando un bajo acoplamiento.

### 3.5. Roles gestionados por IAM, no por flujos
La existencia de los actores mostrados en el diagrama (Usuarios y Administradores) se controla a través de permisos de sistema (IAM) y validación de endpoints, no mediante tareas de usuario dentro de un motor BPM.
