# Laboratorio 8 — Formularios, validación y operaciones CRUD con Spring Boot y Thymeleaf

En una aplicación web no siempre basta con **mostrar** información. También necesitamos que una persona pueda escribir datos, enviarlos, corregir errores y modificar registros existentes. Hasta ahora, **PetMatch Mini** mostraba un catálogo de mascotas almacenadas en una colección Java. En este laboratorio vas a convertirlo en una aplicación interactiva: podrás **registrar, consultar, editar y eliminar mascotas** desde el navegador.

Las clases y herramientas que ya conoces siguen presentes. `Pet` representa los datos de una mascota; `PetController` recibe peticiones HTTP; `PetService` coordina las operaciones; `InMemoryPetRepository` mantiene los objetos; y Thymeleaf construye el HTML. Lo nuevo será aprender **cómo llegan los datos desde un formulario a Java** y qué hacer cuando esos datos no son válidos.

No necesitas conocer bases de datos ni SQL para realizar las prácticas. Los registros seguirán almacenados **en memoria**: los cambios estarán disponibles mientras la aplicación esté ejecutándose y se perderán al reiniciarla. Esta característica permite observar las operaciones web sin incorporar todavía un mecanismo de persistencia.

| Práctica | Aprendizaje principal | Resultado comprobable |
|---|---|---|
| 8.1 | HTTP, formularios HTML, GET, POST y petición/respuesta | Explicar qué envía el navegador y qué recibe el servidor |
| 8.2 | Form DTO, `@ModelAttribute`, `th:object` y `th:field` | Construir un formulario conectado con un objeto Java |
| 8.3 | Bean Validation, `@Valid`, `BindingResult` y errores de Thymeleaf | Detectar entradas incorrectas sin perder los datos introducidos |
| 8.4 | Creación, repositorio modificable, identificadores y Post/Redirect/Get | Registrar una mascota y verla en el catálogo |
| 8.5 | Edición, eliminación, rutas, errores 404 y reutilización de vistas | Completar las operaciones CRUD |
| 8.6 | Pruebas unitarias, pruebas MVC, depuración y reto integrador | Verificar los recorridos correctos y los casos de error |

> **Forma de trabajo:** antes de ejecutar cada ejemplo, predice qué debería suceder; después compruébalo en el navegador o en la terminal. Cuando un resultado sea diferente, sigue el recorrido de la petición y localiza la responsabilidad correspondiente. Los fragmentos señalados como *conceptuales* sirven para comprender una idea; los archivos indicados como *completos* sí están preparados para incorporarse al proyecto.

## Preparación del proyecto

Utiliza el proyecto **PetMatch Mini** construido en el Laboratorio 7. Debe contener, al menos:

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
    │   │   │   └── PetController.java
    │   │   ├── model/Pet.java
    │   │   ├── repository/InMemoryPetRepository.java
    │   │   └── service/PetService.java
    │   └── resources/
    │       ├── application.yaml
    │       ├── static/css/application.css
    │       └── templates/
    │           ├── home.html
    │           ├── about.html
    │           ├── fragments/
    │           │   ├── head.html
    │           │   └── navigation.html
    │           └── pets/
    │               ├── list.html
    │               └── detail.html
    └── test/java/com/example/petmatchlab/
        └── service/PetServiceTests.java
```

Si añadiste mascotas adicionales o cambiaste sus nombres al resolver ejercicios anteriores, conserva esos datos y adapta los ejemplos. En los resultados ilustrativos de este documento se utilizan tres mascotas iniciales: **Luna**, **Milo** y **Nube**.

### Comprobar el punto de partida

Desde la carpeta que contiene `pom.xml`, ejecuta:

**Linux o macOS:**

```bash
./mvnw test
./mvnw spring-boot:run
```

**Windows PowerShell:**

```powershell
.\mvnw.cmd test
.\mvnw.cmd spring-boot:run
```

Comprueba que puedes visitar `http://localhost:8080/pets` y acceder al detalle de una mascota. Detén el servidor con **Ctrl+C** antes de modificar las clases principales. Para editar el código necesitarás **JDK 21**, el mismo utilizado en el proyecto anterior.

> Si tus clases usan otro paquete raíz, adapta todas las declaraciones `package` e instrucciones `import` de este documento. El paquete de los ejemplos será `com.example.petmatchlab` de principio a fin.

### Añadir la dependencia de validación

El proyecto ya utiliza Spring MVC y Thymeleaf. Para trabajar con anotaciones de validación necesitas añadir un *starter* a la sección `<dependencies>` de `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

**No reemplaces las dependencias existentes.** Conserva `spring-boot-starter-webmvc`, `spring-boot-starter-thymeleaf` y `spring-boot-starter-webmvc-test`. Con el `spring-boot-starter-parent` del proyecto, Maven administra las versiones compatibles y normalmente no debes escribir `<version>` en estas dependencias.

Para que los ejemplos sigan siendo compatibles con el proyecto original, se mantiene la configuración de **Spring Boot 4.1.1** y **Java 21**. Se utilizarán clases de `jakarta.validation`, no los paquetes antiguos `javax.validation`.

Verifica que las dependencias se resuelvan correctamente:

```bash
./mvnw test
```

O en PowerShell:

```powershell
.\mvnw.cmd test
```

La primera ejecución después de cambiar el `pom.xml` puede necesitar descargar bibliotecas. Si Maven indica que no pudo resolver una dependencia, revisa el nombre del artefacto, la conexión a los repositorios y el mensaje concreto antes de cambiar versiones al azar.

## Antes de comenzar — Diagnóstico

Responde con tus palabras:

1. En el catálogo actual, ¿de dónde salen las mascotas que aparecen en pantalla?
2. ¿Qué diferencia imaginas entre abrir una página y enviar información desde esa página?
3. Si una persona escribe `-5` como edad, ¿quién debería impedir que se registre?
4. ¿Por qué un formulario de registro no debería permitir que quien lo completa elija libremente el identificador de la mascota?
5. Si modificas una mascota guardada en una colección, ¿se conservará el cambio después de cerrar y reiniciar Java?

Vuelve a estas preguntas al finalizar. No necesitas memorizar las respuestas: debes poder justificarlas siguiendo una petición real.

---

## Práctica 8.1 — HTTP y formularios: del navegador al servidor

### Objetivos

Al terminar esta práctica podrás:

- describir una petición y una respuesta HTTP;
- distinguir los métodos `GET` y `POST`;
- reconocer los atributos `action`, `method` y `name` de un formulario;
- diferenciar una URL de los datos que viajan en el cuerpo de una petición;
- explicar por qué guardar o eliminar información no debe depender de un enlace `GET`.

### 1. Hasta ahora, el navegador solo solicitaba información

En el catálogo construido anteriormente, visitar `/pets` provoca una petición similar a:

```http
GET /pets HTTP/1.1
Host: localhost:8080
```

**Esta representación es conceptual**: los navegadores incluyen otros encabezados y pueden usar versiones distintas de HTTP. Lo importante aquí es reconocer dos partes:

- `GET` expresa que se solicita una representación del recurso.
- `/pets` identifica la dirección a la que se dirige esa petición.

Spring MVC encuentra el método correspondiente en `PetController`. Después, el controlador obtiene datos a través del servicio y devuelve el nombre de una vista. Thymeleaf genera el HTML que se envía como **respuesta**.

```text
Navegador
  └── GET /pets
        └── PetController
              └── PetService
                    └── InMemoryPetRepository
              └── Model + "pets/list"
        └── Thymeleaf genera HTML
  └── Navegador muestra la página
```

Una respuesta satisfactoria suele tener el código de estado **200 OK**. El código es una propiedad de la respuesta HTTP; no es un mensaje definido por el HTML de la página.

### 2. Un formulario también produce una petición

Observa este ejemplo **conceptual**; todavía no debes copiarlo al proyecto:

```html
<form action="/pets" method="post">
    <label for="name">Nombre</label>
    <input id="name" name="name" type="text">

    <label for="species">Especie</label>
    <input id="species" name="species" type="text">

    <button type="submit">Registrar</button>
</form>
```

Cuando una persona escribe `Luna` y `Perro` y presiona **Registrar**, el navegador envía esos valores como parámetros de formulario. De manera simplificada:

```http
POST /pets HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=Luna&species=Perro
```

`application/x-www-form-urlencoded` es un formato habitual de los formularios HTML. Los caracteres especiales se codifican; por ejemplo, un espacio puede representarse como `+`. El navegador se encarga de esa codificación.

**No confundas los nombres:**

| Elemento | Función |
|---|---|
| `action="/pets"` | Indica la ruta que recibirá el envío |
| `method="post"` | Indica el método HTTP para enviar el formulario |
| `name="name"` | Define la clave del dato enviado por ese campo |
| `id="name"` | Identifica el elemento HTML; permite asociarlo al `<label>` |
| `type="text"` | Indica la clase de control que verá la persona |
| `type="submit"` | Convierte el botón en una acción de envío |

El valor de `id` **no sustituye** al atributo `name`: si un control no tiene `name`, normalmente su valor no formará parte de los datos enviados al servidor.

### 3. Diferenciar `GET` y `POST`

En este laboratorio utilizarás:

| Método | Uso | Ejemplo |
|---|---|---|
| `GET` | Consultar una página o representación sin solicitar una modificación de estado | Abrir `/pets/new` para ver un formulario |
| `POST` | Enviar datos para que el servidor procese una operación que puede modificar el estado | Enviar `/pets` para registrar una mascota |

**Mostrar un formulario** y **procesar sus datos** son acciones distintas:

```text
GET /pets/new  → mostrar una plantilla con campos
POST /pets     → recibir datos, validarlos y registrar
```

No uses un enlace como `<a href="/pets/3/delete">Eliminar</a>` para borrar un registro. Los navegadores, buscadores o herramientas automáticas pueden seguir enlaces `GET` sin que exista una intención explícita de modificar información. En el proyecto, la eliminación se implementará mediante un formulario con `method="post"`.

> `POST` no significa que una operación sea automáticamente segura, autenticada o autorizada. En esta aplicación introductoria no se han añadido usuarios ni permisos. Utilízala en tu entorno local; no publiques estos endpoints de modificación sin incorporar las protecciones correspondientes.

### 4. El navegador no habla directamente con el repositorio

Una petición de registro tendrá este recorrido:

```text
Formulario HTML
      │
      │ POST /pets + campos enviados
      ▼
Spring MVC
      │
      ▼
PetController  ── recibe los valores y decide la respuesta HTTP
      │
      ▼
PetService     ── coordina la operación y aplica reglas
      │
      ▼
InMemoryPetRepository ── modifica la colección Java
      │
      ▼
Respuesta HTTP ── muestra errores o redirige a otra página
```

El repositorio **no recibe peticiones HTTP**. Sus métodos son métodos Java normales llamados desde la capa de servicio. Del mismo modo, una plantilla Thymeleaf no debería encargarse de insertar o eliminar objetos en la colección.

### 5. Observar peticiones con el navegador

1. Abre `/pets` y entra a las herramientas de desarrollo del navegador.
2. Localiza la pestaña **Network / Red**.
3. Vuelve a cargar la página y selecciona la petición cuyo nombre o ruta corresponda a `/pets`.
4. Identifica el método HTTP y el código de estado.
5. Repite la observación al abrir `/pets/1`.

Por ahora verás principalmente peticiones `GET` que devuelven HTML. Más adelante repetirás la observación cuando envíes formularios `POST`.

### Antes de ejecutar — Predice

Si una persona abre `/pets/new` y después envía datos a `/pets`, responde:

1. ¿Cuántas peticiones diferentes pueden observarse?
2. ¿Cuál de las dos debería modificar la colección de mascotas?
3. ¿Dónde se encontrarán los valores de `name` y `species` en el formulario enviado por `POST`?
4. ¿Qué debería pasar si la aplicación recibe un método HTTP que no está asociado a esa ruta?

### Ejercicio — Reconoce las partes

En el formulario conceptual anterior, cambia temporalmente `name="species"` por `name="petType"`. Explica qué nombre recibiría el servidor y por qué un objeto Java con propiedad `species` ya no se completaría automáticamente con ese valor. No necesitas ejecutar el formulario todavía.

### Comprobación

Debes poder explicar la diferencia entre `GET /pets/new` y `POST /pets`, por qué `name` es importante y por qué las acciones que modifican información no deben implementarse mediante enlaces `GET`.

---

## Práctica 8.2 — Form DTO y enlace de formularios con objetos Java

### Objetivos

Al finalizar podrás:

- diferenciar el objeto de dominio `Pet` de un objeto de entrada `PetForm`;
- explicar qué significa DTO y para qué sirve en un formulario;
- reconocer la responsabilidad de `@ModelAttribute`;
- utilizar `th:object` y `th:field`;
- mostrar el formulario con una ruta `GET` sin guardar datos todavía.

### 1. ¿Por qué no recibir directamente un `Pet`?

Recuerda la clase `Pet` del laboratorio anterior. Tiene los atributos `id`, `name`, `species`, `age` y `description`, definidos como `final`, y un constructor que recibe todos los valores. Esa decisión era apropiada para el catálogo: una mascota se construía completa y después se consultaba.

En cambio, al abrir un formulario de registro **todavía no existe una mascota registrada**. Primero tenemos valores que una persona está escribiendo. Esos valores pueden estar incompletos o ser incorrectos. Además, el identificador `id` debe decidirlo la aplicación, **no** la persona que completa el formulario.

Por eso crearemos otra clase:

```text
PetForm  → datos recibidos de la interfaz
Pet      → objeto de dominio de una mascota registrada
```

**DTO** significa *Data Transfer Object*, u objeto de transferencia de datos. En este caso, `PetForm` define cuáles valores puede recibir el formulario y ofrece un lugar donde expresar reglas de entrada. Tener dos clases parecidas no significa que sean duplicadas sin propósito: representan responsabilidades diferentes.

### 2. Crear `PetForm`

Crea el paquete `dto` y el archivo `src/main/java/com/example/petmatchlab/dto/PetForm.java`:

```java
package com.example.petmatchlab.dto;

import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;

public class PetForm {

    @NotBlank(message = "El nombre es obligatorio")
    @Size(max = 100, message = "El nombre no puede superar los 100 caracteres")
    private String name;

    @NotBlank(message = "La especie es obligatoria")
    @Size(max = 80, message = "La especie no puede superar los 80 caracteres")
    private String species;

    @NotNull(message = "La edad es obligatoria")
    @Min(value = 0, message = "La edad no puede ser negativa")
    private Integer age;

    @Size(max = 1000, message = "La descripción no puede superar los 1000 caracteres")
    private String description;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSpecies() {
        return species;
    }

    public void setSpecies(String species) {
        this.species = species;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```

**Léela como una clase Java convencional:**

- Sus atributos son `private` para conservar el encapsulamiento.
- Los métodos `get...()` permiten consultar las propiedades.
- Los métodos `set...()` permiten asignar los valores que se reciben desde el formulario.
- Como no se declaró un constructor con parámetros, Java proporciona un constructor sin argumentos. Spring puede crear el objeto vacío y después completar sus propiedades.
- Las anotaciones `@NotBlank`, `@Size`, `@NotNull` y `@Min` describen restricciones. En la siguiente práctica comprobarás cómo se ejecutan.

**¿Por qué `Integer` y no `int` para la edad?** `int` siempre tiene un valor numérico inicial, normalmente `0`, mientras que `Integer` puede representar la ausencia del dato mediante `null`. Un campo de edad sin completar debe distinguirse de una edad válida de cero años.

**¿Por qué `PetForm` no contiene `id`?** El formulario de creación no decide la identidad. En edición, el identificador vendrá de la ruta `/pets/{id}`, no de un campo que cualquiera pueda manipular.

### 3. Crear la ruta que muestra el formulario

Abre `controller/PetController.java`. Añade a su clase el siguiente método, conservando los métodos de listado y detalle:

```java
@GetMapping("/new")
public String createForm(Model model) {
    model.addAttribute("petForm", new PetForm());
    model.addAttribute("editing", false);
    return "pets/form";
}
```

Agrega el import:

```java
import com.example.petmatchlab.dto.PetForm;
```

**¿Qué está pasando?**

1. `@GetMapping("/new")` atiende `GET /pets/new`, porque el controlador ya tiene `@RequestMapping("/pets")`.
2. `new PetForm()` construye el objeto que almacenará los valores del formulario. Todavía no crea ningún `Pet`.
3. `model.addAttribute("petForm", ...)` entrega ese objeto a Thymeleaf bajo la clave `petForm`.
4. `editing = false` indica a la plantilla que estás creando una mascota nueva. Más adelante reutilizarás esa plantilla para editar.
5. `return "pets/form"` pide renderizar `templates/pets/form.html`.

> Es importante declarar `GET /pets/new` antes de probar el formulario. La ruta `/pets/{id}` utilizada en el detalle espera un identificador numérico; `/pets/new` debe resolverse mediante una ruta fija propia.

### 4. Construir una primera versión del formulario

Crea `src/main/resources/templates/pets/form.html` con este contenido **completo**:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Registrar mascota | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <section class="content-panel">
            <h1>Registrar mascota</h1>

            <form th:action="@{/pets}" th:object="${petForm}" method="post">
                <div>
                    <label for="name">Nombre</label>
                    <input id="name" type="text" th:field="*{name}">
                </div>

                <div>
                    <label for="species">Especie</label>
                    <input id="species" type="text" th:field="*{species}">
                </div>

                <div>
                    <label for="age">Edad</label>
                    <input id="age" type="number" th:field="*{age}">
                </div>

                <div>
                    <label for="description">Descripción</label>
                    <textarea id="description" th:field="*{description}"></textarea>
                </div>

                <button type="submit">Registrar</button>
                <a th:href="@{/pets}">Cancelar</a>
            </form>
        </section>
    </main>
</body>
</html>
```

Inicia la aplicación y abre:

```text
http://localhost:8080/pets/new
```

**Resultado esperado:** la página presenta cuatro campos y un botón para registrar. **Todavía no presiones el botón**: la ruta `POST /pets` no está implementada y, en este punto, el servidor no sabe procesar el envío. Es normal que esa petición no funcione hasta completar la práctica 8.4.

### 5. Comprender `th:object` y `th:field`

Observa estas dos expresiones:

```html
<form th:object="${petForm}">
    <input th:field="*{name}">
</form>
```

`th:object="${petForm}"` indica que el formulario utilizará como objeto principal el atributo `petForm` entregado por el controlador.

`th:field="*{name}"` señala la propiedad `name` **de ese objeto seleccionado**. Thymeleaf genera y completa los atributos necesarios del control, incluido el nombre con el que el navegador enviará el dato y su valor cuando exista.

La diferencia puede representarse así:

```text
Model de Spring MVC
    └── "petForm" → PetForm
                       ├── name
                       ├── species
                       ├── age
                       └── description

th:object="${petForm}"  → selecciona el objeto
th:field="*{name}"      → selecciona una propiedad del objeto
```

No confundas `${...}` con `*{...}`: la primera se usa habitualmente para consultar variables del contexto; la segunda consulta propiedades del objeto seleccionado por `th:object`.

Tampoco confundas `th:field` con una operación de almacenamiento. **Thymeleaf no guarda la mascota.** Solo genera un formulario HTML que el navegador podrá enviar a un controlador.

### 6. ¿Cómo se convierte un formulario en un objeto Java?

Spring MVC ofrece el mecanismo llamado *data binding*, o enlace de datos. Cuando el navegador envía los campos:

```text
name=Luna
species=Perro
age=3
description=Le gusta correr
```

Spring MVC puede crear un `PetForm` y asignar sus propiedades utilizando métodos como `setName(...)` y `setAge(...)`.

En el futuro método de recepción verás una declaración semejante a:

```java
@ModelAttribute("petForm") PetForm petForm
```

`@ModelAttribute` le indica a Spring MVC qué objeto debe utilizar para enlazar los datos recibidos. No es necesario leer manualmente cada campo del cuerpo HTTP ni escribir `new PetForm()` dentro del método que recibe el `POST`.

**Importante:** el navegador envía texto. Spring MVC intenta convertir los valores al tipo de cada propiedad; por ejemplo, la cadena `"3"` debe convertirse a `Integer`. Si una persona envía `"abc"` como edad, esa conversión falla. Aprenderás a mostrar ese error en lugar de permitir que interrumpa sin explicación el flujo del formulario.

### Ejercicio — Sigue una propiedad

En el código de `PetForm`, localiza la propiedad `species`, su método de lectura y su método de escritura. Después localiza en el HTML el control `th:field="*{species}"`. Explica la conexión entre ambas piezas.

**Reto:** agrega a la vista una breve explicación debajo del campo de edad, sin añadir todavía nuevos atributos a `Pet`. La explicación pertenece a la presentación, no al modelo del dominio.

### Depuración — Tres problemas comunes

1. Cambia temporalmente `th:object="${petForm}"` por `th:object="${animalForm}"`. ¿Por qué Thymeleaf no encuentra el objeto esperado?
2. Cambia `*{species}` por `*{type}` sin añadir esa propiedad al DTO. ¿Qué error de enlace o resolución de propiedad aparece?
3. Omite el `model.addAttribute("petForm", new PetForm())` del controlador. ¿Qué necesita la plantilla para renderizar correctamente el formulario?

Restaura cada cambio antes de continuar.

### Comprobación

Debes poder explicar por qué existen `Pet` y `PetForm`, qué significa DTO, qué hacen `th:object` y `th:field`, y de dónde salen los valores que Spring colocará en el objeto Java cuando se envíe el formulario.

---
## Práctica 8.3 — Validación: impedir datos incorrectos y explicar los errores

### Objetivos

Al finalizar podrás:

- distinguir la validación HTML de la validación que realiza el servidor;
- interpretar las restricciones `@NotBlank`, `@NotNull`, `@Min` y `@Size`;
- explicar para qué sirven `@Valid` y `BindingResult`;
- mostrar errores junto al campo correspondiente mediante Thymeleaf;
- conservar los valores introducidos cuando un formulario contiene errores;
- reconocer un fallo de conversión de tipos.

### 1. Recibir datos no significa que sean correctos

Imagina que una persona envía:

```text
name=
species=   
age=-3
description=Una descripción cualquiera
```

El navegador puede enviar esos valores y Spring MVC puede intentar convertirlos a un objeto `PetForm`. Pero nada garantiza que representen una mascota válida.

Necesitamos **comprobar los datos antes de registrar el objeto**. En una aplicación web hay varios niveles con responsabilidades diferentes:

```text
Validaciones HTML (ayuda al completar el formulario)
                │
                ▼
Enlace de datos (Spring convierte entradas a tipos Java)
                │
                ▼
Bean Validation (restricciones declaradas en PetForm)
                │
                ▼
Reglas del servicio (protegen la operación también fuera de HTTP)
                │
                ▼
Repositorio (guarda la representación en memoria)
```

La validación HTML puede mejorar la experiencia de uso, pero **no sustituye la validación del servidor**. Una petición puede construirse sin utilizar el formulario del navegador. Del mismo modo, declarar restricciones en `PetForm` no significa que cualquier operación Java se valide automáticamente: veremos dónde se solicita su ejecución.

### 2. Comprender las anotaciones de `PetForm`

La clase creada anteriormente ya contiene reglas. Revisa esta tabla:

| Restricción | Qué exige | Ejemplo que debe rechazarse |
|---|---|---|
| `@NotBlank` | Texto no nulo, no vacío y no formado solo por espacios en blanco | `name="   "` |
| `@NotNull` | Que exista un valor, sin exigir que sea positivo o no vacío | `age=null` |
| `@Min(0)` | Un valor numérico mayor o igual que cero | `age=-2` |
| `@Size(max=100)` | Un texto con longitud máxima de 100 caracteres | Un nombre con 101 caracteres |

**Dos diferencias importantes:**

- `@NotBlank` se utiliza en textos; `@NotNull` también puede emplearse en propiedades que no son texto. Una cadena de espacios cumple `@NotNull`, pero incumple `@NotBlank`.
- `@Size(max=1000)` no hace obligatorio el campo `description`. Si su valor es `null`, esa restricción por sí sola no produce un error. Para convertirlo en obligatorio habría que añadir otras reglas.

El mensaje escrito en cada anotación, por ejemplo `"La edad no puede ser negativa"`, será el texto que podremos mostrar al usuario cuando se incumpla esa condición.

### 3. ¿Qué hace `@Valid`?

Las anotaciones colocadas en `PetForm` describen reglas, pero necesitamos solicitar la validación cuando Spring reciba ese objeto. Para hacerlo, el futuro método `POST` utilizará:

```java
@Valid @ModelAttribute("petForm") PetForm petForm
```

`@Valid` procede de `jakarta.validation.Valid`. Indica que Spring debe aplicar las restricciones de Bean Validation al objeto recibido durante el procesamiento de esta petición.

**No es una garantía general de seguridad ni ejecuta todas las reglas del dominio.** En particular, `@Valid` no verifica automáticamente que una mascota exista, que un usuario tenga permisos ni que se haya guardado algún registro. Solo aplica las restricciones configuradas para el objeto y las validaciones correspondientes.

### 4. ¿Para qué sirve `BindingResult`?

Cuando la entrada es incorrecta, necesitamos conocer los problemas y volver a mostrar el mismo formulario con mensajes útiles. Spring MVC permite recibir los errores en un parámetro `BindingResult`:

```java
@PostMapping
public String create(
    @Valid @ModelAttribute("petForm") PetForm petForm,
    BindingResult bindingResult,
    Model model
) {
    if (bindingResult.hasErrors()) {
        model.addAttribute("editing", false);
        return "pets/form";
    }

    // Valid values will be processed in the next practice.
    return "redirect:/pets";
}
```

**Este bloque es una demostración parcial, no el método definitivo:** aún faltan la creación real y el mensaje de confirmación. Lo importante es observar el orden de los parámetros.

`BindingResult` debe estar **inmediatamente después** del objeto al que corresponde (`@Valid @ModelAttribute("petForm") PetForm petForm`). Spring lo utiliza para registrar errores de validación y de conversión. Si cambias el orden e intercalas otros parámetros, el tratamiento de esos errores puede no ser el esperado.

Con `bindingResult.hasErrors()` puedes preguntar si ocurrió algún problema. En caso afirmativo, devuelve la vista directamente:

```java
return "pets/form";
```

No hagas un `redirect:` en esta situación: una nueva petición perdería el objeto y la información de errores que necesitas mostrar, salvo que implementaras una solución adicional para conservarlos.

### 5. Conservar lo que se escribió

Supón que alguien envía:

```text
Nombre: Luna
Especie: Perro
Edad: -5
Descripción: Juega mucho
```

Solo la edad es incorrecta. Una respuesta útil debe conservar los demás campos para que la persona no necesite introducirlos de nuevo.

Ese comportamiento se obtiene al devolver `"pets/form"` con el **mismo** `PetForm` enlazado y el `BindingResult` que contiene el error. Thymeleaf puede volver a mostrar los valores introducidos gracias a `th:field`.

El servidor **no debe registrar** una mascota cuando `bindingResult.hasErrors()` es verdadero.

### 6. Actualizar la plantilla para mostrar errores

Sustituye `src/main/resources/templates/pets/form.html` por este archivo **completo**. La plantilla ya quedará preparada tanto para crear como para editar; la edición se implementará más adelante.

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Formulario | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <section class="content-panel">
            <h1 th:text="${editing} ? 'Editar mascota' : 'Registrar mascota'">
                Formulario de mascota
            </h1>

            <p class="page-intro">
                Completa los datos de la mascota. La descripción es opcional.
            </p>

            <form th:action="${editing} ? @{/pets/{id}(id=${petId})} : @{/pets}"
                  th:object="${petForm}"
                  method="post"
                  novalidate>

                <p class="form-summary" th:if="${#fields.hasAnyErrors()}"
                   role="alert">
                    Revisa los campos señalados antes de continuar.
                </p>

                <div class="form-field">
                    <label for="name">Nombre</label>
                    <input id="name" type="text"
                           th:field="*{name}"
                           maxlength="100"
                           th:errorclass="field-error"
                           th:attr="aria-invalid=${#fields.hasErrors('name')}">
                    <p class="field-error-message"
                       th:if="${#fields.hasErrors('name')}"
                       th:errors="*{name}" id="name-error" role="alert"></p>
                </div>

                <div class="form-field">
                    <label for="species">Especie</label>
                    <input id="species" type="text"
                           th:field="*{species}"
                           maxlength="80"
                           th:errorclass="field-error"
                           th:attr="aria-invalid=${#fields.hasErrors('species')}">
                    <p class="field-error-message"
                       th:if="${#fields.hasErrors('species')}"
                       th:errors="*{species}" id="species-error" role="alert"></p>
                </div>

                <div class="form-field">
                    <label for="age">Edad en años</label>
                    <input id="age" type="number"
                           th:field="*{age}"
                           min="0"
                           th:errorclass="field-error"
                           th:attr="aria-invalid=${#fields.hasErrors('age')}">
                    <p class="field-error-message"
                       th:if="${#fields.hasErrors('age')}"
                       th:errors="*{age}" id="age-error" role="alert"></p>
                </div>

                <div class="form-field">
                    <label for="description">Descripción (opcional)</label>
                    <textarea id="description" rows="4"
                              th:field="*{description}"
                              maxlength="1000"
                              th:errorclass="field-error"
                              th:attr="aria-invalid=${#fields.hasErrors('description')}">
                    </textarea>
                    <p class="field-error-message"
                       th:if="${#fields.hasErrors('description')}"
                       th:errors="*{description}"
                       id="description-error" role="alert"></p>
                </div>

                <div class="form-actions">
                    <button class="button-link" type="submit"
                            th:text="${editing} ? 'Guardar cambios' : 'Registrar mascota'">
                        Guardar
                    </button>
                    <a class="secondary-link" th:href="@{/pets}">Cancelar</a>
                </div>
            </form>
        </section>
    </main>
</body>
</html>
```

**Observa los elementos nuevos:**

- `th:action` elige la ruta de envío según el atributo `editing` que entrega el controlador. Si es `false`, utiliza `/pets`; cuando sea `true`, utilizará `/pets/{id}`.
- `th:object="${petForm}"` mantiene el objeto que recibe y muestra la información.
- `#fields.hasAnyErrors()` permite presentar un aviso general cuando existe algún problema.
- `#fields.hasErrors('age')` comprueba si un campo concreto tiene errores.
- `th:errors="*{age}"` muestra los mensajes registrados para ese campo.
- `th:errorclass="field-error"` añade una clase CSS al control cuando hay errores. No es una validación nueva: solo cambia la presentación.
- `aria-invalid` informa a tecnologías de asistencia de que el campo tiene un valor incorrecto.

El atributo HTML `novalidate` se utiliza **deliberadamente durante la práctica** para que el navegador no bloquee el envío y puedas observar cómo valida el servidor. Cuando hayas comprobado esos comportamientos, puedes retirarlo para aprovechar también las ayudas de validación del navegador. **La validación del servidor debe permanecer en ambos casos.**

> El enlace a la hoja de estilos se conserva mediante el fragmento `fragments/head` creado anteriormente. Las clases nuevas (`form-field`, `field-error` y otras) recibirán estilos al completar la práctica 8.5. Antes de eso, el formulario funciona aunque algunos elementos se vean sencillos.

### 7. ¿Y si la conversión del número falla?

Una propiedad `Integer age` puede recibir un número válido, por ejemplo `3`. Pero si llega una cadena no convertible como `"abc"`, Spring no podrá asignarla a esa propiedad. El problema se registra como **error de enlace de datos** (*binding error*), y `BindingResult` permite tratarlo junto con los errores de Bean Validation.

No todos los errores nacen de `@Min` o `@NotNull`. Esta diferencia importa porque un mensaje como “La edad debe ser un número válido” describe mejor un error de conversión que “La edad no puede ser negativa”.

Si deseas proporcionar un mensaje específico para ese caso, crea opcionalmente `src/main/resources/messages.properties`:

```properties
typeMismatch.petForm.age=La edad debe ser un número entero válido.
```

Spring MVC puede utilizar esta clave de mensaje de enlace cuando falle la conversión de `age` en el objeto `petForm`. El mensaje específico depende del mecanismo de resolución de mensajes y de las claves de error generadas por Spring; si no defines uno, puede mostrarse el mensaje predeterminado.

**No necesitas este archivo para que la validación funcione.** Puedes añadirlo después de verificar que el formulario ya registra errores de conversión.

### 8. Tabla de comprobaciones

Cuando hayas implementado `POST /pets` en la práctica siguiente, utiliza estos casos:

| Entrada | Resultado esperado |
|---|---|
| Nombre vacío | Mensaje de nombre obligatorio; no se crea mascota |
| Especie con solo espacios | Mensaje de especie obligatoria; no se crea mascota |
| Edad vacía | Mensaje de edad obligatoria; no se crea mascota |
| Edad `-1` | Mensaje de edad negativa; no se crea mascota |
| Edad `abc` enviada manualmente | Error de conversión; no se crea mascota |
| Descripción vacía | Se admite y puede guardarse como ausencia de descripción |
| Datos completos y válidos | Se crea la mascota y se navega a su detalle |

### Antes de ejecutar — Predice

¿Por qué un formulario con edad `-1` debería volver a mostrar `pets/form` y no redirigir al detalle? ¿Qué sucedería con los valores válidos que estaban escritos en otros campos si el controlador hiciera un `redirect:` al detectar ese error?

### Depuración — Un error por vez

1. Mueve `BindingResult` a una posición separada del `PetForm` por un parámetro `Model`. Examina por qué esa firma no permite gestionar los errores como se esperaba.
2. Cambia `th:errors="*{age}"` por `th:errors="*{unknown}"`. ¿Qué propiedad intenta consultar Thymeleaf?
3. Elimina temporalmente `@Valid` del método `POST`. ¿Se aplicarían automáticamente las restricciones escritas en `PetForm` por el solo hecho de existir?

Restaura todo antes de continuar.

### Comprobación

Debes poder distinguir validación de conversión, explicar `@Valid` y `BindingResult`, mostrar un error junto a un campo y justificar por qué una entrada inválida no debe producir una modificación en el repositorio.

---

## Práctica 8.4 — Crear mascotas y actualizar el repositorio en memoria

### Objetivos

Al finalizar podrás:

- convertir un repositorio de solo lectura en uno modificable;
- distinguir `List`, `Map` y sus usos en un repositorio simple;
- asignar identificadores desde el servidor;
- implementar una creación coordinada por el servicio;
- procesar un formulario `POST` y responder con una redirección;
- mostrar una confirmación mediante *flash attributes*;
- explicar el patrón Post/Redirect/Get.

### 1. El repositorio anterior no permite registrar mascotas

En la versión inicial, `InMemoryPetRepository` contenía una lista creada con `List.of(...)`. Esa lista era adecuada para **consultar** tres elementos, pero no para agregar o eliminar registros de manera dinámica: las listas creadas con `List.of(...)` no permiten operaciones que cambien su contenido.

Ahora necesitamos una estructura con la que podamos:

```text
consultar todas las mascotas
buscar una mascota por id
crear una mascota con id nuevo
reemplazar una mascota existente
eliminar una mascota
```

Utilizaremos un `Map<Long, Pet>` basado en `LinkedHashMap`. Un `Map` asocia una **clave** con un **valor**:

```text
Clave (Long)       Valor (Pet)
---------------------------------------
1L                 Pet(1L, "Luna", ...)
2L                 Pet(2L, "Milo", ...)
3L                 Pet(3L, "Nube", ...)
```

La clave será el identificador de la mascota. `LinkedHashMap` conserva el orden de inserción al recorrer sus elementos, lo que ayuda a mantener un catálogo predecible. En este caso no representa una tabla ni se conecta a una base de datos: sigue siendo una colección de Java.

### 2. Sustituir el repositorio por una versión modificable

Reemplaza **todo** el contenido de `src/main/java/com/example/petmatchlab/repository/InMemoryPetRepository.java` por:

```java
package com.example.petmatchlab.repository;

import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

import org.springframework.stereotype.Repository;

import com.example.petmatchlab.model.Pet;

@Repository
public class InMemoryPetRepository {

    private final Map<Long, Pet> pets = new LinkedHashMap<>();
    private long nextId = 4L;

    public InMemoryPetRepository() {
        pets.put(1L, new Pet(1L, "Luna", "Perro", 3,
            "Disfruta los paseos y los juegos."));
        pets.put(2L, new Pet(2L, "Milo", "Gato", 2,
            "Prefiere lugares tranquilos."));
        pets.put(3L, new Pet(3L, "Nube", "Conejo", 1,
            "Es curioso y le gusta explorar."));
    }

    public synchronized List<Pet> findAll() {
        return List.copyOf(pets.values());
    }

    public synchronized Optional<Pet> findById(long id) {
        return Optional.ofNullable(pets.get(id));
    }

    public synchronized Pet create(
        String name,
        String species,
        int age,
        String description
    ) {
        long id = nextId++;
        Pet pet = new Pet(id, name, species, age, description);
        pets.put(id, pet);
        return pet;
    }

    public synchronized Optional<Pet> update(
        long id,
        String name,
        String species,
        int age,
        String description
    ) {
        if (!pets.containsKey(id)) {
            return Optional.empty();
        }

        Pet updated = new Pet(id, name, species, age, description);
        pets.put(id, updated);
        return Optional.of(updated);
    }

    public synchronized boolean delete(long id) {
        return pets.remove(id) != null;
    }
}
```

**Lee el código por partes:**

- `new LinkedHashMap<>()` crea una colección que admite altas, actualizaciones y eliminaciones.
- `pets.put(id, pet)` asocia un identificador con un objeto. Si la clave ya existe, reemplaza el valor asociado.
- `pets.get(id)` obtiene el objeto de una clave, o `null` si no existe.
- `Optional.ofNullable(...)` expresa que la búsqueda puede no encontrar nada.
- `nextId++` entrega el identificador actual y después incrementa el contador para la siguiente inserción.
- `List.copyOf(pets.values())` devuelve una copia no modificable de la colección de valores: una clase que consulta el catálogo no puede alterar el `Map` interno mediante esa lista.
- `pets.remove(id)` elimina la entrada de esa clave y devuelve el valor anterior, o `null` si no existía.

**¿Por qué aparece `synchronized`?** En una aplicación web pueden procesarse peticiones simultáneas. Este modificador evita que dos hilos ejecuten a la vez las operaciones sincronizadas sobre la misma instancia del repositorio. Para este ejemplo es una precaución sencilla; no equivale a implementar transacciones de base de datos ni resuelve necesidades de una aplicación distribuida.

**¿Por qué `nextId` empieza en `4L`?** Porque los datos de ejemplo ocupan los identificadores 1, 2 y 3. Si conservaste una cuarta mascota con id 4, inclúyela en el constructor y ajusta el contador para que el primer registro nuevo reciba un identificador distinto, por ejemplo `5L`.

**¿Por qué no hay setters en `Pet`?** La clase conserva los atributos `final` que conoces. Cuando se edite una mascota, construiremos **otro objeto `Pet` con el mismo id y los nuevos valores** y reemplazaremos la entrada del `Map`. No es necesario cambiar la clase `Pet` ni hacer sus atributos públicos.

### 3. Ampliar el servicio con operaciones y reglas básicas

La capa `Service` debe ser el lugar en el que se prepara una operación antes de solicitar su almacenamiento. En este ejemplo distinguirá entradas obligatorias, aplicará una normalización sencilla y delegará la modificación al repositorio.

Reemplaza **todo** el archivo `src/main/java/com/example/petmatchlab/service/PetService.java` por:

```java
package com.example.petmatchlab.service;

import java.util.List;
import java.util.Optional;

import org.springframework.stereotype.Service;

import com.example.petmatchlab.dto.PetForm;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.repository.InMemoryPetRepository;

@Service
public class PetService {

    private final InMemoryPetRepository petRepository;

    public PetService(InMemoryPetRepository petRepository) {
        this.petRepository = petRepository;
    }

    public List<Pet> findAll() {
        return petRepository.findAll();
    }

    public Optional<Pet> findById(long id) {
        return petRepository.findById(id);
    }

    public Pet create(PetForm form) {
        validateForm(form);

        return petRepository.create(
            form.getName().strip(),
            form.getSpecies().strip(),
            form.getAge(),
            normalizeDescription(form.getDescription())
        );
    }

    public Optional<Pet> update(long id, PetForm form) {
        validateForm(form);

        return petRepository.update(
            id,
            form.getName().strip(),
            form.getSpecies().strip(),
            form.getAge(),
            normalizeDescription(form.getDescription())
        );
    }

    public boolean delete(long id) {
        return petRepository.delete(id);
    }

    private void validateForm(PetForm form) {
        if (form == null) {
            throw new IllegalArgumentException("Pet form is required");
        }
        if (invalidRequiredText(form.getName(), 100)) {
            throw new IllegalArgumentException("Pet name is invalid");
        }
        if (invalidRequiredText(form.getSpecies(), 80)) {
            throw new IllegalArgumentException("Pet species is invalid");
        }
        if (form.getAge() == null || form.getAge() < 0) {
            throw new IllegalArgumentException("Pet age is invalid");
        }
        if (form.getDescription() != null
            && form.getDescription().length() > 1000) {
            throw new IllegalArgumentException("Pet description is too long");
        }
    }

    private boolean invalidRequiredText(String value, int maxLength) {
        return value == null
            || value.strip().isEmpty()
            || value.strip().length() > maxLength;
    }

    private String normalizeDescription(String description) {
        if (description == null || description.isBlank()) {
            return null;
        }
        return description.strip();
    }
}
```

**¿Por qué validar dos veces, en `PetForm` y en el servicio?** Son límites diferentes. `@Valid` es utilizado por Spring MVC para dar mensajes de entrada al usuario. La comprobación explícita del servicio impide que otra clase Java registre valores básicos inválidos si llama al servicio directamente sin pasar por el controlador. No hace falta duplicar aquí todas las reglas imaginables; mantenemos comprobaciones pequeñas y consistentes con el ejemplo.

En una aplicación más extensa puedes utilizar objetos de entrada específicos de los casos de uso o validadores reutilizables para reducir duplicación. No es necesario introducir esa abstracción todavía.

**¿Qué hace `strip()`?** Es un método de `String` que elimina espacios en blanco al principio y al final. Así, el texto `"  Luna  "` se guarda como `"Luna"`. No elimina los espacios que estén entre palabras.

**¿Qué devuelve `create(...)`?** La instancia `Pet` que se acaba de construir y almacenar, incluida su identidad asignada. Eso permitirá redirigir a `/pets/{id}` sin adivinar cuál fue el nuevo id.

**¿Por qué `update(...)` devuelve `Optional<Pet>` y `delete(...)` un `boolean`?** En ambos casos la mascota podría no existir. En edición podemos querer obtener el objeto actualizado; en eliminación solo necesitamos saber si se eliminó una entrada. Son contratos Java explícitos, en lugar de ocultar un fallo.

### 4. Implementar `POST /pets`

Ahora el controlador puede recibir `PetForm`, comprobar errores y llamar al servicio.

Durante esta práctica, conserva los métodos `list`, `detail` y `createForm` existentes y **agrega** este método a `PetController`:

```java
@PostMapping
public String create(
    @Valid @ModelAttribute("petForm") PetForm petForm,
    BindingResult bindingResult,
    Model model,
    RedirectAttributes redirectAttributes
) {
    if (bindingResult.hasErrors()) {
        model.addAttribute("editing", false);
        return "pets/form";
    }

    Pet pet = petService.create(petForm);
    redirectAttributes.addFlashAttribute(
        "successMessage",
        "Mascota registrada correctamente."
    );
    return "redirect:/pets/" + pet.getId();
}
```

Agrega los imports necesarios al controlador:

```java
import jakarta.validation.Valid;

import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import com.example.petmatchlab.model.Pet;
```

Si alguno de esos tipos ya está importado, no repitas el import.

**Lee el recorrido completo:**

1. El navegador envía `POST /pets` con los valores de los campos.
2. Spring MVC construye un `PetForm`, enlaza los valores y aplica `@Valid`.
3. Si `BindingResult` contiene errores, el controlador devuelve `pets/form` sin guardar datos.
4. Si no hay errores, `PetService.create(...)` normaliza y comprueba los valores.
5. `InMemoryPetRepository.create(...)` asigna un id, construye `Pet` y lo guarda en el `Map`.
6. El servicio devuelve el `Pet` creado y el controlador conoce su id.
7. `RedirectAttributes` conserva un mensaje de éxito para la siguiente petición.
8. La respuesta redirige hacia la página de detalle, por ejemplo `/pets/4`.

**Importante:** el controlador no utiliza `new InMemoryPetRepository()`, no llama directamente a `pets.put(...)` y no genera HTML por concatenación. Cada responsabilidad permanece en su componente.

### 5. Comprender Post/Redirect/Get

Observa la diferencia entre devolver una vista después de registrar y redirigir a otra ruta.

**Sin redirección:**

```text
POST /pets
    └── guarda mascota
    └── devuelve HTML inmediatamente

Actualizar página
    └── el navegador puede intentar repetir el POST
```

**Con Post/Redirect/Get (PRG):**

```text
POST /pets
    └── guarda mascota
    └── responde con una redirección
           └── GET /pets/4
                 └── devuelve HTML del detalle

Actualizar página
    └── repite GET, no el POST anterior
```

PRG ayuda a evitar envíos accidentales repetidos al actualizar la página. **No garantiza por sí solo que nunca pueda existir un registro duplicado**: por ejemplo, alguien podría enviar dos peticiones POST independientes. Ese problema requeriría reglas adicionales si el dominio exigiera unicidad.

### 6. ¿Qué es un atributo flash?

Esta línea:

```java
redirectAttributes.addFlashAttribute(
    "successMessage", "Mascota registrada correctamente."
);
```

prepara un atributo que estará disponible temporalmente para la petición que llega después de la redirección. Spring MVC lo utiliza para mensajes de confirmación sin tener que agregarlos como parámetros a la URL.

Para mostrarlo, añade dentro de `<main>` en `templates/pets/detail.html`, antes del contenido de la mascota:

```html
<p class="success-message" role="status"
   th:if="${successMessage}"
   th:text="${successMessage}"></p>
```

La palabra `flash` no significa que el mensaje deba permanecer para siempre. Está pensado para la transición entre peticiones, normalmente después de redirigir. En la práctica siguiente incorporaremos también ese mismo mensaje en la página del catálogo.

### 7. Primera ejecución de un registro completo

Inicia la aplicación y abre:

```text
http://localhost:8080/pets/new
```

Introduce:

```text
Nombre: Kira
Especie: Perro
Edad: 4
Descripción: Le gusta jugar en el parque.
```

Presiona **Registrar mascota**. Si conservas las tres mascotas iniciales, el nuevo objeto recibirá el identificador `4`, y el navegador debería mostrar su detalle en `/pets/4`.

Comprueba también:

- `/pets` muestra ahora **cuatro** mascotas.
- Puedes abrir el detalle de Kira desde el listado.
- Actualizar `/pets/4` no crea otra mascota.
- Si envías nombre vacío o edad negativa, el formulario vuelve a mostrarse con errores y el catálogo no aumenta.
- Si reinicias la aplicación, Kira desaparece porque todavía no existe persistencia permanente.

> Puedes revisar la petición en las herramientas de desarrollo del navegador. Después del envío válido normalmente observarás un `POST /pets`, una respuesta de redirección y una nueva petición `GET /pets/4`. El código exacto de redirección depende de la infraestructura HTTP, pero la secuencia conceptual es la importante.

### 8. Probar la validación sin depender del navegador

Para observar qué ocurre si una petición no respeta las restricciones HTML, utiliza una herramienta HTTP como `curl`, si está disponible. Por ejemplo:

```bash
curl -i -X POST http://localhost:8080/pets \
  --data-urlencode "name=" \
  --data-urlencode "species=Perro" \
  --data-urlencode "age=-1" \
  --data-urlencode "description=Prueba"
```

Esta petición no debería crear una mascota. El servidor debe devolver la vista del formulario con errores. Comprueba después en `/pets` que el catálogo no cambió.

Para provocar un error de conversión:

```bash
curl -i -X POST http://localhost:8080/pets \
  --data-urlencode "name=Prueba" \
  --data-urlencode "species=Gato" \
  --data-urlencode "age=abc"
```

Los comandos muestran un ejemplo para terminales compatibles con continuación de línea mediante `\`. Si utilizas PowerShell y ese formato no funciona, ejecuta los argumentos en una única línea con `curl.exe`.

### Ejercicio — Registra y comprueba

Registra dos mascotas nuevas desde el navegador. Asegúrate de que cada una tenga un identificador diferente. Después, modifica una descripción para incluir espacios al principio y al final y revisa qué texto terminó guardado.

### Laboratorio de errores

1. Cambia temporalmente `nextId = 4L` por `nextId = 2L`. ¿Qué ocurre cuando `pets.put(...)` utiliza una clave que ya existe? Explica por qué el id debe ser único y generado correctamente.
2. Omite `bindingResult.hasErrors()` en el `POST`. ¿Qué riesgo aparece al enviar campos vacíos o no convertibles?
3. Cambia `return "redirect:/pets/" + pet.getId()` por `return "pets/detail"` sin añadir al `Model` la mascota recién creada. ¿Qué información necesitaría la plantilla para funcionar?
4. Intenta añadir directamente un elemento a la lista devuelta por `findAll()`. ¿Por qué esa lista no es modificable?

Corrige cada cambio y conserva un funcionamiento correcto antes de continuar.

### Comprobación

Debes poder explicar de dónde sale el nuevo identificador, qué modifica realmente el `Map`, por qué el servicio recibe los datos antes de almacenar y qué diferencia existe entre devolver una vista y redirigir después de un `POST`.

---
## Práctica 8.5 — Editar, eliminar y completar el CRUD

### Objetivos

Al finalizar podrás:

- explicar qué significa CRUD y relacionarlo con operaciones reales;
- reutilizar una plantilla para crear y editar;
- cargar los datos actuales antes de presentar un formulario de edición;
- modificar un registro manteniendo su identificador;
- eliminar un registro mediante una petición `POST`;
- distinguir errores de entrada de recursos que no existen;
- mostrar confirmaciones y mantener una navegación clara.

### 1. ¿Qué significa CRUD?

**CRUD** resume cuatro operaciones habituales sobre información:

| Letra | Operación | Acción en PetMatch Mini |
|---|---|---|
| C | Create | Registrar una mascota |
| R | Read | Listar mascotas o consultar una por id |
| U | Update | Modificar datos de una mascota |
| D | Delete | Eliminar una mascota |

No es una librería ni una anotación de Spring. Es una forma sencilla de describir funcionalidades. Las operaciones se implementan con métodos Java, rutas HTTP, vistas y la colección utilizada como repositorio.

Ya tienes **C** y **R**. Ahora añadiremos **U** y **D** sin cambiar el modelo `Pet` ni introducir una segunda fuente de datos.

### 2. Mostrar el formulario de edición

La edición comienza cuando una persona visita una dirección como:

```text
GET /pets/2/edit
```

El controlador debe buscar la mascota con id `2` y utilizar sus datos para preparar un `PetForm`:

```text
Pet guardado
    └── id = 2
    └── name = "Milo"
    └── species = "Gato"
    └── age = 2
    └── description = "Prefiere lugares tranquilos."
          │
          ▼
PetForm para editar
    └── name = "Milo"
    └── species = "Gato"
    └── age = 2
    └── description = "Prefiere lugares tranquilos."
```

El `PetForm` no necesita un campo `id`: ese valor sigue identificado por la ruta y no debe confiarse en un valor oculto o editable enviado desde el formulario.

En el futuro método `editForm` usarás:

```java
Pet pet = findPet(id);

PetForm form = new PetForm();
form.setName(pet.getName());
form.setSpecies(pet.getSpecies());
form.setAge(pet.getAge());
form.setDescription(pet.getDescription());
```

Esto es un **mapeo** entre dos objetos Java. Es explícito y fácil de depurar: cada propiedad del objeto guardado se copia al objeto utilizado por el formulario.

### 3. Reutilizar la vista

La plantilla `pets/form.html` creada en la práctica 8.3 ya distingue las dos situaciones mediante `editing`:

```html
<h1 th:text="${editing} ? 'Editar mascota' : 'Registrar mascota'">
    Formulario de mascota
</h1>
```

También determina a dónde se envían los datos:

```html
<form th:action="${editing} ? @{/pets/{id}(id=${petId})} : @{/pets}"
      th:object="${petForm}" method="post" novalidate>
```

Si `editing` es `false`, la acción será `POST /pets` y se creará una mascota.

Si `editing` es `true` y `petId` vale `2`, la acción será `POST /pets/2` y se intentará actualizar la mascota existente.

**¿Por qué ambos envíos son POST?** Los formularios HTML ordinarios admiten `GET` y `POST` como métodos de envío. Aunque HTTP también define otros métodos con semánticas de actualización y eliminación, aquí empleamos rutas diferentes con `POST` para mantener el trabajo centrado en Spring MVC y Thymeleaf. Lo importante es que ninguna modificación se realice mediante `GET`.

### 4. Implementar el controlador completo

Reemplaza **todo** `src/main/java/com/example/petmatchlab/controller/PetController.java` por esta versión. Incluye lo que ya construiste y agrega edición y eliminación; así podrás comparar el archivo final completo con el tuyo.

```java
package com.example.petmatchlab.controller;

import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import com.example.petmatchlab.dto.PetForm;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.service.PetService;

import jakarta.validation.Valid;

@Controller
@RequestMapping("/pets")
public class PetController {

    private final PetService petService;

    public PetController(PetService petService) {
        this.petService = petService;
    }

    @GetMapping
    public String list(Model model) {
        model.addAttribute("pets", petService.findAll());
        return "pets/list";
    }

    @GetMapping("/new")
    public String createForm(Model model) {
        model.addAttribute("petForm", new PetForm());
        model.addAttribute("editing", false);
        return "pets/form";
    }

    @PostMapping
    public String create(
        @Valid @ModelAttribute("petForm") PetForm petForm,
        BindingResult bindingResult,
        Model model,
        RedirectAttributes redirectAttributes
    ) {
        if (bindingResult.hasErrors()) {
            model.addAttribute("editing", false);
            return "pets/form";
        }

        Pet pet = petService.create(petForm);
        redirectAttributes.addFlashAttribute(
            "successMessage", "Mascota registrada correctamente."
        );
        return "redirect:/pets/" + pet.getId();
    }

    @GetMapping("/{id}")
    public String detail(@PathVariable("id") long id, Model model) {
        model.addAttribute("pet", findPet(id));
        return "pets/detail";
    }

    @GetMapping("/{id}/edit")
    public String editForm(@PathVariable("id") long id, Model model) {
        Pet pet = findPet(id);

        PetForm form = new PetForm();
        form.setName(pet.getName());
        form.setSpecies(pet.getSpecies());
        form.setAge(pet.getAge());
        form.setDescription(pet.getDescription());

        model.addAttribute("petForm", form);
        model.addAttribute("editing", true);
        model.addAttribute("petId", id);
        return "pets/form";
    }

    @PostMapping("/{id}")
    public String update(
        @PathVariable("id") long id,
        @Valid @ModelAttribute("petForm") PetForm petForm,
        BindingResult bindingResult,
        Model model,
        RedirectAttributes redirectAttributes
    ) {
        findPet(id);

        if (bindingResult.hasErrors()) {
            model.addAttribute("editing", true);
            model.addAttribute("petId", id);
            return "pets/form";
        }

        Pet updated = petService.update(id, petForm)
            .orElseThrow(() -> new ResponseStatusException(
                HttpStatus.NOT_FOUND, "Pet not found"
            ));

        redirectAttributes.addFlashAttribute(
            "successMessage", "Mascota actualizada correctamente."
        );
        return "redirect:/pets/" + updated.getId();
    }

    @PostMapping("/{id}/delete")
    public String delete(
        @PathVariable("id") long id,
        RedirectAttributes redirectAttributes
    ) {
        if (!petService.delete(id)) {
            throw new ResponseStatusException(
                HttpStatus.NOT_FOUND, "Pet not found"
            );
        }

        redirectAttributes.addFlashAttribute(
            "successMessage", "Mascota eliminada correctamente."
        );
        return "redirect:/pets";
    }

    private Pet findPet(long id) {
        return petService.findById(id)
            .orElseThrow(() -> new ResponseStatusException(
                HttpStatus.NOT_FOUND, "Pet not found"
            ));
    }
}
```

**No cambies las rutas sin actualizar también sus enlaces y formularios.** Las rutas declaradas en este controlador serán:

| Método HTTP | Ruta | Método Java | Resultado |
|---|---|---|---|
| GET | `/pets` | `list` | Vista con el catálogo |
| GET | `/pets/new` | `createForm` | Vista de registro |
| POST | `/pets` | `create` | Crea o vuelve al formulario con errores |
| GET | `/pets/{id}` | `detail` | Vista de detalle o error 404 |
| GET | `/pets/{id}/edit` | `editForm` | Formulario con datos actuales o error 404 |
| POST | `/pets/{id}` | `update` | Actualiza o vuelve al formulario con errores |
| POST | `/pets/{id}/delete` | `delete` | Elimina o responde 404 |

### 5. Seguir un cambio de principio a fin

Supón que quieres modificar la edad de Milo de `2` a `3`.

**Primer paso — Mostrar:**

```text
GET /pets/2/edit
    → findPet(2)
    → PetForm con datos actuales
    → templates/pets/form.html
```

**Segundo paso — Enviar:**

```text
POST /pets/2
    → @ModelAttribute("petForm")
    → @Valid + BindingResult
    → PetService.update(2, form)
    → InMemoryPetRepository.update(...)
    → pets.put(2L, updated)
    → redirect:/pets/2
```

**Tercer paso — Consultar:**

```text
GET /pets/2
    → el repositorio devuelve el objeto actualizado
    → Thymeleaf muestra edad 3
```

El identificador continúa siendo `2`. Esto es una **actualización del registro** y no la creación de una mascota nueva.

### 6. ¿Por qué volver a comprobar la existencia en `POST`?

En `update`, el controlador llama primero a:

```java
findPet(id);
```

Con ello, una URL de edición que señala un registro inexistente devuelve 404, incluso si además el formulario contiene campos inválidos. **No tiene sentido presentar errores de edición para una mascota que no existe.**

Después, el servicio vuelve a consultar el repositorio al actualizar. Esto protege el caso en que el registro deje de existir entre la primera comprobación y la escritura. La pequeña duplicación de consulta hace visible el flujo para el nivel de este ejercicio; no reemplaza controles de concurrencia más sofisticados cuando estos sean necesarios.

El repositorio devuelve `Optional.empty()` si el identificador no existe. El controlador lo traduce a `ResponseStatusException(HttpStatus.NOT_FOUND, ...)` y Spring MVC genera una respuesta con estado **404 Not Found**.

**Distingue estos problemas:**

| Situación | Resultado |
|---|---|
| Una mascota no existe | Respuesta HTTP 404 |
| La edad enviada es negativa | Se vuelve a mostrar el formulario con un mensaje de validación |
| Una mascota existe y los datos son válidos | Se actualiza y redirige a su detalle |
| Una persona intenta consultar una ruta sin método `GET` habilitado | La aplicación no debe ejecutar la operación `POST` por tratarse de una solicitud `GET` |

### 7. Modificar el catálogo para acceder a las operaciones

Sustituye el archivo `src/main/resources/templates/pets/list.html` por esta versión **completa**:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Catálogo | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <header class="page-header">
            <div>
                <h1>Catálogo de mascotas</h1>
                <p class="page-intro">Consulta y administra las mascotas del catálogo.</p>
            </div>
            <a class="button-link" th:href="@{/pets/new}">
                Registrar mascota
            </a>
        </header>

        <p class="success-message" role="status"
           th:if="${successMessage}"
           th:text="${successMessage}"></p>

        <section class="content-panel" th:if="${pets.isEmpty()}">
            <h2>El catálogo está vacío</h2>
            <p>Registra una mascota para comenzar.</p>
            <a class="button-link" th:href="@{/pets/new}">Registrar mascota</a>
        </section>

        <section class="pet-grid" aria-label="Lista de mascotas"
                 th:unless="${pets.isEmpty()}">
            <article class="pet-card" th:each="pet : ${pets}">
                <h2 th:text="${pet.name}">Nombre</h2>
                <p>Especie: <span th:text="${pet.species}">Especie</span></p>
                <p>Edad: <span th:text="${pet.age}">0</span> años</p>

                <div class="form-actions">
                    <a class="button-link"
                       th:href="@{/pets/{id}(id=${pet.id})}">Ver detalle</a>
                    <a class="secondary-link"
                       th:href="@{/pets/{id}/edit(id=${pet.id})}">Editar</a>
                </div>

                <form method="post" class="delete-form"
                      th:action="@{/pets/{id}/delete(id=${pet.id})}">
                    <button type="submit" class="danger-button">
                        Eliminar
                        <span class="visually-hidden"
                              th:text="' ' + ${pet.name}"></span>
                    </button>
                </form>
            </article>
        </section>
    </main>
</body>
</html>
```

Observa que el botón de eliminación está dentro de un `<form>`:

```html
<form method="post" th:action="@{/pets/{id}/delete(id=${pet.id})}">
    <button type="submit">Eliminar</button>
</form>
```

No es un enlace `GET`. Al presionar el botón se envía `POST /pets/{id}/delete` y el servidor realiza la operación.

La clase `visually-hidden` se utilizará para añadir el nombre de la mascota al texto accesible del botón sin repetirlo visualmente. Así, una tecnología de asistencia podrá distinguir “Eliminar Luna” de “Eliminar Milo”, aunque los botones visibles tengan el mismo texto. Definiremos esa clase CSS más adelante.

### 8. Actualizar la página de detalle

Sustituye `src/main/resources/templates/pets/detail.html` por esta versión **completa**:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Detalle | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <p class="success-message" role="status"
           th:if="${successMessage}"
           th:text="${successMessage}"></p>

        <article class="content-panel">
            <h1 th:text="${pet.name}">Nombre</h1>
            <p><strong>Identificador:</strong>
                <span th:text="${pet.id}">0</span></p>
            <p><strong>Especie:</strong>
                <span th:text="${pet.species}">Especie</span></p>
            <p><strong>Edad:</strong>
                <span th:text="${pet.age}">0</span> años</p>
            <p class="muted"
               th:text="${pet.description != null ? pet.description : 'Sin descripción adicional.'}">
                Descripción
            </p>

            <div class="form-actions">
                <a class="button-link"
                   th:href="@{/pets/{id}/edit(id=${pet.id})}">Editar</a>
                <a class="secondary-link" th:href="@{/pets}">Volver al catálogo</a>
            </div>

            <form class="delete-form" method="post"
                  th:action="@{/pets/{id}/delete(id=${pet.id})}">
                <button type="submit" class="danger-button">
                    Eliminar mascota
                </button>
            </form>
        </article>
    </main>
</body>
</html>
```

No se ha duplicado una página para cada mascota. La misma plantilla sigue mostrando los datos del objeto encontrado por el servicio.

**Importante:** en el HTML de ambas vistas los nombres, especies y descripciones se representan mediante `th:text`. Esta directiva escapa caracteres especiales del contenido para que una descripción introducida por una persona no se interprete directamente como HTML. Eso es apropiado para mostrar texto normal y evita una clase frecuente de errores de representación.

### 9. Completar el CSS de los formularios

Conserva el CSS existente en `src/main/resources/static/css/application.css` y **añade al final** estas reglas:

```css
.page-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    flex-wrap: wrap;
    margin-bottom: 1.5rem;
}

.form-field {
    display: flex;
    flex-direction: column;
    gap: 0.4rem;
    margin-block: 1rem;
}

.form-field label {
    font-weight: 700;
}

.form-field input,
.form-field textarea {
    width: 100%;
    max-width: 36rem;
    padding: 0.7rem 0.85rem;
    border: 1px solid #64748b;
    border-radius: 0.6rem;
    font: inherit;
    color: #0f172a;
    background: #fff;
}

.form-field input:focus-visible,
.form-field textarea:focus-visible {
    outline: 3px solid #0ea5e9;
    outline-offset: 2px;
}

.form-field .field-error {
    border: 2px solid #b91c1c;
}

.field-error-message,
.form-summary {
    color: #b91c1c;
    font-weight: 600;
}

.form-summary {
    border-left: 4px solid #b91c1c;
    padding: 0.75rem;
    background: #fee2e2;
}

.form-actions {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 0.8rem;
    margin-top: 1.1rem;
}

.secondary-link {
    display: inline-flex;
    padding: 0.6rem 0.8rem;
    text-decoration: underline;
    color: inherit;
}

.danger-button {
    cursor: pointer;
    padding: 0.65rem 0.95rem;
    border: 2px solid #991b1b;
    border-radius: 0.6rem;
    color: #fff;
    background: #991b1b;
    font: inherit;
    font-weight: 700;
}

.danger-button:hover {
    background: #7f1d1d;
}

.danger-button:focus-visible,
.secondary-link:focus-visible {
    outline: 3px solid #0ea5e9;
    outline-offset: 3px;
}

.delete-form {
    margin-top: 0.9rem;
}

.success-message {
    padding: 0.8rem 1rem;
    border-left: 4px solid #15803d;
    background: #dcfce7;
    color: #14532d;
    font-weight: 600;
    margin-bottom: 1rem;
}

.visually-hidden {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
}
```

Estas reglas no cambian la lógica Java ni sustituyen las validaciones. Proporcionan una presentación más clara para los campos, los errores, los mensajes y los botones destructivos.

Comprueba que los colores de los mensajes sean legibles, que los controles puedan utilizarse con el teclado y que la página siga siendo usable al reducir el ancho del navegador. Si el CSS anterior del proyecto ya incluye reglas con estos nombres, revisa posibles conflictos antes de duplicarlas.

### 10. ¿Qué sucede al eliminar?

Al presionar **Eliminar** sobre Kira, el navegador enviará una petición parecida a:

```text
POST /pets/4/delete
```

El controlador llama:

```java
petService.delete(4L)
```

El servicio delega en:

```java
petRepository.delete(4L)
```

El repositorio elimina la entrada del `Map`. Si la operación fue posible, el controlador añade un mensaje de confirmación y responde con una redirección hacia `/pets`, donde el catálogo ya no debería incluir a Kira.

Si intentas abrir el detalle de la mascota eliminada mediante `/pets/4`, el servidor debe responder con **404**.

**¿Por qué la eliminación no lleva al detalle?** Porque el objeto ya no existe. La redirección a la lista proporciona un lugar válido desde el cual continuar navegando.

### 11. Comprobar errores de identificador

Prueba estas rutas después de ejecutar la aplicación:

| Petición | Qué comprueba |
|---|---|
| `GET /pets/1` | Detalle válido |
| `GET /pets/999999` | La mascota no existe: 404 |
| `GET /pets/999999/edit` | No hay datos para editar: 404 |
| `POST /pets/999999` con datos válidos | No puede actualizar una mascota inexistente: 404 |
| `POST /pets/999999/delete` | No puede eliminar una mascota inexistente: 404 |
| `GET /pets/1/delete` | No es una operación de borrado y no debe eliminar nada |

`GET /pets/abc` es un caso diferente: `abc` no puede convertirse al `long` declarado por `@PathVariable`. La aplicación puede devolver un **400 Bad Request** por el tipo de dato incorrecto, en lugar de tratarlo como el id numérico de una mascota inexistente.

### Ejercicio — Antes y después de editar

1. Abre `/pets/2` y anota nombre, especie, edad y descripción.
2. Ingresa al formulario de edición desde el enlace.
3. Cambia únicamente la edad.
4. Envía el formulario y comprueba que la URL de detalle conserva el mismo id.
5. Vuelve a abrir el formulario y comprueba que muestra la edad actualizada.
6. Intenta enviar una edad negativa y verifica que no se pierde el valor válido de los demás campos.

### Reto — Eliminar y comprobar

Crea una mascota de prueba, abre su detalle y elimínala. Comprueba que desaparece del catálogo y que su ruta anterior devuelve 404. Luego repite la eliminación mediante una petición manual hacia el mismo identificador. ¿Por qué el servicio devuelve `false` la segunda vez?

### Depuración — El problema está en el límite correcto

1. Cambia temporalmente el nombre del atributo de la vista de edición de `"petForm"` a `"pet"`, sin modificar `th:object`. ¿Qué deja de coincidir?
2. Elimina `model.addAttribute("editing", true)` del camino que vuelve al formulario por errores. ¿Qué ruta intentaría utilizar la plantilla al enviarse de nuevo?
3. En `InMemoryPetRepository.update(...)`, reemplaza `pets.put(id, updated)` por `pets.put(nextId++, updated)`. ¿Por qué la operación ya no representa una actualización del registro identificado por `id`?
4. Cambia por un instante el formulario de eliminación a `method="get"`. ¿Por qué el controlador no debería atender ese envío como borrado?

Restaura los archivos después de cada prueba y no dejes cambios intencionalmente incorrectos en el proyecto.

### Comprobación

Debes poder explicar cada letra de CRUD, justificar por qué el id se conserva durante una edición, diferenciar una entrada inválida de un registro inexistente y seguir el recorrido del botón **Eliminar** hasta el método Java que modifica el `Map`.

---
## Práctica 8.6 — Pruebas, depuración y reto integrador

### Objetivos

Al finalizar podrás:

- comprobar la lógica del repositorio y del servicio sin abrir el navegador;
- probar peticiones MVC con `MockMvc`;
- verificar registros válidos, errores de campos y recursos inexistentes;
- diferenciar una prueba unitaria de una prueba que integra varios componentes;
- utilizar resultados de pruebas para detectar regresiones;
- realizar una ampliación pequeña del sistema sin mezclar responsabilidades.

### 1. ¿Por qué no basta con probar manualmente?

Puedes registrar una mascota desde el navegador y comprobar que aparece. Esa verificación es necesaria, pero no demuestra por sí sola que todos los casos funcionen. También conviene comprobar:

- que dos mascotas nuevas reciben identificadores diferentes;
- que la edición conserva el identificador anterior;
- que una búsqueda inexistente no devuelve un objeto inventado;
- que un formulario con edad negativa no registra nada;
- que una eliminación elimina solo el registro indicado;
- que una modificación posterior del código no rompe funcionalidades que ya funcionaban.

Una **prueba automatizada** ejecuta una operación, observa el resultado y lo compara con una expectativa. Cuando la expectativa no se cumple, la prueba falla y ofrece una señal concreta para investigar.

### 2. Probar directamente el repositorio

Para las operaciones sobre colecciones no necesitamos que Spring Boot inicie un servidor. Podemos construir el repositorio con `new`, igual que cualquier otra clase Java.

Crea `src/test/java/com/example/petmatchlab/repository/InMemoryPetRepositoryTests.java`:

```java
package com.example.petmatchlab.repository;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

class InMemoryPetRepositoryTests {

    private InMemoryPetRepository repository;

    @BeforeEach
    void setUp() {
        repository = new InMemoryPetRepository();
    }

    @Test
    void shouldCreatePetWithNewIdentifier() {
        var created = repository.create("Kira", "Perro", 4, "Juguetona");

        assertEquals(4L, created.getId());
        assertEquals("Kira", repository.findById(4L).orElseThrow().getName());
        assertEquals(4, repository.findAll().size());
    }

    @Test
    void shouldAssignDifferentIdentifiersToTwoNewPets() {
        var first = repository.create("Kira", "Perro", 4, null);
        var second = repository.create("Sol", "Gato", 1, null);

        assertTrue(first.getId() != second.getId());
    }

    @Test
    void shouldUpdateExistingPetWithoutChangingIdentifier() {
        var updated = repository.update(2L, "Milo", "Gato", 3, null);

        assertTrue(updated.isPresent());
        assertEquals(2L, updated.orElseThrow().getId());
        assertEquals(3, repository.findById(2L).orElseThrow().getAge());
    }

    @Test
    void shouldReturnEmptyWhenUpdatingMissingPet() {
        var updated = repository.update(999L, "X", "Gato", 1, null);

        assertTrue(updated.isEmpty());
        assertEquals(3, repository.findAll().size());
    }

    @Test
    void shouldDeleteOnlyRequestedPet() {
        assertTrue(repository.delete(2L));

        assertTrue(repository.findById(2L).isEmpty());
        assertTrue(repository.findById(1L).isPresent());
        assertEquals(2, repository.findAll().size());
    }

    @Test
    void shouldReportMissingPetOnDeletion() {
        assertFalse(repository.delete(999L));
    }
}
```

**Explica el patrón:**

1. `@BeforeEach` prepara un repositorio nuevo antes de cada prueba.
2. Cada método `@Test` verifica una situación concreta.
3. `assertEquals(...)` compara un valor esperado con el obtenido.
4. `assertTrue(...)` y `assertFalse(...)` comprueban condiciones.
5. `orElseThrow()` obtiene el objeto cuando la prueba espera que exista; si no existe, la prueba falla y deja visible el problema.

**¿Por qué crear un repositorio nuevo para cada prueba?** Porque el repositorio almacena datos modificables en memoria. Si las pruebas compartieran la misma instancia, una eliminación ejecutada por un método podría alterar lo que espera otro. Cada caso debe tener un punto de partida claro.

Si cambiaste las mascotas iniciales o el valor de `nextId`, adapta los valores esperados manteniendo las mismas intenciones de las pruebas.

### 3. Probar el servicio y su normalización

El servicio utiliza un repositorio recibido por constructor. Gracias a esa dependencia explícita, puedes construir ambos sin iniciar Spring.

Si ya existe `src/test/java/com/example/petmatchlab/service/PetServiceTests.java` del laboratorio anterior, reemplázalo por esta versión ampliada:

```java
package com.example.petmatchlab.service;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import com.example.petmatchlab.dto.PetForm;
import com.example.petmatchlab.repository.InMemoryPetRepository;

class PetServiceTests {

    private PetService service;

    @BeforeEach
    void setUp() {
        service = new PetService(new InMemoryPetRepository());
    }

    @Test
    void shouldReturnAvailablePets() {
        assertEquals(3, service.findAll().size());
    }

    @Test
    void shouldReturnEmptyWhenPetDoesNotExist() {
        assertTrue(service.findById(999L).isEmpty());
    }

    @Test
    void shouldNormalizeNameAndSpeciesWhenCreatingPet() {
        PetForm form = newForm("  Kira  ", "  Perro  ", 4);
        form.setDescription("  Juguetona  ");

        var created = service.create(form);

        assertEquals("Kira", created.getName());
        assertEquals("Perro", created.getSpecies());
        assertEquals("Juguetona", created.getDescription());
    }

    @Test
    void shouldRejectNegativeAgeWhenServiceIsCalledDirectly() {
        PetForm form = newForm("Kira", "Perro", -1);

        assertThrows(IllegalArgumentException.class, () -> service.create(form));
        assertEquals(3, service.findAll().size());
    }

    @Test
    void shouldRejectBlankNameWhenServiceIsCalledDirectly() {
        PetForm form = newForm("   ", "Perro", 2);

        assertThrows(IllegalArgumentException.class, () -> service.create(form));
    }

    @Test
    void shouldUpdateExistingPet() {
        PetForm form = newForm("Milo", "Gato", 5);

        var updated = service.update(2L, form);

        assertTrue(updated.isPresent());
        assertEquals(5, service.findById(2L).orElseThrow().getAge());
    }

    @Test
    void shouldNotCreatePetWhenUpdateTargetDoesNotExist() {
        PetForm form = newForm("Desconocido", "Gato", 5);

        assertTrue(service.update(999L, form).isEmpty());
        assertEquals(3, service.findAll().size());
    }

    private PetForm newForm(String name, String species, Integer age) {
        PetForm form = new PetForm();
        form.setName(name);
        form.setSpecies(species);
        form.setAge(age);
        return form;
    }
}
```

**¿Qué es `assertThrows(...)`?** Comprueba que una operación produce una excepción del tipo esperado. Aquí el servicio debe rechazar una edad negativa cuando otra clase Java lo llama directamente. No queremos que el método guarde la mascota y luego lance un error: también comprobamos que el tamaño de la colección no cambió.

El método `newForm(...)` es un **método auxiliar de prueba**, no una funcionalidad de la aplicación. Evita repetir la construcción de `PetForm` en cada caso. Los nombres y comentarios dentro del código siguen en inglés; los textos que verá el usuario pueden estar en español.

### 4. Probar el controlador sin abrir un navegador

Las pruebas anteriores verifican clases Java. Pero no comprueban si Spring MVC reconoce rutas, enlaza campos y aplica las anotaciones del formulario.

Para eso existe `MockMvc`: permite realizar peticiones simuladas a la capa MVC y comprobar sus respuestas, sin abrir manualmente una ventana del navegador ni iniciar un servidor HTTP real. En esta práctica utilizaremos `@SpringBootTest` para cargar el contexto de la aplicación y `@AutoConfigureMockMvc` para preparar la herramienta de pruebas.

Crea `src/test/java/com/example/petmatchlab/controller/PetControllerIntegrationTests.java`:

```java
package com.example.petmatchlab.controller;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.model;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.redirectedUrlPattern;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.view;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest
@AutoConfigureMockMvc
class PetControllerIntegrationTests {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldRenderCreateForm() throws Exception {
        mockMvc.perform(get("/pets/new"))
            .andExpect(status().isOk())
            .andExpect(view().name("pets/form"))
            .andExpect(model().attributeExists("petForm"));
    }

    @Test
    void shouldRejectInvalidPetFormWithoutRedirect() throws Exception {
        mockMvc.perform(post("/pets")
                .param("name", "")
                .param("species", "   ")
                .param("age", "-1")
                .param("description", "Prueba"))
            .andExpect(status().isOk())
            .andExpect(view().name("pets/form"))
            .andExpect(model().attributeHasFieldErrors(
                "petForm", "name", "species", "age"));
    }

    @Test
    void shouldRedirectAfterValidCreation() throws Exception {
        mockMvc.perform(post("/pets")
                .param("name", "Kira")
                .param("species", "Perro")
                .param("age", "4")
                .param("description", "Juguetona"))
            .andExpect(status().is3xxRedirection())
            .andExpect(redirectedUrlPattern("/pets/*"));
    }

    @Test
    void shouldReturnNotFoundWhenEditingMissingPet() throws Exception {
        mockMvc.perform(get("/pets/999999/edit"))
            .andExpect(status().isNotFound());
    }

    @Test
    void shouldReturnNotFoundWhenDeletingMissingPet() throws Exception {
        mockMvc.perform(post("/pets/999999/delete"))
            .andExpect(status().isNotFound());
    }
}
```

**Recorre una de las pruebas:**

```java
mockMvc.perform(post("/pets")
        .param("name", "")
        .param("species", "   ")
        .param("age", "-1"))
    .andExpect(status().isOk())
    .andExpect(view().name("pets/form"));
```

- `post("/pets")` construye una petición HTTP simulada.
- `.param(...)` incluye un valor de formulario.
- `.perform(...)` ejecuta esa petición a través de la infraestructura MVC de prueba.
- `.andExpect(...)` establece una condición que debe cumplirse.
- `view().name("pets/form")` verifica que se devolvió el formulario para corregir datos.

El código **200 OK** en un envío inválido no significa que se guardó una mascota. Aquí indica que se devolvió correctamente una página HTML con errores. Es distinto del resultado **404** cuando se intenta editar un recurso inexistente.

**Observación sobre los datos de estas pruebas:** el contexto completo de Spring utiliza un repositorio en memoria y puede reutilizarse entre métodos de prueba. Por eso las pruebas MVC de arriba no suponen que el siguiente identificador sea siempre `4L`, ni necesitan que el catálogo tenga exactamente tres registros. Si una prueba necesita comprobar cantidades o identidades precisas tras varias escrituras, deberá preparar y aislar explícitamente sus datos para que el resultado no dependa del orden de ejecución.

### 5. Ejecutar la suite

Desde la raíz del proyecto ejecuta:

```bash
./mvnw test
```

O en PowerShell:

```powershell
.\mvnw.cmd test
```

Maven compilará el código principal y las pruebas y ejecutará los métodos marcados con `@Test`. Si alguno falla, identifica primero:

1. **Qué prueba falló.** Lee su nombre; debería describir una intención concreta.
2. **Qué esperaba.** Busca el `assert...` o `andExpect(...)` correspondiente.
3. **Qué obtuvo.** Inspecciona el mensaje de error, la excepción o la respuesta HTTP.
4. **En qué capa está el problema.** No cambies el HTML si falló una comparación de identificadores dentro del repositorio.
5. **Qué corrección mínima resuelve la causa.** Después vuelve a ejecutar toda la suite.

No elimines una prueba simplemente porque haya descubierto un error de implementación. Una prueba fallida es información valiosa si expresa correctamente el comportamiento que deseas.

### 6. Depuración — Sigue una petición real

Coloca puntos de interrupción en estos lugares:

- `PetController.create(...)`, en la condición `bindingResult.hasErrors()`;
- `PetService.create(...)`, antes de llamar al repositorio;
- `InMemoryPetRepository.create(...)`, en `long id = nextId++;`.

Envía una mascota válida desde el navegador y observa el **Call Stack** del depurador. Deberías encontrar el orden conceptual:

```text
Spring MVC
  → PetController.create
      → PetService.create
          → InMemoryPetRepository.create
```

Después repite el ejercicio con una edad negativa. Si el enlace y la validación funcionan correctamente, el controlador detectará errores y **no** debería llamar a `PetService.create(...)`.

Esto permite comprobar la separación de responsabilidades observando una ejecución, no solo leyendo un diagrama.

### 7. Reto integrador — Mejorar el catálogo sin romper las operaciones

Amplía PetMatch Mini con una funcionalidad de **búsqueda por especie** desde el catálogo, sin modificar el comportamiento del registro, la edición ni la eliminación.

**Requisitos:**

1. Agrega un campo de búsqueda por especie en `pets/list.html` utilizando un formulario con `method="get"`.
2. La búsqueda debe generar una URL como `/pets?species=Gato`.
3. Permite que `PetController.list(...)` reciba ese parámetro como opcional.
4. Implementa en `PetService` el filtrado de las mascotas sin escribir reglas dentro del HTML.
5. Conserva el listado completo cuando no se suministra una especie.
6. Haz que una búsqueda sin coincidencias muestre el estado vacío de la vista.
7. Añade al menos una prueba que compruebe la búsqueda por una especie conocida y otra que compruebe una búsqueda sin coincidencias.

**Antes de implementar, identifica la responsabilidad de cada pieza:**

| Necesidad | Capa apropiada |
|---|---|
| Campo y botón de búsqueda | Plantilla Thymeleaf |
| Recibir el parámetro `species` | Controller |
| Aplicar el criterio de filtrado | Service |
| Proporcionar las mascotas disponibles | Repository |

La búsqueda **no modifica** el catálogo, por lo que `GET` es apropiado para esta operación. No necesitas crear otro formulario DTO para un filtro simple de una sola propiedad.

**Pista de Java:** sobre una lista de mascotas puedes construir otra lista con las coincidencias. Es posible utilizar un ciclo `for` y `ArrayList`, conceptos que ya conoces; no es obligatorio emplear Streams para resolver el ejercicio.

### 8. Diagnóstico final — Comprueba que comprendes el flujo

**Caso A — Registro válido.** Una persona envía `name="Mora"`, `species="Gato"` y `age="2"`.

Explica, en orden, qué hace Spring MVC, qué se guarda en `PetForm`, qué revisa `BindingResult`, qué recibe el servicio, quién asigna el identificador y por qué el navegador termina mostrando un detalle.

**Caso B — Registro inválido.** Una persona envía `name=""`, `species="Gato"` y `age="-2"`.

¿A qué plantilla debe regresar el controlador? ¿Se ejecutará `PetService.create(...)`? ¿Cambiará la cantidad de mascotas almacenadas? ¿Por qué?

**Caso C — Edición.** Existe la mascota con id `2`, y se cambia únicamente su descripción.

¿Por qué conserva el mismo id? ¿Por qué el repositorio reemplaza el valor asociado a la clave `2L` en lugar de insertar uno con una clave nueva?

**Caso D — Eliminación repetida.** Se elimina correctamente la mascota `4` y después se envía otra vez la misma petición de eliminación.

¿Qué devuelve el repositorio en el segundo intento? ¿Cómo debe traducirse ese resultado a HTTP? ¿Por qué `GET /pets/4` tampoco debe funcionar después de eliminarla?

**Caso E — Error de conversión.** Una petición envía `age="abc"`.

¿Es exactamente lo mismo que `@Min(0)`? ¿En qué componente se detecta que el texto no puede convertirse a `Integer`?

### Comprobación

Al terminar deberías poder explicar sin consultar el código: formulario HTML, GET, POST, `@ModelAttribute`, Form DTO, `@Valid`, `BindingResult`, `th:object`, `th:field`, `th:errors`, redirección, flash attribute, CRUD y error 404. También debes poder identificar en qué clase se implementa cada operación y demostrar que funciona desde el navegador y mediante pruebas automatizadas.

---

## Mapa final del proyecto

El resultado del laboratorio conserva la estructura anterior y agrega solo los elementos necesarios para recibir datos y modificar registros:

```text
petmatch-lab/
├── pom.xml
├── mvnw
├── mvnw.cmd
├── src/
│   ├── main/
│   │   ├── java/com/example/petmatchlab/
│   │   │   ├── PetMatchLabApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── HomeController.java
│   │   │   │   └── PetController.java
│   │   │   ├── dto/
│   │   │   │   └── PetForm.java
│   │   │   ├── model/
│   │   │   │   └── Pet.java
│   │   │   ├── repository/
│   │   │   │   └── InMemoryPetRepository.java
│   │   │   └── service/
│   │   │       └── PetService.java
│   │   └── resources/
│   │       ├── application.yaml
│   │       ├── messages.properties             (opcional)
│   │       ├── static/css/application.css
│   │       └── templates/
│   │           ├── home.html
│   │           ├── about.html
│   │           ├── fragments/
│   │           │   ├── head.html
│   │           │   └── navigation.html
│   │           └── pets/
│   │               ├── list.html
│   │               ├── detail.html
│   │               └── form.html
│   └── test/
│       └── java/com/example/petmatchlab/
│           ├── controller/
│           │   └── PetControllerIntegrationTests.java
│           ├── repository/
│           │   └── InMemoryPetRepositoryTests.java
│           └── service/
│               └── PetServiceTests.java
```

**Recorrido de escritura:**

```text
Navegador
  │
  │ POST /pets
  ▼
PetController
  ├── @ModelAttribute → PetForm
  ├── @Valid → validación de entrada
  ├── BindingResult → ¿hay errores?
  │      ├── Sí → pets/form.html con valores y mensajes
  │      └── No
  ▼
PetService
  └── valida y normaliza datos
  ▼
InMemoryPetRepository
  └── crea, reemplaza o elimina objetos Pet en un Map
  ▼
PetController
  └── redirige o responde 404
  ▼
Thymeleaf
  └── genera HTML para el navegador
```

## Revisión del funcionamiento

Realiza una comprobación final con la aplicación iniciada y sin depender de que las pruebas de otro momento hayan dejado registros en memoria:

| Comprobación | Resultado esperado |
|---|---|
| Abrir `/pets` | Se muestra el catálogo inicial |
| Abrir `/pets/new` | Aparece un formulario vacío |
| Registrar con datos válidos | Se crea una mascota y se abre su detalle |
| Actualizar la página después de registrar | No se repite automáticamente el registro |
| Registrar sin nombre | Se muestra error y no se guarda |
| Registrar con edad negativa | Se muestra error y no se guarda |
| Editar mascota existente | Se cargan sus valores actuales |
| Guardar una edición válida | Los cambios aparecen en el detalle y el id se conserva |
| Guardar una edición inválida | Se conserva lo escrito y se muestran los errores |
| Eliminar una mascota | Desaparece del catálogo y se muestra confirmación |
| Abrir el detalle de una mascota eliminada | Respuesta 404 |
| Buscar un id inexistente | Respuesta 404 |
| Reiniciar la aplicación | Se restaura el contenido inicial del repositorio en memoria |
| Ejecutar `./mvnw test` | Las pruebas terminan correctamente |

## Cierre del laboratorio

Has pasado de una aplicación que solo **consulta** objetos Java a una aplicación que también **recibe, valida y modifica** información mediante formularios web. Ahora puedes seguir una petición completa desde los campos HTML hasta el método Java que cambia el estado del catálogo.

La idea principal no consiste en memorizar `@PostMapping` o `th:field`. Consiste en reconocer que **cada parte tiene una responsabilidad diferente**: el formulario recoge datos, Spring MVC los enlaza, la validación detecta problemas, el controlador coordina la respuesta HTTP, el servicio aplica reglas y el repositorio mantiene los objetos.

El proyecto sigue utilizando memoria como almacenamiento. Esa limitación es visible y deliberada: permite distinguir entre **modificar objetos durante la ejecución** y **persistir datos para recuperarlos después de reiniciar la aplicación**.

## Referencias oficiales

- [Spring Framework — `@ModelAttribute`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/modelattrib-method-args.html): enlace de datos, objetos de formulario y tratamiento de errores con `BindingResult`.
- [Spring Framework — Validation](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-validation.html): validación de parámetros en métodos MVC.
- [Spring Framework — Redirect Attributes](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/redirecting-passing-data.html): cómo construir redirecciones y controlar los atributos que se trasladan.
- [Spring Framework — Flash Attributes](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods/flash-attributes.html): mensajes temporales disponibles después de redirigir.
- [Spring Boot — Validation](https://docs.spring.io/spring-boot/reference/io/validation.html): integración con la implementación de Jakarta Bean Validation.
- [Spring Boot — Testing Spring Boot Applications](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html): pruebas con `@SpringBootTest`, `@WebMvcTest` y `MockMvc`.
- [Spring Framework — MockMvc](https://docs.spring.io/spring-framework/reference/testing/mockmvc.html): peticiones simuladas y verificación de respuestas MVC.
- [Thymeleaf 3.1 — Tutorial: Thymeleaf + Spring](https://www.thymeleaf.org/doc/tutorials/3.1/thymeleafspring.html): `th:object`, `th:field`, errores y formularios.
- [MDN — Enviar datos de formularios](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data): método, acción, datos enviados y respuesta del servidor.
- [MDN — Métodos de petición HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods): semántica de `GET`, `POST` y otros métodos.
- [Java SE 21 — `LinkedHashMap`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedHashMap.html): mapas con orden de iteración predecible.
- [Java SE 21 — `Optional`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html): representación explícita de un resultado que puede estar ausente.
