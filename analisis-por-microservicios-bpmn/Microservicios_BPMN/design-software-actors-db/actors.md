\# Justificación de por qué no se realiza un diagrama BPMN para el microservicio Actors



\## ¿Por qué no se realiza un diagrama BPMN?



No se desarrolla un diagrama \*\*BPMN (Business Process Model and Notation)\*\* para el microservicio \*\*Actors\*\*, ya que su propósito principal no es modelar procesos de negocio, sino administrar la información de los actores del sistema.



Este microservicio se encarga de almacenar y gestionar entidades como:



\- Instructores.

\- Aprendices.

\- Empresas.

\- Asignaciones.

\- Contratos.

\- Áreas de formación.



Estas operaciones corresponden principalmente a acciones de tipo \*\*CRUD (Crear, Consultar, Actualizar y Eliminar)\*\*, las cuales no representan un flujo de negocio complejo que requiera ser modelado mediante BPMN.



\## Función del microservicio



El servicio \*\*Actors\*\* actúa como un proveedor de información para otros microservicios del sistema. Su responsabilidad consiste en mantener los datos de los diferentes actores y ofrecerlos cuando otros servicios los necesitan.



No coordina procesos, aprobaciones, validaciones secuenciales ni actividades entre diferentes participantes del negocio.



\## ¿Cuándo se utiliza BPMN?



Los diagramas BPMN se utilizan para representar procesos de negocio que incluyen:



\- Varias actividades consecutivas.

\- Toma de decisiones.

\- Eventos.

\- Participación de diferentes actores.

\- Flujos de trabajo completos.



Ejemplos de procesos donde sí sería apropiado utilizar BPMN son:



\- Registro completo de un aprendiz.

\- Asignación de un instructor a una competencia.

\- Gestión de una etapa productiva.

\- Proceso de matrícula.

\- Proceso de certificación.



En estos casos existe un flujo de actividades claramente definido.



\## Conclusión



No se elabora un diagrama BPMN para el microservicio \*\*Actors\*\* porque su responsabilidad está enfocada en la administración y persistencia de información de los actores del sistema. Sus funcionalidades consisten principalmente en operaciones CRUD y exposición de datos mediante una API, sin ejecutar procesos de negocio complejos que justifiquen la utilización de la notación BPMN.

