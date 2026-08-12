# Microservicio IAM (Identity and Access Management)
## Autenticación, Autorización e Identidad

> **Estado:** 🟡 En revisión  
> **Microservicio:** IAM (Identity and Access Management)

---

# ¿Qué es el microservicio IAM?

El microservicio **IAM (Identity and Access Management)** es el encargado de gestionar la identidad de los usuarios, controlar el acceso al sistema y emitir los tokens de autenticación (JWT).

Es considerado el **servicio de entrada** de la plataforma, ya que **todos los usuarios deben autenticarse primero en IAM antes de acceder a cualquier otro microservicio**.

Una vez autenticado el usuario, los demás microservicios verifican el JWT de forma local, sin necesidad de consultar nuevamente a IAM.

---

# Responsabilidad principal

El microservicio IAM tiene como responsabilidades:

- Autenticar usuarios.
- Emitir Access Token y Refresh Token.
- Administrar usuarios.
- Administrar roles.
- Administrar permisos (Features).
- Gestionar Scopes de acceso.
- Administrar sesiones.
- Recuperación de contraseña.
- Registrar auditoría de autenticaciones.
- Publicar eventos hacia otros microservicios.

---

# ¿Por qué es un microservicio?

IAM cumple con todas las características de una arquitectura basada en microservicios:

- Tiene una única responsabilidad.
- Posee su propia base de datos (`iam_db`).
- Expone una API REST independiente.
- Publica eventos mediante un Broker (RabbitMQ).
- Puede desplegarse de manera independiente.
- Escala horizontalmente.
- Es consumido por los demás microservicios.

---

# Tipo de componente

El componente implementado corresponde a:

- ✅ API REST (`iam-api`)

No corresponde a:

- Worker
- Scheduler
- Workflow
- Gateway
- Notifier

---

# Base de datos

Motor:

- PostgreSQL 16

Base de datos:

```
iam_db
```

Contiene toda la información relacionada con:

- Usuarios
- Roles
- Features
- Módulos
- Refresh Tokens
- Password Reset
- Auditoría de Login

---

# Componentes principales

El microservicio administra las siguientes entidades:

- User
- Module
- Feature
- Role
- RoleFeature
- UserRole
- UserScopeOverride
- RefreshToken
- PasswordResetRequest
- AuditLogin

---

# Funcionalidades principales

## Autenticación

Permite:

- Login
- Refresh Token
- Logout
- Consultar usuario autenticado
- Recuperación de contraseña

Endpoints:

```
POST /auth/login
POST /auth/refresh
POST /auth/logout
GET  /auth/me
POST /auth/password-reset/request
POST /auth/password-reset/confirm
```

---

## Gestión de usuarios

Permite:

- Crear usuario
- Consultar usuarios
- Editar usuario
- Desactivar usuario
- Consultar sesiones activas
- Revocar sesiones

Endpoints:

```
GET    /users
POST   /users
GET    /users/{id}
PUT    /users/{id}
POST   /users/{id}/deactivate
GET    /users/{id}/sessions
DELETE /users/{id}/sessions/{session_id}
```

---

## Gestión de Roles

Permite:

- Consultar roles
- Consultar Features
- Asignar Roles
- Revocar Roles

Endpoints:

```
GET    /roles
GET    /roles/{id}/features
POST   /users/{id}/roles
DELETE /users/{id}/roles/{role_name}
```

---

## Gestión de Módulos

Permite consultar todos los módulos del sistema.

Endpoint:

```
GET /modules
```

---

## Gestión de Scopes

Permite otorgar permisos temporales o excepcionales a un usuario.

Endpoints:

```
GET    /users/{id}/scope-overrides
POST   /users/{id}/scope-overrides
DELETE /users/{id}/scope-overrides/{override_id}
```

---

## Reportes

Permite consultar auditoría de intentos de autenticación.

Endpoint:

```
GET /reports/login-audit
```

---

# Seguridad

IAM implementa:

- JWT
- Refresh Token
- RBAC (Role Based Access Control)
- Features
- Scopes
- Auditoría
- Bloqueo por intentos fallidos
- Recuperación segura de contraseña

---

# Escalabilidad

Este microservicio fue diseñado para ejecutarse en ambientes de alta disponibilidad.

Características:

- Stateless (sin estado en memoria).
- Escalamiento horizontal.
- Balanceador de carga.
- Health Checks.
- Readiness.
- Observabilidad.
- Logs estructurados.
- Trazabilidad distribuida.

---

# Eventos publicados

IAM publica eventos para mantener sincronizados los demás microservicios.

Eventos principales:

- iam.user.created
- iam.user.deactivated
- iam.role.assigned
- iam.session.started

Consumidores:

- audit-service
- actors-service

---

# ¿Necesita un diagrama BPMN?

## Sí.

Aunque es un microservicio técnico, posee procesos de negocio con validaciones, decisiones y reglas que pueden modelarse mediante BPMN.

No todos los endpoints requieren un BPMN; solamente aquellos que representan un proceso completo.

---

# Procesos recomendados para BPMN

| Proceso | ¿Requiere BPMN? |
|----------|-----------------|
| Inicio de sesión (Login) | ✅ Sí |
| Refresh Token | ✅ Sí |
| Logout | ✅ Sí |
| Recuperación de contraseña | ✅ Sí |
| Crear usuario | ✅ Sí |
| Asignar rol | ✅ Sí |
| Desactivar usuario | ✅ Sí |
| Crear Scope Override | ✅ Sí |
| Consultar perfil | Opcional |
| Listar usuarios | No |
| Listar roles | No |
| Listar módulos | No |

---

# BPMN más importante

El BPMN principal del microservicio es el **Proceso de Inicio de Sesión (Login)**.

Flujo general:

```
Usuario
    │
    ▼
Ingresa correo y contraseña
    │
    ▼
IAM recibe la solicitud
    │
    ▼
Buscar usuario
    │
    ▼
¿Existe?
 ├── No → Error 401
 └── Sí
        │
        ▼
¿Cuenta activa?
 ├── No → Error 401
 └── Sí
        │
        ▼
¿Cuenta bloqueada?
 ├── Sí → Error 401
 └── No
        │
        ▼
Validar contraseña
        │
        ▼
¿Contraseña correcta?
 ├── No
 │      │
 │      ▼
 │ Incrementar intentos
 │ Registrar auditoría
 │ Responder Error 401
 │
 └── Sí
        │
        ▼
Calcular Roles
        │
        ▼
Calcular Features
        │
        ▼
Calcular Scopes
        │
        ▼
Generar Access Token
        │
        ▼
Generar Refresh Token
        │
        ▼
Registrar sesión
        │
        ▼
Publicar evento
iam.session.started
        │
        ▼
Enviar respuesta al usuario
```

---

# Conclusión

El microservicio IAM es uno de los componentes más importantes de la arquitectura, ya que es responsable de la autenticación, autorización e identidad de todos los usuarios.

Además de exponer una API REST, implementa procesos de negocio complejos, administra usuarios, roles y permisos, genera tokens JWT, registra auditoría y publica eventos para mantener sincronizados los demás microservicios.

Por esta razón, **sí es recomendable modelar sus procesos principales mediante diagramas BPMN**, especialmente el proceso de autenticación (Login), recuperación de contraseña, creación de usuarios y asignación de roles.