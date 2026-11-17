
# Comprueba tus conocimientos — Desarrollo de aplicaciones con Spring Boot

## Instrucciones

1. Responde individualmente cada pregunta.
2. Justifica las decisiones técnicas cuando el enunciado lo solicite.
3. En las preguntas de selección múltiple, identifica la cantidad de respuestas indicada.
4. En la corrección de código, identifica el problema, presenta la solución y explica los cambios.
5. En los ejercicios de completar código, utiliza sintaxis válida.
6. En los casos profesionales, describe una solución que pueda implementarse y verificarse.
7. Se valorará el razonamiento técnico, no solamente la memorización de conceptos.
8. Utiliza identificadores en inglés para clases, métodos, variables y archivos.

---

# Sección A — Selección múltiple

**Valor: 3 puntos por pregunta.**

## Pregunta 1. Operaciones HTTP en una empresa de logística

Una empresa de mensajería desarrolla una aplicación web para consultar el estado de sus entregas.

Cada entrega tiene un identificador único, una dirección de destino y un estado actual.

El equipo necesita definir una operación que permita consultar la información de una entrega sin modificar sus datos.

**¿Cuál de las siguientes operaciones es la más apropiada?**

Selecciona una respuesta.

A. `POST /deliveries/145` con la intención de consultar la entrega.

B. `GET /deliveries/145` para recuperar la información de la entrega.

C. `GET /deliveries/145/delete` para consultar y eliminar una entrega.

D. `PUT /deliveries/145` enviando el identificador para obtener la información.

## Pregunta 2. Integridad de datos en una plataforma comercial

Una plataforma de comercio electrónico almacena información de proveedores y órdenes de compra en MariaDB.

Cada orden de compra pertenece a un proveedor.

El sistema debe evitar que una operación deje órdenes con referencias inválidas a proveedores.

**¿Cuáles de las siguientes medidas contribuyen a mantener la integridad de los datos?**

Selecciona dos respuestas.

A. Definir una clave foránea que relacione cada orden con su proveedor.

B. Aplicar controles de negocio antes de ejecutar operaciones que puedan afectar registros relacionados.

C. Eliminar automáticamente todas las órdenes cada vez que se modifica el nombre de un proveedor.

D. Eliminar las restricciones de la base de datos para permitir cualquier modificación.

## Pregunta 3. Acceso a información administrativa

Una empresa utiliza Spring Security para proteger una aplicación de gestión interna.

Un empleado ha iniciado sesión correctamente, pero intenta acceder a una sección reservada a administradores.

El usuario está autenticado y no tiene el rol requerido.

**¿Cuál es la respuesta HTTP más apropiada cuando se rechaza ese acceso por falta de permisos?**

Selecciona una respuesta.

A. `200 OK`

B. `201 Created`

C. `403 Forbidden`

D. `500 Internal Server Error`

## Pregunta 4. Reutilización de interfaces web

Un equipo desarrolla una aplicación Spring MVC con Thymeleaf.

La interfaz contiene varias páginas y todas deben mostrar el mismo encabezado y menú de navegación.

El equipo quiere evitar duplicaciones y facilitar cambios posteriores.

**¿Cuáles de las siguientes decisiones son apropiadas?**

Selecciona dos respuestas.

A. Extraer las partes comunes a fragmentos reutilizables de Thymeleaf.

B. Copiar manualmente toda la navegación en cada archivo HTML y modificar cada copia cuando cambie.

C. Mantener los recursos CSS compartidos separados del contenido dinámico de las vistas.

D. Implementar toda la estructura visual dentro de los repositorios JPA.

---

# Sección B — Análisis y explicación

**Valor: 5 puntos por pregunta.**

Responde utilizando tus propias palabras y ejemplos cuando sea necesario.

## Pregunta 5. Arquitectura de una aplicación Spring Boot

Durante los laboratorios desarrollaste PetMatch Mini, una aplicación que comenzó utilizando datos almacenados en memoria y posteriormente incorporó persistencia con MariaDB.

Imagina que ahora otro equipo debe mantener y ampliar la aplicación.

**Explica por qué es importante separar las responsabilidades del controlador, el servicio y el repositorio.**

En tu respuesta analiza:

1. Qué problemas aparecen cuando toda la lógica se concentra en una única clase.
2. Qué ventajas ofrece separar las reglas de negocio del acceso a datos.
3. Qué cambios deberían producirse si la aplicación necesita utilizar otro mecanismo de persistencia.
4. Cómo influye esta separación en las pruebas y el mantenimiento del proyecto.

No es necesario escribir código.

## Pregunta 6. Diagnóstico de una conexión a base de datos

Una empresa desarrolla un sistema de inventarios con Spring Boot y MariaDB.

La aplicación funciona correctamente en el equipo de un desarrollador, pero falla cuando se ejecuta en otro computador.

En los registros aparece un error de conexión a la base de datos.

El equipo sabe que el servidor MariaDB está instalado, pero todavía no ha identificado la causa del problema.

**Describe cómo investigarías esta situación.**

Incluye:

1. Qué elementos comprobarías antes de modificar el código Java.
2. Cómo diferenciarías un problema de conectividad de uno de autenticación.
3. Qué revisarías en la configuración de Spring Boot.
4. Cómo evitarías exponer información sensible durante el diagnóstico.
5. Qué comprobaciones realizarías para confirmar que el problema quedó solucionado.

## Pregunta 7. Calidad y pruebas de software

Una tienda virtual tiene un servicio encargado de procesar devoluciones de productos.

El servicio debe verificar las condiciones de una devolución, modificar el estado correspondiente y conservar los cambios.

Actualmente, el equipo considera que abrir la aplicación y realizar una devolución manualmente es suficiente para comprobar su funcionamiento.

**Explica por qué esta estrategia es insuficiente y cómo mejorarías el proceso de pruebas.**

En tu respuesta diferencia las responsabilidades de:

- Pruebas unitarias.
- Pruebas de integración.
- Pruebas del comportamiento HTTP.

Propón al menos tres casos que deberían comprobarse, incluyendo situaciones de error.

## Pregunta 8. Diagnóstico de errores durante el desarrollo

Un aprendiz trabaja en una aplicación con Java y Spring Boot.

Durante diferentes ejecuciones observa estos problemas:

**Situación A:** Maven no puede compilar el proyecto debido a una referencia a una clase inexistente.

**Situación B:** El proyecto compila, pero Spring Boot no consigue iniciar el contexto de aplicación porque no puede crear uno de sus componentes.

**Situación C:** La aplicación inicia correctamente, pero al visitar una dirección determinada el navegador recibe una respuesta `404`.

**Situación D:** La aplicación abre correctamente, pero una operación produce una excepción durante su ejecución.

**Pregunta:**

Explica en qué etapa ocurre cada problema y cómo comenzarías a investigarlo.

Indica qué información o herramientas utilizarías para encontrar la causa sin modificar el código de manera aleatoria.

---

# Sección C — Diagnóstico y corrección de código

**Valor: 6 puntos por pregunta.**

Cada ejercicio contiene uno o varios problemas intencionales.

Debes identificar los errores, escribir el fragmento corregido y justificar las modificaciones.

Puedes asumir que las clases y dependencias mencionadas existen, salvo cuando el problema del ejercicio indique lo contrario.

## Pregunta 9. Registro de conductores mediante formularios

Una empresa de transporte utiliza Spring MVC para registrar conductores.

El controlador tiene `@RequestMapping("/drivers")` a nivel de clase y dispone de un formulario `DriverForm` con restricciones de Bean Validation.

La vista del formulario se encuentra en `templates/drivers/form.html`.

Actualmente, el método de registro es el siguiente:

```java
@PostMapping
public String create(
    @Valid @ModelAttribute("driverForm") DriverForm form,
    Model model,
    BindingResult bindingResult
) {
    driverService.create(form);

    return "redirect:/drivers";
}
```

**Actividad:**

Corrige el método para que gestione apropiadamente los datos recibidos y los errores de entrada.

Tu solución debe impedir que se ejecute el registro cuando la validación no sea satisfactoria.

Explica los problemas presentes en el método original y las consecuencias que podrían producir en la aplicación.

## Pregunta 10. Consulta de solicitudes de soporte

Una plataforma empresarial permite que sus clientes registren solicitudes de soporte técnico.

La entidad `SupportTicket` tiene una asociación llamada `customer` que identifica al cliente al que pertenece.

El servicio contiene el siguiente método:

```java
@Transactional(readOnly = true)
public SupportTicket findForCustomer(
    Long customerId,
    Long ticketId
) {
    return ticketRepository.findById(ticketId)
        .orElseThrow(() ->
            new ResourceNotFoundException(
                "Ticket not found"
            )
        );
}
```

El equipo detectó que el método puede devolver una solicitud aunque no pertenezca al cliente indicado.

**Actividad:**

Corrige la implementación utilizando las capacidades de consulta de Spring Data JPA.

Presenta los cambios que consideres necesarios en el repositorio y en el servicio.

Explica por qué el método original no cumple el contrato esperado y cómo comprobarías que el problema fue solucionado.

Puedes asumir que `customerId` fue obtenido o validado previamente en el contexto de acceso autorizado.

## Pregunta 11. Configuración de seguridad de una intranet

Una empresa tiene una aplicación web con autenticación mediante formulario y sesiones HTTP.

Las rutas `/public/**` deben ser públicas, mientras que `/admin/**` están reservadas a administradores. Las demás rutas requieren autenticación.

El siguiente fragmento pertenece a su configuración de Spring Security:

```java
@Bean
public SecurityFilterChain securityFilterChain(
    HttpSecurity http
) throws Exception {
    http
        .csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/public/**").permitAll()
            .requestMatchers("/admin/**").permitAll()
            .anyRequest().authenticated()
        )
        .formLogin(Customizer.withDefaults());

    return http.build();
}
```

**Actividad:**

Identifica los problemas de seguridad que presenta esta configuración.

Corrige el código para respetar la política de acceso descrita y conservar las protecciones apropiadas para una aplicación web basada en sesiones.

Justifica las modificaciones realizadas.

## Pregunta 12. Consulta de productos desde una API

Una empresa de almacenamiento tiene una API REST para consultar productos.

El equipo encuentra el siguiente método:

```java
@GetMapping("/{itemId}")
public Item detail(@PathVariable Long itemId) {
    return itemService.findById(itemId)
        .orElse(null);
}
```

El contrato público de la API establece que las consultas deben devolver representaciones `ItemResponse`, obtenidas mediante `ItemApiMapper`, y que los productos inexistentes deben recibir una respuesta HTTP apropiada.

Existe un manejador global que convierte `ResourceNotFoundException` en una respuesta `404`.

**Actividad:**

Corrige el método respetando el contrato de la API.

Explica los problemas que presenta la implementación original y por qué esos problemas pueden afectar a los clientes que consumen el servicio.

---

# Sección D — Completar código

**Valor: 5 puntos por pregunta.**

Completa los espacios en blanco. No cambies innecesariamente la estructura existente.

## Pregunta 13. Construcción de un servicio empresarial

Una empresa de distribución utiliza un servicio para calcular cotizaciones de transporte.

El cálculo depende de un componente llamado `RateRepository`.

Completa el constructor:

```java
@Service
public class PricingService {

    private final RateRepository rateRepository;

    public PricingService(
        ______________________________
    ) {
        this.rateRepository = __________________;
    }
}
```

Después de completar el código, explica brevemente una ventaja de la estrategia utilizada para proporcionar esta dependencia.

## Pregunta 14. Presentación de datos en Thymeleaf

Un hotel tiene una página que muestra sus reservas.

El controlador ya incorporó al modelo una colección denominada `bookings`.

Cada elemento contiene las propiedades `code` y `guestName`.

Completa los atributos Thymeleaf necesarios para recorrer y mostrar los datos:

```html
<table>
    <tbody>
        <tr th:____________="booking : ${bookings}">
            <td th:____________="${booking.code}">
                Booking code
            </td>
            <td th:____________="${booking.guestName}">
                Guest name
            </td>
        </tr>
    </tbody>
</table>
```

Explica qué ocurriría si la colección estuviera vacía.

## Pregunta 15. Configuración de conexión a MariaDB

Un sistema de inventario utiliza Spring Boot y MariaDB.

Las credenciales y la dirección de conexión se proporcionan mediante variables de entorno.

Completa el archivo `application.yaml` utilizando los nombres de variables trabajados en los laboratorios.

```yaml
spring:
  application:
    name: inventory-service

  datasource:
    url: ${____________________}
    username: ${____________________}
    password: ${____________________}

  jpa:
    open-in-view: false
```

Después de completar la configuración, responde:

¿Qué consecuencias podría tener incluir directamente las credenciales reales dentro de un archivo que posteriormente se publica en GitHub?

## Pregunta 16. Asociación entre entidades JPA

Un sistema de facturación almacena órdenes y sus líneas de detalle.

Cada línea de una orden pertenece a una única orden, mientras que una orden puede contener varias líneas.

La clase `OrderLine` ya tiene sus demás atributos correctamente implementados.

Completa las anotaciones de la siguiente asociación:

```java
@Entity
@Table(name = "order_lines")
public class OrderLine {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @____________________(fetch = FetchType.LAZY)
    @____________________(
        name = "order_id",
        nullable = false
    )
    private Order order;
}
```

Después responde:

¿Qué función cumple la columna `order_id` dentro del modelo relacional?

---

# Sección E — Resolución de problemas profesionales

**Valor: 6 puntos por pregunta.**

Analiza cada situación y plantea una solución técnica.

No es necesario desarrollar una aplicación completa. Sin embargo, tu propuesta debe explicar las modificaciones necesarias y cómo comprobarías que funcionan.

Puedes utilizar fragmentos de código, nombres de métodos o estructuras cuando aporten claridad.

## Pregunta 17. Sistema de seguimiento de entregas

### Contexto profesional

Una empresa de transporte tiene una aplicación web desarrollada con Spring Boot, Thymeleaf y MariaDB.

La aplicación almacena cientos de entregas.

Cada entrega contiene:

- Identificador.
- Dirección de destino.
- Fecha de registro.
- Estado de entrega.

Los operadores necesitan consultar las entregas por estado, sin tener que revisar manualmente el listado completo.

La aplicación ya dispone de una entidad `Delivery` y de las operaciones básicas para consultar entregas.

### Requerimiento

Implementar un filtro que permita seleccionar un estado y visualizar únicamente las entregas correspondientes.

### Pregunta

**¿Cómo implementarías esta funcionalidad utilizando la arquitectura existente?**

En tu propuesta identifica:

1. Los cambios necesarios en el acceso a datos.
2. Las responsabilidades del servicio.
3. Cómo recibirías el criterio de búsqueda.
4. Cómo presentarías los resultados en Thymeleaf.
5. Qué debería suceder si no existen entregas para el estado seleccionado.
6. Qué comprobaciones realizarías para verificar el funcionamiento.

Justifica por qué tu propuesta mantiene una separación adecuada de responsabilidades.

## Pregunta 18. Operación de negocio con múltiples cambios

### Contexto profesional

En PetMatch Mini se gestionan solicitudes de apoyo para mascotas.

Una solicitud puede encontrarse en los estados:

- `OPEN`
- `COMPLETED`
- `CANCELLED`

Una funcionalidad permite reemplazar una solicitud abierta por otra.

Para que el reemplazo sea correcto, la solicitud anterior debe dejar de estar abierta y la nueva solicitud debe quedar registrada correctamente.

### Situación problemática

Un desarrollador implementa ambos cambios mediante operaciones independientes.

Durante una ejecución, el primer cambio se guarda correctamente, pero el segundo falla.

La aplicación conserva entonces un resultado incompleto.

### Pregunta

**¿Cómo reorganizarías esta operación para conservar la consistencia de los datos?**

Explica:

1. Dónde debería implementarse la operación completa.
2. Cómo organizarías los cambios necesarios.
3. Cómo manejarías los errores.
4. Qué debería ocurrir con los cambios previos cuando falla una parte de la operación.
5. Qué mecanismos utilizarías para garantizar el comportamiento esperado.
6. Cómo probarías el caso exitoso y los casos de fallo.

Tu propuesta debe respetar las reglas de estados establecidas por el dominio.

## Pregunta 19. Registro seguro de empleados en una aplicación

### Contexto profesional

Una empresa necesita incorporar un módulo de registro e inicio de sesión para su sistema interno de gestión.

La aplicación está construida con Spring Boot, Thymeleaf y MariaDB.

Existen dos tipos de cuenta:

- `EMPLOYEE`: utiliza funcionalidades operativas.
- `ADMIN`: administra usuarios y configuraciones.

El registro ordinario debe crear cuentas de empleado. Las cuentas administrativas se gestionarán mediante un procedimiento interno autorizado.

### Requerimiento

Diseñar una solución que permita registrar cuentas, verificar sus credenciales e iniciar sesión de forma segura.

### Pregunta

**¿Cómo implementarías esta funcionalidad utilizando Spring Security y las tecnologías estudiadas?**

Describe:

1. Qué información almacenarías para representar una cuenta.
2. Cómo recibirías y validarías los datos del registro.
3. Cómo protegerías las contraseñas almacenadas.
4. Cómo impedirías que un empleado se registre asignándose privilegios administrativos.
5. Cómo integrarías las cuentas persistidas con el proceso de autenticación.
6. Qué pruebas utilizarías para verificar registros válidos, entradas incorrectas y accesos no autorizados.

No es necesario desarrollar toda la interfaz, pero sí identificar los componentes principales de la solución.

## Pregunta 20. Integración mediante una API REST

### Contexto profesional

Una empresa de electrodomésticos tiene una aplicación Spring Boot para administrar productos y solicitudes de garantía.

Actualmente, los empleados registran las solicitudes desde una interfaz web con Thymeleaf.

La empresa necesita permitir que una aplicación móvil registre solicitudes de garantía utilizando JSON.

La aplicación ya tiene implementados:

- Una entidad `Product`.
- Una entidad `WarrantyClaim`.
- Persistencia con Spring Data JPA.
- Un servicio `WarrantyService` con las reglas de registro.
- Una interfaz web funcional.

La empresa no desea mantener dos implementaciones independientes de las reglas de negocio.

### Requerimiento

Incorporar una operación:

`POST /api/v1/products/{productId}/warranties`

La operación debe recibir los datos necesarios para registrar una solicitud de garantía y devolver una respuesta que permita identificar el recurso creado.

### Pregunta

**¿Cómo diseñarías e implementarías esta funcionalidad?**

Tu propuesta debe abordar:

1. El contrato JSON de entrada y salida.
2. El componente encargado de recibir la petición HTTP.
3. La reutilización de las reglas de negocio existentes.
4. El tratamiento de datos inválidos y productos inexistentes.
5. La respuesta HTTP para una solicitud registrada correctamente.
6. Las consideraciones de autenticación y autorización del endpoint.
7. Las pruebas necesarias para verificar el contrato y el comportamiento.

Explica cómo evitarías que la nueva interfaz móvil obligue a duplicar la lógica ya implementada para la aplicación web.
