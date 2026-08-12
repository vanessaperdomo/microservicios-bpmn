# Diagrama BPMN general de los microservicios que requieren BPMN

## 1. Alcance

Este documento presenta el alcance del diagrama BPMN general que se elaborará para el proyecto. Su propósito es definir qué microservicios del sistema participarán en la representación general del proceso de negocio y cuáles no, con base en un análisis previo realizado a cada uno de ellos.

---

## 2. Propósito

El objetivo de este documento es justificar por qué se realizará un diagrama BPMN general y qué microservicios serán incluidos en él. Se busca mostrar una visión clara y ordenada del conjunto de procesos de negocio que realmente requieren modelación con BPMN, evitando incluir componentes que no aportan valor al análisis del flujo general.

---

## 3. Análisis previo realizado

Antes de definir el diagrama general, se realizó un análisis de cada microservicio para identificar si su funcionamiento correspondía a un proceso de negocio con características adecuadas para BPMN. Ese análisis permitió clasificar los servicios según si presentaban:

- flujos de negocio con decisiones;
- pasos secuenciales o estados de proceso;
- interacción entre actores y servicios;
- eventos, notificaciones o reglas de negocio;
- necesidad de representación visual del proceso.

Con base en este análisis, se determinaron los microservicios que sí requieren BPMN y los que no.

---

## 4. Microservicios que participan en el diagrama BPMN general

Los microservicios que sí participarán en el diagrama BPMN general son los siguientes:

- Academic Management
- IAM
- Monitoring
- Scheduling
- Reference Data, únicamente en su parte administrativa

Estos servicios fueron seleccionados porque sus funciones implican procesos de negocio que pueden representarse de forma clara mediante BPMN.

---

## 5. Microservicios que no participan en el diagrama general

No se incluirán en el diagrama BPMN general los microservicios que cumplen funciones más transversales, técnicas o de apoyo, tales como:

- Actors
- Audit
- Documents
- Environment

Estos servicios no presentan un flujo de negocio principal que justifique su inclusión en una vista general de BPMN.

---

## 6. ¿Por qué se realiza un diagrama BPMN general?

Se realiza un diagrama BPMN general porque permite visualizar de forma integrada los procesos de negocio más relevantes del sistema, sin entrar en el detalle técnico de cada microservicio. Este tipo de diagrama ayuda a:

- comprender el flujo general del sistema;
- identificar cómo interactúan los microservicios que tienen procesos de negocio;
- mostrar de manera organizada los puntos clave del funcionamiento del sistema;
- facilitar la comunicación con stakeholders, evaluadores o personas no técnicas.

Además, al incluir solo los microservicios que realmente requieren BPMN, el diagrama resulta más limpio, comprensible y útil para la presentación.

---

## 7. Características esperadas del diagrama general

El diagrama BPMN general debe presentarse como una vista de alto nivel, donde cada microservicio participante aparezca una sola vez y se relacione con los demás a través de un flujo lógico de negocio. La idea es que el diagrama muestre:

- un inicio del proceso;
- la participación de los microservicios relevantes;
- el flujo principal de operaciones;
- los puntos de decisión o interacción entre servicios;
- un fin del proceso con resultado definido.

---

## 8. Conclusión

El diagrama BPMN general se elaborará únicamente con los microservicios que, según el análisis previo, requieren BPMN. Esta selección permite construir una representación más clara del negocio, más organizada y más adecuada para la presentación, evitando la inclusión de servicios que no aportan valor a la comprensión del flujo general del sistema.
