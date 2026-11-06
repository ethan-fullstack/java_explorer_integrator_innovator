# Laboratorio 12 — Diseño e implementación de una API REST con Spring Boot, JSON y seguridad

Hasta ahora, **PetMatch Mini** funciona como una aplicación web tradicional: una persona utiliza el navegador, Spring MVC procesa la petición, los servicios ejecutan las reglas de negocio y Thymeleaf construye una página HTML. La aplicación también conserva la información en MariaDB, mantiene las relaciones entre mascotas y solicitudes de apoyo y protege las operaciones mediante Spring Security.

Pero no todos los clientes de una aplicación necesitan recibir páginas HTML. Una aplicación móvil, un programa de escritorio, un cliente HTTP o una segunda interfaz web pueden necesitar **datos estructurados** para construir sus propias pantallas. Para estos casos resulta útil ofrecer una **API HTTP que envíe y reciba JSON**.

En este laboratorio incorporarás una **API REST** a la misma aplicación. **No crearás otro proyecto, otra base de datos ni una segunda copia de las reglas de negocio**. Construirás controladores REST que utilizan los servicios existentes, aprenderás a expresar resultados y errores con códigos HTTP, separarás los objetos persistentes de los datos públicos y configurarás una política de seguridad específica para la API.

Se mantiene la base técnica de los laboratorios anteriores: **Java 21, Spring Boot 4.1.1, Spring MVC, Spring Security 7.1.2, Jakarta Validation, Spring Data JPA, Hibernate y MariaDB**. Las páginas web seguirán utilizando **Thymeleaf y sesiones HTTP**. La nueva API utilizará **HTTP Basic sin sesiones de autenticación** como mecanismo pedagógico, y sus solicitudes enviarán las credenciales en cada petición. **No implementaremos JWT, OAuth2, Swagger, GraphQL ni un servidor independiente**.

Al terminar, una mascota podrá consultarse desde el navegador como HTML y desde un cliente HTTP como JSON, sin que la aplicación mantenga dos conjuntos de reglas para crearla, modificarla o eliminarla.

| Práctica | Aprendizaje principal | Resultado comprobable |
|---|---|---|
| **12.1** | HTTP, recursos, métodos, representaciones y JSON | Interpretar una petición y una respuesta REST sin confundirlas con una página HTML |
| **12.2** | DTO de entrada y salida, validación y mapeo | Crear contratos JSON que no expongan entidades JPA ni información sensible |
| **12.3** | `@RestController`, rutas, `ResponseEntity` y códigos HTTP | Consultar y modificar mascotas y solicitudes reutilizando los servicios existentes |
| **12.4** | Seguridad REST, HTTP Basic, roles, sesiones y CSRF | Separar la política `/api/**` de la política web sin eliminar los controles anteriores |
| **12.5** | Errores JSON, uso de `curl` y análisis de respuestas | Distinguir errores de entrada, recursos ausentes, conflictos y rechazos de seguridad |
| **12.6** | Pruebas automatizadas y reto integrador | Verificar contratos, autenticación, autorización, validación y persistencia |

> **Cómo utilizar los fragmentos:** cuando se indique **archivo completo**, crea ese archivo en la ruta indicada. Si se indica **sustituir**, modifica exclusivamente el elemento señalado del archivo existente. Los ejemplos **conceptuales** sirven para explicar un comportamiento y no deben copiarse como si fueran una implementación completa. El código, sus identificadores y comentarios están escritos en inglés; las explicaciones están en español.

---

## Preparación — Reconocer lo que ya existe

### 1. No empezar desde cero

La aplicación debería contener una estructura equivalente a esta:

```text
src/main/java/com/example/petmatchlab/
├── controller/
│   ├── HomeController.java
│   ├── PetController.java
│   ├── SupportRequestController.java
│   ├── RegistrationController.java
│   ├── LoginController.java
│   ├── AdminController.java
│   └── PageErrorHandler.java
├── dto/
│   ├── PetForm.java
│   ├── SupportRequestForm.java
│   └── RegistrationForm.java
├── exception/
│   ├── BusinessRuleException.java
│   └── ResourceNotFoundException.java
├── model/
│   ├── Pet.java
│   ├── SupportRequest.java
│   ├── SupportType.java
│   ├── RequestStatus.java
│   ├── AppUser.java
│   └── UserRole.java
├── repository/
│   ├── PetRepository.java
│   ├── SupportRequestRepository.java
│   └── AppUserRepository.java
├── security/
│   ├── SecurityConfig.java
│   └── DatabaseUserDetailsService.java
└── service/
    ├── PetService.java
    ├── SupportRequestService.java
    └── RegistrationService.java
```

Además, debe conservar los recursos `templates/`, el archivo `application.yaml`, las pruebas en `src/test/` y el archivo `pom.xml` con las dependencias utilizadas en los laboratorios anteriores.

**No elimines la interfaz MVC** al agregar la API. Las rutas `/pets` y `/pets/{id}/requests` continuarán produciendo HTML, mientras que `/api/v1/pets` y las demás rutas nuevas producirán JSON.

### 2. Comprobar el estado de la aplicación

Antes de hacer cambios:

1. Inicia la aplicación utilizando las variables de entorno de MariaDB.
2. Comprueba el registro y el inicio de sesión de una cuenta `USER`.
3. Comprueba que una cuenta `ADMIN` puede ejecutar una operación administrativa de la web.
4. Abre una mascota y consulta sus solicitudes de apoyo.
5. Comprueba que no se pueden completar solicitudes canceladas.
6. Ejecuta la suite de pruebas existente y registra cualquier fallo previo.

En Linux o macOS:

```bash
./mvnw test
```

En Windows PowerShell:

```powershell
.\mvnw.cmd test
```

**Si una prueba anterior falla, distingue ese problema de los cambios que realizarás ahora.** No desactives controles de seguridad ni borres pruebas para conseguir una ejecución aparentemente correcta.

### 3. Revisar las dependencias

El proyecto ya dispone de Spring MVC, Spring Security, Jackson mediante el soporte web de Spring Boot, Jakarta Validation, JPA y soporte de pruebas. No necesitas instalar un *starter* exclusivo de REST ni sustituir Thymeleaf.

Verifica que `pom.xml` incluye estas dependencias, **sin duplicarlas**:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

Conserva también las dependencias de Thymeleaf, Spring Data JPA, MariaDB, H2 y las pruebas de persistencia que ya existen. El `spring-boot-starter-parent` administra las versiones compatibles, por lo que normalmente no debes escribir versiones individuales para las dependencias administradas.

**No agregues `spring-boot-starter-web` antiguo ni reemplaces `spring-boot-starter-webmvc` simplemente porque un tutorial de otra versión lo muestre.** La implementación de este laboratorio corresponde al conjunto utilizado en Spring Boot 4.1.1.

### Diagnóstico inicial

Contesta estas preguntas antes de avanzar:

1. ¿Qué hace `PetService.create(...)` que no debería repetirse en un controlador REST?
2. ¿Por qué `Pet` es una entidad JPA y `PetForm` no lo es?
3. ¿Qué diferencia hay entre el rol `USER` y el rol `ADMIN`?
4. ¿Por qué una solicitud `CANCELLED` no puede completarse aunque el usuario tenga permiso administrativo?
5. ¿Por qué la contraseña codificada de `AppUser` nunca debería aparecer en una respuesta HTTP?
6. ¿Qué diferencia hay entre la página `/pets` y un futuro recurso `/api/v1/pets`?

---

## Práctica 12.1 — Comprender HTTP, REST y JSON

### Objetivos

Al completar esta práctica podrás:

- identificar método, ruta, cabeceras, cuerpo y código de respuesta;
- distinguir un recurso de su representación;
- explicar para qué se utilizan `GET`, `POST`, `PUT` y `DELETE`;
- interpretar JSON con tipos y estructuras;
- elegir un código HTTP adecuado para una operación.

### 1. Una petición HTTP no siempre solicita HTML

La web que ya conoces recibe peticiones como:

```http
GET /pets HTTP/1.1
Host: localhost:8080
Accept: text/html
```

El servidor puede responder:

```http
HTTP/1.1 200 OK
Content-Type: text/html
```

A continuación envía un documento HTML. El navegador interpreta etiquetas como `<main>`, `<ul>` y `<a>` para mostrar la página.

Un cliente que no necesita dibujar HTML puede solicitar los datos del mismo conjunto de mascotas:

```http
GET /api/v1/pets HTTP/1.1
Host: localhost:8080
Accept: application/json
Authorization: Basic <credentials>
```

La respuesta podría tener este aspecto:

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

```json
[
  {
    "id": 5,
    "name": "Luna",
    "species": "Dog",
    "age": 3,
    "description": "Enjoys walks."
  },
  {
    "id": 8,
    "name": "Milo",
    "species": "Cat",
    "age": 2,
    "description": null
  }
]
```

**Estos son ejemplos conceptuales:** los identificadores y los registros reales dependen de tu MariaDB. `<credentials>` representa una cabecera construida por el cliente y no debe escribirse literalmente como contraseña.

Ambas rutas trabajan sobre mascotas, pero **entregan representaciones diferentes**. Una representación es la forma en que un recurso se comunica a un cliente. La entidad JPA que vive en memoria no tiene que ser idéntica al JSON recibido.

### 2. Leer una petición como un contrato

Toda petición HTTP tiene componentes que cumplen funciones diferentes:

| Componente | Ejemplo | Pregunta que responde |
|---|---|---|
| Método | `GET` | ¿Qué operación se solicita? |
| Ruta | `/api/v1/pets/5` | ¿Sobre qué recurso? |
| Cabecera `Accept` | `application/json` | ¿Qué formato de respuesta admite el cliente? |
| Cabecera `Content-Type` | `application/json` | ¿Qué formato tiene el cuerpo enviado? |
| Cabecera `Authorization` | `Basic ...` | ¿Qué credenciales presenta la petición? |
| Cuerpo | `{"name":"Luna"}` | ¿Qué datos entrega el cliente? |

`Accept` y `Content-Type` no significan lo mismo. Un `POST` puede indicar que **envía JSON** mediante `Content-Type: application/json` y que **espera JSON** mediante `Accept: application/json`.

Si el contenido enviado no corresponde al tipo esperado, la aplicación puede responder `415 Unsupported Media Type`. Si la respuesta no puede producirse en un formato aceptado, puede responder `406 Not Acceptable`.

### 3. JSON no es un objeto Java

JSON significa *JavaScript Object Notation*, pero puede utilizarse desde distintos lenguajes. Una estructura JSON contiene valores simples, objetos y arreglos.

Ejemplo:

```json
{
  "name": "Luna",
  "species": "Dog",
  "age": 3,
  "description": null,
  "available": true,
  "tags": ["friendly", "small"]
}
```

En este ejemplo:

- `"Luna"` es una cadena de texto.
- `3` es un número, no la cadena `"3"`.
- `null` indica que no se proporciona un valor.
- `true` es un booleano.
- `tags` contiene un arreglo.
- Las propiedades de un objeto se escriben entre comillas dobles.

**JSON no permite comentarios ni comas finales**. Esta entrada es inválida:

```text
{
  "name": "Luna", // This is not valid JSON.
}
```

El servidor debe distinguir entre un cuerpo **ilegible** y un cuerpo **bien formado pero inválido para el dominio**. Más adelante ambos producirán `400 Bad Request`, aunque se identificarán mediante códigos de error diferentes.

### 4. Recursos y URI

Para trabajar con mascotas, utilizaremos estas rutas:

```text
/api/v1/pets
/api/v1/pets/{petId}
/api/v1/pets/{petId}/requests
/api/v1/pets/{petId}/requests/{requestId}
```

Los nombres en plural representan colecciones. Un identificador individualiza un elemento de la colección. Las solicitudes están anidadas porque cada `SupportRequest` pertenece a una mascota concreta.

`/api/v1` es un **prefijo de versión del contrato HTTP**, no una versión de Java ni del proyecto Maven. Permite introducir cambios incompatibles en una versión futura sin sustituir de inmediato los clientes anteriores. Es una estrategia de diseño; no crea automáticamente dos versiones funcionales.

### 5. Los métodos HTTP expresan intención

| Método | Intención habitual | Ejemplo |
|---|---|---|
| `GET` | Consultar sin modificar deliberadamente el recurso | Obtener una mascota |
| `POST` | Crear un recurso o ejecutar una acción definida por el servidor | Registrar una mascota |
| `PUT` | Reemplazar el estado representado de un recurso conocido | Actualizar los datos de una mascota |
| `DELETE` | Solicitar la eliminación de un recurso | Eliminar una mascota |

`GET` es un método **seguro** en el sentido HTTP: un cliente no debería utilizarlo para solicitar cambios de estado. Puede producir efectos auxiliares, como registros de acceso, pero no debería completar una solicitud de apoyo al consultar su detalle.

`PUT` y `DELETE` se consideran métodos **idempotentes** en términos de su efecto deseado: repetir una solicitud idéntica tiene, idealmente, el mismo efecto final que ejecutarla una vez. Eso **no significa que todas las respuestas repetidas deban tener el mismo código HTTP**. Por ejemplo, eliminar una mascota puede devolver primero `204` y después `404`.

`POST` no es idempotente de manera general: enviar dos veces una creación podría producir dos recursos. En PetMatch Mini, la regla de negocio sobre solicitudes abiertas puede impedir algunos duplicados, pero no transforma todas las peticiones `POST` en idempotentes.

### 6. ¿Por qué completar y cancelar utilizan POST?

En el laboratorio anterior, `SupportRequest` definió operaciones explícitas:

```java
request.complete();
request.cancel();
```

No existe una operación genérica que permita introducir cualquier estado desde el navegador. Lo mismo ocurrirá en la API:

```text
POST /api/v1/pets/5/requests/12/complete
POST /api/v1/pets/5/requests/12/cancel
```

Son **acciones del dominio**, no un permiso para escribir libremente `status = "COMPLETED"` en JSON. Estas rutas son una decisión REST pragmática: privilegian un contrato claro y preservan las transiciones ya implementadas en el servicio.

### 7. Interpretar códigos HTTP

| Código | Nombre | Uso en este laboratorio |
|---|---|---|
| `200` | OK | Consulta o actualización exitosa con respuesta JSON |
| `201` | Created | Se creó un recurso; se envía la ubicación del nuevo recurso |
| `204` | No Content | Operación exitosa sin cuerpo de respuesta |
| `400` | Bad Request | JSON ilegible o datos que no cumplen el contrato de entrada |
| `401` | Unauthorized | Faltan credenciales válidas para la API |
| `403` | Forbidden | El usuario autenticado no tiene el permiso necesario |
| `404` | Not Found | No existe el recurso solicitado o no pertenece a la ruta indicada |
| `409` | Conflict | El estado actual del recurso impide la operación |

`401` y `403` no son intercambiables. Una cuenta válida con rol `USER` que intenta crear mascotas mediante una ruta reservada a `ADMIN` recibe `403`. Un cliente sin credenciales para una ruta protegida por HTTP Basic recibe `401`.

### 8. Una creación correcta necesita ubicación

Una solicitud de creación puede responder:

```http
HTTP/1.1 201 Created
Location: http://localhost:8080/api/v1/pets/15
Content-Type: application/json
```

```json
{
  "id": 15,
  "name": "Kira",
  "species": "Dog",
  "age": 4,
  "description": null
}
```

La cabecera `Location` identifica la URL del recurso creado. El cuerpo muestra su representación. **No son equivalentes**: un cliente puede seguir `Location` para consultar el recurso posteriormente.

### 9. ¿REST requiere una base de datos nueva?

No. La diferencia relevante se encuentra en la capa de transporte y presentación:

```text
Web HTML                           API JSON
---------                          --------
Navegador                          Cliente HTTP
    ↓                                  ↓
PetController                    PetRestController
    ↓                                  ↓
              PetService
                  ↓
              PetRepository
                  ↓
                MariaDB
```

Los controladores son diferentes porque construyen respuestas diferentes. Los servicios y repositorios no deben duplicarse por esa razón.

### Ejercicio — Predice la respuesta

Antes de consultar la tabla de códigos, razona qué respuesta correspondería a cada situación:

1. Una persona autenticada solicita una mascota existente.
2. Se intenta crear una mascota sin su nombre.
3. Una cuenta `USER` intenta eliminar una mascota.
4. Una cuenta `ADMIN` intenta completar una solicitud `CANCELLED`.
5. Un cliente sin credenciales intenta obtener la lista de mascotas.
6. Se elimina correctamente una mascota que no tiene solicitudes relacionadas.
7. El cliente intenta acceder a una solicitud usando el identificador de otra mascota.

**Comprobación:** puedes explicar la diferencia entre un recurso, una representación, un método HTTP y un código de respuesta.

---
## Práctica 12.2 — Separar entidades JPA, contratos JSON y mapeadores

### Objetivos

Al completar esta práctica podrás:

- explicar qué es un DTO y por qué se utiliza en una API;
- definir objetos de entrada y salida sin exponer entidades;
- validar las entradas JSON con Jakarta Validation;
- convertir datos recibidos en los formularios que los servicios ya conocen;
- comprobar que un cliente no puede elegir identificadores ni estados protegidos.

### 1. El problema de devolver una entidad directamente

La clase `Pet` es una entidad JPA. Conoce su identidad persistente y participa en operaciones del contexto de persistencia. La clase `SupportRequest` además contiene una asociación `@ManyToOne` con `Pet`, estados y métodos de transición.

Podríamos pensar en escribir lo siguiente:

```java
@GetMapping("/api/v1/pets")
public List<Pet> list() {
    return petService.findAll();
}
```

**Este fragmento es conceptual y no debe incorporarse.** Aunque puede funcionar en un ejemplo muy pequeño, crea un acoplamiento innecesario entre los detalles de persistencia y el contrato público de la API.

Considera las consecuencias:

- Agregar una propiedad a una entidad podría modificar el JSON de forma inesperada.
- Una asociación bidireccional podría producir recorridos circulares durante la serialización.
- Una relación `LAZY` podría causar consultas adicionales o un error de inicialización.
- Los campos internos podrían exponerse por accidente.
- Un cliente podría intentar modificar propiedades que no le corresponden.
- La estructura del esquema de datos quedaría atada a la estructura pública de la API.

El riesgo es especialmente claro en `AppUser`: **nunca debes devolver `passwordHash` a un cliente**. Que una contraseña esté codificada no convierte su hash en información pública.

### 2. Tres modelos con responsabilidades diferentes

Para este laboratorio utilizaremos tres representaciones que cumplen funciones distintas:

```text
JSON enviado por el cliente
         ↓
PetRequest (contrato REST)
         ↓
PetApiMapper
         ↓
PetForm (contrato de entrada del servicio)
         ↓
PetService
         ↓
Pet (entidad JPA)
         ↓
PetApiMapper
         ↓
PetResponse (contrato REST)
         ↓
JSON enviado al cliente
```

`PetRequest` representa los campos permitidos en el cuerpo JSON. `PetForm` ya existe porque el servicio construido para la interfaz HTML recibe un objeto de entrada. `PetResponse` define exactamente qué información expondrá la API.

**¿Por qué mantener `PetForm` y agregar `PetRequest`?** En este ejercicio conservamos el contrato del servicio para no romper la web. Un mapper conecta la representación REST con ese contrato. Así aprendemos a introducir otra interfaz sin duplicar el negocio. En una aplicación más grande se podrían introducir comandos de aplicación independientes de los DTO de cada transporte, pero no necesitamos esa abstracción todavía.

### 3. ¿Qué es un `record` de Java?

Java 21 permite declarar tipos de datos inmutables de forma concisa mediante `record`:

```java
public record ExampleResponse(String name, int count) {
}
```

Un `record` proporciona un constructor y métodos de acceso como `name()` y `count()`. Sus componentes son finales. Esto es útil para representar una respuesta que no necesita setters.

No debes confundir un `record` de respuesta con una entidad JPA. La entidad necesita una identidad persistente y un ciclo de vida administrado; el `record` solo representa los datos que la API entrega o recibe.

### 4. Crear el DTO de entrada de mascotas

Crea la carpeta `src/main/java/com/example/petmatchlab/api/dto/`.

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/dto/PetRequest.java`:

```java
package com.example.petmatchlab.api.dto;

import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

public record PetRequest(
    @NotBlank(message = "A pet name is required")
    @Size(max = 100, message = "The name must have at most 100 characters")
    String name,

    @NotBlank(message = "A species is required")
    @Size(max = 80, message = "The species must have at most 80 characters")
    String species,

    @NotNull(message = "An age is required")
    @Min(value = 0, message = "The age cannot be negative")
    Integer age,

    @Size(max = 1000,
          message = "The description must have at most 1000 characters")
    String description
) {
}
```

La anotación `@NotBlank` rechaza texto nulo, vacío o compuesto únicamente por espacios. `@Size` controla la longitud del texto. `@NotNull` exige que la edad se proporcione; `@Min(0)` prohíbe edades negativas.

Usamos `Integer` y no `int` para detectar una edad ausente mediante `@NotNull`. Un primitivo `int` no puede representar `null`, y un valor faltante podría confundirse con cero.

**No existe `id` ni `status` en este DTO.** La base de datos genera los identificadores y los estados de las solicitudes pertenecen al dominio. El cliente solo envía los datos que necesita la operación.

### 5. Crear el DTO de respuesta de mascotas

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/dto/PetResponse.java`:

```java
package com.example.petmatchlab.api.dto;

public record PetResponse(
    Long id,
    String name,
    String species,
    int age,
    String description
) {
}
```

Una respuesta puede incluir el `id` porque ya representa una mascota guardada. Sin embargo, que el cliente conozca un identificador **no demuestra que tenga permiso** para modificarlo.

Cuando el servidor envíe un `PetResponse`, el conversor JSON de Spring MVC utilizará los componentes del record para construir una representación como:

```json
{
  "id": 15,
  "name": "Kira",
  "species": "Dog",
  "age": 4,
  "description": null
}
```

El JSON de respuesta no necesita conservar el mismo orden de propiedades para ser correcto. Los consumidores deben buscar los campos por su nombre, no por su posición textual.

### 6. Crear el mapeador de mascotas

Crea la carpeta `src/main/java/com/example/petmatchlab/api/mapper/`.

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/mapper/PetApiMapper.java`:

```java
package com.example.petmatchlab.api.mapper;

import org.springframework.stereotype.Component;

import com.example.petmatchlab.api.dto.PetRequest;
import com.example.petmatchlab.api.dto.PetResponse;
import com.example.petmatchlab.dto.PetForm;
import com.example.petmatchlab.model.Pet;

@Component
public class PetApiMapper {

    public PetForm toForm(PetRequest request) {
        PetForm form = new PetForm();
        form.setName(request.name());
        form.setSpecies(request.species());
        form.setAge(request.age());
        form.setDescription(request.description());
        return form;
    }

    public PetResponse toResponse(Pet pet) {
        return new PetResponse(
            pet.getId(),
            pet.getName(),
            pet.getSpecies(),
            pet.getAge(),
            pet.getDescription()
        );
    }
}
```

`@Component` permite que Spring inyecte el mapeador en los controladores. El mapper **no tiene acceso a repositorios ni ejecuta operaciones de negocio**. Solo transforma datos.

Observa que `toForm(...)` no crea ni guarda una mascota. El objeto `PetForm` resultante se entregará a `PetService.create(...)` o `PetService.update(...)`, que ya tienen validaciones y reglas propias.

### 7. Crear el DTO de entrada de solicitudes

`SupportRequest` utiliza los enums existentes `SupportType` y `RequestStatus`. El cliente podrá seleccionar un tipo válido de apoyo, pero **no podrá escoger el estado inicial**.

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/dto/SupportRequestRequest.java`:

```java
package com.example.petmatchlab.api.dto;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

import com.example.petmatchlab.model.SupportType;

public record SupportRequestRequest(
    @NotNull(message = "A support type is required")
    SupportType type,

    @NotBlank(message = "A description is required")
    @Size(max = 500,
          message = "The description must have at most 500 characters")
    String description
) {
}
```

Un cuerpo válido sería:

```json
{
  "type": "WALK",
  "description": "Needs a short walk."
}
```

Los valores admitidos por el enum del Laboratorio 10 son `WALK`, `FEEDING` y `COMPANY`. El valor `VETERINARY` **no existe** en ese modelo. Un JSON que lo contenga no podrá convertirse correctamente al tipo `SupportType` y se rechazará antes de ejecutar el servicio.

**No confundas el valor persistido con la etiqueta de la interfaz:** `WALK` es el identificador técnico del enum; la página HTML puede mostrar «Paseo» mediante `getLabel()`.

### 8. Crear el DTO de respuesta de solicitudes

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/dto/SupportRequestResponse.java`:

```java
package com.example.petmatchlab.api.dto;

import com.example.petmatchlab.model.RequestStatus;
import com.example.petmatchlab.model.SupportType;

public record SupportRequestResponse(
    Long id,
    Long petId,
    SupportType type,
    String description,
    RequestStatus status
) {
}
```

Este objeto contiene dos identificadores con significados diferentes: `id` identifica la solicitud y `petId` indica a qué mascota pertenece. El estado `OPEN`, `COMPLETED` o `CANCELLED` describe el resultado persistido y **no es un campo que el cliente pueda modificar libremente mediante el DTO de entrada**.

Ejemplo de respuesta:

```json
{
  "id": 21,
  "petId": 5,
  "type": "WALK",
  "description": "Needs a short walk.",
  "status": "OPEN"
}
```

### 9. Crear el mapeador de solicitudes

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/mapper/SupportRequestApiMapper.java`:

```java
package com.example.petmatchlab.api.mapper;

import org.springframework.stereotype.Component;

import com.example.petmatchlab.api.dto.SupportRequestRequest;
import com.example.petmatchlab.api.dto.SupportRequestResponse;
import com.example.petmatchlab.dto.SupportRequestForm;
import com.example.petmatchlab.model.SupportRequest;

@Component
public class SupportRequestApiMapper {

    public SupportRequestForm toForm(SupportRequestRequest request) {
        SupportRequestForm form = new SupportRequestForm();
        form.setType(request.type());
        form.setDescription(request.description());
        return form;
    }

    public SupportRequestResponse toResponse(
        SupportRequest request,
        long petId
    ) {
        return new SupportRequestResponse(
            request.getId(),
            petId,
            request.getType(),
            request.getDescription(),
            request.getStatus()
        );
    }
}
```

¿Por qué el mapeador recibe `petId` como argumento y no hace esto?

```java
request.getPet().getName();
```

Porque `SupportRequest.pet` está configurado con `FetchType.LAZY`. Acceder a una asociación de forma innecesaria durante la construcción del JSON podría depender del contexto de persistencia e introducir consultas adicionales. Los métodos de `SupportRequestService` ya verifican la mascota y la pertenencia de cada solicitud; el identificador de la ruta, una vez comprobado por el servicio, proporciona el valor necesario para la respuesta.

**Esto no autoriza confiar en cualquier `petId` enviado por un cliente.** El controller debe invocar primero los métodos del servicio que comprueban la relación. El mapper solo trabaja con datos ya obtenidos y validados por la operación correspondiente.

### 10. Crear el contrato de errores

Además de respuestas exitosas, una API necesita una forma consistente de expresar fallos conocidos.

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/dto/ApiErrorResponse.java`:

```java
package com.example.petmatchlab.api.dto;

import java.util.Map;

public record ApiErrorResponse(
    String code,
    String message,
    Map<String, String> fieldErrors
) {
}
```

Este DTO permitirá construir respuestas como:

```json
{
  "code": "validation_failed",
  "message": "The request contains invalid fields",
  "fieldErrors": {
    "name": "A pet name is required",
    "age": "The age cannot be negative"
  }
}
```

`code` es una etiqueta estable para que el cliente reconozca la categoría del fallo; `message` ofrece una explicación general; `fieldErrors` relaciona nombres de campos con errores de validación. Este formato es **un contrato propio del ejercicio**, no la afirmación de que todas las APIs REST deban usar exactamente esas propiedades.

Spring también admite formatos estandarizados de descripción de problemas, como `ProblemDetail`. En este laboratorio utilizaremos un DTO propio porque permite practicar los mismos principios de entrada, salida y conversión sin añadir otra estructura conceptual desde el principio.

### 11. Entrada y salida son responsabilidades distintas

No utilices una única clase con todos los campos para cada operación por comodidad. En particular, evita una petición como:

```json
{
  "id": 99,
  "petId": 5,
  "type": "WALK",
  "description": "Needs a walk.",
  "status": "COMPLETED"
}
```

Si alguien envía esos campos adicionales a nuestra API de creación, **no se convertirán en parámetros de `SupportRequestService.create(...)`**, porque el DTO declarado solo tiene `type` y `description`. Sin embargo, el comportamiento exacto ante propiedades JSON desconocidas puede depender de la configuración del conversor; **no asumas que todas las propiedades desconocidas siempre se rechazarán**. Para una política estricta se necesitaría configurar explícitamente ese comportamiento.

**La protección real del estado** no consiste en esperar que el cliente coopere: el servicio crea la solicitud con `RequestStatus.OPEN` y las transiciones se realizan por métodos controlados del dominio.

### 12. Ejercicio — Diseña un contrato sin escribir el controlador

Imagina una API que registra cuentas. Identifica qué propiedades deberían aparecer en un DTO de registro, cuáles en una respuesta pública y cuáles **nunca** deberían devolverse.

Piensa especialmente en `password`, `confirmPassword`, `passwordHash`, `role` y `enabled`. Distingue la información necesaria para crear una cuenta de la información que podría mostrarse posteriormente a la propia persona autenticada.

### Depuración — Errores de representación

**Problema:** un cliente envía `"age": "three"`. La cadena no es un número y no puede convertirse a `Integer`. El problema ocurre al interpretar el cuerpo HTTP, antes de aplicar las reglas del servicio.

**Problema:** el cliente envía `"age": -1`. El JSON es sintácticamente válido y puede convertirse a `Integer`, pero incumple `@Min(0)`. La validación debe rechazarlo.

**Problema:** se intenta serializar una asociación `LAZY` desde un controlador REST. Revisa que el controlador devuelva un DTO y no una entidad JPA.

**Problema:** el campo `passwordHash` aparece en un JSON. El contrato de salida está exponiendo información que no necesita el cliente. Retira esa propiedad del DTO y comprueba las respuestas mediante pruebas automáticas.

**Comprobación:** dispones de cinco archivos de DTO —`PetRequest`, `PetResponse`, `SupportRequestRequest`, `SupportRequestResponse`, `ApiErrorResponse`— y dos mapeadores que no ejecutan operaciones de persistencia.

---
## Práctica 12.3 — Construir controladores REST sin duplicar el negocio

### Objetivos

Al completar esta práctica podrás:

- diferenciar `@Controller` de `@RestController`;
- recibir JSON con `@RequestBody` y validar con `@Valid`;
- crear respuestas mediante `ResponseEntity`;
- devolver `200`, `201` y `204` según la operación;
- reutilizar `PetService` y `SupportRequestService` sin modificar sus reglas;
- relacionar las rutas REST con los mismos datos de MariaDB.

### 1. ¿Qué cambia entre MVC con vistas y MVC con JSON?

Un controlador web tradicional suele devolver el nombre de una plantilla:

```java
@GetMapping("/pets")
public String list(Model model) {
    model.addAttribute("pets", petService.findAll());
    return "pets/list";
}
```

La cadena `"pets/list"` indica qué plantilla Thymeleaf debe renderizar Spring MVC.

Un controlador REST utiliza `@RestController`. Con esa anotación, el valor retornado se interpreta normalmente como **contenido del cuerpo HTTP**, que Spring serializa al formato correspondiente mediante sus conversores:

```java
@GetMapping("/api/v1/pets")
public List<PetResponse> list() {
    return petService.findAll().stream()
        .map(petApiMapper::toResponse)
        .toList();
}
```

Estos dos fragmentos son comparaciones conceptuales. No crees todavía dos rutas `/pets` ni reemplaces el controlador HTML: crearás un controlador separado para `/api/v1/pets`.

### 2. Comprender tres anotaciones fundamentales

`@RestController` combina el comportamiento de un controlador de Spring MVC con la producción de cuerpos de respuesta. No devuelve nombres de plantillas Thymeleaf.

`@RequestBody` indica que el parámetro procede del **cuerpo de la petición**, no de campos de formulario enviados como parámetros URL. Si el cuerpo es JSON, el conversor HTTP lo interpreta y construye el objeto Java correspondiente.

`@Valid` solicita aplicar las restricciones de Jakarta Validation declaradas sobre el DTO. Si fallan, Spring MVC no debe continuar con la creación o actualización de la entidad.

Ejemplo conceptual:

```java
@PostMapping
public ResponseEntity<PetResponse> create(
    @Valid @RequestBody PetRequest request
) {
    // The complete implementation appears below.
    return null;
}
```

**No copies ese ejemplo como implementación.** El `null` no representa una respuesta correcta; la implementación completa construirá un recurso, una cabecera `Location` y el cuerpo correspondiente.

### 3. Crear la carpeta de controladores REST

Crea la carpeta:

```text
src/main/java/com/example/petmatchlab/api/controller/
```

No reutilices el paquete `controller` de las vistas para copiar los métodos de las operaciones. La separación de paquetes hace visible que ambas interfaces tienen responsabilidades de transporte diferentes.

La regla general será:

```text
api/controller → service → repository → MariaDB
```

No añadiremos `@Transactional` a cada controlador REST ni haremos consultas SQL allí. Los servicios existentes ya definen las unidades de trabajo del dominio.

### 4. Implementar `PetRestController`

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/controller/PetRestController.java`:

```java
package com.example.petmatchlab.api.controller;

import java.net.URI;
import java.util.List;

import jakarta.validation.Valid;

import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.support.ServletUriComponentsBuilder;

import com.example.petmatchlab.api.dto.PetRequest;
import com.example.petmatchlab.api.dto.PetResponse;
import com.example.petmatchlab.api.mapper.PetApiMapper;
import com.example.petmatchlab.exception.ResourceNotFoundException;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.service.PetService;

@RestController
@RequestMapping(
    path = "/api/v1/pets",
    produces = MediaType.APPLICATION_JSON_VALUE
)
public class PetRestController {

    private final PetService petService;
    private final PetApiMapper mapper;

    public PetRestController(PetService petService, PetApiMapper mapper) {
        this.petService = petService;
        this.mapper = mapper;
    }

    @GetMapping
    public List<PetResponse> list() {
        return petService.findAll().stream()
            .map(mapper::toResponse)
            .toList();
    }

    @GetMapping("/{petId}")
    public PetResponse detail(@PathVariable("petId") long petId) {
        Pet pet = petService.findById(petId)
            .orElseThrow(() -> new ResourceNotFoundException(
                "Pet not found"
            ));

        return mapper.toResponse(pet);
    }

    @PostMapping(consumes = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<PetResponse> create(
        @Valid @RequestBody PetRequest request
    ) {
        Pet created = petService.create(mapper.toForm(request));

        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{petId}")
            .buildAndExpand(created.getId())
            .toUri();

        return ResponseEntity.created(location)
            .body(mapper.toResponse(created));
    }

    @PutMapping(
        path = "/{petId}",
        consumes = MediaType.APPLICATION_JSON_VALUE
    )
    public PetResponse update(
        @PathVariable("petId") long petId,
        @Valid @RequestBody PetRequest request
    ) {
        Pet updated = petService.update(petId, mapper.toForm(request))
            .orElseThrow(() -> new ResourceNotFoundException(
                "Pet not found"
            ));

        return mapper.toResponse(updated);
    }

    @DeleteMapping("/{petId}")
    public ResponseEntity<Void> delete(@PathVariable("petId") long petId) {
        if (!petService.delete(petId)) {
            throw new ResourceNotFoundException("Pet not found");
        }

        return ResponseEntity.noContent().build();
    }
}
```

**Este archivo es un controlador REST completo.** Todos sus métodos dependen de `PetService`, que ya existía. Los objetos JSON de respuesta proceden de `PetApiMapper`.

### 5. Leer el método de listado

```java
@GetMapping
public List<PetResponse> list() {
    return petService.findAll().stream()
        .map(mapper::toResponse)
        .toList();
}
```

`findAll()` entrega mascotas desde el servicio. `stream()` permite recorrerlas como un flujo de procesamiento. `map(...)` transforma cada entidad en un `PetResponse`. `toList()` reúne las respuestas en una lista.

Si el catálogo está vacío, el resultado será una colección vacía que se representará como:

```json
[]
```

Un listado vacío no equivale a un recurso individual inexistente: para `GET /api/v1/pets`, **`200 OK` con `[]` es una respuesta válida**.

### 6. Leer el método de detalle

```java
Pet pet = petService.findById(petId)
    .orElseThrow(() -> new ResourceNotFoundException("Pet not found"));
```

`findById` devuelve `Optional<Pet>`. Si hay una mascota, el controlador la transforma. Si no existe, lanza una excepción conocida que el manejador REST convertirá en `404`.

El controlador no debe fabricar un `PetResponse` con valores inventados para ocultar que la mascota no existe.

### 7. Leer la creación y su `Location`

```java
Pet created = petService.create(mapper.toForm(request));
```

El mapper transforma la petición en `PetForm`. El servicio normaliza y comprueba los datos, crea la entidad y la guarda. No hemos escrito esas reglas de nuevo dentro del controlador.

Después construimos la ubicación del recurso:

```java
URI location = ServletUriComponentsBuilder.fromCurrentRequest()
    .path("/{petId}")
    .buildAndExpand(created.getId())
    .toUri();
```

Si el cliente hizo `POST /api/v1/pets` y la base asignó el identificador `18`, `Location` apuntará a `/api/v1/pets/18`, con el esquema y el host correspondientes al contexto de la petición.

En un despliegue detrás de un proxy, la generación de URL absoluta requiere configurar correctamente las cabeceras reenviadas y confiar solo en proxies autorizados. Para el ejercicio local, la URL se construye con la petición recibida por Spring MVC.

El retorno:

```java
return ResponseEntity.created(location)
    .body(mapper.toResponse(created));
```

establece el código `201 Created`, la cabecera `Location` y un cuerpo JSON.

### 8. Leer la actualización

```java
Pet updated = petService.update(petId, mapper.toForm(request))
    .orElseThrow(() -> new ResourceNotFoundException("Pet not found"));
```

La operación `PUT` se utiliza aquí para enviar los datos completos que la aplicación permite editar. En este contrato, el cliente debe incluir `name`, `species` y `age`; `description` es opcional.

No utilizamos `PATCH`, que normalmente permite expresar actualizaciones parciales. **No anuncies soporte de `PATCH` si no has implementado su interpretación y sus validaciones.**

### 9. Leer la eliminación

```java
if (!petService.delete(petId)) {
    throw new ResourceNotFoundException("Pet not found");
}

return ResponseEntity.noContent().build();
```

La eliminación conserva la regla construida en el Laboratorio 10: **una mascota que tiene solicitudes relacionadas no puede eliminarse**. Si el servicio lanza `BusinessRuleException`, el manejador REST devolverá `409 Conflict`.

Si la mascota no existe, `delete(...)` devuelve `false` y el controlador transforma esa situación en `404 Not Found`. Si se elimina correctamente, se responde `204 No Content` **sin cuerpo JSON**. No envíes `{"success":true}` junto a un `204`.

### 10. Implementar `SupportRequestRestController`

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/controller/SupportRequestRestController.java`:

```java
package com.example.petmatchlab.api.controller;

import java.net.URI;
import java.util.List;

import jakarta.validation.Valid;

import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.support.ServletUriComponentsBuilder;

import com.example.petmatchlab.api.dto.SupportRequestRequest;
import com.example.petmatchlab.api.dto.SupportRequestResponse;
import com.example.petmatchlab.api.mapper.SupportRequestApiMapper;
import com.example.petmatchlab.model.SupportRequest;
import com.example.petmatchlab.service.SupportRequestService;

@RestController
@RequestMapping(
    path = "/api/v1/pets/{petId}/requests",
    produces = MediaType.APPLICATION_JSON_VALUE
)
public class SupportRequestRestController {

    private final SupportRequestService requestService;
    private final SupportRequestApiMapper mapper;

    public SupportRequestRestController(
        SupportRequestService requestService,
        SupportRequestApiMapper mapper
    ) {
        this.requestService = requestService;
        this.mapper = mapper;
    }

    @GetMapping
    public List<SupportRequestResponse> list(
        @PathVariable("petId") long petId
    ) {
        return requestService.findAllForPet(petId).stream()
            .map(request -> mapper.toResponse(request, petId))
            .toList();
    }

    @GetMapping("/{requestId}")
    public SupportRequestResponse detail(
        @PathVariable("petId") long petId,
        @PathVariable("requestId") long requestId
    ) {
        SupportRequest request = requestService.findOneForPet(
            petId, requestId
        );
        return mapper.toResponse(request, petId);
    }

    @PostMapping(consumes = MediaType.APPLICATION_JSON_VALUE)
    public ResponseEntity<SupportRequestResponse> create(
        @PathVariable("petId") long petId,
        @Valid @RequestBody SupportRequestRequest body
    ) {
        SupportRequest created = requestService.create(
            petId, mapper.toForm(body)
        );

        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{requestId}")
            .buildAndExpand(created.getId())
            .toUri();

        return ResponseEntity.created(location)
            .body(mapper.toResponse(created, petId));
    }

    @PostMapping("/{requestId}/complete")
    public ResponseEntity<Void> complete(
        @PathVariable("petId") long petId,
        @PathVariable("requestId") long requestId
    ) {
        requestService.complete(petId, requestId);
        return ResponseEntity.noContent().build();
    }

    @PostMapping("/{requestId}/cancel")
    public ResponseEntity<Void> cancel(
        @PathVariable("petId") long petId,
        @PathVariable("requestId") long requestId
    ) {
        requestService.cancel(petId, requestId);
        return ResponseEntity.noContent().build();
    }

    @PutMapping(
        path = "/{requestId}/replace",
        consumes = MediaType.APPLICATION_JSON_VALUE
    )
    public ResponseEntity<SupportRequestResponse> replace(
        @PathVariable("petId") long petId,
        @PathVariable("requestId") long requestId,
        @Valid @RequestBody SupportRequestRequest body
    ) {
        SupportRequest replacement = requestService.replace(
            petId, requestId, mapper.toForm(body)
        );

        URI location = ServletUriComponentsBuilder.fromCurrentContextPath()
            .path("/api/v1/pets/{petId}/requests/{requestId}")
            .buildAndExpand(petId, replacement.getId())
            .toUri();

        return ResponseEntity.created(location)
            .body(mapper.toResponse(replacement, petId));
    }
}
```

**Este archivo también es completo.** Utiliza exactamente los métodos que `SupportRequestService` ya tenía al terminar el Laboratorio 10: `findAllForPet`, `findOneForPet`, `create`, `complete`, `cancel` y `replace`.

### 11. El recurso anidado no es una tabla anidada

La ruta:

```text
/api/v1/pets/5/requests/21
```

no significa que MariaDB guarde físicamente la tabla `support_requests` dentro de la fila `pets`.

Significa que el cliente identifica una solicitud en el **contexto de una mascota**. La relación se guarda mediante una clave foránea. El servicio comprueba la pertenencia usando el método de repositorio que ya se explicó:

```java
findByIdAndPet_Id(requestId, petId)
```

Si el `requestId` existe, pero está asociado a otra mascota, la búsqueda no devuelve el recurso solicitado en ese contexto. Responder `404` evita dar por válida una asociación que no existe.

### 12. Los cambios de estado siguen siendo operaciones del dominio

En los métodos de completar y cancelar, el controlador ejecuta:

```java
requestService.complete(petId, requestId);
```

O bien:

```java
requestService.cancel(petId, requestId);
```

No utiliza:

```java
request.setStatus(RequestStatus.COMPLETED);
```

Esa última línea **no es válida en el diseño existente**, porque `SupportRequest` no expone un setter público de estado; lo protege mediante `complete()` y `cancel()`. Tampoco debemos añadir ese setter para facilitar la API.

El servicio conserva los límites transaccionales, las consultas y las transiciones válidas. La API únicamente cambia **cómo se solicita la operación y cómo se comunica el resultado**.

### 13. ¿Por qué `replace` produce una solicitud nueva?

En el Laboratorio 10, `replace(...)` cancela una solicitud abierta y crea otra en la misma transacción. Por eso el controlador REST devuelve `201 Created` y la ubicación **de la solicitud nueva**.

Ejemplo conceptual:

```text
PUT /api/v1/pets/5/requests/21/replace

Antes:
  request 21 → OPEN

Después:
  request 21 → CANCELLED
  request 22 → OPEN

HTTP 201 Created
Location: /api/v1/pets/5/requests/22
```

En este caso `PUT` se utiliza sobre una **acción de sustitución definida por el dominio**, no como un reemplazo directo del recurso identificado en la URL. Es una elección pedagógica que permite reutilizar el caso de uso transaccional existente; **no debemos afirmar que el endpoint es idempotente**: repetirlo podría fallar porque la solicitud anterior ya fue cancelada. En una API cuyo contrato necesita idempotencia estricta para esta operación, sería más apropiado diseñar otra interfaz, por ejemplo un recurso de operaciones con una clave de idempotencia.

### 14. Tabla de endpoints implementados

| Método | Ruta | Respuesta correcta | Servicio utilizado |
|---|---|---|---|
| `GET` | `/api/v1/pets` | `200` + colección | `PetService.findAll` |
| `GET` | `/api/v1/pets/{petId}` | `200` + mascota | `PetService.findById` |
| `POST` | `/api/v1/pets` | `201` + `Location` | `PetService.create` |
| `PUT` | `/api/v1/pets/{petId}` | `200` + mascota | `PetService.update` |
| `DELETE` | `/api/v1/pets/{petId}` | `204` sin cuerpo | `PetService.delete` |
| `GET` | `/api/v1/pets/{petId}/requests` | `200` + colección | `SupportRequestService.findAllForPet` |
| `GET` | `/api/v1/pets/{petId}/requests/{requestId}` | `200` + solicitud | `SupportRequestService.findOneForPet` |
| `POST` | `/api/v1/pets/{petId}/requests` | `201` + `Location` | `SupportRequestService.create` |
| `POST` | `/api/v1/pets/{petId}/requests/{requestId}/complete` | `204` sin cuerpo | `SupportRequestService.complete` |
| `POST` | `/api/v1/pets/{petId}/requests/{requestId}/cancel` | `204` sin cuerpo | `SupportRequestService.cancel` |
| `PUT` | `/api/v1/pets/{petId}/requests/{requestId}/replace` | `201` + nueva ubicación | `SupportRequestService.replace` |

En esta tabla solo aparecen rutas que **sí se implementaron**. No agregues `PATCH`, filtros, paginación, ordenación configurable ni rutas de usuarios si todavía no están programadas.

### 15. ¿Qué ocurrirá al probar ahora?

Si ejecutas el proyecto antes de ajustar Spring Security, las rutas `/api/**` quedan cubiertas por la cadena web del Laboratorio 11, que termina con:

```java
.anyRequest().authenticated()
```

Un cliente HTTP podría recibir una redirección al formulario de login o enfrentarse a la protección CSRF de la web. **Eso no significa que el controlador REST deba manejar sesiones o desactivar CSRF de toda la aplicación**. Indica que aún falta definir una política específica para las nuevas rutas.

La siguiente práctica separará la autenticación web mediante formularios de la autenticación de clientes REST mediante HTTP Basic.

### Ejercicio — Recorrer una creación sin programar de nuevo

Dibuja el recorrido de `POST /api/v1/pets` hasta MariaDB. Identifica dónde ocurre cada acción:

1. Convertir el JSON en `PetRequest`.
2. Verificar `@NotBlank` y `@Min`.
3. Transformar `PetRequest` en `PetForm`.
4. Normalizar el nombre de la mascota.
5. Guardar la entidad.
6. Recuperar el identificador generado.
7. Construir el cuerpo `PetResponse`.
8. Devolver `201 Created` con `Location`.

**Comprobación:** dispones de dos controladores REST que reutilizan los servicios existentes; no creaste servicios paralelos ni copiaste reglas de negocio al paquete `api`.

---
## Práctica 12.4 — Separar la seguridad de la API y la web

### Objetivos

Al completar esta práctica podrás:

- explicar por qué una aplicación puede necesitar varias cadenas de filtros;
- identificar qué cadena atiende cada petición;
- configurar HTTP Basic para rutas `/api/**`;
- conservar el login por formulario, las sesiones y CSRF de la web;
- proteger operaciones de escritura mediante los roles existentes;
- reconocer los límites de seguridad del ejemplo.

### 1. La autenticación ocurre antes del controlador

Cuando un cliente envía:

```http
POST /api/v1/pets HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic <credentials>
```

no debería suponerse que Spring llama inmediatamente a `PetRestController.create(...)`.

Primero interviene la infraestructura de Spring Security. Debe elegir la política aplicable, verificar las credenciales y comprobar si el usuario tiene el rol necesario.

```text
Petición HTTP
      ↓
FilterChainProxy
      ↓
Selecciona SecurityFilterChain
      ↓
Autentica credenciales
      ↓
Comprueba permisos
      ↓
PetRestController
      ↓
PetService
      ↓
PetRepository
```

La seguridad de la ruta no elimina las reglas de negocio. Una cuenta `ADMIN` puede tener permiso para invocar `complete`, pero **no puede completar una solicitud que ya está cancelada**: el servicio y la entidad seguirán rechazando esa transición.

### 2. Dos interfaces, dos políticas

La aplicación tendrá dos grupos de peticiones:

| Interfaz | Rutas | Autenticación | Sesión de autenticación | CSRF |
|---|---|---|---|---|
| Web | `/pets`, `/login`, `/register`, `/admin` | Formulario de acceso | Sí | Activada |
| API | `/api/**` | HTTP Basic | Política sin sesiones | Desactivada **solo** en esa cadena |

**No mezcles estas configuraciones.** Si desactivas CSRF en la cadena web para facilitar una petición REST, estarías eliminando una protección que incorporaste precisamente para los formularios y las sesiones del navegador.

### 3. ¿Qué contiene HTTP Basic?

HTTP Basic utiliza una cabecera `Authorization` cuyo valor contiene `Basic` seguido de una representación Base64 de `usuario:contraseña`:

```text
username:password
        ↓
Base64 encoding
        ↓
Authorization: Basic ...
```

**Base64 no es cifrado.** Una persona que pueda leer la cabecera puede recuperar las credenciales. Por eso HTTP Basic debe utilizarse **solo sobre HTTPS** cuando el servicio viaja por una red que no es de confianza. En el desarrollo local con `localhost` se puede observar su funcionamiento, pero no debe desplegarse un servicio accesible por Internet mediante HTTP sin TLS.

El servidor sigue utilizando `DatabaseUserDetailsService` y el `PasswordEncoder` construidos en el laboratorio anterior. No agregaremos una tabla de usuarios exclusiva para la API ni almacenaremos contraseñas de otra manera.

### 4. ¿Qué significa `STATELESS`?

La política:

```java
.sessionManagement(session -> session
    .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
)
```

indica que Spring Security no creará ni utilizará una sesión HTTP para recuperar y conservar la autenticación de esa cadena.

En el ejemplo, el cliente debe proporcionar las credenciales HTTP Basic en cada petición protegida. Esto **no significa que toda la aplicación se vuelva stateless**: la web conserva su sesión y su login por formulario mediante la otra cadena.

Tampoco implica que las operaciones sean inmutables: `POST`, `PUT` y `DELETE` siguen modificando información persistida. *Stateless* se refiere aquí a la manera de mantener el contexto de autenticación entre peticiones, no a la ausencia de cambios en MariaDB.

### 5. ¿Por qué no necesitamos JWT?

Un token JWT puede representar afirmaciones firmadas que una aplicación verifica, pero **no es un requisito para crear una API REST**. También existen otros mecanismos de autenticación. En este laboratorio utilizamos HTTP Basic porque permite estudiar el contrato HTTP, la autenticación y los roles con las cuentas que ya tenemos.

No añadas una librería JWT por imitación. Eso implicaría diseñar emisión, firma, validación, caducidad, revocación y protección de tokens, responsabilidades que no se requieren para los casos de uso de este laboratorio.

### 6. Reemplazar la configuración de seguridad

Abre el archivo existente `src/main/java/com/example/petmatchlab/security/SecurityConfig.java`.

**Sustituye su contenido completo** por la versión que sigue. Conserva su ubicación y el bean de `PasswordEncoder`, pero ahora habrá **dos** beans `SecurityFilterChain`, con orden y alcance explícitos.

**Archivo completo** `src/main/java/com/example/petmatchlab/security/SecurityConfig.java`:

```java
package com.example.petmatchlab.security;

import jakarta.servlet.DispatcherType;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Bean
    @Order(1)
    public SecurityFilterChain apiSecurityFilterChain(HttpSecurity http)
            throws Exception {
        http
            .securityMatcher("/api/**")
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers(
                    HttpMethod.GET,
                    "/api/v1/pets",
                    "/api/v1/pets/{petId}",
                    "/api/v1/pets/{petId}/requests",
                    "/api/v1/pets/{petId}/requests/{requestId}"
                ).hasAnyRole("USER", "ADMIN")
                .requestMatchers(
                    HttpMethod.POST,
                    "/api/v1/pets/{petId}/requests"
                ).hasAnyRole("USER", "ADMIN")
                .requestMatchers(
                    HttpMethod.POST,
                    "/api/v1/pets"
                ).hasRole("ADMIN")
                .requestMatchers(
                    HttpMethod.PUT,
                    "/api/v1/pets/{petId}"
                ).hasRole("ADMIN")
                .requestMatchers(
                    HttpMethod.DELETE,
                    "/api/v1/pets/{petId}"
                ).hasRole("ADMIN")
                .requestMatchers(
                    HttpMethod.POST,
                    "/api/v1/pets/{petId}/requests/{requestId}/complete",
                    "/api/v1/pets/{petId}/requests/{requestId}/cancel"
                ).hasRole("ADMIN")
                .requestMatchers(
                    HttpMethod.PUT,
                    "/api/v1/pets/{petId}/requests/{requestId}/replace"
                ).hasRole("ADMIN")
                .anyRequest().denyAll()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .csrf(csrf -> csrf.disable())
            .httpBasic(Customizer.withDefaults());

        return http.build();
    }

    @Bean
    @Order(2)
    public SecurityFilterChain webSecurityFilterChain(HttpSecurity http)
            throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .dispatcherTypeMatchers(
                    DispatcherType.FORWARD, DispatcherType.ERROR
                ).permitAll()
                .requestMatchers(
                    "/", "/about", "/login", "/register",
                    "/css/**", "/images/**", "/js/**", "/favicon.ico"
                ).permitAll()
                .requestMatchers("/admin", "/admin/**").hasRole("ADMIN")
                .requestMatchers(
                    HttpMethod.GET,
                    "/pets/new",
                    "/pets/{petId}/edit",
                    "/pets/{petId}/requests/{requestId}/replace"
                ).hasRole("ADMIN")
                .requestMatchers(
                    HttpMethod.POST,
                    "/pets",
                    "/pets/{petId}",
                    "/pets/{petId}/delete",
                    "/pets/{petId}/requests/{requestId}/complete",
                    "/pets/{petId}/requests/{requestId}/cancel",
                    "/pets/{petId}/requests/{requestId}/replace"
                ).hasRole("ADMIN")
                .requestMatchers(
                    HttpMethod.POST,
                    "/pets/{petId}/requests"
                ).hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .loginProcessingUrl("/login")
                .usernameParameter("email")
                .passwordParameter("password")
                .defaultSuccessUrl("/pets", true)
                .failureUrl("/login?error")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login?logout")
                .invalidateHttpSession(true)
                .permitAll()
            );

        return http.build();
    }
}
```

Esta configuración reproduce las reglas de la web construidas en el Laboratorio 11 y agrega la política REST. **No borres** `DatabaseUserDetailsService` ni cambies el rol por defecto de las cuentas registradas.

### 7. Comprender `@Order` y `securityMatcher`

Spring Security utiliza `FilterChainProxy` para seleccionar la primera cadena que coincida con la petición.

```text
GET /api/v1/pets
      ↓
SecurityFilterChain @Order(1)
securityMatcher("/api/**")
      ↓ coincide
HTTP Basic + reglas REST
```

En cambio:

```text
GET /pets
      ↓
SecurityFilterChain @Order(1)
securityMatcher("/api/**")
      ↓ no coincide
SecurityFilterChain @Order(2)
      ↓
Login web + sesión + CSRF
```

Una petición **no recorre completas ambas cadenas**. La primera cadena cuyo matcher coincide determina la política aplicable.

`securityMatcher("/api/**")` determina **qué peticiones seleccionan la cadena**. Los `requestMatchers(...)` dentro de `authorizeHttpRequests` deciden **qué permisos se necesitan** después de haber seleccionado esa cadena.

Confundir estas dos responsabilidades puede dejar rutas sin proteger o aplicar a una API una política de login que no corresponde.

### 8. Leer las reglas de la API en orden

La secuencia de autorizaciones importa. Hemos colocado primero las rutas de consulta, después la creación de solicitudes permitida a ambos roles y finalmente las operaciones administrativas.

Ejemplos:

```text
GET  /api/v1/pets                  → USER o ADMIN
POST /api/v1/pets                  → solo ADMIN
POST /api/v1/pets/5/requests       → USER o ADMIN
POST /api/v1/pets/5/requests/8/cancel → solo ADMIN
```

La cláusula:

```java
.anyRequest().denyAll()
```

niega por defecto las solicitudes que no coinciden con las reglas declaradas dentro de la cadena REST. Es una decisión deliberada para que añadir accidentalmente un método REST no lo vuelva accesible antes de definir su política. Como consecuencia, algunas rutas o métodos **no declarados** pueden recibir `403` en la capa de seguridad en vez de llegar a la capa MVC para devolver `404` o `405`. Para un contrato API público más amplio habría que decidir expresamente cómo gestionar esas respuestas; aquí priorizamos una política de acceso explícita.

### 9. Lo que permanece en la cadena web

Compara las partes que no cambiaron:

```java
.formLogin(form -> form
    .loginPage("/login")
    .loginProcessingUrl("/login")
    .usernameParameter("email")
    .passwordParameter("password")
    .defaultSuccessUrl("/pets", true)
    .failureUrl("/login?error")
    .permitAll()
)
```

El formulario HTML sigue enviando `email` y `password` a `/login`. Spring Security comprueba los datos mediante el `UserDetailsService` que consulta `AppUserRepository`. Una autenticación correcta establece el contexto correspondiente en la sesión web.

También se conserva:

```java
.logout(logout -> logout
    .logoutUrl("/logout")
    .logoutSuccessUrl("/login?logout")
    .invalidateHttpSession(true)
    .permitAll()
)
```

`POST /logout` sigue siendo la operación adecuada para cerrar la sesión desde el formulario Thymeleaf. No la transformes en un enlace GET ni desactives CSRF para evitar enviar su token.

### 10. CSRF de la API y CSRF de los formularios

En la cadena API aparece:

```java
.csrf(csrf -> csrf.disable())
```

**Su alcance está limitado a `/api/**` por `securityMatcher`.** Los formularios web mantienen CSRF activado. Para la API de este ejercicio, que no utiliza autenticación mediante cookies de sesión y espera credenciales explícitas en la cabecera, hemos decidido no requerir tokens CSRF.

Esta elección tiene un límite importante: **HTTP Basic también puede utilizarse desde navegadores que almacenen o reenvíen credenciales automáticamente**. Si la API se utiliza desde un navegador de esa forma, debe evaluarse el riesgo de CSRF de acuerdo con los mecanismos reales de autenticación y las capacidades del cliente. No debemos convertir «API stateless» en la regla universal «desactivar CSRF siempre es seguro».

Para una aplicación real, además de HTTPS, deben revisarse el almacenamiento de credenciales, las cabeceras del navegador, los permisos efectivos, los controles contra abuso y la exposición de datos.

### 11. Autorización de ruta no es propiedad de recurso

La política del Laboratorio 11 distingue `USER` y `ADMIN`. **Todavía no existe una relación persistente que indique quién es propietario de cada mascota o de cada solicitud**.

Por tanto, el modelo actual permite que un `USER` autenticado consulte mascotas y solicitudes conforme a las rutas permitidas, pero **no demuestra que los registros pertenezcan a esa cuenta**. Añadir `Authentication` al controlador sin un modelo de propiedad y una regla de acceso no resolvería ese problema.

No afirmes que esta API garantiza privacidad por usuario. Para conseguirla sería necesario incorporar, por ejemplo, una relación `Pet.owner`, consultas filtradas por la cuenta autenticada y verificaciones en la capa de servicio. Esa ampliación se puede estudiar como ejercicio posterior, pero no forma parte del comportamiento implementado aquí.

### 12. Comprobar la separación sin cambiar el código del dominio

Con la aplicación iniciada, predice estas respuestas:

| Petición | Sin credenciales | `USER` válido | `ADMIN` válido |
|---|---|---|---|
| `GET /api/v1/pets` | `401` | `200` | `200` |
| `POST /api/v1/pets` | `401` | `403` | Puede llegar al controlador |
| `POST /api/v1/pets/5/requests` | `401` | Puede llegar al controlador | Puede llegar al controlador |
| `POST /api/v1/pets/5/requests/8/complete` | `401` | `403` | Puede llegar al servicio |
| `GET /pets` | Redirección al login | Página HTML | Página HTML |

«Puede llegar al controlador» no significa que el resultado será siempre exitoso. Una mascota inexistente, datos inválidos o una regla de negocio incumplida pueden producir otros códigos después de superar la seguridad.

### Ejercicio — Reconoce el punto de rechazo

Una petición con un usuario válido intenta eliminar una mascota que tiene solicitudes. Distingue dos escenarios:

- La cuenta posee rol `USER`: Spring Security debe rechazarla con `403` antes de ejecutar `PetService.delete(...)`.
- La cuenta posee rol `ADMIN`: la política HTTP permite la llamada, pero `PetService.delete(...)` detecta el historial y lanza `BusinessRuleException`; la API responderá `409`.

Esta comparación muestra que **autorización y reglas de negocio son capas complementarias**, no mecanismos equivalentes.

### Depuración — La política aplicada no es la esperada

Si una petición API redirige a `/login`, revisa el alcance y el orden de las cadenas. Comprueba que la ruta comienza por `/api/`, que el bean `apiSecurityFilterChain` está presente y que tiene prioridad sobre la cadena web.

Si un POST web funciona sin token CSRF después de la modificación, revisa que no hayas desactivado CSRF en la cadena equivocada.

Si una petición API `POST` de `USER` crea mascotas, comprueba los `requestMatchers` y verifica que el endpoint `/api/v1/pets` requiere `ADMIN`.

Si una cuenta `ADMIN` recibe `403`, comprueba que el registro de la base tiene realmente rol `ADMIN` y que el adaptador `DatabaseUserDetailsService` construye la autoridad `ROLE_ADMIN`. El formulario público de registro **no permite elegir ese rol**.

**Comprobación:** las rutas REST utilizan HTTP Basic sin sesión de autenticación; las páginas web mantienen el formulario, la sesión y CSRF; ambas interfaces comparten el mismo repositorio de cuentas y los mismos servicios del dominio.

---
## Práctica 12.5 — Errores JSON y comprobación con clientes HTTP

### Objetivos

Al completar esta práctica podrás:

- devolver errores estructurados desde una API REST;
- diferenciar una excepción de dominio de un error de validación;
- reconocer los errores generados antes de llegar al controlador;
- consultar y modificar recursos desde `curl` u otro cliente HTTP;
- verificar códigos, cabeceras y cuerpos, no solo el contenido visible.

### 1. Una API no debe devolver la plantilla de error de Thymeleaf

La aplicación ya cuenta con `PageErrorHandler`, un `@ControllerAdvice` construido para transformar excepciones del dominio en **páginas HTML**. Su responsabilidad sigue siendo correcta para las rutas web.

Pero si un cliente solicita:

```http
GET /api/v1/pets/999999 HTTP/1.1
Accept: application/json
```

espera una respuesta de la API, no un documento HTML pensado para un navegador. Necesitamos un manejador de errores REST que produzca el DTO `ApiErrorResponse`.

Los errores de seguridad como `401` y `403` pueden producirse **en los filtros, antes del controlador**. Esos errores siguen el tratamiento de Spring Security y **no tienen por qué usar el mismo cuerpo JSON que los errores manejados por un `@RestControllerAdvice`**. La distinción importa para no prometer uniformidad que el código no implementa.

### 2. Diferentes fallos necesitan respuestas diferentes

| Situación | Componente que la detecta | Código previsto |
|---|---|---|
| Nombre vacío | Validación de `PetRequest` | `400` |
| Edad negativa | Validación de `PetRequest` | `400` |
| JSON con sintaxis inválida | Conversor HTTP de Spring MVC | `400` |
| Valor de enum desconocido | Conversor de JSON | `400` |
| Identificador de ruta que no puede convertirse a `long` | Enlace de parámetros MVC | `400` |
| Mascota inexistente | Servicio/controlador | `404` |
| Solicitud que pertenece a otra mascota | Servicio de solicitudes | `404` |
| Completar una solicitud cancelada | Regla de dominio | `409` |
| Eliminar mascota con solicitudes | Regla de negocio | `409` |
| Credenciales ausentes o incorrectas | Spring Security | `401` |
| Rol insuficiente | Spring Security | `403` |

**No conviertas todas las excepciones en `400`**. Un conflicto de estado no es un fallo de sintaxis. Y no captures `Exception` genéricamente para responder siempre `200`: eso ocultaría fallos reales.

### 3. Crear el manejador REST

Crea la carpeta:

```text
src/main/java/com/example/petmatchlab/api/error/
```

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/api/error/RestApiExceptionHandler.java`:

```java
package com.example.petmatchlab.api.error;

import java.util.LinkedHashMap;
import java.util.Map;

import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import org.springframework.http.ResponseEntity;
import org.springframework.http.converter.HttpMessageNotReadableException;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.method.annotation.MethodArgumentTypeMismatchException;

import com.example.petmatchlab.api.controller.PetRestController;
import com.example.petmatchlab.api.controller.SupportRequestRestController;
import com.example.petmatchlab.api.dto.ApiErrorResponse;
import com.example.petmatchlab.exception.BusinessRuleException;
import com.example.petmatchlab.exception.ResourceNotFoundException;

@RestControllerAdvice(assignableTypes = {
    PetRestController.class,
    SupportRequestRestController.class
})
@Order(Ordered.HIGHEST_PRECEDENCE)
public class RestApiExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiErrorResponse> handleValidation(
        MethodArgumentNotValidException exception
    ) {
        Map<String, String> errors = new LinkedHashMap<>();

        for (FieldError error : exception.getBindingResult().getFieldErrors()) {
            String message = error.getDefaultMessage() == null
                ? "Invalid field value"
                : error.getDefaultMessage();
            errors.putIfAbsent(error.getField(), message);
        }

        return ResponseEntity.badRequest().body(new ApiErrorResponse(
            "validation_failed",
            "The request contains invalid fields",
            errors
        ));
    }

    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ApiErrorResponse> handleUnreadableBody() {
        return ResponseEntity.badRequest().body(new ApiErrorResponse(
            "invalid_json",
            "The request body is not valid JSON for this operation",
            Map.of()
        ));
    }

    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ApiErrorResponse> handleInvalidPathVariable() {
        return ResponseEntity.badRequest().body(new ApiErrorResponse(
            "invalid_parameter",
            "A path parameter has an invalid value",
            Map.of()
        ));
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ApiErrorResponse> handleInvalidArgument() {
        return ResponseEntity.badRequest().body(new ApiErrorResponse(
            "invalid_request",
            "The request contains invalid data",
            Map.of()
        ));
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiErrorResponse> handleNotFound() {
        return ResponseEntity.status(404).body(new ApiErrorResponse(
            "not_found",
            "The requested resource does not exist",
            Map.of()
        ));
    }

    @ExceptionHandler(BusinessRuleException.class)
    public ResponseEntity<ApiErrorResponse> handleBusinessRule() {
        return ResponseEntity.status(409).body(new ApiErrorResponse(
            "business_conflict",
            "The operation conflicts with the current resource state",
            Map.of()
        ));
    }
}
```

**Este archivo es completo.** Los métodos están limitados a los dos controladores REST creados en la práctica anterior. `@Order(Ordered.HIGHEST_PRECEDENCE)` indica que este tratamiento tiene prioridad respecto de otros consejos de controladores aplicables, incluido `PageErrorHandler`.

¿Por qué es importante la prioridad? `@RestController` es una especialización de `@Controller`; por tanto, los selectores por anotaciones de un `@ControllerAdvice` de páginas podrían coincidir también con controladores REST. Configurar explícitamente el alcance y el orden evita que una excepción de la API termine en una vista HTML.

**No es necesario eliminar ni reemplazar `PageErrorHandler`.** Las rutas HTML continúan utilizándolo para los errores de las operaciones web.

### 4. Leer la validación de campos

En el manejador aparece:

```java
for (FieldError error : exception.getBindingResult().getFieldErrors()) {
    String message = error.getDefaultMessage() == null
        ? "Invalid field value"
        : error.getDefaultMessage();
    errors.putIfAbsent(error.getField(), message);
}
```

`MethodArgumentNotValidException` representa el fallo de validación de un argumento, como `@Valid @RequestBody PetRequest`.

Cada `FieldError` identifica el campo y contiene un mensaje. `putIfAbsent` conserva el primer mensaje cuando un mismo campo incumple varias restricciones. **Este contrato muestra un mensaje por campo**; no es una representación exhaustiva de todas las violaciones posibles. Si el cliente necesita mostrar todas las violaciones, puede diseñarse una colección de mensajes por campo.

### 5. Por qué no enviamos errores internos completos

Las excepciones de conversión de JSON pueden incluir nombres de clases, rutas de propiedades o detalles internos del conversor. Una excepción de base de datos puede incluir nombres de tablas o restricciones.

En el cuerpo de error exponemos un mensaje controlado:

```json
{
  "code": "invalid_json",
  "message": "The request body is not valid JSON for this operation",
  "fieldErrors": {}
}
```

Esto **no impide que la aplicación registre información útil para depurar** en sus logs configurados. Simplemente evita trasladar indiscriminadamente detalles internos al consumidor de la API.

Tampoco intentamos manejar cualquier excepción con una respuesta inventada: los fallos inesperados deben investigarse y conservar su semántica de error del servidor.

### 6. Preparar un cliente HTTP

Puedes utilizar `curl` desde una terminal o un cliente gráfico como Postman o Insomnia. El mecanismo es el mismo: construir una petición HTTP y observar la respuesta.

Para comprobar la versión de `curl`:

```bash
curl --version
```

Si utilizas Windows PowerShell, puedes llamar explícitamente a `curl.exe` para evitar diferencias con posibles alias de comandos.

Los ejemplos siguientes utilizan `http://localhost:8080`. Sustituye las cuentas por las que **tú mismo hayas registrado**. Una cuenta pública recién registrada tiene rol `USER`; para ejecutar operaciones administrativas necesitarás una cuenta `ADMIN` preparada mediante el procedimiento controlado del laboratorio anterior.

**No escribas contraseñas reales en archivos de documentación, repositorios, capturas públicas ni comandos que queden almacenados en el historial.** En los comandos `curl -u "email@example.com"`, al no proporcionar el carácter `:` y la contraseña, `curl` puede solicitarla de forma interactiva.

### 7. Consultar mascotas desde la API

Con una cuenta `USER` válida:

```bash
curl -i \
  --user "user@example.com" \
  --header "Accept: application/json" \
  http://localhost:8080/api/v1/pets
```

El parámetro `-i` muestra **cabeceras y cuerpo**. Comprueba que el resultado sea `200 OK`, que el contenido sea JSON y que la respuesta represente una colección.

Si existen mascotas registradas desde Thymeleaf, también deben aparecer en la API, porque ambas interfaces consultan `PetService.findAll()` y la misma MariaDB.

### 8. Consultar una mascota concreta

Utiliza un identificador que exista realmente:

```bash
curl -i \
  --user "user@example.com" \
  --header "Accept: application/json" \
  http://localhost:8080/api/v1/pets/5
```

**El `5` es un ejemplo.** Consulta primero la lista y utiliza un identificador de tu base. No suponemos que todas las instalaciones tienen una mascota con `id=5`.

Prueba después con un identificador que no exista:

```bash
curl -i \
  --user "user@example.com" \
  --header "Accept: application/json" \
  http://localhost:8080/api/v1/pets/999999999
```

La API debe devolver `404` y un JSON con `code` igual a `not_found`, siempre que la solicitud llegue al controlador con autenticación válida.

### 9. Crear una mascota mediante JSON

Con una cuenta `ADMIN`:

```bash
curl -i \
  --user "admin@example.com" \
  --header "Accept: application/json" \
  --header "Content-Type: application/json" \
  --request POST \
  --data '{"name":"Kira","species":"Dog","age":4,"description":"Enjoys walks."}' \
  http://localhost:8080/api/v1/pets
```

Comprueba:

1. Código `201 Created`.
2. Cabecera `Location` que termina en el identificador generado.
3. Cuerpo JSON con `id`, `name`, `species`, `age` y `description`.
4. Registro nuevo visible también en la web tradicional.
5. Fila correspondiente en MariaDB.

**No copies el identificador de un ejemplo:** utiliza el valor real devuelto por la operación de creación.

### 10. Actualizar una mascota

Utiliza el `id` real de una mascota de prueba que puedas modificar:

```bash
curl -i \
  --user "admin@example.com" \
  --header "Content-Type: application/json" \
  --request PUT \
  --data '{"name":"Kira","species":"Dog","age":5,"description":"Enjoys long walks."}' \
  http://localhost:8080/api/v1/pets/15
```

Si la mascota existe y se cumplen las reglas, recibirás `200 OK` con la representación actualizada. Confirma el cambio mediante un `GET` posterior y mediante la página HTML de esa mascota.

### 11. Crear y administrar solicitudes de apoyo

Para una mascota existente, una cuenta `USER` puede crear una solicitud:

```bash
curl -i \
  --user "user@example.com" \
  --header "Content-Type: application/json" \
  --request POST \
  --data '{"type":"WALK","description":"Needs a short walk."}' \
  http://localhost:8080/api/v1/pets/5/requests
```

Si la mascota existe y no tiene otra solicitud abierta, el resultado será `201 Created`, con un cuerpo que contiene `status: "OPEN"` y una cabecera `Location` para consultar la solicitud creada.

**Regla del dominio:** si esa mascota ya tiene una solicitud abierta, el servicio debe rechazar la creación con `409 Conflict`. No lo soluciones enviando un `status` fabricado por el cliente.

Para completar una solicitud abierta, utiliza una cuenta `ADMIN`:

```bash
curl -i \
  --user "admin@example.com" \
  --request POST \
  http://localhost:8080/api/v1/pets/5/requests/21/complete
```

El resultado correcto de una transición válida es `204 No Content`. Puedes verificar el nuevo estado mediante un `GET` a la URL de la solicitud.

No supongas que los identificadores `5` y `21` existen en tu base. Sustitúyelos por los valores obtenidos de tus respuestas reales.

### 12. Probar validación con una petición incorrecta

Envía una edad negativa:

```bash
curl -i \
  --user "admin@example.com" \
  --header "Content-Type: application/json" \
  --request POST \
  --data '{"name":"Kira","species":"Dog","age":-4}' \
  http://localhost:8080/api/v1/pets
```

Espera `400 Bad Request` y una respuesta con `code: "validation_failed"` y un error asociado al campo `age`.

Prueba ahora con un enum inexistente:

```bash
curl -i \
  --user "user@example.com" \
  --header "Content-Type: application/json" \
  --request POST \
  --data '{"type":"VETERINARY","description":"Needs assistance."}' \
  http://localhost:8080/api/v1/pets/5/requests
```

El cuerpo tiene sintaxis JSON correcta, pero `VETERINARY` no puede convertirse al enum `SupportType`. El conversor debe rechazarlo con `400`, y nuestro manejador utiliza el código `invalid_json` para esta categoría de error de lectura/convertibilidad.

### 13. Probar autenticación y permisos

Prueba una consulta sin credenciales:

```bash
curl -i \
  --header "Accept: application/json" \
  http://localhost:8080/api/v1/pets
```

La respuesta esperada es `401 Unauthorized`. Con HTTP Basic suele incluir una cabecera `WWW-Authenticate` que indica el desafío de autenticación.

Ahora intenta crear una mascota utilizando una cuenta `USER`:

```bash
curl -i \
  --user "user@example.com" \
  --header "Content-Type: application/json" \
  --request POST \
  --data '{"name":"Kira","species":"Dog","age":4}' \
  http://localhost:8080/api/v1/pets
```

La respuesta esperada es `403 Forbidden`. **No debe crearse ningún registro**. El rechazo procede de la política HTTP, antes de ejecutar `PetService.create(...)`.

### 14. Probar CSRF en las dos interfaces

En la API, el `POST` de una cuenta autorizada puede ejecutarse sin token CSRF porque esa cadena se configuró expresamente de esa manera.

En la web tradicional, una petición `POST /pets` desde una sesión `ADMIN` requiere el token CSRF. La plantilla Thymeleaf y Spring Security continúan incorporando y verificando esa protección. No deduzcas que un `POST /pets` sin cookie y sin token será aceptado: puede ser rechazado por autenticación, autorización o CSRF según las circunstancias.

La conclusión correcta es: **la política depende de qué cadena se selecciona, no de la palabra `POST` por sí sola**.

### 15. Inspeccionar directamente MariaDB

Tras crear una mascota y una solicitud desde la API, ejecuta consultas de solo lectura:

```sql
SELECT id, name, species, age, description
FROM pets
ORDER BY id DESC;

SELECT id, pet_id, type, description, status
FROM support_requests
ORDER BY id DESC;
```

La segunda consulta corresponde a las columnas del modelo del Laboratorio 10. No necesitas consultar las tablas de usuarios para mostrar las contraseñas codificadas ni exponer datos de autenticación en la API.

Comprueba que los cambios también aparecen en la interfaz HTML. **Una API y una web que comparten servicios y repositorios deben reflejar el mismo estado persistido**, salvo que sus políticas de visibilidad definan expresamente diferencias.

### 16. Una API no necesita una página por recurso

Los navegadores pueden utilizar múltiples plantillas para presentar mascotas y solicitudes. La API no necesita crear `templates/api/pet.html` ni `templates/api/request.html`.

Los DTO, los controladores REST y los conversores de Spring MVC son suficientes para producir las representaciones JSON. Agregar Thymeleaf a la respuesta REST introduciría otra responsabilidad sin necesidad.

### Ejercicio — Elaborar una tabla de verificación

Para cada solicitud realizada con un cliente HTTP, registra:

- método y ruta;
- cuenta o rol utilizado;
- código HTTP esperado;
- código HTTP obtenido;
- cabeceras relevantes (`Content-Type`, `Location`, `WWW-Authenticate` cuando corresponda);
- estructura del cuerpo recibido;
- efecto observable en MariaDB, si la operación modifica datos.

Cuando el resultado no coincida, **no cambies la expectativa inmediatamente**. Determina primero si falló la autenticación, la autorización, la validación, la búsqueda del recurso, la regla de negocio o la persistencia.

### Depuración — El resultado parece contradictorio

**`302 Found` con destino `/login` al llamar la API:** revisa el prefijo de la ruta y la selección de cadenas de seguridad. Puede haberse aplicado la política web por error.

**`401 Unauthorized` con credenciales que parecen correctas:** comprueba el correo normalizado, la contraseña registrada, el estado `enabled` y el adaptador `DatabaseUserDetailsService`. El rol no se evalúa antes de autenticar correctamente.

**`403 Forbidden` con credenciales válidas:** comprueba si la ruta está reservada a `ADMIN`. Si es una petición web, verifica además el token CSRF.

**`400 Bad Request` con `invalid_json`:** el cuerpo puede ser JSON mal formado o contener un valor que no pueda convertirse a un tipo Java, como un enum desconocido.

**`400 Bad Request` con `validation_failed`:** la deserialización funcionó, pero algún campo incumple una restricción declarada mediante Jakarta Validation.

**`404 Not Found` después de crear una solicitud:** comprueba los identificadores utilizados y que la solicitud pertenezca a la mascota de la URL. No combines arbitrariamente un `requestId` con un `petId` diferente.

**`409 Conflict` al completar una solicitud:** comprueba su estado actual. Si está `COMPLETED` o `CANCELLED`, el problema es una transición prohibida.

**`415 Unsupported Media Type`:** verifica que un `POST` o `PUT` con JSON incluya `Content-Type: application/json`.

**`500 Internal Server Error`:** no lo conviertas automáticamente en `400`. Examina los logs y la causa real; puede tratarse de una configuración, una consulta, una transacción o un defecto de programación.

**Comprobación:** puedes utilizar un cliente HTTP para demostrar operaciones correctas, errores controlados y rechazo de accesos no autorizados, distinguiendo claramente la respuesta HTTP del estado de MariaDB.

---
## Práctica 12.6 — Pruebas automatizadas, depuración y reto integrador

### Objetivos

Al completar esta práctica podrás:

- probar mapeadores sin iniciar Spring;
- verificar rutas REST con `MockMvc`;
- utilizar credenciales HTTP Basic reales contra cuentas almacenadas en H2;
- comprobar roles, validación, respuestas JSON y cambios persistidos;
- distinguir la evidencia de una prueba automatizada de una comprobación manual contra MariaDB;
- identificar problemas de contrato, seguridad y lógica de negocio.

### 1. ¿Qué debemos probar en una API?

Probar solo que `GET /api/v1/pets` devuelve un código `200` no es suficiente. También debemos comprobar **quién puede utilizar la ruta**, **qué datos se reciben**, **qué estructura se devuelve** y **qué sucede cuando la operación no está permitida**.

Organiza las comprobaciones por responsabilidades:

| Responsabilidad | Prueba | Resultado que buscamos |
|---|---|---|
| Mapeo | Unidad de `PetApiMapper` | Los valores pasan correctamente entre DTO y `PetForm` |
| Autenticación | GET con HTTP Basic válido e inválido | `200` o `401` según las credenciales |
| Autorización | POST administrativo con cuenta `USER` | `403` sin crear datos |
| Validación | POST con edad negativa | `400` y error del campo `age` |
| Recurso inexistente | GET con id desconocido | `404` y código de error JSON |
| Creación | POST válido con `ADMIN` | `201`, `Location` y registro en H2 |
| Relación | Solicitud en URL de otra mascota | `404` |
| Estado | Completar una solicitud cancelada | `409` |
| Transacción | Reemplazar una solicitud abierta | Anterior `CANCELLED`, nueva `OPEN` |
| Separación web/API | POST web sin CSRF | `403` |

Las pruebas de este apartado usan **H2 aislada**, configurada previamente en `src/test/resources/application.yaml`. No requieren que la base de desarrollo MariaDB esté disponible ni deben modificarla.

### 2. Comprobar la base de pruebas

Verifica que existe `src/test/resources/application.yaml` con una configuración equivalente a:

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:petmatch-test;MODE=MySQL;DB_CLOSE_DELAY=-1
    driver-class-name: org.h2.Driver
    username: sa
    password: ""
  jpa:
    hibernate:
      ddl-auto: create-drop
    open-in-view: false
    show-sql: false
```

**Este archivo es exclusivo del classpath de pruebas.** Si tu archivo de pruebas anterior utiliza otro nombre de base H2 o incorpora opciones adicionales coherentes, consérvalo; no necesitas duplicarlo.

`MODE=MySQL` aproxima ciertas características de sintaxis de MariaDB/MySQL, pero **no convierte H2 en MariaDB**. Las diferencias de tipos, restricciones y comportamiento del motor real siguen requiriendo comprobación con MariaDB.

No pongas `ddl-auto: create-drop` en `src/main/resources/application.yaml` si esa configuración conecta a una base de datos con información que quieres conservar.

### 3. Crear pruebas unitarias del mapeador

Crea el **archivo completo** `src/test/java/com/example/petmatchlab/api/mapper/PetApiMapperTests.java`:

```java
package com.example.petmatchlab.api.mapper;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNull;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import com.example.petmatchlab.api.dto.PetRequest;
import com.example.petmatchlab.api.dto.PetResponse;
import com.example.petmatchlab.dto.PetForm;
import com.example.petmatchlab.model.Pet;

class PetApiMapperTests {

    private PetApiMapper mapper;

    @BeforeEach
    void setUp() {
        mapper = new PetApiMapper();
    }

    @Test
    void shouldMapJsonRequestFieldsToExistingServiceForm() {
        PetRequest request = new PetRequest(
            "Luna", "Dog", 3, "Enjoys walks."
        );

        PetForm form = mapper.toForm(request);

        assertEquals("Luna", form.getName());
        assertEquals("Dog", form.getSpecies());
        assertEquals(Integer.valueOf(3), form.getAge());
        assertEquals("Enjoys walks.", form.getDescription());
    }

    @Test
    void shouldKeepOptionalDescriptionNull() {
        PetRequest request = new PetRequest("Milo", "Cat", 2, null);

        PetForm form = mapper.toForm(request);

        assertNull(form.getDescription());
    }

    @Test
    void shouldMapEntityToResponseWithoutChangingTheEntity() {
        Pet pet = new Pet("Kira", "Dog", 4, null);

        PetResponse response = mapper.toResponse(pet);

        assertEquals("Kira", response.name());
        assertEquals("Dog", response.species());
        assertEquals(4, response.age());
        assertNull(response.description());
    }
}
```

Estas pruebas no inician Spring Boot ni utilizan un servidor HTTP. Construyen el mapeador como un objeto Java ordinario para comprobar una transformación pura de datos.

**No hay ninguna razón para simular un repositorio en esta prueba**: el mapper no depende de un repositorio. Si un mapper necesita comenzar una transacción o consultar MariaDB, probablemente has mezclado responsabilidades.

### 4. Una prueba HTTP debe incluir la seguridad real

`MockMvc` permite construir peticiones que atraviesan Spring MVC y los filtros de seguridad en un contexto de prueba. Para comprobar HTTP Basic de forma realista, utilizaremos cuentas almacenadas en H2 y el `PasswordEncoder` del proyecto.

No confundas dos estrategias:

- `.with(user("someone").roles("ADMIN"))` crea una identidad simulada para una petición de prueba; es útil para comprobar autorización, pero **no verifica la contraseña contra la base**.
- `.with(httpBasic(email, password))` construye la cabecera HTTP Basic y permite que el mecanismo de autenticación real consulte el `UserDetailsService` configurado.

En las pruebas siguientes elegiremos la segunda estrategia para verificar **autenticación y autorización juntas**.

### 5. Preparar cuentas de prueba sin abrir el registro público a `ADMIN`

El constructor de `AppUser` establece siempre el rol `USER` y no ofrece un setter público de rol. Eso protege el registro de cuentas. En el Laboratorio 11, la cuenta administrativa de desarrollo se preparaba mediante un cambio controlado en la base de datos.

Para las pruebas utilizaremos el mismo principio, pero **solo en la base H2 temporal**: registraremos una cuenta ordinaria, registraremos otra cuenta para la prueba administrativa y actualizaremos el rol de esta segunda cuenta con SQL de preparación.

**No agregues un endpoint `/register-admin`, un campo `role` en `RegistrationForm` ni un método público para elevar privilegios.** Eso cambiaría negativamente el modelo de seguridad que ya construiste.

### 6. Crear las pruebas de integración REST

Crea la carpeta:

```text
src/test/java/com/example/petmatchlab/api/controller/
```

Crea el **archivo completo** `src/test/java/com/example/petmatchlab/api/controller/RestApiIntegrationTests.java`:

```java
package com.example.petmatchlab.api.controller;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.httpBasic;
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.user;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.delete;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.header;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInstance;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
import org.springframework.http.MediaType;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;

import com.example.petmatchlab.model.AppUser;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.model.RequestStatus;
import com.example.petmatchlab.model.SupportRequest;
import com.example.petmatchlab.model.SupportType;
import com.example.petmatchlab.repository.AppUserRepository;
import com.example.petmatchlab.repository.PetRepository;
import com.example.petmatchlab.repository.SupportRequestRepository;

@SpringBootTest
@AutoConfigureMockMvc
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class RestApiIntegrationTests {

    private static final String USER_EMAIL = "api-user-test@example.com";
    private static final String ADMIN_EMAIL = "api-admin-test@example.com";
    private static final String PASSWORD = "ExamplePassword123!";

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private AppUserRepository userRepository;

    @Autowired
    private PetRepository petRepository;

    @Autowired
    private SupportRequestRepository requestRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @BeforeAll
    void prepareAccounts() {
        userRepository.findByEmail(USER_EMAIL)
            .ifPresent(userRepository::delete);
        userRepository.findByEmail(ADMIN_EMAIL)
            .ifPresent(userRepository::delete);

        userRepository.saveAndFlush(new AppUser(
            "API User", USER_EMAIL, passwordEncoder.encode(PASSWORD)
        ));

        AppUser admin = userRepository.saveAndFlush(new AppUser(
            "API Admin", ADMIN_EMAIL, passwordEncoder.encode(PASSWORD)
        ));

        jdbcTemplate.update(
            "UPDATE app_users SET role = 'ADMIN' WHERE id = ?",
            admin.getId()
        );
    }

    @Test
    void shouldRequireAuthenticationForApiRead() throws Exception {
        mockMvc.perform(get("/api/v1/pets"))
            .andExpect(status().isUnauthorized());
    }

    @Test
    void shouldRejectIncorrectBasicPassword() throws Exception {
        mockMvc.perform(get("/api/v1/pets")
                .with(httpBasic(USER_EMAIL, "WrongPassword123!")))
            .andExpect(status().isUnauthorized());
    }

    @Test
    void shouldAllowRegisteredUserToReadPets() throws Exception {
        mockMvc.perform(get("/api/v1/pets")
                .with(httpBasic(USER_EMAIL, PASSWORD)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$").isArray());
    }

    @Test
    void shouldRejectPetCreationForUserRole() throws Exception {
        mockMvc.perform(post("/api/v1/pets")
                .with(httpBasic(USER_EMAIL, PASSWORD))
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name":"Kira","species":"Dog","age":4}
                    """))
            .andExpect(status().isForbidden());
    }

    @Test
    void shouldCreatePetWithLocationForAdmin() throws Exception {
        MvcResult result = mockMvc.perform(post("/api/v1/pets")
                .with(httpBasic(ADMIN_EMAIL, PASSWORD))
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name":"Kira","species":"Dog","age":4}
                    """))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.name").value("Kira"))
            .andExpect(jsonPath("$.age").value(4))
            .andReturn();

        String location = result.getResponse().getHeader("Location");
        assertNotNull(location);

        long petId = Long.parseLong(
            location.substring(location.lastIndexOf('/') + 1)
        );

        assertEquals("Kira", petRepository.findById(petId)
            .orElseThrow().getName());
    }

    @Test
    void shouldReportInvalidPetFieldsAsJson() throws Exception {
        mockMvc.perform(post("/api/v1/pets")
                .with(httpBasic(ADMIN_EMAIL, PASSWORD))
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name":"Kira","species":"Dog","age":-2}
                    """))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value("validation_failed"))
            .andExpect(jsonPath("$.fieldErrors.age").exists());
    }

    @Test
    void shouldReturnJsonNotFoundForMissingPet() throws Exception {
        mockMvc.perform(get("/api/v1/pets/{id}", Long.MAX_VALUE)
                .with(httpBasic(USER_EMAIL, PASSWORD)))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.code").value("not_found"));
    }

    @Test
    void shouldAllowUserToCreateSupportRequest() throws Exception {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Nube", "Rabbit", 2, null)
        );

        mockMvc.perform(post("/api/v1/pets/{id}/requests", pet.getId())
                .with(httpBasic(USER_EMAIL, PASSWORD))
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"type":"WALK","description":"Needs a walk."}
                    """))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.status").value("OPEN"))
            .andExpect(jsonPath("$.petId").value(pet.getId().intValue()));
    }

    @Test
    void shouldRejectUnknownSupportType() throws Exception {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Milo", "Cat", 2, null)
        );

        mockMvc.perform(post("/api/v1/pets/{id}/requests", pet.getId())
                .with(httpBasic(USER_EMAIL, PASSWORD))
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"type":"VETERINARY","description":"Needs help."}
                    """))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.code").value("invalid_json"));
    }

    @Test
    void shouldRejectRequestUnderAnotherPet() throws Exception {
        Pet first = petRepository.saveAndFlush(
            new Pet("First", "Dog", 2, null)
        );
        Pet second = petRepository.saveAndFlush(
            new Pet("Second", "Cat", 3, null)
        );
        SupportRequest request = requestRepository.saveAndFlush(
            new SupportRequest(first, SupportType.WALK, "Needs a walk.")
        );

        mockMvc.perform(get(
                "/api/v1/pets/{petId}/requests/{requestId}",
                second.getId(), request.getId()
            ).with(httpBasic(USER_EMAIL, PASSWORD)))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.code").value("not_found"));
    }

    @Test
    void shouldCompleteOpenRequestWithoutResponseBody() throws Exception {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Luna", "Dog", 3, null)
        );
        SupportRequest request = requestRepository.saveAndFlush(
            new SupportRequest(pet, SupportType.COMPANY, "Needs company.")
        );

        mockMvc.perform(post(
                "/api/v1/pets/{petId}/requests/{requestId}/complete",
                pet.getId(), request.getId()
            ).with(httpBasic(ADMIN_EMAIL, PASSWORD)))
            .andExpect(status().isNoContent());

        assertEquals(RequestStatus.COMPLETED,
            requestRepository.findById(request.getId())
                .orElseThrow().getStatus());
    }

    @Test
    void shouldRejectCompletingCompletedRequest() throws Exception {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Kira", "Dog", 4, null)
        );
        SupportRequest request = new SupportRequest(
            pet, SupportType.FEEDING, "Needs feeding."
        );
        request.complete();
        request = requestRepository.saveAndFlush(request);

        mockMvc.perform(post(
                "/api/v1/pets/{petId}/requests/{requestId}/complete",
                pet.getId(), request.getId()
            ).with(httpBasic(ADMIN_EMAIL, PASSWORD)))
            .andExpect(status().isConflict())
            .andExpect(jsonPath("$.code").value("business_conflict"));
    }

    @Test
    void shouldReplaceRequestUsingExistingTransactionalService()
            throws Exception {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Toby", "Dog", 2, null)
        );
        SupportRequest previous = requestRepository.saveAndFlush(
            new SupportRequest(pet, SupportType.WALK, "Short walk.")
        );

        mockMvc.perform(put(
                "/api/v1/pets/{petId}/requests/{requestId}/replace",
                pet.getId(), previous.getId()
            ).with(httpBasic(ADMIN_EMAIL, PASSWORD))
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"type":"COMPANY","description":"Stay together."}
                    """))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.status").value("OPEN"));

        assertEquals(RequestStatus.CANCELLED,
            requestRepository.findById(previous.getId())
                .orElseThrow().getStatus());
    }

    @Test
    void shouldRejectDeletingPetWithRequestHistory() throws Exception {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Nina", "Cat", 4, null)
        );
        requestRepository.saveAndFlush(
            new SupportRequest(pet, SupportType.COMPANY, "Stay nearby.")
        );

        mockMvc.perform(delete("/api/v1/pets/{id}", pet.getId())
                .with(httpBasic(ADMIN_EMAIL, PASSWORD)))
            .andExpect(status().isConflict())
            .andExpect(jsonPath("$.code").value("business_conflict"));

        assertTrue(petRepository.existsById(pet.getId()));
    }

    @Test
    void shouldKeepCsrfProtectionOnWebForms() throws Exception {
        mockMvc.perform(post("/pets")
                .with(user(ADMIN_EMAIL).roles("ADMIN"))
                .param("name", "Kira")
                .param("species", "Dog")
                .param("age", "4"))
            .andExpect(status().isForbidden());
    }
}
```

**Esta clase de pruebas es extensa porque verifica un contrato integrado**, no porque un único método de producción deba realizar muchas tareas. Cada prueba presenta una intención concreta mediante su nombre.

### 7. Leer la preparación de cuentas

La anotación:

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
```

permite ejecutar el método `@BeforeAll` sobre la instancia de prueba, que Spring prepara mediante inyección de dependencias. De ese modo se pueden crear las cuentas de prueba una vez para esta clase.

La cuenta de usuario se crea con `new AppUser(...)`; el constructor asigna `USER` como rol inicial. La cuenta administrativa se crea primero como una cuenta ordinaria y después recibe el rol `ADMIN` mediante:

```java
jdbcTemplate.update(
    "UPDATE app_users SET role = 'ADMIN' WHERE id = ?",
    admin.getId()
);
```

**Esta operación solo prepara registros temporales de H2 para la suite.** No es un endpoint, no está disponible a visitantes ni forma parte del código productivo. La consulta es parametrizada; el identificador se transmite por separado en vez de concatenarse al SQL.

No añadas ese `UPDATE` al proceso de registro público. La aplicación real debe mantener los privilegios administrados por mecanismos controlados.

### 8. Comprender las pruebas de `Location`

El caso `shouldCreatePetWithLocationForAdmin` ejecuta una creación real a través de `MockMvc`. Después extrae el último segmento de la URL devuelta en la cabecera `Location` y consulta la entidad por ese identificador.

Esto es más confiable que suponer:

```java
assertEquals(1L, created.getId());
```

En una base relacional, los identificadores pueden tener saltos. El cliente debe utilizar el identificador **realmente devuelto**, no inventar uno según el número de pruebas ejecutadas.

### 9. ¿Qué demuestra `shouldRejectPetCreationForUserRole`?

La prueba envía una petición con **credenciales correctas** de una cuenta que solo tiene rol `USER`:

```java
.with(httpBasic(USER_EMAIL, PASSWORD))
```

Espera `403 Forbidden`. Por tanto, la prueba no se limita a detectar una contraseña inválida: comprueba que **una identidad autenticada puede seguir careciendo de permiso** para ejecutar una operación.

La prueba de HTTP Basic es diferente de un test que utiliza simplemente una autoridad simulada. Aquí se evalúa el rol que recupera el adaptador `DatabaseUserDetailsService` desde el repositorio de cuentas.

### 10. ¿Qué demuestra la prueba de transición?

En `shouldRejectCompletingCompletedRequest`, una solicitud se crea y se completa durante la preparación de datos:

```java
request.complete();
```

Luego se envía una segunda operación `complete` desde una cuenta `ADMIN`. La ruta está autorizada, pero el servicio y el dominio deben impedir la transición repetida. La respuesta esperada es `409`, no `403`.

La prueba evidencia que **tener el rol correcto no elimina las reglas de estado**. El mismo comportamiento debe observarse desde la interfaz HTML y desde la API.

### 11. Probar la separación de CSRF

La prueba `shouldKeepCsrfProtectionOnWebForms` utiliza una cuenta administrativa simulada para la web, pero omite deliberadamente el token CSRF. Espera `403`.

Por contraste, las pruebas REST que ejecutan `POST` con HTTP Basic y el rol adecuado **no envían un token CSRF** y pueden completar la operación. Esto demuestra el alcance de las dos cadenas de seguridad, no una desactivación global.

### 12. Ejecutar la suite

En Linux o macOS:

```bash
./mvnw test
```

En Windows PowerShell:

```powershell
.\mvnw.cmd test
```

Si necesitas ejecutar solo la clase REST con Maven Surefire, utiliza:

```bash
./mvnw -Dtest=RestApiIntegrationTests test
```

En PowerShell, si tu entorno interpreta de forma especial algún argumento, puedes pasarlo como una cadena:

```powershell
.\mvnw.cmd '-Dtest=RestApiIntegrationTests' test
```

Una suite exitosa muestra que los casos de prueba definidos cumplen sus expectativas **en el entorno de prueba**. No demuestra por sí sola que la URL de MariaDB, el usuario de base de datos o los permisos del servidor de desarrollo estén correctamente configurados.

### 13. Laboratorio de errores — Ubica la capa afectada

**Caso A: `HttpMessageNotReadableException` al enviar JSON.** El cliente envió un cuerpo ilegible o incompatible con el tipo del DTO. Revisa `Content-Type`, la sintaxis JSON y los valores de enums.

**Caso B: `MethodArgumentNotValidException`.** El JSON pudo convertirse a un objeto, pero una restricción como `@NotBlank` o `@Min` falló. Comprueba que existe `@Valid` junto a `@RequestBody`.

**Caso C: una respuesta API contiene HTML.** Revisa qué cadena de seguridad se seleccionó y qué `@ControllerAdvice` resolvió la excepción. Comprueba el alcance y la prioridad de `RestApiExceptionHandler`.

**Caso D: el usuario se autentica en la web, pero un cliente API sin cabecera recibe `401`.** Este es el comportamiento esperado de la configuración: la API no reutiliza automáticamente la sesión de autenticación de la web.

**Caso E: un `POST /api/v1/pets` devuelve `403` con una cuenta registrada.** Recuerda que el registro público asigna rol `USER`; esa ruta exige `ADMIN`. No alteres el formulario público para permitir que la persona escoja su rol.

**Caso F: se creó una solicitud con un estado inesperado.** Revisa que el controlador convierta `SupportRequestRequest` en `SupportRequestForm` y que la operación invoque `SupportRequestService.create(...)`. El estado inicial procede del constructor de `SupportRequest`.

**Caso G: `LazyInitializationException` al producir JSON.** Comprueba si se devolvió una entidad JPA directamente o si un mapper intenta navegar `request.pet.name` fuera del contexto de persistencia. Los DTO actuales evitan esa navegación.

**Caso H: una prueba devuelve `404` aunque el servicio aparentemente encontró algo.** Comprueba los dos identificadores de una solicitud anidada: `petId` y `requestId` deben corresponder a la misma relación.

**Caso I: las pruebas antiguas esperaban respuestas sin autenticación.** Las pruebas MVC del Laboratorio 11 ya debían adaptarse a la seguridad. No quites los filtros para que vuelvan a pasar: prepara las identidades necesarias y verifica por separado la autorización y el CSRF.

### 14. Reto integrador — Una API coherente para dos clientes

Realiza el siguiente recorrido utilizando **la aplicación web** y **la API REST**, y comprueba que ambos clientes trabajan sobre los mismos datos:

1. Inicia sesión como `ADMIN` en la web y registra una mascota de prueba.
2. Consulta esa mascota mediante `GET /api/v1/pets/{petId}` con HTTP Basic.
3. Utiliza una cuenta `USER` para crear una solicitud de apoyo mediante `POST` JSON.
4. Consulta la solicitud desde su listado HTML.
5. Intenta completar la solicitud desde la API usando `USER`; confirma el rechazo `403`.
6. Completa la solicitud desde la API usando `ADMIN`; confirma `204`.
7. Comprueba el nuevo estado desde la web HTML.
8. Intenta completar otra vez la misma solicitud; confirma `409`.
9. Intenta eliminar la mascota desde la API; confirma que el historial impide la eliminación.
10. Verifica las filas relacionadas mediante consultas SQL y reinicia la aplicación para comprobar persistencia.

**No modifiques la lógica de los servicios para fabricar resultados diferentes según el cliente.** La web y la API deben aplicar las mismas reglas del dominio.

### 15. Ampliación opcional — Filtrar sin romper el contrato existente

Diseña, como ejercicio, una consulta de mascotas por especie:

```text
GET /api/v1/pets?species=Dog
```

Antes de implementarla, define estas decisiones:

- ¿Qué ocurre si el parámetro `species` no se envía?
- ¿Se compararán los valores sin distinguir mayúsculas y minúsculas?
- ¿El parámetro permite espacios o valores vacíos?
- ¿Quién define la consulta: el repositorio o el controlador?
- ¿Qué contrato conserva el endpoint existente sin filtro?
- ¿Cómo comprobarás que el resultado no contiene mascotas de otra especie?

Una implementación mantenible debería ubicar la consulta en el repositorio y la coordinación en el servicio, dejando al controlador la interpretación del parámetro HTTP y la construcción de la respuesta. **No es obligatorio implementar este filtro para completar los objetivos principales del laboratorio.**

### 16. Ampliación opcional — Respuestas de error estandarizadas

Investiga el soporte de Spring MVC para `ProblemDetail`, asociado al formato de problemas HTTP definido por RFC 9457. Compara sus campos habituales (`type`, `title`, `status`, `detail`, `instance`) con nuestro `ApiErrorResponse`.

No sustituyas automáticamente el manejador de errores construido por una configuración copiada de otra versión. Determina antes qué contrato espera el cliente y cómo quedarán representados los errores de validación. En Spring Boot y Spring Framework existen comportamientos de autoconfiguración que pueden requerir adaptar el orden y alcance de los `@ControllerAdvice`.

### 17. Diagnóstico final — Explica una petición completa

Explica sin mirar las respuestas anteriores qué sucede cuando se envía:

```http
POST /api/v1/pets HTTP/1.1
Authorization: Basic <credentials>
Content-Type: application/json
```

```json
{
  "name": "Kira",
  "species": "Dog",
  "age": 4
}
```

Debes poder identificar cada paso:

```text
Cliente HTTP
    ↓
SecurityFilterChain de /api/**
    ↓
HTTP Basic + DatabaseUserDetailsService
    ↓
Comprobación del rol ADMIN
    ↓
PetRestController.create(...)
    ↓
PetRequest + Jakarta Validation
    ↓
PetApiMapper.toForm(...)
    ↓
PetService.create(...)
    ↓
PetRepository.save(...)
    ↓
JPA / Hibernate / MariaDB
    ↓
PetApiMapper.toResponse(...)
    ↓
201 Created + Location + JSON
```

Responde también: ¿qué cambia si el cliente envía una edad negativa?, ¿qué cambia si la cuenta tiene rol `USER`?, ¿qué cambia si no envía credenciales?, ¿qué cambia si se trata de una solicitud que ya fue cancelada?

**Comprobación final:** puedes razonar sobre el contrato HTTP, la conversión JSON, los permisos, las reglas de negocio y la persistencia como responsabilidades diferentes y verificables.

---
## Mapa final del proyecto

Los nuevos componentes REST se incorporan a la aplicación existente; **no reemplazan los controladores HTML, los servicios ni los repositorios**.

```text
petmatch-lab/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/com/example/petmatchlab/
│   │   │   ├── api/
│   │   │   │   ├── controller/
│   │   │   │   │   ├── PetRestController.java
│   │   │   │   │   └── SupportRequestRestController.java
│   │   │   │   ├── dto/
│   │   │   │   │   ├── PetRequest.java
│   │   │   │   │   ├── PetResponse.java
│   │   │   │   │   ├── SupportRequestRequest.java
│   │   │   │   │   ├── SupportRequestResponse.java
│   │   │   │   │   └── ApiErrorResponse.java
│   │   │   │   ├── error/
│   │   │   │   │   └── RestApiExceptionHandler.java
│   │   │   │   └── mapper/
│   │   │   │       ├── PetApiMapper.java
│   │   │   │       └── SupportRequestApiMapper.java
│   │   │   ├── controller/
│   │   │   │   ├── HomeController.java
│   │   │   │   ├── PetController.java
│   │   │   │   ├── SupportRequestController.java
│   │   │   │   ├── RegistrationController.java
│   │   │   │   ├── LoginController.java
│   │   │   │   ├── AdminController.java
│   │   │   │   └── PageErrorHandler.java
│   │   │   ├── dto/
│   │   │   │   ├── PetForm.java
│   │   │   │   ├── SupportRequestForm.java
│   │   │   │   └── RegistrationForm.java
│   │   │   ├── model/
│   │   │   │   ├── Pet.java
│   │   │   │   ├── SupportRequest.java
│   │   │   │   ├── SupportType.java
│   │   │   │   ├── RequestStatus.java
│   │   │   │   ├── AppUser.java
│   │   │   │   └── UserRole.java
│   │   │   ├── repository/
│   │   │   │   ├── PetRepository.java
│   │   │   │   ├── SupportRequestRepository.java
│   │   │   │   └── AppUserRepository.java
│   │   │   ├── security/
│   │   │   │   ├── SecurityConfig.java
│   │   │   │   └── DatabaseUserDetailsService.java
│   │   │   ├── service/
│   │   │   │   ├── PetService.java
│   │   │   │   ├── SupportRequestService.java
│   │   │   │   └── RegistrationService.java
│   │   │   └── exception/
│   │   │       ├── BusinessRuleException.java
│   │   │       └── ResourceNotFoundException.java
│   │   └── resources/
│   │       ├── application.yaml
│   │       ├── templates/
│   │       │   ├── pets/
│   │       │   ├── requests/
│   │       │   ├── auth/
│   │       │   └── fragments/
│   │       └── static/
│   └── test/
│       ├── java/com/example/petmatchlab/
│       │   ├── api/
│       │   │   ├── controller/RestApiIntegrationTests.java
│       │   │   └── mapper/PetApiMapperTests.java
│       │   ├── controller/
│       │   ├── security/
│       │   ├── service/
│       │   └── repository/
│       └── resources/application.yaml
└── mvnw
```

Esta vista muestra los archivos principales del recorrido. **No elimina** otros archivos existentes, como clases de inicio, fragmentos, hojas de estilo o pruebas construidas previamente.

### Dos entradas, una sola lógica de negocio

```text
                    ┌─────────────────────────────┐
                    │      PetMatch Mini          │
                    └─────────────────────────────┘
                           │             │
                     Web HTML         API JSON
                           │             │
                      /pets         /api/v1/pets
                           │             │
                   SecurityFilterChain seleccionada
                           │             │
                 PetController    PetRestController
                           │             │
                           └──────┬──────┘
                                  │
                              PetService
                                  │
                             PetRepository
                                  │
                                MariaDB
```

El recorrido equivalente de solicitudes utiliza `SupportRequestController` y `SupportRequestRestController`, pero converge en `SupportRequestService`, `SupportRequestRepository` y la misma base de datos.

## Revisión del funcionamiento

Utiliza esta tabla como lista de comprobación. **No basta con abrir una URL y observar texto JSON**: verifica los códigos, los permisos y los cambios reales cuando corresponda.

| Comprobación | Resultado esperado |
|---|---|
| La página `/pets` sigue funcionando con una sesión válida | HTML con Thymeleaf |
| `GET /api/v1/pets` con HTTP Basic válido | `200` y arreglo JSON |
| `GET /api/v1/pets` sin credenciales | `401` |
| `GET /api/v1/pets/{petId}` con id inexistente | `404` con `not_found` |
| `POST /api/v1/pets` con `USER` | `403`, sin inserción |
| `POST /api/v1/pets` con `ADMIN` y cuerpo válido | `201`, `Location` y nuevo registro |
| `POST /api/v1/pets` con `age=-1` | `400`, `validation_failed` |
| `POST /api/v1/pets` con JSON ilegible | `400`, `invalid_json` |
| `PUT /api/v1/pets/{petId}` con `ADMIN` | `200` y actualización persistida |
| `DELETE /api/v1/pets/{petId}` sin solicitudes y con `ADMIN` | `204` sin cuerpo |
| `DELETE` de mascota con historial | `409` con `business_conflict` |
| `GET` de solicitudes de mascota inexistente | `404` |
| `POST` de solicitud con `USER` y datos válidos | `201` y estado `OPEN` |
| `POST` de segunda solicitud abierta | `409`, sin duplicar la solicitud |
| `GET` de una solicitud bajo otra mascota | `404` |
| `POST` de completar con `USER` | `403` |
| `POST` de completar con `ADMIN` y estado `OPEN` | `204` y estado `COMPLETED` |
| `POST` de completar con estado `COMPLETED` | `409` |
| `PUT` de reemplazo válido con `ADMIN` | `201`, anterior `CANCELLED`, nueva `OPEN` |
| `POST /pets` web sin CSRF | `403` |
| Cuentas del registro público | Rol inicial `USER` |
| Consultas API de mascotas | No exponen entidades `AppUser` ni contraseñas |
| Reinicio de la aplicación conectada a MariaDB | Los registros y estados persisten |

## Errores frecuentes y cómo interpretarlos

### «El navegador ve HTML, pero el cliente API obtiene una redirección»

No significa necesariamente que el controlador REST esté mal escrito. Revisa si se selecciona la cadena web y si la ruta contiene el prefijo `/api/`. La diferencia entre el login de formularios y HTTP Basic está configurada en `SecurityConfig`.

### «Todo POST de la API devuelve 403»

Primero comprueba el rol del usuario, no añadas tokens CSRF indiscriminadamente. En la API, un `USER` solo puede crear solicitudes de apoyo y consultar recursos. Las operaciones de mascotas y los cambios de estado requieren `ADMIN`.

### «Un campo de JSON no aparece en la respuesta»

Compara el contrato de entrada y el de salida. El mapper no devuelve necesariamente todas las propiedades recibidas. Por ejemplo, `PetRequest` no permite establecer un `id` y `PetResponse` sí lo incorpora después de persistir la entidad.

### «El API devuelve 500 al convertir una relación»

Revisa que los controladores REST utilicen `SupportRequestResponse` en vez de devolver `SupportRequest` directamente. La serialización de entidades con relaciones perezosas puede causar errores o consultas adicionales.

### «Un administrador puede ejecutar una ruta, pero recibe 409»

La autorización fue exitosa, pero la regla de negocio rechazó la operación. Una solicitud completada no puede completarse de nuevo y una mascota con solicitudes relacionadas no puede eliminarse.

### «La petición REST crea una solicitud para la mascota equivocada»

Comprueba que el servicio reciba el `petId` de la URL y que busque la mascota existente. No sustituyas esa búsqueda por un objeto fabricado con un identificador recibido del cliente.

### «Las pruebas REST pasan, pero MariaDB no muestra los registros»

Las pruebas utilizan H2 y pueden crear y descartar datos dentro del entorno de prueba. Para verificar MariaDB debes iniciar la aplicación normal con las variables de entorno correctas y ejecutar la operación sobre ese proceso.

### «El servidor ignora un campo adicional enviado por el cliente»

Un DTO declara qué propiedades recibe el contrato. La configuración del conversor determina cómo tratar campos desconocidos. Ignorar un campo extra no significa que el servidor lo haya aplicado. Comprueba la respuesta y las reglas ejecutadas.

### «Los tests siempre dependen del identificador 1»

El identificador de una entidad persistente no es un contador que debas predecir. Recupéralo de `saveAndFlush(...)`, del cuerpo JSON o de la cabecera `Location`.

### «La aplicación tiene una API REST, así que está lista para Internet»

Una API funcional no equivale a una plataforma lista para un despliegue público. El ejemplo no incorpora todas las medidas de operación y seguridad necesarias para un entorno expuesto. En particular, HTTP Basic requiere HTTPS; el modelo actual no define pertenencia de recursos a usuarios y no incluye límites de consumo ni protección contra abuso. Reconoce estas limitaciones antes de utilizar la aplicación con datos reales.

## Cierre del laboratorio

Has incorporado una segunda interfaz HTTP a PetMatch Mini sin romper la aplicación web construida anteriormente. Ahora el mismo sistema puede ofrecer **HTML a través de Thymeleaf** y **JSON mediante controladores REST**.

La idea más importante no es la cantidad de endpoints creados. Es la **separación de responsabilidades**: las rutas y los formatos pertenecen a los controladores, los contratos de entrada y salida a los DTO, las conversiones a los mapeadores, las reglas de negocio a los servicios y entidades, y el almacenamiento a los repositorios y MariaDB.

También has comprobado que **autenticación, autorización y reglas del dominio no son sinónimos**. Una cuenta puede tener credenciales válidas y carecer del rol necesario; una cuenta autorizada puede intentar una transición prohibida; y un cliente puede enviar JSON correcto en su sintaxis, pero inválido para la operación.

Una API confiable debe expresar esas diferencias mediante **respuestas HTTP significativas**, no mediante excepciones ocultas o respuestas `200` para todos los resultados. Las pruebas permiten comprobar ese contrato sin sustituir la verificación de la aplicación conectada al motor real de base de datos.

## Referencias oficiales

- [Spring Boot 4.1 — Servlet Web Applications](https://docs.spring.io/spring-boot/reference/web/servlet.html): controladores REST, integración con Spring MVC y representación JSON.
- [Spring Boot — Testing Spring Boot Applications](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html): pruebas del contexto, integración MVC y servicios.
- [Spring Framework — `@ResponseBody` y `@RestController`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/responsebody.html): retorno de datos como cuerpo HTTP.
- [Spring Framework — `@RequestBody`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/requestbody.html): interpretación de cuerpos de entrada.
- [Spring Framework — `ResponseEntity`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/responseentity.html): cabeceras, códigos y cuerpos de respuesta.
- [Spring Framework — `@ControllerAdvice`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-advice.html): tratamiento compartido de excepciones.
- [Spring Framework — Error Responses](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html): gestión de errores HTTP y `ProblemDetail`.
- [Spring Security 7.1 — Java Configuration](https://docs.spring.io/spring-security/reference/servlet/configuration/java.html): múltiples cadenas de seguridad y orden de selección.
- [Spring Security 7.1 — Servlet Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html): `FilterChainProxy` y filtros.
- [Spring Security 7.1 — Session Management](https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html): política `STATELESS` y sesiones.
- [Spring Security 7.1 — CSRF](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html): riesgos de solicitudes entre sitios.
- [Spring Security 7.1 — HTTP Basic](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/basic.html): autenticación HTTP Basic.
- [Spring Security 7.1 — MockMvc with HTTP Basic](https://docs.spring.io/spring-security/reference/servlet/test/mockmvc/http-basic.html): pruebas de autenticación mediante cabeceras.
- [Spring Security 7.1 — Testing with CSRF](https://docs.spring.io/spring-security/reference/servlet/test/mockmvc/csrf.html): pruebas con tokens válidos e inválidos.
- [Spring Data JPA — Core Concepts](https://docs.spring.io/spring-data/jpa/reference/repositories/core-concepts.html): repositorios y operaciones persistentes.
- [Jakarta Bean Validation 3.1](https://jakarta.ee/specifications/bean-validation/3.1/): anotaciones de restricciones de entrada.
- [Java SE 21 — Record Classes](https://docs.oracle.com/en/java/javase/21/language/records.html): estructura y características de `record`.
- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110): métodos, códigos, cabeceras e idempotencia.
- [RFC 7617 — The Basic HTTP Authentication Scheme](https://www.rfc-editor.org/rfc/rfc7617): formato y consideraciones de seguridad de HTTP Basic.
- [RFC 9457 — Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457): formato de problemas HTTP para APIs.
