# Laboratorio 10 — Relaciones JPA, reglas de negocio, transacciones y estados con Spring Boot

Una aplicación que guarda mascotas en una base de datos ya puede resolver operaciones CRUD: registrar, consultar, modificar y eliminar información. Sin embargo, las aplicaciones también necesitan **relacionar registros** y **controlar qué operaciones tienen sentido**. No basta con que una fila exista: debemos saber a qué corresponde y qué acciones están permitidas sobre ella.

En este laboratorio ampliarás **PetMatch Mini** con solicitudes de apoyo para mascotas. Una mascota podrá tener varias solicitudes a lo largo de su historial, pero cada solicitud pertenecerá a una mascota concreta. Además, cada solicitud tendrá un estado que determinará si puede completarse, cancelarse o reemplazarse.

Seguirás utilizando Java 21, Spring Boot 4.1.1, Spring MVC, Thymeleaf, MariaDB, JPA, Hibernate y Spring Data JPA. El recorrido va desde las claves foráneas de SQL hasta los servicios transaccionales y las comprobaciones automatizadas. El objetivo es entender **por qué existe cada pieza**, no copiar anotaciones sin reconocer sus consecuencias.

| Práctica | Aprendizaje principal | Resultado comprobable |
|---|---|---|
| 10.1 | Relaciones entre tablas, cardinalidad y reglas del dominio | Diseñar la relación entre mascotas y solicitudes antes de escribir anotaciones |
| 10.2 | Asociaciones JPA, claves foráneas, enums y entidades | Persistir solicitudes relacionadas con una mascota real |
| 10.3 | Repositorios, consultas derivadas y límites del acceso a datos | Consultar solicitudes sin mezclar SQL con controladores |
| 10.4 | Servicios, formularios, estados y reglas de negocio | Registrar, consultar, completar y cancelar solicitudes correctamente |
| 10.5 | Transacciones, consistencia y rollback | Reemplazar una solicitud mediante dos modificaciones que forman una unidad |
| 10.6 | Excepciones, pruebas, depuración y reto integrador | Comprobar errores, asociaciones, cambios de estado y resultados persistidos |

> **Forma de trabajo:** en cada paso distingue entre la *regla del problema*, el *código Java que la representa* y el *mecanismo de persistencia que la ejecuta*. Cuando aparezca una anotación, identifica primero qué problema resuelve. Los fragmentos rotulados como **archivo completo** se escriben tal como aparecen; los ejemplos conceptuales se estudian, pero no se añaden al proyecto.

## Preparación — Reconocer la aplicación existente

Parte del proyecto `petmatch-lab` que ya permite administrar mascotas desde una interfaz web y guardarlas en MariaDB. Debes contar con:

- JDK 21 y Maven Wrapper (`mvnw` o `mvnw.cmd`).
- Spring Boot 4.1.1, Spring MVC, Thymeleaf, Bean Validation y Spring Data JPA.
- Una base MariaDB de práctica configurada mediante `application.yaml` y variables de entorno `DB_URL`, `DB_USERNAME` y `DB_PASSWORD`.
- `Pet`, `PetRepository`, `PetService`, `PetForm`, `PetController` y las plantillas `pets/` funcionando.
- Las dependencias de pruebas y la configuración de H2 dentro de `src/test/resources/application.yaml`.

Los archivos fundamentales del proyecto existente son:

```text
src/main/java/com/example/petmatchlab/
├── PetMatchLabApplication.java
├── controller/
│   ├── HomeController.java
│   └── PetController.java
├── dto/
│   └── PetForm.java
├── model/
│   └── Pet.java
├── repository/
│   └── PetRepository.java
└── service/
    └── PetService.java

src/main/resources/
├── application.yaml
├── static/css/application.css
└── templates/
    ├── fragments/
    └── pets/
        ├── list.html
        ├── detail.html
        └── form.html
```

**No crees un proyecto nuevo.** La ampliación utiliza la misma configuración y los mismos datos de mascotas. Conserva el CRUD existente: en este laboratorio solo modificarás la eliminación de mascotas para proteger las relaciones con las nuevas solicitudes.

### Revisión de dependencias

En `pom.xml` deben existir las dependencias equivalentes a estas (no dupliques una dependencia que ya esté declarada):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>test</scope>
</dependency>
```

Estas dependencias ya forman parte del recorrido de persistencia. Las relaciones, los servicios transaccionales y las consultas derivadas no requieren instalar una biblioteca adicional. **No reemplaces todo el `pom.xml` con este fragmento:** conserva su `parent`, su configuración de Java y el plugin de Spring Boot.

### Diagnóstico inicial

Antes de escribir código, responde:

1. ¿Por qué el identificador de una mascota es distinto de su nombre?
2. ¿Cómo se evita registrar una solicitud para una mascota que no existe?
3. Si guardas una solicitud y después cambias el nombre de su mascota, ¿debería perderse la relación?
4. ¿Qué diferencia hay entre que un formulario tenga datos válidos y que una operación esté permitida?
5. Si una operación debe modificar dos filas y falla al modificar la segunda, ¿qué resultado esperas de la primera?
6. ¿Por qué eliminar una mascota puede ser más delicado cuando otras tablas la referencian?

---

## Práctica 10.1 — Modelar relaciones y reglas antes de programar

### Objetivos

Al completar esta práctica podrás:

- diferenciar una relación conceptual de una referencia entre objetos Java;
- identificar relaciones **uno a muchos** y **muchos a uno**;
- explicar qué es una **clave foránea**;
- distinguir una restricción de datos de una regla de negocio;
- proponer estados y transiciones válidas para una solicitud.

### 1. Del objeto aislado al conjunto de objetos relacionados

Hasta ahora cada mascota se representa con un objeto `Pet` y una fila en `pets`:

```text
pets
-----------------------------------------------------
id | name | species | age | description
 1 | Luna | Dog     |   3 | Enjoys walks.
 2 | Milo | Cat     |   2 | Likes quiet places.
```

Ahora queremos registrar solicitudes como:

- Luna necesita un paseo.
- Luna necesita ayuda con su alimentación.
- Milo necesita compañía.

Podríamos guardar el nombre de la mascota como texto dentro de cada solicitud. Pero si cambiamos `Luna` por `Luna María`, ¿cómo sabríamos qué solicitudes pertenecían a la misma mascota? ¿Qué sucedería si existieran dos mascotas llamadas `Luna`?

**El nombre no es una referencia confiable.** Ya contamos con algo diseñado para identificar registros: la clave primaria `id`.

### 2. Dos tablas relacionadas

El diseño relacional será:

```text
pets
---------------------------------------------------
id (PK) | name  | species | age | description
       1 | Luna  | Dog     | 3   | ...
       2 | Milo  | Cat     | 2   | ...

support_requests
------------------------------------------------------------------
id (PK) | pet_id (FK) | type    | description        | status
      10 |           1 | WALK    | Morning walk       | OPEN
      11 |           1 | FEEDING | Help with feeding  | COMPLETED
      12 |           2 | COMPANY | Visit and play     | CANCELLED
```

Aquí aparecen dos tipos de identificador:

- **Clave primaria (PK):** `support_requests.id` identifica una solicitud individual.
- **Clave foránea (FK):** `support_requests.pet_id` contiene el identificador de una mascota existente en `pets.id`.

La clave foránea no guarda una copia de toda la mascota: guarda la **referencia a su fila**.

### 3. ¿Qué significa uno a muchos?

Una mascota puede tener varias solicitudes:

```text
Pet (id=1, Luna)
    |
    +---- SupportRequest (id=10)
    |
    +---- SupportRequest (id=11)
```

Observa la relación desde ambos lados:

| Pregunta | Respuesta | Nombre de la relación |
|---|---|---|
| ¿Cuántas solicitudes puede tener una mascota? | Cero, una o varias | Uno a muchos (`1:N`) |
| ¿A cuántas mascotas pertenece una solicitud? | Exactamente una | Muchos a uno (`N:1`) |

La cardinalidad expresa cuántos elementos pueden asociarse. **No obliga a guardar una lista dentro de `Pet`.** En una base relacional basta con que la tabla de solicitudes tenga `pet_id`. Más adelante veremos cómo se representa esa relación con JPA.

### 4. ¿Por qué no basta una columna de texto?

Imagina una solicitud que contiene:

```text
petName = "Luna"
```

Con ese valor, el programa no puede distinguir de forma confiable dos mascotas con el mismo nombre. Tampoco obtiene automáticamente una restricción que impida usar el nombre de una mascota inexistente.

En cambio:

```text
pet_id = 1
```

permite solicitar a MariaDB una relación con `pets.id`. Una restricción de clave foránea puede rechazar una solicitud que apunte a una mascota inexistente.

**Predice:** si existe una mascota con `id = 1`, ¿puede una solicitud referirse a `pet_id = 1` aunque la mascota cambie de nombre? Sí: su clave primaria se conserva durante la edición normal de sus datos.

### 5. Restricciones de la base y reglas de la aplicación

Necesitamos diferenciar:

**Regla de integridad de datos:**

> Cada solicitud debe pertenecer a una mascota existente.

Esta regla se refleja en la clave foránea y en el mapeo de JPA.

**Regla de negocio:**

> Una mascota no debe tener más de una solicitud abierta en el flujo normal de la aplicación.

Esta es una decisión del comportamiento del catálogo. No se cumple automáticamente porque `pet_id` sea una clave foránea. Debemos comprobarla en el servicio y reconocer después sus límites ante peticiones concurrentes.

**Otra regla de negocio:**

> Una solicitud completada o cancelada no puede volver a modificarse mediante las operaciones normales.

Esta regla depende del **estado**, no de una simple relación SQL.

### 6. Representar el ciclo de vida

Utilizaremos estos estados:

```text
                 +-------------------+
                 |       OPEN        |
                 +-------------------+
                    |            |
           complete |            | cancel
                    v            v
              COMPLETED      CANCELLED
```

Una solicitud comienza siempre en `OPEN`. Desde allí puede pasar a `COMPLETED` o `CANCELLED`. Los estados finales no tienen nuevas transiciones en este modelo.

| Estado actual | Acción | Resultado |
|---|---|---|
| OPEN | Completar | COMPLETED |
| OPEN | Cancelar | CANCELLED |
| COMPLETED | Completar | Operación no permitida |
| COMPLETED | Cancelar | Operación no permitida |
| CANCELLED | Completar | Operación no permitida |
| CANCELLED | Cancelar | Operación no permitida |

Este comportamiento es una **máquina de estados sencilla**. No requiere una biblioteca especial: es posible expresarlo mediante un `enum`, métodos Java y excepciones.

### 7. ¿Qué ocurre al eliminar una mascota?

Si una mascota tiene solicitudes, sus filas contienen `pet_id` que la referencia. Eliminar esa mascota sin una política clara puede producir un error de integridad referencial o, con otras configuraciones, eliminar accidentalmente información relacionada.

En esta aplicación utilizaremos una regla explícita:

> Una mascota con solicitudes registradas no puede eliminarse, aunque sus solicitudes estén completadas o canceladas.

La razón es conservar el historial. La interfaz debe informar que existe una restricción, y el servicio debe comprobarla **antes** de ejecutar la eliminación.

No configuraremos un borrado en cascada. Eso permite aprender que una relación no significa que la eliminación de una entidad deba destruir a las demás.

### 8. Interpretar una consulta con JOIN

Esta consulta SQL es conceptual por ahora; funcionará cuando exista la tabla nueva:

```sql
SELECT
    r.id,
    p.name AS pet_name,
    r.type,
    r.status
FROM support_requests AS r
JOIN pets AS p ON p.id = r.pet_id
ORDER BY r.id;
```

Descompón la consulta:

- `FROM support_requests AS r`: comienza con las solicitudes.
- `JOIN pets AS p`: incorpora las mascotas relacionadas.
- `ON p.id = r.pet_id`: especifica qué filas deben coincidir.
- `SELECT`: elige qué columnas aparecerán en el resultado.

La base de datos realiza la combinación; no necesitamos guardar el nombre de la mascota en cada solicitud.

### Ejercicio — Diseña antes de implementar

Dibuja dos tablas con tres mascotas y cinco solicitudes. Haz que dos solicitudes correspondan a una misma mascota. Después explica:

1. Cuál es la clave primaria de cada tabla.
2. Dónde se encuentra la clave foránea.
3. Por qué una solicitud no debería tener dos valores de `pet_id`.
4. Si el estado `OPEN` puede reemplazarse por `COMPLETED` mediante un formulario de edición de mascota.
5. Qué responderías al intentar eliminar una mascota que conserva solicitudes históricas.

### Depuración — Identifica el error conceptual

- Alguien propone usar el nombre de la mascota como clave foránea. ¿Qué problema aparece cuando dos mascotas comparten nombre?
- Otro propone duplicar todos los datos de `Pet` en cada solicitud. ¿Qué información quedaría desactualizada si cambia la mascota?
- Una tercera propuesta intenta eliminar automáticamente las solicitudes al eliminar una mascota. ¿Respeta la regla de conservar el historial?

### Comprobación

Debes poder explicar la diferencia entre `id` y `pet_id`, identificar el lado `1` y el lado `N`, describir una clave foránea y diferenciar una regla de integridad de una regla de negocio.

---

## Práctica 10.2 — Implementar la relación con entidades JPA

### Objetivos

Al finalizar podrás:

- representar una relación muchos a uno con `@ManyToOne` y `@JoinColumn`;
- comprender `FetchType.LAZY` sin confundirlo con una consulta;
- persistir enumeraciones mediante `@Enumerated(EnumType.STRING)`;
- explicar para qué sirve un constructor protegido sin argumentos en una entidad;
- comprobar en MariaDB las columnas y la clave foránea resultantes.

### 1. Crear el tipo de solicitud

Antes de escribir la entidad, define los valores permitidos para el tipo de apoyo. Crea el **archivo completo** `src/main/java/com/example/petmatchlab/model/SupportType.java`:

```java
package com.example.petmatchlab.model;

public enum SupportType {
    WALK("Paseo"),
    FEEDING("Alimentación"),
    COMPANY("Compañía");

    private final String label;

    SupportType(String label) {
        this.label = label;
    }

    public String getLabel() {
        return label;
    }
}
```

**¿Qué es un `enum`?** Es un tipo Java que representa un conjunto limitado de valores conocidos. `SupportType` solo admite `WALK`, `FEEDING` o `COMPANY`. No admite cualquier texto arbitrario.

El `label` permite mostrar un nombre comprensible en la interfaz. El nombre técnico del valor sigue siendo estable en Java.

**Predice:** ¿qué ocurriría si alguien escribiera `SupportType.VET` sin declarar `VET` en el enum? El proyecto no compilaría porque ese valor no existe.

### 2. Crear el estado de las solicitudes

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/model/RequestStatus.java`:

```java
package com.example.petmatchlab.model;

public enum RequestStatus {
    OPEN("Abierta"),
    COMPLETED("Completada"),
    CANCELLED("Cancelada");

    private final String label;

    RequestStatus(String label) {
        this.label = label;
    }

    public String getLabel() {
        return label;
    }
}
```

**Diferencia importante:** `SupportType` describe **qué ayuda se solicita**; `RequestStatus` describe **en qué situación se encuentra la solicitud**. Que ambos sean enums no significa que expresen lo mismo.

### 3. Añadir una excepción de reglas del dominio

Necesitaremos señalar operaciones que no están permitidas. Crea la carpeta `exception` y el **archivo completo** `src/main/java/com/example/petmatchlab/exception/BusinessRuleException.java`:

```java
package com.example.petmatchlab.exception;

public class BusinessRuleException extends RuntimeException {

    public BusinessRuleException(String message) {
        super(message);
    }
}
```

Esto es Java básico aplicado al dominio: la clase hereda de `RuntimeException` y permite comunicar por qué una operación no tiene sentido.

No significa que la aplicación haya fallado por una conexión a MariaDB. Una excepción de negocio expresa una restricción conocida del problema.

### 4. Crear la entidad relacionada

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/model/SupportRequest.java`:

```java
package com.example.petmatchlab.model;

import com.example.petmatchlab.exception.BusinessRuleException;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.EnumType;
import jakarta.persistence.Enumerated;
import jakarta.persistence.FetchType;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.JoinColumn;
import jakarta.persistence.ManyToOne;
import jakarta.persistence.Table;

@Entity
@Table(name = "support_requests")
public class SupportRequest {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "pet_id", nullable = false)
    private Pet pet;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private SupportType type;

    @Column(nullable = false, length = 500)
    private String description;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private RequestStatus status;

    protected SupportRequest() {
    }

    public SupportRequest(Pet pet, SupportType type, String description) {
        if (pet == null || type == null
            || description == null || description.isBlank()) {
            throw new IllegalArgumentException("Invalid support request data");
        }

        this.pet = pet;
        this.type = type;
        this.description = description.strip();
        this.status = RequestStatus.OPEN;
    }

    public void complete() {
        ensureOpen();
        status = RequestStatus.COMPLETED;
    }

    public void cancel() {
        ensureOpen();
        status = RequestStatus.CANCELLED;
    }

    private void ensureOpen() {
        if (status != RequestStatus.OPEN) {
            throw new BusinessRuleException(
                "La solicitud no está abierta y no puede cambiar de estado."
            );
        }
    }

    public boolean isOpen() {
        return status == RequestStatus.OPEN;
    }

    public Long getId() {
        return id;
    }

    public Pet getPet() {
        return pet;
    }

    public SupportType getType() {
        return type;
    }

    public String getDescription() {
        return description;
    }

    public RequestStatus getStatus() {
        return status;
    }
}
```

Este archivo reúne dos conceptos diferentes: **el mapeo a la base de datos** y **el comportamiento propio de una solicitud**. Examínalos por separado.

### 5. Interpretar la clave foránea en Java

Observa:

```java
@ManyToOne(fetch = FetchType.LAZY, optional = false)
@JoinColumn(name = "pet_id", nullable = false)
private Pet pet;
```

Cada parte tiene un significado:

| Elemento | Función |
|---|---|
| `Pet pet` | En Java, la solicitud referencia un objeto `Pet` |
| `@ManyToOne` | Varias solicitudes pueden pertenecer a una misma mascota |
| `@JoinColumn(name = "pet_id")` | La tabla `support_requests` contiene la columna que referencia a `pets` |
| `nullable = false` | La columna no admite valores SQL `NULL` |
| `optional = false` | La asociación se declara obligatoria en el modelo JPA |
| `FetchType.LAZY` | Indica que no es necesario cargar inmediatamente la mascota al cargar una solicitud |

**No existe una columna SQL que almacene un objeto Java completo.** JPA transforma la referencia `Pet pet` en el valor de clave foránea `pet_id`.

**Cuidado con `LAZY`:** es una estrategia de carga. Si intentas acceder a una relación perezosa fuera de una sesión de persistencia disponible, puedes obtener un error de inicialización. Por eso el controlador y las vistas de este laboratorio trabajan con los datos de la mascota recuperada expresamente, en vez de depender de `request.pet.name` después de cerrar la transacción.

### 6. ¿Por qué el constructor recibe un objeto `Pet`?

Este fragmento corresponde a la creación de una solicitud:

```java
SupportRequest request = new SupportRequest(
    existingPet,
    SupportType.WALK,
    "Necesita un paseo corto."
);
```

El argumento `existingPet` debe ser una entidad que el servicio haya recuperado mediante `PetRepository`. No es suficiente que el navegador envíe `pet_id = 999` y que el programa invente una nueva mascota con ese identificador.

**El formulario envía el identificador; el servicio consulta y obtiene la entidad existente.** Esa separación permite controlar referencias inexistentes y ubicar la validación en la capa correspondiente.

### 7. Enums persistidos como texto

Observa:

```java
@Enumerated(EnumType.STRING)
@Column(nullable = false, length = 20)
private RequestStatus status;
```

Con `EnumType.STRING`, el valor se almacena como texto similar a `OPEN`, `COMPLETED` o `CANCELLED`.

Existe otra opción, `EnumType.ORDINAL`, que almacena el índice numérico del valor. Eso puede resultar peligroso si luego cambias el orden de las constantes. Para este modelo utilizamos `STRING` porque los valores almacenados son más fáciles de reconocer y menos dependientes del orden del código.

El texto que muestra la interfaz (`"Abierta"`) no tiene por qué coincidir con el que se almacena (`"OPEN"`). Esta distinción permite mantener nombres técnicos estables y presentar etiquetas legibles.

### 8. Una entidad no es un formulario

`SupportRequest` es una entidad persistente. Contiene campos que el navegador **no debería controlar directamente**:

```text
id       → generado por la base de datos
pet      → buscada y verificada por el servicio
status   → asignado por el dominio y sus transiciones
type     → seleccionado por la persona
description → escrito por la persona
```

Más adelante crearemos un `SupportRequestForm` que contenga solamente `type` y `description`.

Esta separación evita que un usuario pueda enviar libremente `status=COMPLETED` o reemplazar la relación de una entidad existente mediante un formulario diseñado para otra operación.

### 9. ¿Hace falta `@OneToMany` en `Pet`?

Existe una anotación complementaria:

```java
@OneToMany(mappedBy = "pet")
private List<SupportRequest> requests = new ArrayList<>();
```

**Este fragmento es conceptual: no lo agregues a `Pet.java` en esta práctica.**

`@OneToMany` permitiría recorrer una colección de solicitudes desde el objeto `Pet`. El atributo `mappedBy = "pet"` señala que la asociación se mantiene mediante el campo `pet` de `SupportRequest`; no crea una segunda clave foránea.

Sin embargo, **no necesitas una relación bidireccional** para consultar todas las solicitudes de una mascota. Puedes preguntárselo directamente al repositorio de solicitudes utilizando el identificador de la mascota.

Elegimos una asociación unidireccional `SupportRequest → Pet` porque permite explicar la cardinalidad con menos estado mutable y evita exponer colecciones perezosas a las plantillas. No confundas esta simplificación con una limitación de JPA.

### 10. Cascada no es sinónimo de relación

La anotación `@ManyToOne` no indica automáticamente que al eliminar una mascota deban eliminarse todas sus solicitudes.

JPA permite configurar operaciones en cascada (`cascade`) y, para determinadas relaciones, eliminación de huérfanos (`orphanRemoval`). Pero esas opciones deben responder a una política explícita del dominio.

Aquí las solicitudes constituyen historial. Por eso **no añadiremos `CascadeType.REMOVE` ni `orphanRemoval = true`**. La eliminación de mascotas se bloqueará cuando existan solicitudes relacionadas.

### 11. Crear el esquema en MariaDB

Conserva la configuración de desarrollo usada previamente:

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: update
    open-in-view: false
    show-sql: true
```

Este ejemplo recuerda las propiedades relevantes; no borres otras propiedades válidas que ya tengas, como `spring.application.name` o `server.port`.

Inicia la aplicación usando las mismas variables de entorno que ya configuraste. Como ahora existe una segunda entidad JPA, Hibernate puede intentar crear la tabla `support_requests` y su relación con `pets`.

Consulta MariaDB:

```sql
USE petmatch_lab;

SHOW TABLES;

DESCRIBE support_requests;

SHOW CREATE TABLE support_requests;
```

Busca estas características en el resultado:

- Una columna `id` con identificador generado.
- Una columna `pet_id` que no admite nulos.
- Columnas `type`, `description` y `status`.
- Una referencia de clave foránea desde `pet_id` hasta `pets.id`.

La sintaxis exacta del SQL generado depende del dialecto y la versión de Hibernate/MariaDB. **No copies una sentencia hipotética como si fuera el esquema real:** comprueba el resultado de `SHOW CREATE TABLE`.

> **Precaución:** `ddl-auto: update` es una comodidad de desarrollo para evolucionar un esquema introductorio. No es un sistema de migraciones versionadas y no garantiza transformar cualquier estructura previa de forma segura. No cambies a `create` o `create-drop` sobre una base con información que quieras conservar.

### Ejercicio — Sigue una asociación

Imagina que una solicitud tiene `id = 15` y `pet_id = 3`. Responde:

1. ¿Qué valor aparece en `support_requests.pet_id`?
2. ¿Qué tipo de Java devuelve `request.getPet()`?
3. ¿Cuál sería la consecuencia de eliminar la fila de la mascota `3` sin resolver la relación?
4. ¿Qué ocurre si se cambia el nombre de la mascota `3`?
5. ¿Qué representa `mappedBy` si posteriormente agregas el lado `@OneToMany`?

### Depuración — Errores de mapeo

- `Not a managed type`: revisa que `SupportRequest` tenga `@Entity`, que esté en un paquete escaneado y que uses anotaciones `jakarta.persistence`.
- `Unknown column 'pet_id'`: compara el esquema real con `@JoinColumn(name = "pet_id")`.
- Error de clave foránea: comprueba que el `Pet` exista y que los tipos de identificador coincidan.
- Error por valor de enum: verifica que los textos guardados coincidan con los nombres de las constantes declaradas.

### Comprobación

Debes poder ubicar la clave foránea tanto en SQL como en Java, explicar por qué la asociación se carga de forma perezosa y distinguir los valores de un enum de su etiqueta para la interfaz.

---

## Práctica 10.3 — Consultar relaciones mediante repositorios

### Objetivos

Al terminar podrás:

- crear un repositorio Spring Data JPA para una entidad relacionada;
- interpretar consultas derivadas por propiedades anidadas;
- buscar solicitudes por mascota y estado;
- diferenciar `find`, `exists` y `count`;
- reconocer cuándo una consulta pertenece al repositorio y cuándo una decisión pertenece al servicio.

### 1. Un repositorio por responsabilidad de persistencia

Ya existe:

```java
public interface PetRepository extends JpaRepository<Pet, Long> {
}
```

Ese repositorio se ocupa de la entidad `Pet`. No necesitamos agregar allí todos los métodos de solicitudes, porque tenemos otra entidad con sus propias consultas.

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/repository/SupportRequestRepository.java`:

```java
package com.example.petmatchlab.repository;

import java.util.List;
import java.util.Optional;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.petmatchlab.model.RequestStatus;
import com.example.petmatchlab.model.SupportRequest;

public interface SupportRequestRepository
    extends JpaRepository<SupportRequest, Long> {

    List<SupportRequest> findByPet_IdOrderByIdDesc(Long petId);

    Optional<SupportRequest> findByIdAndPet_Id(Long id, Long petId);

    boolean existsByPet_Id(Long petId);

    boolean existsByPet_IdAndStatus(Long petId, RequestStatus status);
}
```

La interfaz no contiene ninguna implementación manual. Spring Data JPA interpreta los nombres y crea las consultas necesarias.

### 2. ¿Qué significa el guion bajo en `Pet_Id`?

Observa:

```java
findByPet_IdOrderByIdDesc(Long petId)
```

No existe un atributo llamado `pet_id` en la entidad Java. Existe:

```java
private Pet pet;
```

Y dentro de `Pet` existe:

```java
private Long id;
```

El nombre de la consulta recorre la relación:

```text
SupportRequest.pet.id
```

El guion bajo (`_`) delimita explícitamente el recorrido de propiedades. Por eso el método puede interpretarse como:

> Obtener solicitudes cuyo `pet.id` coincide con el valor recibido y ordenarlas por `id` descendente.

No se trata de un nombre de columna SQL incrustado en el método; las consultas derivadas se construyen a partir de **propiedades de entidades Java**.

### 3. `findByIdAndPet_Id`: validar pertenencia

Este método representa dos condiciones:

```java
Optional<SupportRequest> findByIdAndPet_Id(Long id, Long petId);
```

Su significado es:

```text
buscar solicitud
  con id solicitado
  Y cuyo pet.id coincide con la mascota de la ruta
```

Imagina la dirección:

```text
/pets/1/requests/25
```

Si la solicitud `25` pertenece a la mascota `2`, **no debe tratarse como una solicitud de la mascota `1`**. La consulta combinada evita que el servicio actúe sobre una asociación distinta de la indicada.

Una consulta que no encuentra coincidencia devuelve `Optional.empty()`. Eso permite al servicio responder que no se encontró el recurso solicitado dentro de ese contexto.

### 4. `existsBy...`: preguntar sin recuperar toda la lista

Compara:

```java
boolean existsByPet_Id(Long petId);
```

con:

```java
List<SupportRequest> findByPet_IdOrderByIdDesc(Long petId);
```

La primera pregunta si existe al menos una solicitud asociada. La segunda recupera las solicitudes y las devuelve como objetos.

Si solo necesitas decidir si se permite eliminar una mascota, `existsByPet_Id(...)` comunica mejor la intención y evita recuperar objetos que no se van a utilizar.

Para la regla de una solicitud abierta utilizamos:

```java
boolean existsByPet_IdAndStatus(Long petId, RequestStatus status);
```

El servicio podrá preguntar:

```java
requestRepository.existsByPet_IdAndStatus(petId, RequestStatus.OPEN)
```

y decidir si admite una nueva solicitud.

**El repositorio responde una pregunta sobre datos; el servicio interpreta la respuesta como una regla de negocio.**

### 5. Por qué no agregamos SQL al controlador

Esta separación sigue siendo válida:

```text
Browser
  |
  v
SupportRequestController  → rutas y formularios
  |
  v
SupportRequestService     → reglas y operaciones
  |
  v
SupportRequestRepository  → consultas y persistencia
  |
  v
Hibernate → MariaDB
```

Si el controlador ejecutara consultas SQL directamente, tendría responsabilidades de transporte HTTP, acceso a datos y decisiones de negocio al mismo tiempo. Eso complicaría el mantenimiento y las pruebas.

### 6. Comprobar SQL después de guardar datos

Cuando existan solicitudes, consulta:

```sql
SELECT
    r.id,
    r.pet_id,
    p.name AS pet_name,
    r.type,
    r.status
FROM support_requests AS r
INNER JOIN pets AS p ON p.id = r.pet_id
ORDER BY r.pet_id, r.id DESC;
```

Compara el resultado con los listados que construiremos en la interfaz. Deben coincidir los identificadores y las relaciones, incluso si la forma de presentación es diferente.

No construyas SQL concatenando valores recibidos de formularios. Spring Data JPA realiza la parametrización de las consultas derivadas. Una consulta con parámetros es conceptualmente distinta de formar una sentencia mediante concatenación de texto no confiable.

### Ejercicio — Escoge el método correcto

Indica qué método utilizarías en cada caso:

1. Mostrar todas las solicitudes de la mascota `7`.
2. Comprobar si la mascota `7` tiene alguna solicitud histórica.
3. Buscar la solicitud `32` únicamente si pertenece a la mascota `7`.
4. Comprobar si la mascota `7` ya tiene una solicitud `OPEN`.

Después escribe una frase que explique por qué el controlador no debería elegir directamente si una mascota puede eliminarse.

### Depuración — Consulta no reconocida

Si Spring muestra una excepción al iniciar la aplicación indicando que no encuentra una propiedad para una consulta derivada, revisa el nombre del método, los atributos reales de la entidad y la separación con `_`.

Por ejemplo, `findByAnimalId(...)` no corresponde a nuestro modelo porque `SupportRequest` no tiene un campo `animal`; tiene un campo `pet`.

### Comprobación

Debes poder leer los cuatro métodos del repositorio en español sin necesitar memorizar el nombre de una consulta SQL concreta.

---
## Práctica 10.4 — Servicios, formularios y reglas de negocio

### Objetivos

Al finalizar esta práctica podrás:

- recibir un formulario sin exponer directamente una entidad JPA;
- comprobar que una mascota existe antes de crearle una solicitud;
- establecer el estado inicial desde el dominio, no desde el navegador;
- impedir una segunda solicitud abierta en el flujo habitual;
- ejecutar transiciones de estado válidas e informar sobre las inválidas;
- mantener las reglas en `Service` aunque la operación provenga de una página web.

### 1. Una operación es más que guardar un objeto

La creación de una solicitud sigue esta secuencia:

```text
Navegador
    |
    | POST /pets/7/requests
    v
SupportRequestController
    |
    | SupportRequestForm
    v
SupportRequestService
    |
    +--> buscar Pet 7
    +--> comprobar que no tenga una solicitud OPEN
    +--> construir SupportRequest con estado OPEN
    +--> guardar mediante el repositorio
    |
    v
SupportRequestRepository
    |
    v
MariaDB
```

El controlador comprende HTTP y las plantillas; el servicio comprende las reglas del caso de uso. El controlador puede solicitar datos al servicio para preparar una vista, pero no debe apropiarse de las decisiones centrales del dominio ni acceder directamente a las consultas de persistencia.

### 2. Definir los datos que recibe el formulario

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/dto/SupportRequestForm.java`:

```java
package com.example.petmatchlab.dto;

import com.example.petmatchlab.model.SupportType;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

public class SupportRequestForm {

    @NotNull(message = "Selecciona el tipo de apoyo")
    private SupportType type;

    @NotBlank(message = "Escribe una descripción")
    @Size(max = 500, message = "La descripción no puede superar 500 caracteres")
    private String description;

    public SupportType getType() {
        return type;
    }

    public void setType(SupportType type) {
        this.type = type;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```

Este DTO no recibe `id`, `pet` ni `status`:

- El **id** se genera al persistir la entidad.
- La **mascota** se recupera usando el `petId` de la ruta y se verifica en el servicio.
- El **estado** lo establece el constructor de `SupportRequest`.
- El usuario únicamente elige el **tipo de apoyo** y escribe la **descripción**.

**Predice:** si alguien agrega manualmente un campo oculto `<input name="status" value="COMPLETED">` al HTML, ¿debería poder completar una solicitud al crearla? No: `SupportRequestForm` no expone esa propiedad y la operación de creación utiliza `RequestStatus.OPEN`.

### 3. Diferenciar recurso inexistente y operación inválida

Ya contamos con `BusinessRuleException`. Crearemos otra excepción para referencias inexistentes. Crea el **archivo completo** `src/main/java/com/example/petmatchlab/exception/ResourceNotFoundException.java`:

```java
package com.example.petmatchlab.exception;

public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

Estas dos excepciones comunican fallos distintos:

| Situación | Excepción | Respuesta HTTP que ofreceremos |
|---|---|---|
| No existe una mascota o una solicitud dentro de la ruta indicada | `ResourceNotFoundException` | 404 Not Found |
| La solicitud existe, pero su estado prohíbe la operación | `BusinessRuleException` | 409 Conflict |

**Un dato válido puede producir una operación inválida.** Una descripción correcta no autoriza crear una segunda solicitud `OPEN` para una mascota que ya tiene una abierta.

### 4. Implementar el servicio

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/service/SupportRequestService.java`:

```java
package com.example.petmatchlab.service;

import java.util.List;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.petmatchlab.dto.SupportRequestForm;
import com.example.petmatchlab.exception.BusinessRuleException;
import com.example.petmatchlab.exception.ResourceNotFoundException;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.model.RequestStatus;
import com.example.petmatchlab.model.SupportRequest;
import com.example.petmatchlab.repository.PetRepository;
import com.example.petmatchlab.repository.SupportRequestRepository;

@Service
public class SupportRequestService {

    private final PetRepository petRepository;
    private final SupportRequestRepository requestRepository;

    public SupportRequestService(
        PetRepository petRepository,
        SupportRequestRepository requestRepository
    ) {
        this.petRepository = petRepository;
        this.requestRepository = requestRepository;
    }

    @Transactional(readOnly = true)
    public List<SupportRequest> findAllForPet(long petId) {
        findPet(petId);
        return requestRepository.findByPet_IdOrderByIdDesc(petId);
    }

    @Transactional(readOnly = true)
    public SupportRequest findOneForPet(long petId, long requestId) {
        return findRequest(petId, requestId);
    }

    @Transactional
    public SupportRequest create(long petId, SupportRequestForm form) {
        validateForm(form);
        Pet pet = findPet(petId);

        if (requestRepository.existsByPet_IdAndStatus(
            petId, RequestStatus.OPEN
        )) {
            throw new BusinessRuleException(
                "Esta mascota ya tiene una solicitud abierta."
            );
        }

        SupportRequest request = new SupportRequest(
            pet,
            form.getType(),
            form.getDescription().strip()
        );

        return requestRepository.save(request);
    }

    @Transactional
    public void complete(long petId, long requestId) {
        SupportRequest request = findRequest(petId, requestId);
        request.complete();
    }

    @Transactional
    public void cancel(long petId, long requestId) {
        SupportRequest request = findRequest(petId, requestId);
        request.cancel();
    }

    private Pet findPet(long petId) {
        return petRepository.findById(petId)
            .orElseThrow(() -> new ResourceNotFoundException(
                "No existe la mascota indicada."
            ));
    }

    private SupportRequest findRequest(long petId, long requestId) {
        return requestRepository.findByIdAndPet_Id(requestId, petId)
            .orElseThrow(() -> new ResourceNotFoundException(
                "No existe esa solicitud para la mascota indicada."
            ));
    }

    private void validateForm(SupportRequestForm form) {
        if (form == null || form.getType() == null
            || form.getDescription() == null
            || form.getDescription().isBlank()
            || form.getDescription().length() > 500) {
            throw new IllegalArgumentException("Invalid support request form");
        }
    }
}
```

Este servicio ya puede utilizarse desde Java o desde un controlador. No importa de qué pantalla proceda la operación: las reglas se encuentran en un único lugar.

### 5. Leer el servicio con calma

**Primero: inyección de dependencias.**

```java
public SupportRequestService(
    PetRepository petRepository,
    SupportRequestRepository requestRepository
) {
    this.petRepository = petRepository;
    this.requestRepository = requestRepository;
}
```

Spring administra los repositorios y proporciona sus implementaciones al constructor. El servicio no construye una conexión JDBC ni ejecuta `new PetRepository()`; además, una interfaz no puede instanciarse directamente.

**Segundo: comprobar la mascota.**

```java
Pet pet = findPet(petId);
```

Un identificador recibido desde el navegador no demuestra que exista la mascota. Por eso el servicio obtiene la entidad real con `PetRepository` y responde de forma explícita cuando no existe.

**Tercero: comprobar otra solicitud abierta.**

```java
requestRepository.existsByPet_IdAndStatus(petId, RequestStatus.OPEN)
```

El repositorio indica si existe el dato. El servicio decide que, en ese caso, la operación `create` está prohibida.

**Cuarto: crear la entidad.**

```java
new SupportRequest(pet, form.getType(), form.getDescription().strip())
```

No existe un parámetro de estado porque el propio constructor asigna `OPEN`.

**Quinto: guardar.**

```java
requestRepository.save(request)
```

El repositorio persiste la nueva entidad. Al terminar satisfactoriamente la transacción, los cambios se confirman en MariaDB.

### 6. ¿Por qué dos niveles de validación?

El DTO contiene restricciones como `@NotNull`, `@NotBlank` y `@Size`. Spring MVC las aplica mediante `@Valid` al procesar el formulario.

El servicio también realiza una comprobación básica con `validateForm(...)`. Esto protege su contrato cuando alguien lo llama directamente desde una prueba, otro servicio u otra entrada que no utilice el mismo formulario web.

No confundas ambas comprobaciones con reglas del dominio:

```text
¿El texto está vacío?                 → validez de los datos
¿Tiene más de 500 caracteres?         → validez de los datos
¿La mascota existe?                   → referencia a un recurso
¿Ya existe una solicitud abierta?     → regla de negocio
¿Puede completarse una cancelada?     → transición de estado
```

La anotación `@Valid` no sabe si existe otra solicitud abierta. Esa pregunta pertenece al servicio y necesita consultar el repositorio.

### 7. La transición vive dentro de la entidad

Observa el método del servicio:

```java
@Transactional
public void complete(long petId, long requestId) {
    SupportRequest request = findRequest(petId, requestId);
    request.complete();
}
```

La operación de cambio de estado se concentra en el objeto:

```java
public void complete() {
    ensureOpen();
    status = RequestStatus.COMPLETED;
}
```

Esto evita que cada consumidor del servicio deba recordar por separado la misma comprobación.

**¿Por qué no aparece `requestRepository.save(request)`?** Dentro de una transacción JPA, la entidad recuperada mediante el repositorio está administrada por el contexto de persistencia. Hibernate puede detectar el cambio y sincronizarlo al confirmar la transacción; este comportamiento se conoce como **dirty checking**.

La anotación `@Transactional` establece el límite de la unidad de trabajo. La veremos con mayor detalle en la siguiente práctica. Fuera de un contexto de persistencia adecuado, modificar un objeto Java no garantiza guardar sus cambios en MariaDB.

### 8. Mostrar las solicitudes de una mascota

Crea la carpeta `src/main/resources/templates/requests/`.

La interfaz utilizará las siguientes rutas para mostrar páginas y procesar operaciones:

| Ruta | Función |
|---|---|
| `GET /pets/{petId}/requests` | Lista las solicitudes asociadas a la mascota |
| `GET /pets/{petId}/requests/new` | Muestra el formulario de creación |
| `POST /pets/{petId}/requests` | Valida y registra una nueva solicitud |
| `POST /pets/{petId}/requests/{requestId}/complete` | Completa una solicitud abierta |
| `POST /pets/{petId}/requests/{requestId}/cancel` | Cancela una solicitud abierta |

La dirección contiene `petId` porque las solicitudes siempre se consultan dentro de una mascota. No inventaremos una segunda mascota al enviar el formulario.

### 9. Crear el controlador

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/controller/SupportRequestController.java`:

```java
package com.example.petmatchlab.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import com.example.petmatchlab.dto.SupportRequestForm;
import com.example.petmatchlab.exception.BusinessRuleException;
import com.example.petmatchlab.exception.ResourceNotFoundException;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.model.SupportType;
import com.example.petmatchlab.service.PetService;
import com.example.petmatchlab.service.SupportRequestService;

import jakarta.validation.Valid;

@Controller
@RequestMapping("/pets/{petId}/requests")
public class SupportRequestController {

    private final PetService petService;
    private final SupportRequestService requestService;

    public SupportRequestController(
        PetService petService,
        SupportRequestService requestService
    ) {
        this.petService = petService;
        this.requestService = requestService;
    }

    @GetMapping
    public String list(@PathVariable("petId") long petId, Model model) {
        model.addAttribute("pet", findPet(petId));
        model.addAttribute("requests", requestService.findAllForPet(petId));
        return "requests/list";
    }

    @GetMapping("/new")
    public String newForm(@PathVariable("petId") long petId, Model model) {
        model.addAttribute("supportRequestForm", new SupportRequestForm());
        return prepareForm(model, petId);
    }

    @PostMapping
    public String create(
        @PathVariable("petId") long petId,
        @Valid @ModelAttribute("supportRequestForm") SupportRequestForm form,
        BindingResult bindingResult,
        Model model,
        RedirectAttributes redirectAttributes
    ) {
        findPet(petId);

        if (bindingResult.hasErrors()) {
            return prepareForm(model, petId);
        }

        try {
            requestService.create(petId, form);
        } catch (BusinessRuleException exception) {
            bindingResult.reject("request.rule", exception.getMessage());
            return prepareForm(model, petId);
        }

        redirectAttributes.addFlashAttribute(
            "successMessage", "Solicitud registrada correctamente."
        );
        return "redirect:/pets/" + petId + "/requests";
    }

    @PostMapping("/{requestId}/complete")
    public String complete(
        @PathVariable("petId") long petId,
        @PathVariable("requestId") long requestId,
        RedirectAttributes redirectAttributes
    ) {
        requestService.complete(petId, requestId);
        redirectAttributes.addFlashAttribute(
            "successMessage", "Solicitud completada correctamente."
        );
        return "redirect:/pets/" + petId + "/requests";
    }

    @PostMapping("/{requestId}/cancel")
    public String cancel(
        @PathVariable("petId") long petId,
        @PathVariable("requestId") long requestId,
        RedirectAttributes redirectAttributes
    ) {
        requestService.cancel(petId, requestId);
        redirectAttributes.addFlashAttribute(
            "successMessage", "Solicitud cancelada correctamente."
        );
        return "redirect:/pets/" + petId + "/requests";
    }

    private String prepareForm(Model model, long petId) {
        model.addAttribute("pet", findPet(petId));
        model.addAttribute("supportTypes", SupportType.values());
        model.addAttribute("formAction", "/pets/" + petId + "/requests");
        model.addAttribute("formTitle", "Nueva solicitud de apoyo");
        model.addAttribute("submitLabel", "Registrar solicitud");
        return "requests/form";
    }

    private Pet findPet(long petId) {
        return petService.findById(petId)
            .orElseThrow(() -> new ResourceNotFoundException(
                "No existe la mascota indicada."
            ));
    }
}
```

En este controlador, `prepareForm(...)` concentra los datos que la vista necesita, incluso cuando una validación falla. Así evitamos devolver `requests/form` sin la mascota, las opciones del enum o la dirección del formulario. El controlador consulta la mascota a través de `PetService`, sin acceder directamente a `PetRepository`.

**Fíjate en la posición de `BindingResult`:** aparece inmediatamente después del parámetro anotado con `@Valid @ModelAttribute`. Esto permite que Spring MVC entregue los errores de esa entrada al mismo método.

**Observa también `catch (BusinessRuleException exception)`:** intercepta específicamente una regla conocida durante la creación y la convierte en un error visible en el formulario. No es un `catch (Exception)` que silencie problemas inesperados, y no intenta convertir una excepción de conexión a la base de datos en un error de formulario.

### 10. Crear el formulario Thymeleaf

Crea el **archivo completo** `src/main/resources/templates/requests/form.html`:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Solicitud | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <section class="content-panel">
            <h1 th:text="${formTitle}">Nueva solicitud de apoyo</h1>
            <p>
                Mascota:
                <strong th:text="${pet.name}">Luna</strong>
            </p>

            <form method="post"
                  th:action="${formAction}"
                  th:object="${supportRequestForm}">

                <div class="form-summary"
                     role="alert"
                     th:if="${#fields.hasGlobalErrors()}">
                    <p th:each="error : ${#fields.globalErrors()}"
                       th:text="${error}">Error de la solicitud.</p>
                </div>

                <div class="form-field">
                    <label for="type">Tipo de apoyo</label>
                    <select id="type" th:field="*{type}"
                            th:errorclass="field-error"
                            aria-describedby="type-error">
                        <option value="">Selecciona una opción</option>
                        <option th:each="supportType : ${supportTypes}"
                                th:value="${supportType.name()}"
                                th:text="${supportType.label}">
                            Tipo
                        </option>
                    </select>
                    <p id="type-error" class="field-error-message"
                       th:if="${#fields.hasErrors('type')}"
                       th:errors="*{type}">Error de tipo.</p>
                </div>

                <div class="form-field">
                    <label for="description">Descripción</label>
                    <textarea id="description"
                              rows="5" maxlength="500"
                              th:field="*{description}"
                              th:errorclass="field-error"
                              aria-describedby="description-error"></textarea>
                    <p id="description-error" class="field-error-message"
                       th:if="${#fields.hasErrors('description')}"
                       th:errors="*{description}">
                        Error de descripción.
                    </p>
                </div>

                <div class="form-actions">
                    <button type="submit" class="button-link"
                            th:text="${submitLabel}">
                        Registrar solicitud
                    </button>
                    <a class="secondary-link"
                       th:href="@{/pets/{petId}/requests(petId=${pet.id})}">
                        Volver a solicitudes
                    </a>
                </div>
            </form>
        </section>
    </main>
</body>
</html>
```

Esta plantilla contiene un solo formulario reutilizable. Los datos que cambian entre operaciones se entregan como atributos del `Model`: `formAction`, `formTitle` y `submitLabel`.

La expresión `th:field="*{type}"` se vincula a `SupportRequestForm.type`; Thymeleaf utiliza el objeto seleccionado por `th:object`.

El elemento `<select>` no determina cuál estado tendrá la solicitud. Solo permite elegir el tipo de ayuda. La entidad siempre comienza con `OPEN`.

### 11. Crear el listado de solicitudes

Crea el **archivo completo** `src/main/resources/templates/requests/list.html`:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Solicitudes | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <section class="content-panel">
            <h1>Solicitudes de apoyo</h1>
            <p>
                Mascota:
                <strong th:text="${pet.name}">Luna</strong>
            </p>

            <p class="success-message" role="status"
               th:if="${successMessage}"
               th:text="${successMessage}">Operación correcta.</p>

            <p>
                <a class="button-link"
                   th:href="@{/pets/{petId}/requests/new(petId=${pet.id})}">
                    Crear solicitud
                </a>
                <a class="secondary-link"
                   th:href="@{/pets/{petId}(petId=${pet.id})}">
                    Volver a la mascota
                </a>
            </p>

            <section th:if="${requests.isEmpty()}">
                <h2>Sin solicitudes</h2>
                <p>Esta mascota todavía no tiene solicitudes registradas.</p>
            </section>

            <section th:unless="${requests.isEmpty()}"
                     aria-label="Solicitudes de apoyo">
                <article class="request-card"
                         th:each="request : ${requests}">
                    <h2>
                        Solicitud
                        <span th:text="${request.id}">0</span>
                    </h2>
                    <p>
                        <strong>Tipo:</strong>
                        <span th:text="${request.type.label}">Paseo</span>
                    </p>
                    <p>
                        <strong>Descripción:</strong>
                        <span th:text="${request.description}">Descripción</span>
                    </p>
                    <p>
                        <strong>Estado:</strong>
                        <span th:text="${request.status.label}">Abierta</span>
                    </p>

                    <div class="form-actions" th:if="${request.open}">
                        <form method="post"
                              th:action="@{/pets/{petId}/requests/{requestId}/complete(petId=${pet.id},requestId=${request.id})}">
                            <button type="submit" class="button-link">
                                Completar
                            </button>
                        </form>
                        <form method="post"
                              th:action="@{/pets/{petId}/requests/{requestId}/cancel(petId=${pet.id},requestId=${request.id})}">
                            <button type="submit" class="danger-button">
                                Cancelar
                            </button>
                        </form>
                    </div>
                </article>
            </section>
        </section>
    </main>
</body>
</html>
```

La condición `th:if="${request.open}"` utiliza el método Java `isOpen()` de la entidad. Si la solicitud está completada o cancelada, Thymeleaf no genera esos formularios.

**Importante:** ocultar botones en la interfaz no impide que alguien envíe una petición POST manualmente. La protección real también está en `SupportRequest.complete()` y `SupportRequest.cancel()` y en las comprobaciones del servicio. Una vista no constituye una frontera de autorización.

### 12. Enlazar las solicitudes desde la mascota

En `src/main/resources/templates/pets/detail.html`, conserva todo el contenido existente y añade cerca de los enlaces de navegación:

```html
<a class="button-link"
   th:href="@{/pets/{id}/requests(id=${pet.id})}">
    Ver solicitudes de apoyo
</a>
```

Este es un **fragmento para insertar**, no un reemplazo del archivo completo. El enlace utiliza el mismo identificador `pet.id` que ya presenta la vista de detalle.

### 13. Añadir estilos sin duplicar el tema existente

Al final de `src/main/resources/static/css/application.css` puedes incorporar estas reglas si todavía no dispones de estilos equivalentes:

```css
.request-card {
    padding: 1rem;
    margin-block: 1rem;
    border: 1px solid #64748b;
    border-radius: 0.7rem;
}

.request-card .form-actions form {
    margin: 0;
}

.form-field select {
    width: 100%;
    max-width: 36rem;
    padding: 0.7rem 0.85rem;
    border: 1px solid #64748b;
    border-radius: 0.6rem;
    font: inherit;
    background: #fff;
    color: #0f172a;
}

.form-field select:focus-visible {
    outline: 3px solid #0ea5e9;
    outline-offset: 2px;
}
```

Los estilos apoyan la lectura y la navegación por teclado, pero no reemplazan la validación. Comprueba que los enlaces, los botones y los campos tengan nombres visibles y puedan utilizarse mediante **Tab** y **Enter**.

### 14. Probar el flujo desde el navegador

Con Spring Boot y MariaDB iniciados:

1. Abre `/pets` y selecciona una mascota existente. No supongas que su id tiene un valor fijo.
2. Abre **Ver solicitudes de apoyo**.
3. Comprueba que aparece el mensaje de listado vacío.
4. Selecciona **Crear solicitud**.
5. Envía el formulario sin elegir un tipo y sin escribir descripción. Deben aparecer errores de validación.
6. Elige un tipo, escribe una descripción válida y registra la solicitud.
7. Comprueba que aparece con estado **Abierta**.
8. Intenta crear otra solicitud para la misma mascota. Debe aparecer un error de regla de negocio.
9. Completa o cancela la solicitud actual. Su estado debe cambiar y desaparecerán los botones de transición.
10. Ahora registra otra solicitud para la misma mascota: es válida porque no existe otra `OPEN`.

**No esperes un identificador consecutivo concreto**. La base de datos asigna ids, y estos pueden presentar saltos si hubo inserciones previas, intentos fallidos u otras operaciones.

### 15. Comprobarlo en MariaDB

Ejecuta:

```sql
SELECT
    id,
    pet_id,
    type,
    description,
    status
FROM support_requests
ORDER BY pet_id, id DESC;
```

Después compara la tabla con la interfaz. Al completar una solicitud, la fila correspondiente debe conservar su `id` y `pet_id`; lo que cambia es `status`.

**Predice:** si reinicias el servidor web, ¿desaparecerán las solicitudes? No deberían, porque las filas confirmadas están guardadas en MariaDB y no en una colección temporal.

### Ejercicio — Regla frente a validación

Clasifica estas situaciones:

- El campo `description` tiene solo espacios.
- La mascota indicada en la URL no existe.
- Se intenta completar una solicitud cancelada.
- Se intenta crear una segunda solicitud abierta.
- Se intenta consultar una solicitud de otra mascota cambiando el id de la URL.

Para cada caso, indica qué pieza debería detectarlo y qué respuesta puede ofrecer la aplicación.

### Depuración — El formulario funciona, pero no registra

1. Si `th:field="*{type}"` muestra un error de conversión, compara los valores del `<option>` con las constantes del enum.
2. Si regresa al formulario sin opciones, comprueba que `prepareForm(...)` añadió `supportTypes`.
3. Si la lista siempre aparece vacía, comprueba qué `petId` se envió y si existen filas con ese valor en MariaDB.
4. Si los botones desaparecen antes de completar la solicitud, revisa el resultado de `isOpen()` y el estado persistido.
5. Si el navegador responde 500 ante una regla del dominio, todavía falta la traducción de excepciones a respuestas web que completaremos más adelante.

### Comprobación

Debes poder explicar el recorrido completo de un formulario hasta una fila SQL, señalar dónde se asigna `OPEN` y justificar por qué `@Valid` no sustituye las reglas de negocio.

---
## Práctica 10.5 — Transacciones y operaciones que modifican varias filas

### Objetivos

Al finalizar podrás:

- explicar el concepto de **transacción** y las operaciones *commit* y *rollback*;
- identificar un límite transaccional en una clase `Service`;
- comprender cuándo JPA sincroniza cambios en entidades administradas;
- distinguir una transacción de una garantía de exclusividad ante operaciones concurrentes;
- implementar una operación que modifica dos solicitudes como una sola unidad lógica.

### 1. ¿Por qué necesitamos una transacción?

Hasta ahora la creación de una solicitud afecta principalmente a una fila:

```text
registrar solicitud
   → INSERT en support_requests
```

Pero algunas operaciones necesitan varios cambios relacionados. Supón que existe una solicitud abierta y quieres **reemplazarla por otra sin perder el historial**. La operación debe:

1. Identificar la solicitud abierta original.
2. Cambiar su estado a `CANCELLED`.
3. Crear una nueva solicitud con estado `OPEN`.
4. Confirmar ambos cambios juntos.

El objetivo es evitar que la solicitud anterior quede cancelada **sin** que la nueva llegue a existir si ocurre un fallo durante la creación.

```text
ANTES
  solicitud 25 → OPEN

REEMPLAZAR CORRECTAMENTE
  solicitud 25 → CANCELLED
  solicitud 26 → OPEN

SI FALLA LA SEGUNDA MODIFICACIÓN
  solicitud 25 → OPEN
  solicitud 26 → no existe
```

El último escenario es el resultado que esperamos cuando la operación completa se revierte.

### 2. Unidad de trabajo, commit y rollback

Una **transacción** agrupa cambios de datos que se confirman o deshacen como una unidad lógica.

- **Commit:** confirma los cambios que forman parte de la transacción.
- **Rollback:** revierte los cambios no confirmados de esa transacción.
- **Atomicidad:** la unidad se confirma por completo o se deshace, en vez de conservar solo una parte de ella.

Esto no significa que toda la aplicación ejecute una única transacción. Cada operación significativa debe tener un límite adecuado. Tampoco significa que una transacción siempre bloquee a otros usuarios hasta que termine: el aislamiento y la concurrencia son cuestiones diferentes.

### 3. Entender `@Transactional` sin magia

En Spring podemos expresar ese límite con:

```java
@Transactional
public SupportRequest replace(...) {
    // Read and modify existing data.
    // Insert the replacement.
}
```

La anotación de Spring se importa desde:

```java
import org.springframework.transaction.annotation.Transactional;
```

Cuando otro componente invoca un método transaccional de un bean administrado por Spring a través de su mecanismo habitual de interceptación, Spring abre o participa en una transacción, ejecuta el método y coordina su confirmación o reversión.

**Importante:** no debes imaginar que Java transforma cada instrucción de ese método en un `COMMIT`. En el caso normal, los cambios se confirman cuando la transacción termina exitosamente.

### 4. ¿Qué significa `@Transactional(readOnly = true)`?

En el servicio ya aparece:

```java
@Transactional(readOnly = true)
public List<SupportRequest> findAllForPet(long petId) {
    findPet(petId);
    return requestRepository.findByPet_IdOrderByIdDesc(petId);
}
```

`readOnly = true` comunica que la operación está diseñada para consultar datos. Según la tecnología de persistencia, Spring y Hibernate pueden aplicar optimizaciones o indicaciones adicionales.

**No lo trates como una barrera universal que impide cualquier escritura.** La protección de los datos no se obtiene simplemente poniendo `readOnly = true`: la lógica del programa y los permisos de base de datos siguen importando.

### 5. Añadir el método para reemplazar una solicitud

Abre `src/main/java/com/example/petmatchlab/service/SupportRequestService.java`. **Dentro de la clase**, después de los métodos `create`, `complete` y `cancel`, agrega este método completo:

```java
@Transactional
public SupportRequest replace(
    long petId,
    long requestId,
    SupportRequestForm form
) {
    validateForm(form);
    Pet pet = findPet(petId);
    SupportRequest previous = findRequest(petId, requestId);

    previous.cancel();

    SupportRequest replacement = new SupportRequest(
        pet,
        form.getType(),
        form.getDescription().strip()
    );

    return requestRepository.save(replacement);
}
```

**No reemplaces el resto del servicio.** Este fragmento amplía la clase que ya contiene las consultas y transiciones.

Revisa los pasos:

1. `validateForm(form)` rechaza datos inválidos antes de intentar modificar información.
2. `findPet(petId)` confirma que la mascota existe.
3. `findRequest(petId, requestId)` confirma que la solicitud pertenece a esa mascota.
4. `previous.cancel()` verifica que estaba `OPEN` y cambia su estado a `CANCELLED`.
5. `new SupportRequest(...)` prepara una solicitud nueva en estado `OPEN`.
6. `requestRepository.save(replacement)` solicita su persistencia.
7. El retorno exitoso permite que la transacción confirme los cambios.

Si `previous.cancel()` rechaza una solicitud ya completada, la operación termina con una excepción de negocio y no se inserta ninguna solicitud nueva.

Si la inserción falla con una excepción que provoca rollback, la cancelación anterior también se revierte. El cambio sobre `previous` está administrado por JPA dentro de la misma transacción.

### 6. ¿Por qué no se llama a `save(previous)`?

La entidad `previous` fue recuperada mediante `findRequest(...)` dentro del método transaccional. Al ser una entidad administrada, Hibernate puede detectar que su atributo `status` cambió.

La idea es:

```text
SELECT solicitud anterior
       |
       v
Entidad administrada
       |
       v
previous.cancel()
       |
       v
Hibernate detecta el cambio
       |
       v
UPDATE al sincronizar/confirmar
```

En cambio, `replacement` es un objeto nuevo que necesita ser entregado al contexto de persistencia mediante `save(...)`.

**Predice:** si eliminas `@Transactional` del método y dependes de operaciones de repositorio separadas, ¿puedes asegurar que ambos cambios formen una sola unidad? No. Un conjunto de llamadas independientes no garantiza que el cambio anterior y la inserción compartan la misma transacción.

### 7. Reutilizar el formulario para reemplazar

La plantilla `requests/form.html` ya recibe `formAction`, `formTitle` y `submitLabel`. Eso permite usar el mismo HTML con valores iniciales diferentes, sin copiar todos sus campos a otra página.

Abre `SupportRequestController.java` y añade estos dos métodos **dentro de la clase**, antes de `prepareForm(...)`:

```java
@GetMapping("/{requestId}/replace")
public String replaceForm(
    @PathVariable("petId") long petId,
    @PathVariable("requestId") long requestId,
    Model model
) {
    var request = requestService.findOneForPet(petId, requestId);

    if (!request.isOpen()) {
        throw new BusinessRuleException(
            "Solo se puede reemplazar una solicitud abierta."
        );
    }

    SupportRequestForm form = new SupportRequestForm();
    form.setType(request.getType());
    form.setDescription(request.getDescription());
    model.addAttribute("supportRequestForm", form);

    return prepareReplacementForm(model, petId, requestId);
}

@PostMapping("/{requestId}/replace")
public String replace(
    @PathVariable("petId") long petId,
    @PathVariable("requestId") long requestId,
    @Valid @ModelAttribute("supportRequestForm") SupportRequestForm form,
    BindingResult bindingResult,
    Model model,
    RedirectAttributes redirectAttributes
) {
    if (bindingResult.hasErrors()) {
        var request = requestService.findOneForPet(petId, requestId);
        if (!request.isOpen()) {
            throw new BusinessRuleException(
                "Solo se puede reemplazar una solicitud abierta."
            );
        }
        return prepareReplacementForm(model, petId, requestId);
    }

    requestService.replace(petId, requestId, form);

    redirectAttributes.addFlashAttribute(
        "successMessage", "La solicitud fue reemplazada correctamente."
    );
    return "redirect:/pets/" + petId + "/requests";
}
```

Añade también este método privado **dentro de la misma clase**, después de `prepareForm(...)`:

```java
private String prepareReplacementForm(
    Model model,
    long petId,
    long requestId
) {
    prepareForm(model, petId);
    model.addAttribute(
        "formAction",
        "/pets/" + petId + "/requests/" + requestId + "/replace"
    );
    model.addAttribute("formTitle", "Reemplazar solicitud abierta");
    model.addAttribute("submitLabel", "Reemplazar solicitud");
    return "requests/form";
}
```

El método reutiliza `prepareForm(...)` para cargar la mascota y las opciones del enum; luego cambia la dirección y las etiquetas que necesita el formulario de reemplazo.

**¿Por qué no cambia el identificador de la solicitud original?** Porque no estamos editando ese registro. Estamos **cancelando una solicitud histórica y creando una nueva**, con un id propio.

### 8. Añadir el enlace desde la lista

En `templates/requests/list.html`, dentro de la sección:

```html
<div class="form-actions" th:if="${request.open}">
```

agrega un enlace, conservando los dos formularios de completar y cancelar:

```html
<a class="secondary-link"
   th:href="@{/pets/{petId}/requests/{requestId}/replace(petId=${pet.id},requestId=${request.id})}">
    Reemplazar
</a>
```

La interfaz solo presenta este enlace para solicitudes abiertas. El servicio sigue comprobando el estado por sí mismo, incluso si alguien intenta abrir o enviar una ruta manualmente.

### 9. Seguir el resultado en la base de datos

1. Registra una solicitud nueva y anota su id.
2. Selecciona **Reemplazar**.
3. Modifica el tipo o la descripción.
4. Envía el formulario.
5. Comprueba en la lista que la solicitud original ahora dice **Cancelada** y la nueva dice **Abierta**.
6. Ejecuta:

```sql
SELECT id, pet_id, type, description, status
FROM support_requests
WHERE pet_id = 1
ORDER BY id DESC;
```

Sustituye el `1` por el identificador real de la mascota con la que trabajaste. Deberías observar al menos dos solicitudes distintas: la original cancelada y el reemplazo abierto.

### 10. ¿Qué provoca un rollback?

Por defecto, Spring aplica rollback ante excepciones no comprobadas (`RuntimeException`) y errores (`Error`). Las excepciones comprobadas (*checked exceptions*) no tienen exactamente el mismo comportamiento predeterminado, aunque puede configurarse mediante las opciones de `@Transactional`.

Aquí `BusinessRuleException` extiende `RuntimeException`. Si se lanza dentro de un método transaccional, el resultado normal es que se reviertan las modificaciones no confirmadas de esa transacción.

Hay tres matices importantes:

- **No ocultar la excepción:** si capturas un error y continúas como si nada hubiera pasado, la transacción puede terminar confirmándose. No utilices bloques `catch` vacíos.
- **No suponer que todas las llamadas internas son interceptadas:** cuando un método de una clase llama directamente a otro método de esa misma clase, esa llamada no atraviesa necesariamente el proxy transaccional de Spring. Por eso conviene ubicar límites públicos claros para los casos de uso.
- **No confundir `flush` con `commit`:** `flush` sincroniza SQL pendiente con la base de datos, pero no significa que la transacción ya se haya confirmado definitivamente.

### 11. Transacción no significa control total de concurrencia

La comprobación:

```java
existsByPet_IdAndStatus(petId, RequestStatus.OPEN)
```

funciona para el flujo secuencial del laboratorio. Sin embargo, dos solicitudes concurrentes podrían consultar al mismo tiempo y ambas observar que todavía no existe una solicitud abierta.

**`@Transactional` no convierte automáticamente esa comprobación en una garantía global de unicidad.** Para una aplicación con concurrencia real habría que definir una estrategia adicional, por ejemplo restricciones adecuadas en base de datos y mecanismos de bloqueo o coordinación, y comprobar su comportamiento con el motor utilizado.

Del mismo modo, el patrón *consultar si hay solicitudes y luego eliminar una mascota* necesita considerar carreras concurrentes en un sistema con múltiples operaciones simultáneas. La clave foránea continúa siendo una última defensa de integridad.

Comprender este límite es más importante que copiar opciones avanzadas de bloqueo sin conocer sus consecuencias.

### 12. Ejercicio — Dos cambios, una operación

Predice el resultado de los siguientes casos:

1. Se reemplaza una solicitud `OPEN` con datos válidos.
2. Se intenta reemplazar una solicitud `COMPLETED`.
3. El texto del nuevo formulario está vacío.
4. La solicitud indicada pertenece a otra mascota.
5. Se produce un fallo de persistencia después de cancelar la solicitud anterior, antes de confirmar toda la operación.

Para cada caso, indica el estado final esperado de la solicitud original y si debería existir una solicitud nueva.

### Depuración — El error no siempre está donde aparece

- Si aparecen dos solicitudes `OPEN`, revisa qué comprobaciones existen, si se invocaron desde una transacción y si hubo solicitudes concurrentes.
- Si `cancel()` modifica la entidad pero la base no cambia, comprueba dónde se obtiene la entidad y si la operación ocurre en un contexto de persistencia transaccional.
- Si una excepción deja cambios parciales, comprueba que la operación completa tenga un límite `@Transactional` real y que la excepción no haya sido absorbida.
- Si la página de reemplazo intenta crear una solicitud en vez de reemplazar, compara `formAction` con la ruta `@PostMapping("/{requestId}/replace")`.

### Comprobación

Debes poder explicar por qué `replace(...)` constituye **un caso de uso transaccional**, diferenciar `flush` de `commit` y justificar por qué el servicio concentra el cambio de estado y la inserción.

---
## Práctica 10.6 — Excepciones, pruebas y reto integrador

### Objetivos

Al terminar esta práctica podrás:

- traducir excepciones conocidas del dominio a respuestas HTTP claras;
- impedir eliminar una mascota que tiene solicitudes relacionadas;
- verificar transiciones de estado mediante pruebas unitarias;
- comprobar consultas y asociaciones con pruebas JPA;
- observar un rollback real mediante una prueba de integración;
- probar respuestas 404 y 409 sin depender exclusivamente del navegador.

### 1. Una excepción de dominio necesita una respuesta web

En Java, cuando una operación no puede continuar, una excepción puede comunicar el motivo. Pero una persona que utiliza el navegador necesita además una respuesta comprensible.

Considera estos casos:

```text
GET /pets/999/requests
    → la mascota no existe
    → 404 Not Found

POST /pets/7/requests/25/complete
    → la solicitud existe, pero ya está CANCELLED
    → 409 Conflict
```

Los códigos HTTP expresan situaciones distintas:

| Código | Significado en este laboratorio |
|---|---|
| 200 OK | El servidor generó correctamente una página |
| 302 Found | Se redirigió al navegador después de una operación correcta |
| 400 Bad Request | Un parámetro no tiene el formato necesario, por ejemplo un identificador no numérico |
| 404 Not Found | No existe el recurso solicitado dentro del contexto indicado |
| 409 Conflict | El recurso existe, pero una regla de negocio prohíbe la operación |
| 500 Internal Server Error | Ocurrió un fallo inesperado que no debe ocultarse como un error de formulario |

**Importante:** una regla del negocio no equivale necesariamente a una entrada sintácticamente inválida. Por ejemplo, completar una solicitud cancelada no es lo mismo que escribir letras donde se esperaba un número.

### 2. Separar el tratamiento de errores del controlador

Spring MVC permite manejar excepciones con `@ExceptionHandler`. Si agrupamos el tratamiento en una clase anotada con `@ControllerAdvice`, podremos usarlo desde varios controladores sin copiar los mismos bloques de código.

Crea el **archivo completo** `src/main/java/com/example/petmatchlab/controller/PageErrorHandler.java`:

```java
package com.example.petmatchlab.controller;

import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;

import com.example.petmatchlab.exception.BusinessRuleException;
import com.example.petmatchlab.exception.ResourceNotFoundException;

@ControllerAdvice(annotations = Controller.class)
public class PageErrorHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public String handleNotFound(
        ResourceNotFoundException exception,
        Model model
    ) {
        model.addAttribute("errorTitle", "Recurso no encontrado");
        model.addAttribute("errorMessage", exception.getMessage());
        return "errors/status";
    }

    @ExceptionHandler(BusinessRuleException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public String handleConflict(
        BusinessRuleException exception,
        Model model
    ) {
        model.addAttribute("errorTitle", "Operación no permitida");
        model.addAttribute("errorMessage", exception.getMessage());
        return "errors/status";
    }
}
```

La anotación `@ControllerAdvice(annotations = Controller.class)` limita este tratamiento a controladores anotados con `@Controller`. No estamos construyendo una respuesta JSON ni una API REST: se trata de páginas HTML.

**Por qué no usamos `catch (Exception)` para todo:** los problemas inesperados de programación, SQL o configuración deben seguir siendo investigables. Traducir cualquier error a 409 ocultaría información importante y produciría respuestas engañosas.

### 3. Crear la página de error

Crea la carpeta `src/main/resources/templates/errors/` y dentro el **archivo completo** `status.html`:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('No se pudo completar | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <section class="content-panel" role="alert">
            <h1 th:text="${errorTitle}">Operación no disponible</h1>
            <p th:text="${errorMessage}">
                La operación solicitada no pudo completarse.
            </p>
            <a class="button-link" th:href="@{/pets}">
                Volver al catálogo
            </a>
        </section>
    </main>
</body>
</html>
```

El código HTTP se establece en el método del manejador mediante `@ResponseStatus`. La plantilla únicamente describe qué se mostrará a la persona.

Al recibir `BusinessRuleException`, el navegador obtiene una respuesta **409** y una página con una explicación concreta. Al recibir `ResourceNotFoundException`, obtiene **404**.

### 4. Proteger la eliminación de mascotas relacionadas

El proyecto ya incluye el método `delete(long id)` de `PetService`. Hasta ahora podía eliminar una mascota sin considerar solicitudes relacionadas.

Modificaremos el servicio para que **una mascota con solicitudes no pueda eliminarse**, aunque todas sus solicitudes estén en estado final.

Abre `src/main/java/com/example/petmatchlab/service/PetService.java`. Añade los imports que no existan:

```java
import org.springframework.transaction.annotation.Transactional;

import com.example.petmatchlab.exception.BusinessRuleException;
import com.example.petmatchlab.repository.SupportRequestRepository;
```

En la clase, después del campo `petRepository`, agrega:

```java
private final SupportRequestRepository supportRequestRepository;
```

Reemplaza el **constructor anterior** por este:

```java
public PetService(
    PetRepository petRepository,
    SupportRequestRepository supportRequestRepository
) {
    this.petRepository = petRepository;
    this.supportRequestRepository = supportRequestRepository;
}
```

Por último, reemplaza **únicamente el método `delete(long id)`** por:

```java
@Transactional
public boolean delete(long id) {
    return petRepository.findById(id).map(pet -> {
        if (supportRequestRepository.existsByPet_Id(id)) {
            throw new BusinessRuleException(
                "No se puede eliminar una mascota que tiene solicitudes."
            );
        }

        petRepository.delete(pet);
        return true;
    }).orElse(false);
}
```

No cambies los métodos `create`, `update`, `findAll` ni `findById` del servicio.

### 5. Leer la regla de eliminación

La diferencia está aquí:

```java
if (supportRequestRepository.existsByPet_Id(id)) {
    throw new BusinessRuleException(...);
}
```

El repositorio responde si existe alguna referencia a la mascota. El servicio decide prohibir la eliminación.

Si no existe la mascota, `delete(...)` conserva su contrato anterior y devuelve `false`. Si existe y no tiene solicitudes, devuelve `true` después de eliminarla. Si existe y tiene solicitudes, lanza `BusinessRuleException`.

El controlador `PetController` puede conservar su código para la ruta:

```text
POST /pets/{id}/delete
```

Cuando el servicio lanza la excepción, `PageErrorHandler` se encarga de producir la respuesta 409.

La restricción de clave foránea de MariaDB protege la integridad si alguna operación intenta borrar una mascota todavía referenciada. **La regla del servicio ofrece una explicación de negocio antes de depender de un error SQL.** Ambas protecciones son complementarias, y siguen existiendo riesgos de concurrencia que requieren decisiones adicionales en sistemas más exigentes.

### 6. Actualizar las pruebas anteriores de `PetService`

El constructor de `PetService` ahora requiere dos dependencias. En `src/test/java/com/example/petmatchlab/service/PetServiceTests.java` agrega:

```java
import com.example.petmatchlab.repository.SupportRequestRepository;
```

Junto al mock existente de `PetRepository`, añade:

```java
@Mock
private SupportRequestRepository requestRepository;
```

Y en `setUp()` sustituye:

```java
service = new PetService(repository);
```

por:

```java
service = new PetService(repository, requestRepository);
```

Esto no cambia el significado de las pruebas originales, pero sí adapta su construcción a la nueva dependencia. Los mocks no realizan consultas SQL: responden según lo definido en cada test.

Añade esta prueba dentro de `PetServiceTests`:

```java
@Test
void shouldRejectPetDeletionWhenRequestsExist() {
    Pet existing = new Pet("Luna", "Dog", 3, null);
    when(repository.findById(8L)).thenReturn(Optional.of(existing));
    when(requestRepository.existsByPet_Id(8L)).thenReturn(true);

    assertThrows(BusinessRuleException.class, () -> service.delete(8L));

    verify(repository, never()).delete(any(Pet.class));
}
```

Asegúrate de añadir este import si no existe:

```java
import com.example.petmatchlab.exception.BusinessRuleException;
```

Las importaciones estáticas `assertThrows`, `when`, `verify`, `never` y `any` ya aparecen en las pruebas de servicio anteriores. Si tu archivo difiere, importa explícitamente las funciones que utilices.

La prueba verifica la **decisión**: cuando existen solicitudes, el servicio no invoca la eliminación de `PetRepository`.

### 7. Probar los estados sin iniciar Spring Boot

Las transiciones `complete()` y `cancel()` contienen reglas Java que podemos comprobar directamente. No es necesario iniciar MariaDB para estudiar esa lógica.

Crea el **archivo completo** `src/test/java/com/example/petmatchlab/model/SupportRequestTests.java`:

```java
package com.example.petmatchlab.model;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.Test;

import com.example.petmatchlab.exception.BusinessRuleException;

class SupportRequestTests {

    @Test
    void shouldBeginInOpenState() {
        SupportRequest request = newRequest();

        assertEquals(RequestStatus.OPEN, request.getStatus());
        assertTrue(request.isOpen());
    }

    @Test
    void shouldCompleteAnOpenRequest() {
        SupportRequest request = newRequest();

        request.complete();

        assertEquals(RequestStatus.COMPLETED, request.getStatus());
    }

    @Test
    void shouldCancelAnOpenRequest() {
        SupportRequest request = newRequest();

        request.cancel();

        assertEquals(RequestStatus.CANCELLED, request.getStatus());
    }

    @Test
    void shouldRejectCompletingAnAlreadyCompletedRequest() {
        SupportRequest request = newRequest();
        request.complete();

        assertThrows(BusinessRuleException.class, request::complete);
        assertEquals(RequestStatus.COMPLETED, request.getStatus());
    }

    @Test
    void shouldRejectCancellingACompletedRequest() {
        SupportRequest request = newRequest();
        request.complete();

        assertThrows(BusinessRuleException.class, request::cancel);
        assertEquals(RequestStatus.COMPLETED, request.getStatus());
    }

    @Test
    void shouldRejectCompletingACancelledRequest() {
        SupportRequest request = newRequest();
        request.cancel();

        assertThrows(BusinessRuleException.class, request::complete);
        assertEquals(RequestStatus.CANCELLED, request.getStatus());
    }

    private SupportRequest newRequest() {
        Pet pet = new Pet("Luna", "Dog", 3, null);
        return new SupportRequest(pet, SupportType.WALK, "Needs a walk");
    }
}
```

**Recuerda:** estas son pruebas unitarias del comportamiento de una entidad Java. Una prueba exitosa no demuestra por sí sola que el mapeo JPA o una conexión MariaDB funcionen.

### 8. Probar la relación con Spring Data JPA

Para comprobar que el repositorio interpreta correctamente la asociación, necesitamos una base de datos de prueba. Se utilizará H2 con la configuración de `src/test/resources/application.yaml`.

Crea el **archivo completo** `src/test/java/com/example/petmatchlab/repository/SupportRequestRepositoryTests.java`:

```java
package com.example.petmatchlab.repository;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.List;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.data.jpa.test.autoconfigure.DataJpaTest;

import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.model.RequestStatus;
import com.example.petmatchlab.model.SupportRequest;
import com.example.petmatchlab.model.SupportType;

import jakarta.persistence.EntityManager;

@DataJpaTest
class SupportRequestRepositoryTests {

    @Autowired
    private PetRepository petRepository;

    @Autowired
    private SupportRequestRepository requestRepository;

    @Autowired
    private EntityManager entityManager;

    @Test
    void shouldPersistRequestWithExistingPet() {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Luna", "Dog", 3, null)
        );
        SupportRequest saved = requestRepository.saveAndFlush(
            new SupportRequest(pet, SupportType.WALK, "Needs a walk")
        );
        Long requestId = saved.getId();
        Long petId = pet.getId();

        entityManager.clear();
        SupportRequest found = requestRepository.findById(requestId)
            .orElseThrow();

        assertNotNull(found.getId());
        assertEquals(petId, found.getPet().getId());
        assertEquals(RequestStatus.OPEN, found.getStatus());
    }

    @Test
    void shouldFindRequestsOnlyForSelectedPet() {
        Pet first = petRepository.saveAndFlush(
            new Pet("Luna", "Dog", 3, null)
        );
        Pet second = petRepository.saveAndFlush(
            new Pet("Milo", "Cat", 2, null)
        );

        requestRepository.saveAndFlush(
            new SupportRequest(first, SupportType.WALK, "Walk")
        );
        requestRepository.saveAndFlush(
            new SupportRequest(second, SupportType.COMPANY, "Company")
        );

        List<SupportRequest> found =
            requestRepository.findByPet_IdOrderByIdDesc(first.getId());

        assertEquals(1, found.size());
        assertEquals(SupportType.WALK, found.get(0).getType());
        assertTrue(requestRepository.existsByPet_Id(first.getId()));
        assertFalse(requestRepository.existsByPet_Id(Long.MAX_VALUE));
    }

    @Test
    void shouldNotFindRequestUnderAnotherPet() {
        Pet first = petRepository.saveAndFlush(
            new Pet("Luna", "Dog", 3, null)
        );
        Pet second = petRepository.saveAndFlush(
            new Pet("Milo", "Cat", 2, null)
        );
        SupportRequest saved = requestRepository.saveAndFlush(
            new SupportRequest(first, SupportType.FEEDING, "Feed")
        );

        assertTrue(requestRepository.findByIdAndPet_Id(
            saved.getId(), second.getId()
        ).isEmpty());
    }
}
```

Esta clase demuestra que la relación puede persistirse, que `pet_id` tiene efecto sobre las consultas y que una solicitud de una mascota no aparece como si perteneciera a otra.

**Nota sobre `@DataJpaTest`:** estas pruebas suelen ejecutarse dentro de una transacción que se revierte al terminar cada caso. No las interpretes como un modo de poblar de datos tu MariaDB habitual.

### 9. Probar el servicio con dependencias simuladas

La siguiente prueba verifica una regla sin iniciar MariaDB. Crea el **archivo completo** `src/test/java/com/example/petmatchlab/service/SupportRequestServiceTests.java`:

```java
package com.example.petmatchlab.service;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

import java.util.Optional;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import com.example.petmatchlab.dto.SupportRequestForm;
import com.example.petmatchlab.exception.BusinessRuleException;
import com.example.petmatchlab.exception.ResourceNotFoundException;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.model.RequestStatus;
import com.example.petmatchlab.model.SupportRequest;
import com.example.petmatchlab.model.SupportType;
import com.example.petmatchlab.repository.PetRepository;
import com.example.petmatchlab.repository.SupportRequestRepository;

@ExtendWith(MockitoExtension.class)
class SupportRequestServiceTests {

    @Mock
    private PetRepository petRepository;

    @Mock
    private SupportRequestRepository requestRepository;

    private SupportRequestService service;

    @BeforeEach
    void setUp() {
        service = new SupportRequestService(petRepository, requestRepository);
    }

    @Test
    void shouldRejectSecondOpenRequest() {
        Pet pet = new Pet("Luna", "Dog", 3, null);
        when(petRepository.findById(7L)).thenReturn(Optional.of(pet));
        when(requestRepository.existsByPet_IdAndStatus(
            7L, RequestStatus.OPEN
        )).thenReturn(true);

        assertThrows(
            BusinessRuleException.class,
            () -> service.create(7L, validForm())
        );

        verify(requestRepository, never()).save(any(SupportRequest.class));
    }

    @Test
    void shouldRejectMissingPetBeforeSaving() {
        when(petRepository.findById(999L)).thenReturn(Optional.empty());

        assertThrows(
            ResourceNotFoundException.class,
            () -> service.create(999L, validForm())
        );

        verify(requestRepository, never()).save(any(SupportRequest.class));
    }

    @Test
    void shouldRejectCompletingRequestOwnedByAnotherPet() {
        when(requestRepository.findByIdAndPet_Id(15L, 7L))
            .thenReturn(Optional.empty());

        assertThrows(
            ResourceNotFoundException.class,
            () -> service.complete(7L, 15L)
        );
    }

    @Test
    void shouldCancelOpenRequest() {
        Pet pet = new Pet("Luna", "Dog", 3, null);
        SupportRequest request = new SupportRequest(
            pet, SupportType.WALK, "Needs a walk"
        );
        when(requestRepository.findByIdAndPet_Id(15L, 7L))
            .thenReturn(Optional.of(request));

        service.cancel(7L, 15L);

        assertEquals(RequestStatus.CANCELLED, request.getStatus());
    }

    @Test
    void shouldRejectInvalidFormBeforeAccessingRepositories() {
        SupportRequestForm form = validForm();
        form.setDescription("   ");

        assertThrows(
            IllegalArgumentException.class,
            () -> service.create(7L, form)
        );

        verify(petRepository, never()).findById(7L);
    }

    private SupportRequestForm validForm() {
        SupportRequestForm form = new SupportRequestForm();
        form.setType(SupportType.WALK);
        form.setDescription("Needs a walk");
        return form;
    }
}
```

Estas pruebas comprueban que el servicio toma decisiones apropiadas y colabora con los repositorios, pero no simulan internamente el funcionamiento de Hibernate.

### 10. Verificar una operación transaccional completa

Las pruebas anteriores permiten comprobar reglas y consultas. Ahora necesitamos verificar qué ocurre cuando una transacción modifica información real de la base de datos de pruebas.

Crea el **archivo completo** `src/test/java/com/example/petmatchlab/service/SupportRequestTransactionIntegrationTests.java`:

```java
package com.example.petmatchlab.service;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.support.TransactionTemplate;

import com.example.petmatchlab.dto.SupportRequestForm;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.model.RequestStatus;
import com.example.petmatchlab.model.SupportRequest;
import com.example.petmatchlab.model.SupportType;
import com.example.petmatchlab.repository.PetRepository;
import com.example.petmatchlab.repository.SupportRequestRepository;

@SpringBootTest
class SupportRequestTransactionIntegrationTests {

    @Autowired
    private PetRepository petRepository;

    @Autowired
    private SupportRequestRepository requestRepository;

    @Autowired
    private SupportRequestService requestService;

    @Autowired
    private PlatformTransactionManager transactionManager;

    @Test
    void shouldReplaceRequestWithTwoConsistentChanges() {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Luna", "Dog", 3, null)
        );
        SupportRequest original = requestRepository.saveAndFlush(
            new SupportRequest(pet, SupportType.WALK, "Original")
        );

        SupportRequestForm form = new SupportRequestForm();
        form.setType(SupportType.FEEDING);
        form.setDescription("Replacement");

        SupportRequest replacement = requestService.replace(
            pet.getId(), original.getId(), form
        );

        assertNotEquals(original.getId(), replacement.getId());
        assertEquals(RequestStatus.CANCELLED,
            requestRepository.findById(original.getId())
                .orElseThrow().getStatus());
        assertEquals(RequestStatus.OPEN,
            requestRepository.findById(replacement.getId())
                .orElseThrow().getStatus());
    }

    @Test
    void shouldRollbackWhenFailureOccursInsideTransaction() {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Milo", "Cat", 2, null)
        );
        SupportRequest original = requestRepository.saveAndFlush(
            new SupportRequest(pet, SupportType.COMPANY, "Original")
        );
        Long requestId = original.getId();
        TransactionTemplate transaction =
            new TransactionTemplate(transactionManager);

        assertThrows(IllegalStateException.class, () ->
            transaction.executeWithoutResult(status -> {
                SupportRequest managed = requestRepository.findById(requestId)
                    .orElseThrow();
                managed.cancel();
                requestRepository.flush();
                throw new IllegalStateException("Simulated failure");
            })
        );

        assertEquals(RequestStatus.OPEN,
            requestRepository.findById(requestId)
                .orElseThrow().getStatus());
    }
}
```

**Dos pruebas, dos preguntas:**

- `shouldReplaceRequestWithTwoConsistentChanges` invoca el caso de uso completo y comprueba sus dos resultados persistidos.
- `shouldRollbackWhenFailureOccursInsideTransaction` utiliza `TransactionTemplate` para provocar deliberadamente una excepción después de un cambio que ya fue sincronizado con la base. Luego consulta de nuevo y comprueba que el estado volvió a `OPEN`.

El error simulado pertenece únicamente a la prueba: **no añadas una excepción artificial al código de producción**. La prueba demuestra la semántica de rollback de la infraestructura transaccional, mientras la primera prueba comprueba la operación real de reemplazo.

### 11. Verificar respuestas HTTP con MockMvc

`MockMvc` ejecuta peticiones a través de la infraestructura Spring MVC en un entorno de pruebas. Permite inspeccionar el estado HTTP y la vista seleccionada sin abrir manualmente el navegador.

Crea el **archivo completo** `src/test/java/com/example/petmatchlab/controller/SupportRequestControllerIntegrationTests.java`:

```java
package com.example.petmatchlab.controller;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.view;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
import org.springframework.test.web.servlet.MockMvc;

import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.model.SupportRequest;
import com.example.petmatchlab.model.SupportType;
import com.example.petmatchlab.repository.PetRepository;
import com.example.petmatchlab.repository.SupportRequestRepository;

@SpringBootTest
@AutoConfigureMockMvc
class SupportRequestControllerIntegrationTests {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private PetRepository petRepository;

    @Autowired
    private SupportRequestRepository requestRepository;

    @Test
    void shouldReturnNotFoundForUnknownPet() throws Exception {
        mockMvc.perform(get("/pets/{petId}/requests", Long.MAX_VALUE))
            .andExpect(status().isNotFound())
            .andExpect(view().name("errors/status"));
    }

    @Test
    void shouldRejectInvalidRequestForm() throws Exception {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Kira", "Dog", 4, null)
        );

        mockMvc.perform(post("/pets/{petId}/requests", pet.getId())
                .param("type", "")
                .param("description", ""))
            .andExpect(status().isOk())
            .andExpect(view().name("requests/form"));
    }

    @Test
    void shouldReturnConflictForInvalidStateTransition() throws Exception {
        Pet pet = petRepository.saveAndFlush(
            new Pet("Nube", "Rabbit", 1, null)
        );
        SupportRequest request = new SupportRequest(
            pet, SupportType.COMPANY, "Needs company"
        );
        request.cancel();
        SupportRequest saved = requestRepository.saveAndFlush(request);

        mockMvc.perform(post(
                "/pets/{petId}/requests/{requestId}/complete",
                pet.getId(), saved.getId()
            ))
            .andExpect(status().isConflict())
            .andExpect(view().name("errors/status"));
    }
}
```

Al observar la prueba `shouldRejectInvalidRequestForm`, identifica por qué el resultado esperado es `200 OK` y la vista `requests/form`: el servidor pudo mostrar correctamente el formulario con errores, aunque **no** haya creado una solicitud válida. En cambio, un cambio de estado prohibido produce una respuesta `409 Conflict`.

### 12. Ejecutar las pruebas

Desde la carpeta raíz de `petmatch-lab`:

**Linux o macOS:**

```bash
./mvnw test
```

**Windows (PowerShell):**

```powershell
.\mvnw.cmd test
```

En este proyecto las pruebas utilizan la configuración de `src/test/resources/application.yaml`, con H2 y un esquema temporal. La aplicación normal continúa conectándose a MariaDB mediante las variables de entorno del entorno de desarrollo.

Revisa el resultado del ejecutor de pruebas. Si alguna falla, no elimines el test para obtener una salida verde: compara lo esperado, el método Java y los datos de prueba que preparó el caso.

**Límite de las pruebas:** H2 y MariaDB no son motores idénticos. Una suite correcta con H2 es útil para verificar Java, JPA y Spring, pero conviene comprobar también el comportamiento real de claves foráneas, tipos y transacciones en MariaDB.

### 13. Ejercicios de depuración

**Caso A — Aparece una excepción de clave foránea.**

Una solicitud intenta guardarse con una mascota inexistente. Investiga de dónde provino el identificador y si el servicio recuperó `Pet` desde `PetRepository` antes de crear `SupportRequest`.

**Caso B — El catálogo muestra la solicitud equivocada.**

Una ruta de la mascota `3` permite actuar sobre una solicitud que pertenece a la mascota `5`. Comprueba que se utiliza `findByIdAndPet_Id`, no una búsqueda que solo reciba el id de la solicitud.

**Caso C — La plantilla falla al acceder a `request.pet.name`.**

Recuerda que la asociación está marcada `LAZY` y la configuración desactiva `open-in-view`. No hagas que la plantilla dependa de cargar asociaciones después de cerrar la transacción; reutiliza el objeto `pet` obtenido expresamente o prepara datos de presentación dentro del servicio.

**Caso D — Completar una solicitud cancelada provoca un 500.**

Comprueba que el error sea `BusinessRuleException`, que exista `PageErrorHandler` y que no haya un `catch` general que reemplace o esconda la excepción.

**Caso E — Se borra una mascota con historial.**

Revisa que `PetService` reciba `SupportRequestRepository`, que el método `delete(...)` consulte `existsByPet_Id` y que no hayas añadido una cascada de eliminación ajena a las reglas del ejercicio.

**Caso F — Un test de servicio ya no compila.**

Compara cómo construía `PetService` antes y después de añadir la segunda dependencia. Actualiza la configuración del mock y conserva las pruebas anteriores.

### 14. Reto integrador — Historial y consistencia

Completa la siguiente comprobación usando una mascota creada desde la interfaz:

1. Registra una solicitud de paseo para esa mascota.
2. Comprueba en MariaDB que `pet_id` corresponde al id de la mascota.
3. Intenta registrar una segunda solicitud abierta y verifica que la aplicación informa el conflicto sin crear otra fila.
4. Reemplaza la solicitud abierta con una solicitud de otro tipo.
5. Comprueba que la original está `CANCELLED` y la nueva está `OPEN`.
6. Completa la nueva solicitud e intenta completarla por segunda vez: debe producir `409 Conflict`.
7. Intenta eliminar la mascota desde su formulario de eliminación: la aplicación debe impedirlo porque existen solicitudes asociadas.
8. Reinicia la aplicación y comprueba que el historial sigue disponible.
9. Ejecuta las pruebas automatizadas y explica qué comprueba cada grupo de pruebas.

**Condición importante:** para ninguna de estas operaciones necesitas modificar a mano la tabla `support_requests`. El navegador llama a controladores que utilizan servicios y repositorios; las consultas SQL sirven para verificar los resultados.

### 15. Ampliación opcional — Una consulta derivada más

Agrega al repositorio un método para listar las solicitudes de una mascota que tienen un estado determinado:

```java
List<SupportRequest> findByPet_IdAndStatusOrderByIdDesc(
    Long petId,
    RequestStatus status
);
```

Después incorpora en `SupportRequestService` un método de lectura que reciba la mascota y el estado, valide que existe la mascota y delegue en esta consulta.

Antes de añadir filtros a Thymeleaf, pruébalo desde una prueba JPA con solicitudes en distintos estados. Debes poder explicar por qué una consulta por estado pertenece al repositorio y por qué decidir **si se permite una transición** sigue siendo responsabilidad del dominio y el servicio.

### Comprobación final

El funcionamiento esperado es:

| Comprobación | Resultado esperado |
|---|---|
| Crear solicitud para mascota existente | Se guarda una fila con `pet_id` correcto y estado `OPEN` |
| Crear solicitud sin descripción | Se muestra el formulario con errores y no se guarda |
| Crear solicitud para mascota inexistente | Respuesta 404 |
| Crear segunda solicitud abierta | Error visible sin insertar otra fila |
| Completar solicitud abierta | La solicitud pasa a `COMPLETED` |
| Cancelar solicitud abierta | La solicitud pasa a `CANCELLED` |
| Completar una solicitud cancelada | Respuesta 409 y estado sin cambios |
| Actuar sobre solicitud de otra mascota | Respuesta 404 |
| Reemplazar solicitud abierta | La original pasa a `CANCELLED` y aparece una nueva `OPEN` |
| Fallo transaccional antes de confirmar | Los cambios de esa transacción se revierten |
| Eliminar mascota sin solicitudes | Se elimina mediante el CRUD existente |
| Eliminar mascota con solicitudes | Respuesta 409 y mascota conservada |
| Consultar después de reiniciar | Las filas confirmadas siguen en MariaDB |
| Ejecutar pruebas | Pruebas de entidad, repositorio, servicio, transacciones y MVC terminan correctamente |

---

## Mapa final del proyecto

Al terminar, el proyecto contendrá estos componentes principales:

```text
petmatch-lab/
├── pom.xml
├── mvnw
├── mvnw.cmd
└── src/
    ├── main/
    │   ├── java/com/example/petmatchlab/
    │   │   ├── PetMatchLabApplication.java
    │   │   ├── controller/
    │   │   │   ├── HomeController.java
    │   │   │   ├── PetController.java
    │   │   │   ├── SupportRequestController.java
    │   │   │   └── PageErrorHandler.java
    │   │   ├── dto/
    │   │   │   ├── PetForm.java
    │   │   │   └── SupportRequestForm.java
    │   │   ├── exception/
    │   │   │   ├── BusinessRuleException.java
    │   │   │   └── ResourceNotFoundException.java
    │   │   ├── model/
    │   │   │   ├── Pet.java
    │   │   │   ├── SupportRequest.java
    │   │   │   ├── SupportType.java
    │   │   │   └── RequestStatus.java
    │   │   ├── repository/
    │   │   │   ├── PetRepository.java
    │   │   │   └── SupportRequestRepository.java
    │   │   └── service/
    │   │       ├── PetService.java
    │   │       └── SupportRequestService.java
    │   └── resources/
    │       ├── application.yaml
    │       ├── static/css/application.css
    │       └── templates/
    │           ├── home.html
    │           ├── about.html
    │           ├── fragments/
    │           │   ├── head.html
    │           │   └── navigation.html
    │           ├── pets/
    │           │   ├── list.html
    │           │   ├── detail.html
    │           │   └── form.html
    │           ├── requests/
    │           │   ├── list.html
    │           │   └── form.html
    │           └── errors/
    │               └── status.html
    └── test/
        ├── java/com/example/petmatchlab/
        │   ├── controller/
        │   │   ├── PetControllerIntegrationTests.java
        │   │   └── SupportRequestControllerIntegrationTests.java
        │   ├── model/
        │   │   └── SupportRequestTests.java
        │   ├── repository/
        │   │   ├── PetRepositoryTests.java
        │   │   └── SupportRequestRepositoryTests.java
        │   └── service/
        │       ├── PetServiceTests.java
        │       ├── SupportRequestServiceTests.java
        │       └── SupportRequestTransactionIntegrationTests.java
        └── resources/
            └── application.yaml
```

La relación persiste en dos tablas principales:

```text
pets (1)
    |
    | pets.id = support_requests.pet_id
    v
support_requests (N)
```

Y el recorrido de una operación completa es:

```text
HTML / Thymeleaf
       |
       | POST
       v
SupportRequestController
       |
       | SupportRequestForm + @Valid
       v
SupportRequestService
       |
       | Reglas de negocio + @Transactional
       v
SupportRequest / Pet
       |
       | Estado y relación
       v
SupportRequestRepository / PetRepository
       |
       | JPA + Hibernate + JDBC
       v
MariaDB
```

## Diagnóstico final — Comprueba que comprendiste

Responde sin utilizar únicamente definiciones memorizadas:

1. ¿Por qué la columna `pet_id` vive en `support_requests` y no en `pets`?
2. ¿Qué representa `@ManyToOne` y qué representa `@JoinColumn`?
3. ¿Cuándo es útil `@OneToMany(mappedBy = "pet")` y por qué no fue imprescindible aquí?
4. ¿Por qué `SupportRequestForm` no contiene un campo `status`?
5. ¿Qué diferencia hay entre una restricción `@NotBlank` y una regla como «solo una solicitud abierta»?
6. ¿Qué ventaja ofrece `findByIdAndPet_Id(...)` frente a consultar solo por id?
7. ¿Por qué el servicio no permite eliminar una mascota con solicitudes?
8. ¿Qué ocurre si llamas dos veces a `complete()` sobre la misma solicitud?
9. ¿Por qué el método `replace(...)` usa `@Transactional`?
10. ¿Qué sucede con el primer cambio si la segunda parte de una transacción falla y se produce rollback?
11. ¿Por qué `readOnly = true` no debe entenderse como un sistema de permisos?
12. ¿Qué diferencia existe entre una respuesta 404 y una respuesta 409?
13. ¿Qué demuestra una prueba unitaria que no demuestra una prueba de repositorio? ¿Y a la inversa?
14. ¿Por qué una base de pruebas H2 no garantiza por sí sola que todo funcionará exactamente igual en MariaDB?
15. ¿Qué riesgo existe si dos personas intentan crear simultáneamente la primera solicitud abierta de una mascota?

## Cierre del laboratorio

Has incorporado un problema que una aplicación CRUD sencilla todavía no resolvía: **los datos están relacionados y tienen reglas que limitan sus cambios**. Ahora una solicitud pertenece a una mascota persistida, conserva su historial y solo acepta transiciones de estado válidas.

Has separado nuevamente las responsabilidades: JPA describe la relación; el repositorio responde consultas; el dominio protege los cambios de estado; el servicio coordina reglas y transacciones; Spring MVC transforma operaciones y excepciones en respuestas web; y Thymeleaf presenta la información.

La idea central es que **persistir datos correctamente no consiste solamente en ejecutar `save()`**. Consiste en mantener relaciones válidas, impedir cambios indebidos, agrupar operaciones coherentes y comprobar el comportamiento tanto cuando todo funciona como cuando ocurre un fallo.

## Referencias oficiales

- [Jakarta Persistence 3.2 — `@ManyToOne`](https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/manytoone): definición de asociaciones muchos a uno.
- [Jakarta Persistence 3.2 — `@OneToMany`](https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/onetomany): asociaciones uno a muchos, `mappedBy` y cascadas.
- [Jakarta Persistence 3.2 — `@JoinColumn`](https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/joincolumn): columnas de unión y claves foráneas.
- [Jakarta Persistence 3.2 — `@Enumerated`](https://jakarta.ee/specifications/persistence/3.2/apidocs/jakarta.persistence/jakarta/persistence/enumerated): persistencia de valores enum.
- [Spring Data JPA — Query Methods](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html): consultas derivadas por nombres de métodos.
- [Spring Data — Property Expressions](https://docs.spring.io/spring-data/jpa/reference/repositories/query-methods-details.html#repositories.query-methods.query-property-expressions): navegación de propiedades anidadas en consultas.
- [Spring Framework — Declarative Transactions](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html): `@Transactional`, propagación, rollback y comportamiento de proxy.
- [Spring Framework — Transaction Management](https://docs.spring.io/spring-framework/reference/data-access/transaction.html): administración de transacciones e infraestructura relacionada.
- [Spring Framework — MVC Exception Handling](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-exceptionhandler.html): `@ExceptionHandler` y `@ControllerAdvice`.
- [Spring Boot — SQL Databases](https://docs.spring.io/spring-boot/reference/data/sql.html): JPA, bases SQL y `open-in-view`.
- [Spring Boot — Database Initialization](https://docs.spring.io/spring-boot/how-to/data-initialization.html): configuración y precauciones de `ddl-auto`.
- [Spring Boot — Testing](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html): pruebas con contexto Spring y acceso a datos.
- [Jakarta Bean Validation 3.1](https://jakarta.ee/specifications/bean-validation/3.1/): restricciones declarativas para datos de entrada.
- [MariaDB — Foreign Keys](https://mariadb.com/docs/server/architecture/server-constraints/foreign-key-constraints): claves foráneas e integridad referencial.
- [Java SE 21 — Java Language Specification](https://docs.oracle.com/javase/specs/jls/se21/html/index.html): clases, herencia, excepciones y enumeraciones.
