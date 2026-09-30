# Post-contenido — Unidad 5: Integración en Aplicaciones Web

## Descripción
Repositorio del post-contenido de la Unidad 5 de Patrones de Diseño
de Software. Un único proyecto Spring Boot (reservas-labs-api) para
la reserva de laboratorios de cómputo, con dos partes: una API REST
en capas (Entity, Repository, Service, Controller) sobre H2, y una
vista Thymeleaf (MVC clásico) que reutiliza el mismo Service.

## Parte 1 — Repository, Service y Controller REST
LaboratorioRepository y ReservaRepository extienden JpaRepository;
ReservaRepository agrega una consulta JPQL propia para detectar
solapamientos de horario. ReservaService concentra las reglas de
negocio (solapamiento, horario de atención, duración, cancelación
tardía). ReservaController y LaboratorioController exponen
/api/reservas y /api/laboratorios. Ver paquetes model/, repository/,
service/, exception/ y controller/.

## Parte 2 — Vista MVC con Thymeleaf
ReservaWebController expone /reservas con Thymeleaf, inyectando la
MISMA instancia de ReservaService que usa la API REST — sin Service
duplicado. ReservaWebExceptionHandler maneja las mismas excepciones
de dominio que GlobalRestExceptionHandler, con presentación distinta
(redirección con mensaje en vez de JSON). Ver paquete web/ y
templates/reservas/.

## Cómo ejecutar
```
$ mvn clean package
$ mvn spring-boot:run
```
API REST: http://localhost:8080/api/reservas
Vista MVC: http://localhost:8080/reservas

## Decisiones de diseño

### Punto de decisión 1 — Ubicación de la validación de solapamiento
En el Repository se realiza el filtrado de solapamientos de delega en la consulta JPQL “ReservaRepository.buscarSolapamientos()” ejecutando la evolución directa dentro del motor de base de datos. Evitando la carga de memoria de la aplicación en todas las reservas. Mientras que en el Service en “RservaService” evalua la lista resultante enviada del Repository y lanza las excepciones del dominio “ReservaConflictExcption” con un mensaje. Pero si se usa el Controller para llamar al método “buscarSolapamientos()” se viola la separación de capas y el controlador debe asumir la lógica del dominio, además provoca código duplicado por parte de los controladores REST.

### Punto de decisión 2 — Reglas con y sin apoyo del Repository
En este caso la hora de atención y duración, específicamente en base a las reglas de atención según el horario (si esta dentro o fuera) y las duraciones de las sesiones de laboratorio permitidas (el máximo y mínimo), son campos del objeto Reserva que esta creado, solo se deben realizar validaciones sin mencionar el Repository. Esto es debido que son restricciones intrínsecas del propio objeto, en este caso serian lógica de negocio que se realiza en Service. Pero se podría utilizar con apoyo de Repository si se aplica la validación con otros objetos persistentes en la base de datos, como corroborar que en tal hora no este ocupado el laboratorio.

### Punto de decisión 3 — Cómo comparten Service el Controller MVC y el REST
Realizar inyecciones de clase se debe realizar de forma preventiva ante futuras reglas de negocio (no especulaciones), en este caso se realiza una inyección de “ReservaController” y “ReservaWebController” se permite porque ambos controladores delegan las operaciones de creación y cancelación directamente a un solo servicios. Pero esta regla de inyección de clases no es posible en Service (mas riesgo de duplicidad y cambios severos), donde si se duplica un Service (ósea una clase) y no se abarcaron cambios a futuros el trabajo de ajustes será el doble debido al doble Service existente y posibles riesgos de discrepancias entre la API REST y la interfaz web.

### Punto de decisión 4 — Manejo de errores consistente entre MVC y REST
Poseer un vocabulario común de dominio frente a excepciones esperadas por la capa Service se considera resultado de un análisis completo de la lógica de negocio, no necesariamente debe ser erróneo sino mas bien conocer los posibles fallos o errores dentro de los criterios e intenciones. Ahora en este contexto de manejas las excepciones de manera separada para separar las diferentes excepciones esperadas de la lógica condicional en base a las clases que toman como referencia

## Herramientas utilizadas
- Java 17, Spring Boot 3.2, Spring Data JPA, H2, Thymeleaf
- Apache Maven, Postman/curl, Git, GitHub

## Conclusiones
Se comprendio de forma mas agil el desarrollo o la usabilidad de la capa de Service, la diferencia o aplicacion del Repository para condicionamientos de objetos (saber utilizarlo y cuando no) y el como instalar la version deseada de Spring Boot para Java. Lo que se dificulto fue el como distribuir los archivos y carpetas, al igual que los comandos de terminal para su ejecucion.