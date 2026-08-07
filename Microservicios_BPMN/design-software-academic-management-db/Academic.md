# Microservicio **Academic** con BPMN y Camunda

En un sistema de microservicios, el servicio **Academic** suele encargarse de la lógica académica (cursos, inscripciones, instructores, etc.). Estos flujos implican varios pasos de negocio (validar estudiante, verificar curso, requisitos, cupos, etc.) que **BPMN** (Business Process Model and Notation) puede modelar claramente. De hecho, BPMN es “una notación gráfica estandarizada que permite el modelado de procesos de negocio, en un formato de flujo de trabajo”. Su objetivo es servir de lenguaje común fácil de leer tanto para analistas como para desarrolladores, cerrando la brecha entre diseño e implementación de procesos.

Además, BPMN define el flujo (almacenado como un archivo XML ejecutable) *sin generar código*, enfocándose solo en la secuencia de actividades. Esto encaja perfectamente con **microservicios**: cada tarea BPMN puede delegarse a un servicio externo, manteniendo el acoplamiento bajo. En otras palabras, el motor de BPMN orquesta el proceso (enviar la tarea al microservicio adecuado), y el código de negocio permanece en los propios servicios (como nuestro microservicio Academic). Camunda es una plataforma Java de código abierto que ejecuta BPMN y facilita esta orquestación.

*Ejemplo ilustrativo de diagrama BPMN:* flujo de proceso con inicio (círculo verde), tareas (rectángulos) y decisiones (rombos). El proceso de matrícula de un estudiante seguiría un esquema similar: tras el **inicio**, se validan datos, se toman decisiones (por ejemplo, ¿hay cupo?) y finalmente se guarda la inscripción y se notifican eventos.

## Flujo de negocio (ejemplo: matriculación)

Consideremos el proceso de **matricular a un estudiante en un curso** en el microservicio Academic. Un BPMN simplificado podría ser:

- **Inicio:** Llega una solicitud de matrícula (`Start Event`).  
- **Validar Estudiante:** Tarea de servicio que verifica que exista el estudiante (Delegate Java).  
- **Validar Curso:** Tarea de servicio que verifica que el curso existe y está activo.  
- **Verificar Requisitos (opcional):** Tarea que comprueba prerrequisitos del curso.  
- **Verificar Cupos:** Tarea que revisa si el curso tiene cupo disponible.  
- **Guardar Matrícula:** Tarea de servicio que persiste la matrícula en BD.  
- **Publicar Evento:** Tarea que envía un evento (por ejemplo, con Kafka) avisando que hubo una nueva matrícula.  
- **Fin:** Proceso finaliza con éxito.  

Entre tareas, usamos **gateways** para tomar decisiones. Por ejemplo, si *el estudiante no existe*, vamos a un End Event con error; si *no hay cupo*, podríamos terminar el flujo o registrar en lista de espera. Esto se define en el diagrama BPMN (no mostramos el XML completo aquí, pero puedes guardarlo en `academic-enrollment.bpmn` en `src/main/resources`).

## Implementación en Spring Boot con Camunda

A continuación se muestra cómo integrar BPMN en el microservicio Academic usando Spring Boot + Camunda. La idea es exponer un endpoint REST que inicia el proceso BPMN y luego delegar las tareas a componentes Java.

1. **Dependencias (pom.xml):**  
   Agrega Camunda BPM al proyecto. Por ejemplo, en Maven:
   ```xml
   <dependencies>
     <!-- Spring Boot Web y JPA -->
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
     </dependency>
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-jpa</artifactId>
     </dependency>
     <!-- Camunda BPM para Spring Boot -->
     <dependency>
       <groupId>org.camunda.bpm.springboot</groupId>
       <artifactId>camunda-bpm-spring-boot-starter</artifactId>
       <version>7.20.0</version>
     </dependency>
     <!-- (Opcional) RabbitMQ/Kafka para eventos -->
   </dependencies>
   ```

2. **Definir el proceso BPMN:**  
   Usando Camunda Modeler o similar, crea `academic-enrollment.bpmn` en `src/main/resources`. Asigna un **process key** (p.ej. `academic-enrollment`). Las tareas de servicio (`<serviceTask>`) deben referenciar los *delegates* Java que implementaremos. Ejemplo de fragmento BPMN:
   ```xml
   <bpmn:process id="academic-enrollment" name="Academic Enrollment">
     <bpmn:startEvent id="Start"/>
     <bpmn:serviceTask id="ValidateStudent" name="Validar Estudiante" camunda:class="com.ejemplo.academic.ValidateStudentDelegate"/>
     <bpmn:exclusiveGateway id="Gateway_StudentExists"/>
       <bpmn:sequenceFlow id="sf1" sourceRef="ValidateStudent" targetRef="Gateway_StudentExists"/>
       <!-- caminos del gateway... -->
     <bpmn:serviceTask id="ValidateCourse" name="Validar Curso" camunda:class="com.ejemplo.academic.ValidateCourseDelegate"/>
     <!-- más tareas y gateways -->
     <bpmn:serviceTask id="SaveEnrollment" name="Guardar Matr\u00edcula" camunda:class="com.ejemplo.academic.SaveEnrollmentDelegate"/>
     <bpmn:serviceTask id="PublishEvent" name="Publicar Evento" camunda:class="com.ejemplo.academic.PublishEventDelegate"/>
     <bpmn:endEvent id="End"/>
   </bpmn:process>
   ```
   *(No olvides definir secuencias y condiciones en cada gateway.)*

3. **Controlador REST:**  
   Crea un endpoint en `AcademicController` (o `EnrollmentController`) para iniciar el proceso. Ejemplo:
   ```java
   @RestController
   @RequestMapping("/academic")
   @RequiredArgsConstructor
   public class AcademicController {
       private final RuntimeService runtimeService;

       @PostMapping("/enroll")
       public ResponseEntity<String> enroll(@RequestBody EnrollmentRequest req) {
           // Asignar variables de proceso
           Map<String, Object> vars = new HashMap<>();
           vars.put("studentId", req.getStudentId());
           vars.put("courseId", req.getCourseId());

           // Iniciar instancia del proceso BPMN por su key
           runtimeService.startProcessInstanceByKey("academic-enrollment", vars);
           return ResponseEntity.ok("Proceso de matrícula iniciado");
       }
   }
   ```
   Donde `EnrollmentRequest` es una clase simple con `studentId` y `courseId` (p.ej. usando Lombok `@Data`):
   ```java
   @Data
   public class EnrollmentRequest {
       private Long studentId;
       private Long courseId;
   }
   ```

4. **Repositorios y Entidades:**  
   Configura entidades JPA mínimas. Por ejemplo, `Student`, `Course` y `Enrollment`:
   ```java
   @Entity
   public class Student {
       @Id @GeneratedValue private Long id;
       private String name;
       // getters/setters...
   }
   @Entity
   public class Course {
       @Id @GeneratedValue private Long id;
       private String title;
       private int capacity;
       // getters/setters...
   }
   @Entity
   public class Enrollment {
       @Id @GeneratedValue private Long id;
       private Long studentId;
       private Long courseId;
       // getters/setters...
   }
   ```
   E interfaces `StudentRepository`, `CourseRepository`, `EnrollmentRepository` extienden `JpaRepository<...>`. El motor de Spring Data crea automáticamente las tablas.

5. **Delegados BPMN (JavaDelegates):**  
   Cada tarea de servicio en el BPMN ejecuta código Java. Usa `@Component` y la interfaz `JavaDelegate`. Algunos ejemplos:

   - **Validar estudiante:** verifica existencia en DB (arroja error si no existe).
     ```java
     @Component("validateStudent")
     @RequiredArgsConstructor
     public class ValidateStudentDelegate implements JavaDelegate {
         private final StudentRepository studentRepo;

         @Override
         public void execute(DelegateExecution ex) {
             Long sid = (Long) ex.getVariable("studentId");
             if (!studentRepo.existsById(sid)) {
                 // Lanza un error de negocio para interrumpir el proceso
                 throw new BpmnError("STUDENT_NOT_FOUND");
             }
         }
     }
     ```
     En el gateway posterior puedes preguntar `${studentExists}` si creaste esa variable, o manejar la excepción para terminar con fin de error.

   - **Validar curso:** parecido al anterior, usando `CourseRepository`:
     ```java
     @Component("validateCourse")
     @RequiredArgsConstructor
     public class ValidateCourseDelegate implements JavaDelegate {
         private final CourseRepository courseRepo;

         @Override
         public void execute(DelegateExecution ex) {
             Long cid = (Long) ex.getVariable("courseId");
             if (!courseRepo.existsById(cid)) {
                 throw new BpmnError("COURSE_NOT_FOUND");
             }
         }
     }
     ```

   - **Verificar prerrequisitos (opcional):** podrías inyectar lógica extra aquí. 

   - **Verificar cupos:** comprueba la capacidad del curso.
     ```java
     @Component("checkCapacity")
     @RequiredArgsConstructor
     public class CheckCapacityDelegate implements JavaDelegate {
         private final CourseRepository courseRepo;

         @Override
         public void execute(DelegateExecution ex) {
             Long cid = (Long) ex.getVariable("courseId");
             Course course = courseRepo.findById(cid).orElse(null);
             if (course == null || course.getCapacity() <= 0) {
                 // No hay cupos: establecer variable para gateway
                 ex.setVariable("hasCapacity", false);
             } else {
                 ex.setVariable("hasCapacity", true);
             }
         }
     }
     ```
     En el BPMN, conecta este delegate a un gateway con condiciones `${hasCapacity}` (sí) y `${!hasCapacity}` (no).

   - **Guardar matrícula:** crea el registro de inscripción.
     ```java
     @Component("saveEnrollment")
     @RequiredArgsConstructor
     public class SaveEnrollmentDelegate implements JavaDelegate {
         private final EnrollmentRepository enrollmentRepo;

         @Override
         public void execute(DelegateExecution ex) {
             Enrollment e = new Enrollment();
             e.setStudentId((Long) ex.getVariable("studentId"));
             e.setCourseId((Long) ex.getVariable("courseId"));
             enrollmentRepo.save(e);
         }
     }
     ```

   - **Publicar evento:** emite un evento al sistema de mensajería (Kafka/RabbitMQ).
     ```java
     @Component("publishEvent")
     @RequiredArgsConstructor
     public class PublishEventDelegate implements JavaDelegate {
         private final RabbitTemplate rabbitTemplate; // o KafkaTemplate

         @Override
         public void execute(DelegateExecution ex) {
             Long sid = (Long) ex.getVariable("studentId");
             Long cid = (Long) ex.getVariable("courseId");
             // Construye el evento (p. ej. JSON sencillo)
             String event = "{\"studentId\":" + sid + ",\"courseId\":" + cid + "}";
             rabbitTemplate.convertAndSend("academic.exchange", "academic.enrolled", event);
         }
     }
     ```
     Esto permitirá que otros microservicios (p.ej. Notification) reaccionen a la nueva matrícula.

6. **Ejecutar y probar:**  
   - Inicia la aplicación Spring Boot normalmente (`mvn spring-boot:run`). Camunda arranca incorporado.  
   - Accede a [http://localhost:8080/engine-rest](http://localhost:8080/engine-rest) o al **Cockpit** de Camunda (`/camunda`) si lo habilitaste.  
   - Prueba el endpoint REST con `curl` o Postman:
     ```bash
     curl -X POST -H "Content-Type: application/json" \
       -d '{"studentId": 1, "courseId": 101}' \
       http://localhost:8080/academic/enroll
     ```
     Deberías ver “Proceso de matrícula iniciado”. Internamente Camunda ejecuta el BPMN, llama a los delegates y persiste la matrícula.  

  

## Resumen de la implementación

- BPMN **modela el flujo de negocio**, no lo codifica en secuencias `if/then`; el motor Camunda lo interpreta directamente.  
- Cada tarea del proceso se enlaza a un componente Java (`JavaDelegate`), donde implementas la lógica (validaciones, persistencia, etc.).  
- El flujo queda claro en el diagrama y es fácil de modificar o extender (añadir pasos, paralelismos, temporizadores, etc.) sin mezclarlo con el código de negocio.  
- Camunda proporciona trazabilidad: puedes ver instancias de procesos, tareas pendientes y variables en su interfaz administrativa.  

Con este enfoque, tu microservicio **Academic** se convierte en un *orquestador* de su propio proceso de matrícula, delegando cada paso a componentes especializados. Esto mejora la claridad del código y facilita cambios futuros en el flujo de negocio. 

**Referencias:** BPMN es la notación estándar para procesos de negocio y es ideal para orquestar microservicios, ya que separa el flujo (XML ejecutable) de la lógica de cada servicio. Sigue la documentación de Camunda para más detalles sobre cómo crear y desplegar procesos BPMN en Spring Boot.