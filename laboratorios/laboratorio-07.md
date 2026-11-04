# Laboratorio 7 — De Java a la web: arquitectura en capas, Spring Boot, Spring MVC y Thymeleaf

Hasta ahora has escrito programas Java que se ejecutan desde una terminal. Sabes crear clases y objetos, utilizar métodos, trabajar con colecciones y manejar errores. Una aplicación web también utiliza esas habilidades, pero añade una forma diferente de interactuar: una persona abre una dirección en el navegador y recibe una página HTML generada por una aplicación que se ejecuta en un servidor.

En este laboratorio construirás **PetMatch Mini**, un catálogo web de mascotas. Empezarás organizando clases en paquetes, crearás una aplicación con Spring Boot, conectarás controladores con servicios mediante inyección de dependencias y, finalmente, mostrarás datos dinámicos en páginas creadas con Thymeleaf. Cada práctica deja una parte funcional que utilizarás en la siguiente.

No necesitas conocimientos previos de Spring, Maven ni motores de plantillas. Sí debes poder reconocer una clase, un constructor, un método, una lista y una excepción en Java. También conviene conocer la estructura básica de un documento HTML; aquí se explicarán los elementos utilizados.

| Práctica | Aprendizaje principal | Resultado comprobable |
|---|---|---|
| 7.1 | Paquetes, responsabilidades, MVC y arquitectura en capas | Identificar dónde pertenece cada parte de una aplicación |
| 7.2 | Spring Boot, Maven, dependencias y estructura del proyecto | Iniciar un servidor web desde un proyecto Java |
| 7.3 | Spring MVC, controladores, rutas, modelos y vistas | Abrir dos páginas mediante direcciones HTTP |
| 7.4 | Componentes Spring, servicios, repositorio en memoria e inyección de dependencias | Consultar datos sin mezclarlos con la presentación |
| 7.5 | Thymeleaf, expresiones, listas, condiciones y rutas con parámetros | Mostrar un catálogo y el detalle de cada mascota |
| 7.6 | Fragments, navegación, CSS, accesibilidad y comprobaciones | Completar una interfaz coherente y verificar sus casos principales |

> **Forma de trabajo:** en cada práctica, intenta explicar qué hará el código **antes de ejecutarlo**. Después ejecútalo, observa el resultado, modifica una parte y comprueba el efecto. Si aparece un error, identifica primero si ocurre al compilar, al iniciar Spring Boot o al abrir una ruta en el navegador.

## Preparación del entorno

Necesitas un **JDK 21**, un editor con soporte Java (por ejemplo, Visual Studio Code), un navegador y acceso a las dependencias de Maven. Para crear el proyecto se utilizará [Spring Initializr](https://start.spring.io/). El proyecto incluirá **Maven Wrapper** (`mvnw` y `mvnw.cmd`), por lo que no hace falta instalar Maven globalmente si el wrapper puede descargar lo necesario.

Comprueba en la terminal:

```bash
java --version
javac --version
```

Ambos comandos deben reconocer Java 21. Si el editor o la terminal utiliza otra versión, revisa su configuración antes de continuar. No confundas el **JDK**, que proporciona herramientas para compilar y ejecutar, con **Maven**, que organiza el proyecto y administra dependencias.

En Linux/macOS utilizarás `./mvnw`; en Windows PowerShell, `.\mvnw.cmd`. En este documento se muestran ambos cuando importa la diferencia. Todos los comandos Maven se ejecutan **desde la carpeta donde está `pom.xml`**.

## Antes de comenzar — Diagnóstico

Responde con tus palabras, sin buscar todavía las respuestas:

1. Si una clase muestra un menú y otra calcula resultados, ¿qué ventaja tiene separarlas?
2. ¿Para qué sirve escribir `package` al inicio de una clase Java?
3. ¿Qué diferencia imaginas entre ejecutar un programa de consola y abrir una aplicación desde un navegador?
4. Si una página muestra tres mascotas, ¿de dónde podrían provenir sus datos?
5. ¿Debería una página HTML decidir las reglas de una aplicación? ¿Por qué?

Conserva las respuestas para compararlas con tus explicaciones al finalizar.

---

## Práctica 7.1 — Paquetes, responsabilidades, MVC y arquitectura en capas

### Objetivos

Al finalizar podrás:

- explicar para qué se utilizan los paquetes Java;
- relacionar el nombre de un paquete con la ubicación de un archivo;
- distinguir la responsabilidad de presentar, coordinar y consultar datos;
- describir MVC sin confundirlo con una estructura de carpetas;
- reconocer un recorrido básico entre Controller, Service, Repository, Model y View.

### 1. El problema: una clase que hace todo

Imagina una aplicación de consola para consultar mascotas. Podrías escribir una sola clase que contiene la lista, busca una mascota y muestra el resultado:

```java
import java.util.List;

public class PetCatalogConsole {

    public static void main(String[] args) {
        List<String> petNames = List.of("Luna", "Milo", "Nube");

        for (String petName : petNames) {
            System.out.println(petName);
        }
    }
}
```

El programa es válido y su salida será:

```text
Luna
Milo
Nube
```

Para un ejemplo pequeño no existe problema. Pero imagina que luego necesitas mostrar los nombres en una página web, buscar una mascota por identificador, cambiar la manera de almacenar los datos o comprobar una regla. Si todo vive en `main`, cualquier cambio obliga a modificar la misma clase.

**Separar responsabilidades** significa asignar tareas distintas a piezas distintas. No se trata de crear la mayor cantidad posible de archivos, sino de que cada clase tenga un propósito comprensible.

### 2. Qué es un paquete Java

Un paquete agrupa clases bajo un nombre. Por ejemplo:

```java
package com.example.petmatchlab.model;
```

La clase que declare ese paquete puede guardarse en una ruta como:

```text
src/main/java/com/example/petmatchlab/model/Pet.java
```

La palabra `package` **no crea automáticamente la carpeta**. El paquete declarado y la ubicación del archivo deben ser coherentes con la estructura utilizada por el proyecto. En un proyecto Maven, el código principal se ubica normalmente bajo `src/main/java`.

Cuando otra clase necesita utilizar `Pet` desde otro paquete, puede escribir:

```java
import com.example.petmatchlab.model.Pet;
```

`package` declara a qué paquete pertenece una clase; `import` permite referirse a tipos de otros paquetes usando su nombre corto. No son instrucciones que impriman resultados ni descarguen bibliotecas.

**Distingue tres conceptos:**

| Concepto | Ejemplo | Significado |
|---|---|---|
| Carpeta | `model/` | Ubicación de archivos en el proyecto |
| Paquete | `com.example.petmatchlab.model` | Espacio de nombres de las clases Java |
| Dependencia | Thymeleaf | Biblioteca que el proyecto necesita para una funcionalidad |

Los paquetes ayudan a organizar **nuestro código**. Las dependencias proporcionan **código de bibliotecas externas**; Maven las gestionará en la siguiente práctica.

### 3. Una responsabilidad por componente

Para el catálogo que construiremos, utilizaremos estas piezas:

| Pieza | Pregunta que responde | Ejemplo |
|---|---|---|
| `Pet` (modelo de dominio) | ¿Qué datos describen una mascota? | Nombre, especie, edad y descripción |
| `InMemoryPetRepository` | ¿De dónde se obtienen los datos? | Una lista de objetos Java |
| `PetService` | ¿Qué operaciones ofrece el catálogo? | Obtener todas las mascotas o buscar una |
| `PetController` | ¿Qué hacer cuando llega una petición web? | Preparar datos y seleccionar una vista |
| Plantilla Thymeleaf | ¿Cómo se representa la información? | HTML con tarjetas y enlaces |

El nombre **Repository** no implica necesariamente una base de datos. En este laboratorio la fuente de datos será una lista en memoria. La ventaja de aislarla consiste en que otras partes no necesitan conocer cómo se guardan los elementos.

### 4. ¿Qué es una arquitectura por capas?

Una *capa* agrupa responsabilidades semejantes. Puedes imaginar el recorrido de una petición:

```text
Navegador
   |
   | GET /pets
   v
Controller  (recibe la petición y prepara la respuesta)
   |
   v
Service     (coordina una operación de la aplicación)
   |
   v
Repository  (consulta los datos)
   |
   v
Lista de objetos Pet
   |
   | Los datos regresan al Controller
   v
Model de Spring MVC + plantilla Thymeleaf
   |
   v
HTML que recibe el navegador
```

No interpretes las flechas como si cada capa fuera necesariamente un equipo o un servidor separado. Todas estas clases pueden ejecutarse dentro de **la misma aplicación Java**.

Un controlador no debería recorrer directamente la colección interna del repositorio ni construir HTML concatenando cadenas. Un servicio no debería decidir colores, encabezados ni enlaces de navegación. La plantilla no debería implementar las reglas del catálogo.

### 5. MVC: Model, View y Controller

**MVC** significa *Model–View–Controller*:

- **Model:** información que el controlador pone a disposición de la vista. En Spring MVC suele ser un objeto `Model` que transporta atributos. **No es lo mismo** que una clase de dominio como `Pet`.
- **View:** la página o plantilla que representa la información; utilizaremos Thymeleaf.
- **Controller:** recibe una petición, coordina el trabajo necesario y determina qué vista se presenta.

Es posible combinar MVC con arquitectura por capas. MVC describe especialmente cómo se organiza la interacción web; Service y Repository permiten separar otras responsabilidades de la aplicación.

### Ejercicio — ¿Dónde ubicarías el cambio?

Sin escribir código todavía, decide dónde buscarías la solución:

| Cambio solicitado | Componente más apropiado |
|---|---|
| Cambiar el texto del título de la página | ¿? |
| Agregar otra mascota a los datos de ejemplo | ¿? |
| Obtener una lista de mascotas desde una operación reutilizable | ¿? |
| Crear la ruta `/pets` | ¿? |
| Agregar el atributo `species` al objeto | ¿? |
| Cambiar el estilo de los botones | ¿? |

**Comprueba tu razonamiento:** título y estilos → vista/recursos estáticos; datos iniciales → repositorio; operación del catálogo → servicio; ruta → controlador; atributos de la mascota → modelo de dominio.

### Reto — Explica la separación

Dibuja cuatro cajas con los nombres `PetController`, `PetService`, `InMemoryPetRepository` y `Pet`. Conecta las cajas según las llamadas que esperas encontrar. Después añade `pets/list.html` para representar la salida al navegador.

### Comprobación

Explica por qué `package` no equivale a una dependencia, por qué un Controller no es el lugar correcto para almacenar toda la información y en qué se diferencia un modelo de dominio del `Model` que utiliza Spring MVC.

---

## Práctica 7.2 — Spring Boot y Maven: crear la primera aplicación web

### Objetivos

Al finalizar podrás:

- explicar para qué sirven Spring Framework, Spring Boot y Maven;
- reconocer `pom.xml`, un starter y Maven Wrapper;
- identificar el método `main` de la aplicación;
- reconocer la función de `@SpringBootApplication`;
- iniciar la aplicación y verificar su respuesta desde el navegador.

### 1. Del programa de consola al servidor web

En una aplicación de consola, `main` ejecuta instrucciones como `System.out.println` y el programa puede terminar cuando esas instrucciones concluyen. En una aplicación web, el programa inicia componentes que quedan preparados para **atender peticiones**.

Cuando escribes `http://localhost:8080/` en tu navegador:

1. **`http`** indica el protocolo de comunicación.
2. **`localhost`** se refiere al equipo donde se ejecuta la aplicación.
3. **`8080`** es el puerto de red donde escucha el servidor de esta aplicación.
4. **`/`** es la ruta solicitada.
5. El navegador envía una petición HTTP y espera una respuesta, que puede contener HTML.

El navegador **no ejecuta el código Java del servidor**. Java se ejecuta en el proceso de Spring Boot y envía una respuesta que el navegador puede interpretar.

### 2. ¿Qué son Spring Framework y Spring Boot?

**Spring Framework** proporciona funcionalidades para construir aplicaciones Java, por ejemplo administrar componentes, resolver sus dependencias y atender peticiones con Spring MVC.

**Spring Boot** simplifica la preparación de una aplicación basada en Spring. A partir de las dependencias y la configuración, prepara automáticamente muchas piezas habituales, incluido un servidor web embebido cuando usas el starter correspondiente.

Una analogía útil: Spring Framework ofrece los mecanismos; Spring Boot ayuda a reunirlos y configurarlos para poner en marcha una aplicación concreta. Esto **no significa** que Spring Boot elimine Spring ni que escriba por ti toda la lógica de negocio.

### 3. ¿Qué es Maven?

**Maven** es una herramienta de construcción y gestión de dependencias. En los primeros programas Java podías compilar una clase con `javac`. En un proyecto que utiliza varias bibliotecas, Maven ayuda a obtenerlas, establecer un proyecto reproducible, compilar y ejecutar pruebas.

Su archivo principal se llama **`pom.xml`** (*Project Object Model*). Allí se indica, entre otras cosas:

- identificación del proyecto: grupo, artefacto y versión;
- versión de Java;
- bibliotecas que necesita;
- complementos para construir o ejecutar.

**Maven no es Spring Boot:** Maven construye el proyecto; Spring Boot proporciona funcionalidades para ejecutar la aplicación web. Tampoco es una base de datos.

### 4. Crear el proyecto con Spring Initializr

Visita [start.spring.io](https://start.spring.io/) y configura:

| Campo | Valor |
|---|---|
| Project | Maven |
| Language | Java |
| Spring Boot | `4.1.1` |
| Group | `com.example` |
| Artifact | `petmatch-lab` |
| Name | `petmatch-lab` |
| Package name | `com.example.petmatchlab` |
| Packaging | Jar |
| Java | 21 |

Selecciona las dependencias **Spring Web MVC** y **Thymeleaf**. Descarga el proyecto, descomprímelo y abre la carpeta `petmatch-lab` desde el editor.

> Las opciones visibles de Spring Initializr pueden cambiar. Si no ofrece la versión indicada, genera un proyecto compatible y comprueba después el `pom.xml` con el ejemplo completo que aparece a continuación. No combines versiones de Spring Framework manualmente ni agregues dependencias solo porque aparezcan en otro ejemplo.

### 5. Reconocer la estructura creada

La estructura inicial será semejante a esta:

```text
petmatch-lab/
├── .mvn/
│   └── wrapper/
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/example/petmatchlab/
    │   │   └── PetmatchLabApplication.java
    │   └── resources/
    │       ├── application.properties
    │       ├── static/
    │       └── templates/
    └── test/
        └── java/
```

Es normal que el nombre exacto de la clase inicial cambie según los campos utilizados. **En este laboratorio usaremos `PetMatchLabApplication`**: si Initializr genera `PetmatchLabApplication`, renombra el archivo y la clase para que ambos coincidan. Conserva el paquete `com.example.petmatchlab`.

`src/main/java` contiene el código principal; `src/main/resources` contiene configuración, plantillas y archivos estáticos; `src/test` contiene pruebas. `target/` aparecerá cuando Maven compile: es una carpeta generada, no debes escribir ahí el código del proyecto.

### 6. Configurar `pom.xml`

Abre `pom.xml` y comprueba que su contenido principal sea equivalente a este proyecto completo:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.1</version>
        <relativePath/>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>petmatch-lab</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>petmatch-lab</name>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
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
            <artifactId>spring-boot-starter-webmvc-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

Lee las partes principales:

| Elemento | Significado introductorio |
|---|---|
| `<parent>` | Proporciona convenciones y versiones administradas por Spring Boot |
| `<groupId>` | Identificador del grupo al que pertenece el proyecto |
| `<artifactId>` | Nombre técnico del artefacto construido |
| `<version>` | Versión de este proyecto o de su parent, según la ubicación |
| `<java.version>` | Versión de Java que utilizará la construcción |
| `<dependency>` | Biblioteca o conjunto de bibliotecas que necesita el proyecto |
| `<scope>test</scope>` | Dependencia utilizada en las pruebas, no para el funcionamiento habitual |
| `<plugin>` | Extensión de Maven para una tarea de construcción o ejecución |

Un **starter** agrupa dependencias relacionadas. `spring-boot-starter-webmvc` permite construir la aplicación web con Spring MVC y un servidor embebido. `spring-boot-starter-thymeleaf` integra el motor de plantillas. El starter de pruebas permitirá verificar el proyecto sin incluir esas herramientas en el código principal.

No agregamos números de versión a esos starters porque el parent de Spring Boot administra las versiones compatibles. Esto evita tener que seleccionar una versión independiente de cada biblioteca.

### 7. Configurar `application.yaml`

En `src/main/resources` elimina `application.properties` si está vacío y crea **`application.yaml`**:

```yaml
spring:
  application:
    name: petmatch-lab

server:
  port: 8080
```

YAML organiza opciones mediante **indentación**. Aquí `port` está dentro de `server` y `name` dentro de `spring.application`. Utiliza espacios consistentes, no tabulaciones.

No confundas `application.yaml` con un archivo Java: Spring Boot lee esos valores al iniciar. Si el puerto `8080` ya está ocupado, puedes elegir otro puerto disponible y utilizarlo también al escribir la dirección en el navegador.

### 8. Reconocer el punto de entrada

En `src/main/java/com/example/petmatchlab/PetMatchLabApplication.java` escribe:

```java
package com.example.petmatchlab;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class PetMatchLabApplication {

    public static void main(String[] args) {
        SpringApplication.run(PetMatchLabApplication.class, args);
    }
}
```

Aquí hay conceptos conocidos y nuevos:

- `public static void main(String[] args)` continúa siendo el punto de entrada Java.
- `PetMatchLabApplication.class` es una referencia al tipo de la clase, no una instancia creada con `new`.
- `SpringApplication.run(...)` inicia el contexto de Spring y la aplicación.
- `@SpringBootApplication` activa funciones de configuración y búsqueda de componentes relacionadas con la aplicación.

La clase principal se encuentra en el paquete raíz `com.example.petmatchlab` para que Spring pueda descubrir los componentes que crearemos en sus subpaquetes.

### 9. Iniciar el servidor

En Linux o macOS, desde la raíz del proyecto:

```bash
./mvnw spring-boot:run
```

En Windows PowerShell:

```powershell
.\mvnw.cmd spring-boot:run
```

El wrapper puede descargar Maven y dependencias en la primera ejecución. Si la red no permite la descarga, revisa la conectividad o la configuración del entorno; repetir el mismo comando no corrige una dependencia inaccesible.

Cuando el servidor arranque, busca en la terminal indicaciones de que la aplicación ha comenzado y que el servidor escucha en el puerto configurado. Prueba en el navegador:

```text
http://localhost:8080/
```

**Es posible que aparezca un error 404 o una página de error. Es lo esperado en esta etapa:** el servidor puede estar activo, pero todavía no has creado una ruta `/`. Un error 404 significa que no se encontró un recurso para esa dirección; no demuestra por sí solo que el servidor haya fallado.

Para detener el proceso vuelve a la terminal y presiona `Ctrl + C`.

### 10. Compilar, probar y empaquetar

Desde la raíz del proyecto también puedes usar:

```bash
./mvnw compile
./mvnw test
./mvnw package
```

Estas son tareas distintas: `compile` compila, `test` ejecuta las pruebas disponibles y `package` produce el artefacto del proyecto. Los archivos construidos aparecen bajo `target/`. **No es necesario ejecutar los tres comandos antes de cada cambio**: selecciona el que responda a lo que deseas comprobar.

### Laboratorio de errores

1. Cambia temporalmente el puerto a otro disponible en `application.yaml` y comprueba que debes actualizar también la dirección en el navegador.
2. Escribe mal una dependencia del `pom.xml`. Ejecuta Maven y lee el mensaje relacionado con su resolución. Después restaura el identificador correcto.
3. Mueve temporalmente la clase principal a un paquete que no corresponde con su declaración `package`. Observa el error en el editor o al compilar y corrige la organización.

Provoca los errores **uno a la vez** y devuelve el proyecto a un estado funcional antes de continuar.

### Comprobación

Debes poder explicar la función de Spring Boot, Maven, `pom.xml`, Maven Wrapper, `application.yaml`, `@SpringBootApplication`, `main` y el puerto `8080` sin utilizar la frase «porque el framework lo hace» como única explicación.

---

## Práctica 7.3 — Spring MVC: rutas, controladores y vistas

### Objetivos

Al finalizar podrás:

- distinguir petición HTTP, ruta, controlador y respuesta;
- utilizar `@Controller` y `@GetMapping`;
- devolver un nombre de vista y encontrar su archivo HTML;
- utilizar `Model` para enviar un valor a la plantilla;
- explicar por qué `return "home"` no imprime la palabra `home`.

### 1. ¿Qué es una ruta?

Una **ruta** es una dirección que la aplicación reconoce. Por ejemplo, `/` representa el inicio y `/about` representa una página informativa. Cuando el navegador solicita una dirección mediante HTTP, Spring MVC debe identificar qué método Java corresponde a esa petición.

En este laboratorio utilizaremos principalmente **GET**, el método HTTP habitual para consultar páginas. Una respuesta HTTP contiene, entre otras cosas, un **código de estado** y un **cuerpo**. Si todo sale bien, el código suele ser `200`; si la ruta no existe, suele ser `404`.

**Spring MVC** es la parte de Spring Framework que relaciona peticiones web con métodos de controladores y vistas. Su infraestructura incluye un componente llamado `DispatcherServlet`, encargado de coordinar el procesamiento de las peticiones. No necesitas crearlo manualmente.

### 2. Crear el primer controlador

Dentro del paquete principal crea `controller/HomeController.java`:

```java
package com.example.petmatchlab.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";
    }

    @GetMapping("/about")
    public String about() {
        return "about";
    }
}
```

Lee el código paso a paso:

1. `@Controller` indica que esta clase participa como controlador web administrado por Spring.
2. `@GetMapping("/")` relaciona una petición GET a `/` con el método `home()`.
3. `return "home"` devuelve **el nombre lógico de una vista**, no un mensaje de texto directo.
4. `@GetMapping("/about")` relaciona otra dirección con `about()`.
5. `return "about"` indica que Spring deberá resolver una segunda plantilla.

Una anotación es información que aparece con `@` y que un framework o una herramienta puede interpretar. No es un comentario. Tampoco convierte por sí sola cualquier clase en controlador: depende de que Spring la encuentre al iniciar.

### 3. Crear las dos primeras vistas

Dentro de `src/main/resources/templates/` crea `home.html`:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Inicio | PetMatch Mini</title>
</head>
<body>
    <main>
        <h1>PetMatch Mini</h1>
        <p>Un catálogo web de mascotas construido con Java.</p>
        <a th:href="@{/about}" href="/about">Conocer el proyecto</a>
    </main>
</body>
</html>
```

Crea `about.html`:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Acerca de | PetMatch Mini</title>
</head>
<body>
    <main>
        <h1>Acerca de PetMatch Mini</h1>
        <p>Este catálogo muestra información de mascotas de ejemplo.</p>
        <a th:href="@{/}" href="/">Volver al inicio</a>
    </main>
</body>
</html>
```

**¿Qué significan estas etiquetas?** `<!DOCTYPE html>` declara HTML moderno; `<html>` es el elemento raíz; `<head>` contiene metadatos y el título de la pestaña; `<body>` contiene lo visible; `<main>` identifica el contenido principal; `<h1>` es un encabezado y `<a>` crea un enlace.

`xmlns:th` declara el espacio de nombres utilizado por los atributos de Thymeleaf. `th:href` calcula el enlace al procesar la plantilla en el servidor; el `href` normal sirve como valor de referencia cuando el archivo se abre sin Thymeleaf. En una aplicación real, comprueba los enlaces mediante el servidor, no únicamente abriendo archivos HTML desde el explorador de archivos.

### Antes de ejecutar

Predice qué dirección abrirá cada enlace y qué método Java atenderá la petición. Después inicia la aplicación y visita:

```text
http://localhost:8080/
http://localhost:8080/about
http://localhost:8080/unknown
```

Las dos primeras rutas deben mostrar HTML. La tercera debe devolver un estado de recurso no encontrado. Comprueba la dirección y el contenido visible, no solamente que el navegador muestre una página.

### 4. ¿Cómo encuentra Thymeleaf una vista?

Con la configuración habitual, cuando el controlador devuelve `"home"`, Spring Boot y Thymeleaf buscan una plantilla equivalente a:

```text
src/main/resources/templates/home.html
```

Si devuelve `"pets/list"`, buscarán:

```text
src/main/resources/templates/pets/list.html
```

El nombre del método (`home`) y el nombre de la vista (`home`) **no tienen que coincidir**. En el ejemplo son iguales porque facilitan la lectura; lo que importa para la resolución es la cadena devuelta por el controlador.

### 5. Introducir `Model`: pasar datos a la vista

Modifica solamente el método `home` del controlador:

```java
@GetMapping("/")
public String home(Model model) {
    model.addAttribute("message", "Bienvenido al catálogo de mascotas");
    return "home";
}
```

Agrega el import necesario:

```java
import org.springframework.ui.Model;
```

En `home.html`, sustituye el párrafo de bienvenida por:

```html
<p th:text="${message}">Mensaje de bienvenida.</p>
```

Ahora el recorrido es:

```text
GET /
  -> HomeController.home(Model)
  -> model.addAttribute("message", ...)
  -> return "home"
  -> Thymeleaf procesa home.html
  -> el navegador recibe el HTML resultante
```

`Model` es un contenedor de atributos que Spring MVC pone a disposición de la vista. La cadena `"message"` es la clave con la que Thymeleaf encontrará el valor. **El `Model` de Spring MVC no equivale a la clase `Pet`** que crearemos después.

**¿Qué es `${message}`?** Es una expresión de variable de Thymeleaf. En esta aplicación Spring MVC, Thymeleaf utiliza el lenguaje de expresiones de Spring para obtener el valor del atributo `message`. El navegador recibe el resultado procesado, no necesita interpretar esa expresión.

### Ejercicio — Añadir un segundo atributo

Agrega al `Model` un atributo `projectName` y muéstralo dentro de `<strong>` en la página de inicio usando `th:text`. Cambia su valor en Java y comprueba el resultado en el navegador.

### Depuración — ¿Por qué no se ve la página?

Provoca cada error por separado:

- Devuelve `"homes"` en lugar de `"home"` y observa qué ocurre cuando la plantilla no existe.
- Escribe en la plantilla `${welcomeMessage}` sin agregar ese atributo al `Model`; explica la diferencia con una ruta inexistente.
- Cambia la ruta a `@GetMapping("/welcome")` y sigue intentando visitar `/`; identifica cuál de las dos direcciones quedó registrada.

### Comprobación

Explica la diferencia entre `@Controller` y `@GetMapping`, por qué el método devuelve una cadena, dónde está la plantilla HTML, qué representa `Model` y cuándo se procesa `${message}`.

---

## Práctica 7.4 — Componentes, servicios e inyección de dependencias

### Objetivos

Al finalizar podrás:

- crear una clase de dominio con atributos, constructor y métodos de consulta;
- mantener un conjunto de objetos en una colección Java;
- explicar qué es un componente administrado por Spring (*bean*);
- reconocer `@Repository`, `@Service` y la inyección mediante constructor;
- describir por qué no necesitas crear manualmente el servicio dentro del controlador.

### 1. Representar una mascota con una clase Java

Comienza con algo que ya conoces: una clase y sus objetos. Crea `src/main/java/com/example/petmatchlab/model/Pet.java`:

```java
package com.example.petmatchlab.model;

public class Pet {

    private final long id;
    private final String name;
    private final String species;
    private final int age;
    private final String description;

    public Pet(long id, String name, String species, int age, String description) {
        this.id = id;
        this.name = name;
        this.species = species;
        this.age = age;
        this.description = description;
    }

    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getSpecies() {
        return species;
    }

    public int getAge() {
        return age;
    }

    public String getDescription() {
        return description;
    }
}
```

**Lee la clase desde lo conocido:**

- Los atributos privados representan datos que pertenecen a cada mascota.
- El constructor recibe sus valores iniciales.
- `this.id = id` distingue el atributo del objeto del parámetro recibido.
- Los métodos `get...()` permiten consultar los datos desde otras clases.
- `final` indica que esos atributos no pueden recibir una nueva asignación después de inicializarse. En esta versión del catálogo no necesitamos modificar los objetos.

El identificador `id` distingue mascotas incluso si dos tienen el mismo nombre. Utilizamos `long` para representar un número entero; los valores de ejemplo llevarán el sufijo `L`, como `1L`.

**Importante:** `Pet` es una clase Java normal, sin `@Service`, `@Controller` ni `@Repository`. Un objeto de dominio no tiene que ser administrado directamente por Spring para ser utilizado dentro de la aplicación.

### 2. Crear la fuente de datos en memoria

Crea `repository/InMemoryPetRepository.java`:

```java
package com.example.petmatchlab.repository;

import java.util.List;
import java.util.Optional;

import org.springframework.stereotype.Repository;

import com.example.petmatchlab.model.Pet;

@Repository
public class InMemoryPetRepository {

    private final List<Pet> pets = List.of(
        new Pet(1L, "Luna", "Perro", 3, "Disfruta los paseos y los juegos."),
        new Pet(2L, "Milo", "Gato", 2, "Prefiere lugares tranquilos."),
        new Pet(3L, "Nube", "Conejo", 1, "Es curioso y le gusta explorar.")
    );

    public List<Pet> findAll() {
        return pets;
    }

    public Optional<Pet> findById(long id) {
        for (Pet pet : pets) {
            if (pet.getId() == id) {
                return Optional.of(pet);
            }
        }

        return Optional.empty();
    }
}
```

Este es un ejemplo de repositorio muy sencillo. Todavía **no hay una base de datos**:

- `List.of(...)` crea una lista con tres objetos `Pet`.
- `findAll()` devuelve la lista de mascotas.
- `findById(long id)` recorre la lista y compara identificadores.
- Cuando encuentra la mascota, devuelve `Optional.of(pet)`.
- Cuando no la encuentra, devuelve `Optional.empty()`.

**¿Qué es `Optional<Pet>`?** Es un tipo de Java que permite representar explícitamente dos situaciones: *hay una mascota* o *no hay una mascota*. Su contenido, cuando existe, es un `Pet`. Así, el método no necesita inventar una mascota inexistente ni devolver `null` sin explicar esa posibilidad. En la siguiente práctica aprenderás cómo traducir ese resultado a una página o a un error HTTP.

**¿Por qué se llama `InMemory`?** Porque los datos están en la memoria del proceso Java. Si modificas el código de los elementos y reinicias la aplicación, se construirá la nueva lista. El navegador no está almacenando esos objetos y el programa no los guarda permanentemente.

`@Repository` indica que la clase cumple el papel de acceso a datos. Spring puede detectarla y administrarla como componente. No existe la obligación de utilizar siempre `@Repository` para una lista Java; aquí resulta apropiado porque esta clase es el punto al que se acude para **obtener datos** y queremos reconocer esa responsabilidad.

### 3. Crear la capa de servicio

Crea `service/PetService.java`:

```java
package com.example.petmatchlab.service;

import java.util.List;
import java.util.Optional;

import org.springframework.stereotype.Service;

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
}
```

Ahora tienes dos piezas con funciones distintas:

```text
PetService                 InMemoryPetRepository
-----------                ---------------------
Ofrece operaciones    ---> Consulta objetos guardados
al resto del sistema        en una lista
```

Quizá te preguntes: «¿Para qué sirve un servicio que en este ejemplo únicamente delega?». Aquí nos ayuda a **identificar y practicar los límites de la arquitectura**. En una aplicación más extensa, el servicio puede coordinar reglas y operaciones. No debes interpretar este ejemplo como una regla absoluta según la cual todo método necesita un servicio adicional.

### 4. ¿Qué significa que Spring administre un componente?

Con Java básico has creado objetos así:

```java
InMemoryPetRepository repository = new InMemoryPetRepository();
PetService service = new PetService(repository);
```

Estas instrucciones son Java válido. Crean dos objetos y entregan el primero al constructor del segundo. **No las agregues a los archivos del proyecto**: sirven para entender el mecanismo.

Spring dispone de un contenedor de componentes, conocido como **IoC container**. Durante el inicio, identifica clases como `@Service` y `@Repository`, crea las instancias necesarias y organiza sus dependencias. Una instancia que administra el contenedor recibe el nombre de **bean**.

La expresión **inversión de control (IoC)** significa, en este contexto, que la creación y conexión de determinados objetos ya no está escrita manualmente en el método `main` o en cada controlador: queda a cargo del contenedor.

**Inyección de dependencias (DI)** es la forma de proporcionar a una clase un objeto que necesita para trabajar. En nuestro servicio:

```java
private final InMemoryPetRepository petRepository;

public PetService(InMemoryPetRepository petRepository) {
    this.petRepository = petRepository;
}
```

- `PetService` **depende de** `InMemoryPetRepository`.
- El constructor exige recibir esa dependencia.
- Spring encuentra el repositorio administrado y se lo proporciona al crear el servicio.
- `final` evita reemplazar después la referencia almacenada en el atributo.

Cuando una clase administrada por Spring tiene **un solo constructor**, no es necesario agregar `@Autowired` a ese constructor. La inyección sigue ocurriendo. Esta forma evita esconder las dependencias en atributos que Spring modifica por reflexión.

### 5. Reconocer los componentes que ya existen

Observa lo construido:

| Clase | Anotación | Administrada por Spring | Responsabilidad |
|---|---|---|---|
| `Pet` | Ninguna | No directamente | Datos del dominio |
| `InMemoryPetRepository` | `@Repository` | Sí | Acceso a datos en memoria |
| `PetService` | `@Service` | Sí | Operaciones del catálogo |
| `HomeController` | `@Controller` | Sí | Páginas de inicio e información |

`@Service`, `@Repository` y `@Controller` son especializaciones de `@Component` que comunican mejor el papel de cada clase. No basta con escribir la anotación si la clase está fuera del área que Spring examina en busca de componentes.

### 6. Construir un controlador que utilice el servicio

Crea `controller/PetController.java`:

```java
package com.example.petmatchlab.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import com.example.petmatchlab.service.PetService;

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
}
```

**Observa qué responsabilidad tiene cada línea nueva:**

- `@RequestMapping("/pets")` establece un prefijo de ruta para el controlador.
- `@GetMapping` sin ruta adicional atiende `GET /pets`.
- El constructor recibe `PetService`; el controlador **no escribe** `new PetService(...)`.
- `petService.findAll()` solicita los datos a la capa de servicio.
- `model.addAttribute("pets", ...)` entrega la lista a la vista.
- `return "pets/list"` selecciona la plantilla `templates/pets/list.html`.

**No ejecutes todavía `/pets` sin crear la plantilla:** el controlador ya tendrá su ruta, pero fallará la resolución de la vista si `pets/list.html` no existe. Es el siguiente paso.

Crea la carpeta `templates/pets` y, dentro, el primer `list.html`:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mascotas | PetMatch Mini</title>
</head>
<body>
    <main>
        <h1>Mascotas del catálogo</h1>
        <p>Hay <strong th:text="${pets.size()}">0</strong> mascotas disponibles.</p>
        <a th:href="@{/}" href="/">Volver al inicio</a>
    </main>
</body>
</html>
```

Inicia la aplicación y abre:

```text
http://localhost:8080/pets
```

Deberías ver que el total es **3**. El HTML no contiene el número definitivo: lo obtiene a partir de la lista que el controlador colocó en `Model`.

### Antes de ejecutar — Sigue la dependencia

Intenta responder antes de mirar el navegador:

1. ¿Qué objeto construye directamente el `PetController` con `new`?
2. ¿Quién crea el `PetService` cuando inicia la aplicación?
3. ¿En qué clase se encuentran los tres `new Pet(...)`?
4. ¿Cuál es la clave del atributo agregado al `Model`?
5. ¿Qué método del servicio se invoca al visitar `/pets`?

### Ejercicio — Un nuevo elemento

Agrega una cuarta mascota dentro de la lista del repositorio, con identificador `4L`. Reinicia la aplicación si es necesario y comprueba que `/pets` ahora muestre **4** sin modificar el controlador ni la plantilla. Explica qué demuestra ese comportamiento acerca de la separación de responsabilidades.

### Laboratorio de errores

Trabaja con una copia o restaura cada cambio:

1. Elimina temporalmente `@Service` de `PetService`. Intenta iniciar y analiza el error de dependencia que no puede resolverse.
2. Cambia en el controlador la clave `"pets"` por `"animals"`, sin cambiar el HTML. Explica por qué deja de funcionar la expresión `${pets.size()}`.
3. Coloca `PetService` fuera del paquete raíz de la aplicación y observa por qué Spring ya no lo detecta automáticamente.
4. Repite un identificador de una mascota. ¿Qué resultado devuelve `findById` cuando existen dos coincidencias? ¿Por qué los identificadores del catálogo deberían ser únicos?

### Comprobación

Debes poder diferenciar una clase Java normal de un bean de Spring, explicar para qué se usan `@Repository` y `@Service`, identificar las dependencias del constructor y seguir la llamada `PetController → PetService → InMemoryPetRepository`.

---

## Práctica 7.5 — Thymeleaf: construir páginas dinámicas

### Objetivos

Al finalizar podrás:

- distinguir una plantilla HTML del HTML final que recibe el navegador;
- utilizar expresiones `${...}` y `@{...}`;
- generar elementos repetidos con `th:each`;
- mostrar contenido según una condición con `th:if` y `th:unless`;
- recuperar un valor de ruta con `@PathVariable`;
- responder con estado 404 cuando no existe la mascota solicitada.

### 1. ¿Por qué no escribir una página para cada mascota?

Podrías crear `luna.html`, `milo.html` y `nube.html`, pero repetirías el diseño. Si después tienes muchas mascotas, no sería práctico mantener una página por cada objeto.

**Thymeleaf** es un motor de plantillas que procesa HTML en el servidor. Combina una plantilla con datos provenientes de Java y produce el HTML que recibe el navegador.

```text
pets (List<Pet>)
       +
pets/list.html (plantilla)
       |
       v
Thymeleaf (procesa en el servidor)
       |
       v
HTML final con una tarjeta por mascota
```

Thymeleaf **no es una biblioteca JavaScript que el navegador necesite ejecutar**. Sus atributos `th:*` se interpretan en el servidor durante la generación de la respuesta.

### 2. Cuatro expresiones que debes reconocer

| Sintaxis | Qué hace | Ejemplo |
|---|---|---|
| `${...}` | Consulta un valor disponible en el contexto de la plantilla | `${pet.name}` |
| `@{...}` | Construye una URL | `@{/pets}` |
| `~{...}` | Señala una plantilla o fragmento reutilizable | `~{fragments/navigation :: navigation}` |
| `#{...}` | Consulta mensajes definidos en recursos de internacionalización | `#{app.title}` |

En esta práctica usarás principalmente `${...}` y `@{...}`. Los fragments aparecerán en la siguiente.

Recuerda que `${pet.name}` consulta una propiedad del objeto. Para una clase Java convencional, Thymeleaf puede obtenerla mediante el método `getName()`. **No es necesario hacer públicos los atributos privados**.

### 3. `th:text`: mostrar contenido de forma segura

Este fragmento de HTML:

```html
<h2 th:text="${pet.name}">Nombre de ejemplo</h2>
```

sustituye el contenido del `<h2>` por el nombre de la mascota cuando Thymeleaf procesa la página. Si el nombre es `Luna`, el navegador recibirá un encabezado con ese texto.

`th:text` escapa los caracteres con significado especial en HTML. Por ejemplo, si un nombre contiene `<`, no debe interpretarse como una etiqueta nueva. Para mostrar texto normal, prefiere `th:text` en lugar de `th:utext` (que inserta contenido sin escapar y no es necesario aquí).

### 4. `th:each`: repetir un elemento por cada objeto

En el controlador, `model.addAttribute("pets", petService.findAll())` deja disponible una colección llamada `pets`. En Thymeleaf:

```html
<ul>
    <li th:each="pet : ${pets}" th:text="${pet.name}">Nombre</li>
</ul>
```

- `${pets}` es la lista completa.
- `pet` es el nombre que recibirá **cada elemento** durante la repetición.
- `th:each` repite el elemento `<li>` por cada mascota.
- `th:text` sustituye el contenido de cada `<li>` por el nombre correspondiente.

Con los datos del repositorio, la lista tendrá tres elementos: Luna, Milo y Nube. **No escribas esos nombres directamente en el HTML**: perderías el beneficio de la vista dinámica.

### 5. Crear las tarjetas del catálogo

Sustituye el contenido completo de `templates/pets/list.html` por:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mascotas | PetMatch Mini</title>
</head>
<body>
    <main>
        <h1>Catálogo de mascotas</h1>
        <p>Conoce las mascotas registradas en este ejemplo.</p>

        <section th:if="${pets.isEmpty()}">
            <h2>El catálogo está vacío</h2>
            <p>Vuelve cuando haya mascotas para mostrar.</p>
        </section>

        <section th:unless="${pets.isEmpty()}" aria-label="Lista de mascotas">
            <article th:each="pet : ${pets}">
                <h2 th:text="${pet.name}">Nombre de la mascota</h2>
                <p>Especie: <span th:text="${pet.species}">Especie</span></p>
                <p>Edad: <span th:text="${pet.age}">0</span> años</p>
                <a th:href="@{/pets/{id}(id=${pet.id})}" href="#">Ver detalle</a>
            </article>
        </section>

        <a th:href="@{/}" href="/">Volver al inicio</a>
    </main>
</body>
</html>
```

**Analiza lo que acabas de construir:**

- `<section>` agrupa contenido relacionado; `<article>` representa cada tarjeta con información independiente.
- `th:if="${pets.isEmpty()}"` muestra el mensaje de vacío si la colección no tiene elementos.
- `th:unless="${pets.isEmpty()}"` muestra el catálogo en el caso contrario. Es una manera de expresar «si no se cumple la condición».
- `th:each="pet : ${pets}"` produce un artículo por mascota.
- Cada atributo mostrado consulta una propiedad del objeto correspondiente.

El método `isEmpty()` pertenece a la colección Java; Thymeleaf lo invoca sobre el objeto entregado por el controlador. **No está consultando una base de datos desde la plantilla**.

### 6. ¿Cómo se forma el enlace de detalle?

Observa:

```html
<a th:href="@{/pets/{id}(id=${pet.id})}">Ver detalle</a>
```

Esta expresión indica que `{id}` es una parte variable de la ruta. Si la mascota tiene `id = 2`, Thymeleaf puede generar un enlace como:

```text
/pets/2
```

En lugar de concatenar manualmente cadenas para producir URLs, `@{...}` utiliza las expresiones de enlace de Thymeleaf. El enlace todavía necesita que exista un método del controlador que atienda esa ruta.

### 7. `@PathVariable`: leer un identificador desde la dirección

Amplía `controller/PetController.java` con los imports siguientes:

```java
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.server.ResponseStatusException;
```

Debajo del método `list`, agrega:

```java
@GetMapping("/{id}")
public String detail(@PathVariable("id") long id, Model model) {
    var pet = petService.findById(id)
        .orElseThrow(() -> new ResponseStatusException(
            HttpStatus.NOT_FOUND,
            "Pet not found"
        ));

    model.addAttribute("pet", pet);
    return "pets/detail";
}
```

**Lee cada parte:**

1. La clase tiene `@RequestMapping("/pets")` y el método declara `@GetMapping("/{id}")`: juntos atienden `/pets/1`, `/pets/2`, etc.
2. `@PathVariable("id")` obtiene el valor colocado en la posición `{id}`.
3. Spring convierte el texto numérico de la ruta al tipo `long` del parámetro.
4. `petService.findById(id)` busca el objeto y devuelve `Optional<Pet>`.
5. `orElseThrow(...)` extrae la mascota si existe; si no, lanza una excepción con estado HTTP 404.
6. `model.addAttribute("pet", pet)` expone **una mascota** (no una lista) a la vista.
7. `return "pets/detail"` selecciona el archivo `templates/pets/detail.html`.

La palabra `var` permite que Java deduzca el tipo local de la variable por la expresión de inicialización. Aquí el tipo inferido es `Pet`. No significa que Java deje de comprobar tipos. Si te resulta más claro, puedes escribir `Pet pet = ...` y agregar el import de la clase `Pet`.

`()-> ...` forma parte de una expresión lambda de Java. Aquí representa una función sin argumentos que construye la excepción **solo cuando** no se encuentra la mascota. Su propósito es evitar tratar un resultado ausente como si existiera.

### 8. Crear la vista de detalle

Crea `src/main/resources/templates/pets/detail.html`:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Detalle | PetMatch Mini</title>
</head>
<body>
    <main>
        <article>
            <h1 th:text="${pet.name}">Nombre</h1>
            <p>Identificador: <span th:text="${pet.id}">0</span></p>
            <p>Especie: <span th:text="${pet.species}">Especie</span></p>
            <p>Edad: <span th:text="${pet.age}">0</span> años</p>
            <p th:text="${pet.description}">Descripción de la mascota.</p>
        </article>
        <a th:href="@{/pets}" href="/pets">Volver al catálogo</a>
    </main>
</body>
</html>
```

Visita `/pets`, pulsa un enlace de detalle y comprueba que la información corresponde a la mascota seleccionada. Después modifica el identificador manualmente en la barra de direcciones.

| Dirección | Resultado esperado |
|---|---|
| `/pets` | Catálogo de mascotas |
| `/pets/1` | Detalle de Luna |
| `/pets/2` | Detalle de Milo |
| `/pets/3` | Detalle de Nube |
| `/pets/999` | Respuesta HTTP 404 |

Si agregaste una cuarta mascota en la práctica anterior, `/pets/4` también debe resolver su detalle.

### 9. Diferenciar tres tipos de error

No todos los problemas de una página web significan lo mismo:

| Situación | Significado | Dónde revisar |
|---|---|---|
| `/unknown` | No hay una ruta asociada | Controladores y anotaciones de ruta |
| `/pets/999` | La ruta existe, pero el dato solicitado no | Búsqueda del servicio y respuesta 404 |
| `return "pets/missing"` | El controlador devuelve un nombre de plantilla inexistente | Archivos dentro de `templates/` |

El último problema suele producir un fallo al resolver o procesar la vista, no un «dato no encontrado». Revisa la consola para ver el mensaje exacto de tu ejecución.

### 10. Comprobar el caso de catálogo vacío

Para probar `th:if` y `th:unless`, sustituye **temporalmente** la lista de `InMemoryPetRepository` por:

```java
private final List<Pet> pets = List.of();
```

Ejecuta y visita `/pets`. Ahora debe mostrarse el mensaje **El catálogo está vacío** y no deben aparecer tarjetas. Restaura después la lista original. Esta prueba comprueba que la interfaz representa correctamente dos estados distintos, no solo el catálogo con datos.

### Ejercicio — Nueva información en la interfaz

Agrega un pequeño texto en el detalle que diga **«Datos de la mascota»** y muestra el identificador mediante `th:text`. Después modifica la descripción de un objeto `Pet` en el repositorio y comprueba que la página cambia sin editar el HTML.

### Reto — Explica el recorrido completo

Describe qué sucede al pulsar **Ver detalle** en la tarjeta de Milo, usando correctamente estos términos: `th:href`, petición HTTP, `@PathVariable`, `PetController`, `PetService`, `InMemoryPetRepository`, `Optional`, `Model`, Thymeleaf y respuesta HTML.

### Depuración — Tres fallos diferentes

1. Cambia temporalmente `@GetMapping("/{id}")` por `@GetMapping("/{petId}")` sin modificar `@PathVariable("id")`. Identifica por qué los nombres deben corresponder.
2. Cambia el atributo del controlador de `"pet"` por `"animal"` sin modificar la plantilla de detalle. Explica qué expresión ya no puede resolverse.
3. Cambia `th:href` para enviar siempre el valor `1` en lugar de `${pet.id}`. Explica por qué todos los enlaces llevan a Luna aunque las tarjetas muestren otros nombres.

### Comprobación

Debes poder explicar la diferencia entre una lista de objetos y una plantilla repetida, entre `th:text` y `th:each`, entre `${...}` y `@{...}`, y entre una ruta desconocida y una mascota inexistente.

---

## Práctica 7.6 — Fragments, navegación, estilos y reto integrador

### Objetivos

Al finalizar podrás:

- identificar contenido repetido entre varias páginas;
- crear y reutilizar fragments con `th:fragment` y `th:replace`;
- pasar un parámetro a un fragmento;
- separar los estilos CSS de los documentos HTML;
- construir una navegación coherente y adaptable a distintas pantallas;
- verificar rutas, contenido dinámico y situaciones de error.

### 1. Un problema visible: HTML repetido

Observa los archivos `home.html`, `about.html`, `pets/list.html` y `pets/detail.html`. Cada uno tiene un `<head>` con etiquetas semejantes. También todos necesitan enlaces hacia el inicio y el catálogo.

Si copias el mismo encabezado en cuatro archivos y luego quieres cambiar el nombre de la aplicación o añadir una hoja de estilos, tendrás que repetir el cambio cuatro veces. Eso aumenta la posibilidad de dejar páginas diferentes sin querer.

**Reutilizar un fragmento** significa definir una porción de plantilla una sola vez e insertarla donde haga falta. Thymeleaf ofrece esta funcionalidad sin necesitar una biblioteca adicional de layouts.

### 2. Crear un fragmento para el encabezado

Crea `src/main/resources/templates/fragments/head.html`:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:fragment="head(pageTitle)">
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title th:text="${pageTitle}">PetMatch Mini</title>
    <link rel="stylesheet" th:href="@{/css/application.css}">
</head>
</html>
```

Analiza la línea:

```html
<head th:fragment="head(pageTitle)">
```

- `th:fragment` define una región reutilizable.
- `head` es el nombre que elegimos para el fragmento.
- `pageTitle` es un parámetro que puede recibir un valor diferente en cada página.
- `th:text="${pageTitle}"` coloca ese valor dentro de `<title>`.

**No es un método Java ni un bean de Spring.** Es una definición dentro de una plantilla Thymeleaf.

### 3. Utilizar el fragmento desde una página

En cualquier plantilla, sustituye su `<head>...</head>` original por:

```html
<head th:replace="~{fragments/head :: head('Inicio | PetMatch Mini')}"></head>
```

La expresión `~{fragments/head :: head(...)}` señala el archivo `templates/fragments/head.html`, el fragmento llamado `head` y el valor del parámetro que recibe.

`th:replace` **reemplaza el elemento que lo contiene** por el fragmento seleccionado. En este caso, el resultado contiene un `<head>` completo y no un `<head>` dentro de otro. Existe también `th:insert`, que inserta el fragmento *dentro* del elemento contenedor; no son exactamente equivalentes.

En las demás vistas cambia el texto del título:

```html
<head th:replace="~{fragments/head :: head('Acerca de | PetMatch Mini')}"></head>
<head th:replace="~{fragments/head :: head('Catálogo | PetMatch Mini')}"></head>
<head th:replace="~{fragments/head :: head('Detalle | PetMatch Mini')}"></head>
```

Esas líneas representan **alternativas para páginas distintas**, no cuatro etiquetas `<head>` que deban escribirse juntas en un mismo documento.

### 4. Crear la navegación compartida

Crea `templates/fragments/navigation.html`:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<body>
    <nav th:fragment="navigation" class="site-navigation" aria-label="Navegación principal">
        <a class="brand" th:href="@{/}" href="/">PetMatch Mini</a>
        <div class="navigation-links">
            <a th:href="@{/}" href="/">Inicio</a>
            <a th:href="@{/pets}" href="/pets">Mascotas</a>
            <a th:href="@{/about}" href="/about">Acerca de</a>
        </div>
    </nav>
</body>
</html>
```

Este fragmento no recibe parámetros porque la navegación será igual en todas las páginas. `aria-label` aporta un nombre descriptivo a la región de navegación. Las clases CSS, como `site-navigation`, sirven para aplicar estilos, no son anotaciones Java.

Para utilizarlo dentro de `<body>`:

```html
<nav th:replace="~{fragments/navigation :: navigation}"></nav>
```

Ahora es posible cambiar los enlaces principales en **un solo archivo**.

### 5. Separar los estilos en un archivo CSS

Crea `src/main/resources/static/css/application.css`:

```css
:root {
    font-family: system-ui, sans-serif;
    color: #172033;
    background: #f5f7fb;
    line-height: 1.5;
}

* {
    box-sizing: border-box;
}

body {
    margin: 0;
}

a {
    color: #1757a6;
}

a:focus-visible {
    outline: 3px solid #ffae42;
    outline-offset: 3px;
}

.site-navigation {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
    padding: 1rem max(1rem, calc((100vw - 1100px) / 2));
    background: #ffffff;
    border-bottom: 1px solid #dbe3ed;
}

.brand {
    font-weight: 800;
    text-decoration: none;
    color: #17365f;
}

.navigation-links {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
}

.page-container {
    width: min(100% - 2rem, 1100px);
    margin-inline: auto;
    padding-block: 2rem;
}

.page-intro {
    max-width: 65ch;
    color: #465267;
}

.pet-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(min(100%, 250px), 1fr));
    gap: 1rem;
    margin-block: 1.5rem;
}

.pet-card,
.content-panel {
    background: #ffffff;
    border: 1px solid #dbe3ed;
    border-radius: 12px;
    padding: 1.5rem;
}

.pet-card h2,
.content-panel h1 {
    margin-top: 0;
}

.button-link {
    display: inline-block;
    padding: 0.7rem 1rem;
    border-radius: 8px;
    color: #ffffff;
    background: #1757a6;
    text-decoration: none;
    font-weight: 600;
}

.button-link:hover {
    background: #103e78;
}

.muted {
    color: #58657a;
}

@media (max-width: 620px) {
    .site-navigation {
        flex-direction: column;
        align-items: flex-start;
    }
}
```

**¿Por qué `static/css` y no `templates/css`?** Los archivos bajo `static/` se sirven como recursos estáticos. El navegador los solicita por una ruta como `/css/application.css`. Las plantillas en `templates/` se procesan para producir HTML y normalmente se utilizan a través de un controlador.

**¿Qué hace `@media`?** Es una regla CSS que aplica ciertos estilos cuando se cumple una condición de visualización; aquí ayuda a ordenar la navegación en pantallas estrechas. No es una anotación de Spring. La propiedad `:focus-visible` hace más clara la posición del foco al navegar con teclado.

Abre `http://localhost:8080/css/application.css` para comprobar que el archivo está disponible. Ver código CSS en esa dirección es normal: se trata de un recurso, no de una página de Thymeleaf.

### 6. Actualizar la página de inicio

Sustituye `templates/home.html` por esta versión completa:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Inicio | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <section class="content-panel">
            <h1>Bienvenido a PetMatch Mini</h1>
            <p class="page-intro" th:text="${message}">
                Bienvenido al catálogo de mascotas.
            </p>
            <p>Explora un catálogo de ejemplo construido con Java y Spring Boot.</p>
            <a class="button-link" th:href="@{/pets}" href="/pets">Explorar mascotas</a>
        </section>
    </main>
</body>
</html>
```

Este archivo presupone que el método `home(Model model)` sigue agregando el atributo `message`, como hiciste en la práctica 7.3. Si lo eliminaste, restáuralo antes de comprobar la vista.

### 7. Actualizar la página informativa

Sustituye `templates/about.html` por:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Acerca de | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <section class="content-panel">
            <h1>Acerca de PetMatch Mini</h1>
            <p>Esta aplicación presenta un catálogo de mascotas de ejemplo.</p>
            <p>Los datos se consultan mediante un servicio y se muestran con Thymeleaf.</p>
            <a class="button-link" th:href="@{/pets}" href="/pets">Ver catálogo</a>
        </section>
    </main>
</body>
</html>
```

Observa que la estructura básica es la misma, pero el contenido de `<main>` sigue perteneciendo a cada página. **Reutilizar no significa que todas las páginas tengan que ser idénticas**.

### 8. Actualizar el catálogo

Sustituye `templates/pets/list.html` por esta versión completa:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Catálogo | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <h1>Catálogo de mascotas</h1>
        <p class="page-intro">Selecciona una mascota para conocer sus detalles.</p>

        <section class="content-panel" th:if="${pets.isEmpty()}">
            <h2>El catálogo está vacío</h2>
            <p>Vuelve cuando haya mascotas para mostrar.</p>
        </section>

        <section class="pet-grid"
                 aria-label="Lista de mascotas"
                 th:unless="${pets.isEmpty()}">
            <article class="pet-card" th:each="pet : ${pets}">
                <h2 th:text="${pet.name}">Nombre</h2>
                <p>Especie: <span th:text="${pet.species}">Especie</span></p>
                <p>Edad: <span th:text="${pet.age}">0</span> años</p>
                <a class="button-link"
                   th:href="@{/pets/{id}(id=${pet.id})}"
                   href="#">Ver detalle</a>
            </article>
        </section>
    </main>
</body>
</html>
```

El HTML contiene **una tarjeta** marcada con `th:each`, pero Thymeleaf produce tantas tarjetas como elementos tenga la colección. El CSS organiza esas tarjetas con una cuadrícula adaptable al espacio disponible.

### 9. Actualizar la página de detalle

Sustituye `templates/pets/detail.html` por:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Detalle | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <article class="content-panel">
            <h1 th:text="${pet.name}">Nombre</h1>
            <p><strong>Identificador:</strong> <span th:text="${pet.id}">0</span></p>
            <p><strong>Especie:</strong> <span th:text="${pet.species}">Especie</span></p>
            <p><strong>Edad:</strong> <span th:text="${pet.age}">0</span> años</p>
            <p class="muted" th:text="${pet.description}">Descripción.</p>
            <a class="button-link" th:href="@{/pets}" href="/pets">Volver al catálogo</a>
        </article>
    </main>
</body>
</html>
```

Ahora las cuatro páginas utilizan el mismo encabezado y la misma navegación. Si cambias el nombre de un enlace en `fragments/navigation.html`, el cambio debe verse en todas las vistas que incluyen ese fragmento.

### 10. Reconocer características técnicas de la interfaz

Una interfaz no queda bien resuelta únicamente porque tenga colores. Comprueba estos detalles:

| Característica | Cómo se aplica | Por qué importa |
|---|---|---|
| HTML semántico | `<nav>`, `<main>`, `<section>`, `<article>`, encabezados | Facilita reconocer la estructura de la página |
| Navegación consistente | Un fragmento compartido | Evita enlaces diferentes entre páginas |
| Diseño adaptable | CSS Grid y `@media` | Permite utilizar pantallas de distintos tamaños |
| Título de página | Parámetro `pageTitle` | Identifica la página en la pestaña del navegador |
| Foco visible | `:focus-visible` | Ayuda a navegar con teclado |
| Estado vacío | `th:if` y `th:unless` | Evita presentar una sección vacía sin explicación |
| Errores HTTP | Respuesta 404 | Diferencia un recurso inexistente de uno válido |

Prueba la navegación usando **Tab** y **Enter**, sin ratón. Comprueba que puedas identificar cuál enlace tiene el foco. Reduce el ancho de la ventana y observa cómo se reorganizan las tarjetas.

### 11. Reto integrador — Completa una funcionalidad

Agrega un enlace a `/pets` desde el inicio si aún no está presente y crea una nueva mascota en `InMemoryPetRepository` con un identificador distinto de los existentes. Asegúrate de que:

1. Aparece automáticamente en el catálogo.
2. Su enlace **Ver detalle** conduce a la mascota correcta.
3. Su nombre, especie, edad y descripción proceden del objeto Java.
4. El catálogo sigue mostrando correctamente las mascotas originales.
5. No necesitas duplicar una plantilla HTML para incorporar el nuevo elemento.
6. Los enlaces de navegación continúan funcionando desde cualquier página.

**Ampliación opcional:** agrega a `Pet` el atributo `size` como texto, inclúyelo en el constructor, define `getSize()` y muéstralo tanto en el catálogo como en el detalle. Actualiza todas las llamadas a `new Pet(...)`. Antes de escribir, identifica qué archivos sí deben cambiar y cuáles deberían mantenerse intactos.

### 12. Una primera comprobación automatizada

Una prueba automatizada ejecuta código y comprueba un resultado esperado. No es obligatorio levantar el servidor ni abrir el navegador para verificar una operación Java pura.

Crea `src/test/java/com/example/petmatchlab/service/PetServiceTests.java`:

```java
package com.example.petmatchlab.service;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.Test;

import com.example.petmatchlab.repository.InMemoryPetRepository;

class PetServiceTests {

    private final PetService petService = new PetService(new InMemoryPetRepository());

    @Test
    void shouldReturnAvailablePets() {
        assertTrue(!petService.findAll().isEmpty());
    }

    @Test
    void shouldFindPetWithKnownId() {
        var pet = petService.findById(1L);

        assertTrue(pet.isPresent());
        assertEquals("Luna", pet.orElseThrow().getName());
    }

    @Test
    void shouldReturnEmptyWhenPetDoesNotExist() {
        assertTrue(petService.findById(999L).isEmpty());
    }
}
```

Estas pruebas utilizan **JUnit**, disponible mediante la dependencia de pruebas del proyecto. `@Test` marca un método como prueba. `assertTrue` verifica una condición y `assertEquals` compara un resultado con el valor esperado.

Observa algo importante: la prueba construye `PetService` mediante `new` y le entrega un repositorio. **Puede hacerlo porque sus dependencias están claras en el constructor.** En el funcionamiento web, Spring crea y conecta esos componentes; en esta prueba Java, controlamos su creación directamente para comprobar la lógica sin iniciar el servidor.

Si cambiaste el nombre de Luna o su identificador durante los ejercicios, adapta el caso conocido al dato que realmente conserve tu catálogo. La prueba debe reflejar un requisito verificable, no un valor que ya no exista.

Ejecuta:

```bash
./mvnw test
```

O en Windows PowerShell:

```powershell
.\mvnw.cmd test
```

Si una prueba falla, lee el nombre del método que falló y la diferencia entre valor esperado y valor obtenido. **No elimines la prueba para hacer desaparecer el error**: corrige la implementación o ajusta un dato de prueba que hayas cambiado intencionalmente.

### 13. Comprobaciones finales de navegación y errores

Inicia de nuevo la aplicación y recorre estas direcciones:

| Comprobación | Resultado esperado |
|---|---|
| `/` | Inicio con navegación compartida y mensaje dinámico |
| `/about` | Página informativa con el mismo encabezado |
| `/pets` | Tarjetas generadas desde los datos del repositorio |
| `/pets/1` | Detalle de la mascota con identificador 1 |
| `/pets/999` | Respuesta HTTP 404 |
| `/unknown` | Ruta no encontrada (404) |
| `/css/application.css` | Recurso CSS accesible |

Además, verifica que los vínculos puedan utilizarse con teclado, que las páginas tengan títulos diferenciados y que ninguna muestre literalmente expresiones `${...}` o `@{...}` donde debería aparecer contenido procesado.

### Depuración — Diagnósticos de interfaz

**Problema 1 — No se aplican los estilos.** La página HTML funciona, pero el CSS no carga. Revisa si el archivo existe bajo `static/css`, si el fragmento del encabezado tiene el enlace correcto y si `/css/application.css` responde.

**Problema 2 — Se repite una etiqueta.** El documento termina con un `<head>` anidado. Comprueba si utilizaste `th:insert` donde necesitabas `th:replace` y revisa el elemento raíz del fragmento.

**Problema 3 — Un enlace abre una dirección incorrecta.** Comprueba que el `id` del enlace provenga de la mascota que se está recorriendo y que coincida con el nombre de variable de la ruta.

**Problema 4 — El controlador no puede iniciar.** Lee el mensaje de Spring relacionado con una dependencia faltante y comprueba las anotaciones, los paquetes y el constructor.

**Problema 5 — La página carga, pero no tiene datos.** Comprueba la clave utilizada en `model.addAttribute`, el nombre consultado por la plantilla y el contenido de la lista del repositorio.

---

## Mapa final del proyecto

Al terminar, el proyecto debe estar organizado de forma semejante a esta:

```text
petmatch-lab/
├── .mvn/
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/example/petmatchlab/
    │   │   ├── PetMatchLabApplication.java
    │   │   ├── controller/
    │   │   │   ├── HomeController.java
    │   │   │   └── PetController.java
    │   │   ├── model/
    │   │   │   └── Pet.java
    │   │   ├── repository/
    │   │   │   └── InMemoryPetRepository.java
    │   │   └── service/
    │   │       └── PetService.java
    │   └── resources/
    │       ├── application.yaml
    │       ├── static/
    │       │   └── css/
    │       │       └── application.css
    │       └── templates/
    │           ├── home.html
    │           ├── about.html
    │           ├── fragments/
    │           │   ├── head.html
    │           │   └── navigation.html
    │           └── pets/
    │               ├── list.html
    │               └── detail.html
    └── test/
        └── java/com/example/petmatchlab/service/
            └── PetServiceTests.java
```

**¿Qué no debe aparecer en el proyecto?** No necesitas crear una clase `Main` adicional para imprimir los resultados de la web, copiar HTML dentro del controlador, crear una plantilla por mascota ni duplicar la lista dentro de `PetService`.

## Revisión del funcionamiento

Antes de considerar terminado el laboratorio, verifica:

- [ ] El proyecto utiliza Java 21 y puede compilarse con Maven Wrapper.
- [ ] La clase principal está en el paquete raíz de la aplicación.
- [ ] Los paquetes y los nombres de clase reflejan responsabilidades diferentes.
- [ ] El servidor inicia sin errores de componentes faltantes.
- [ ] `HomeController` resuelve `/` y `/about`.
- [ ] `PetController` resuelve `/pets` y `/pets/{id}`.
- [ ] El controlador no almacena una lista de mascotas ni construye HTML como cadenas Java.
- [ ] `PetService` recibe el repositorio por constructor.
- [ ] El repositorio conserva datos en memoria y permite consultar identificadores.
- [ ] El catálogo recorre los datos con `th:each` y muestra un estado vacío adecuado.
- [ ] Los enlaces de detalle utilizan identificadores dinámicos.
- [ ] Los identificadores inexistentes generan una respuesta 404.
- [ ] El encabezado y la navegación se reutilizan mediante fragments.
- [ ] Las hojas de estilo están en `static/css` y las vistas en `templates`.
- [ ] La navegación se utiliza con teclado y se adapta a una pantalla estrecha.
- [ ] Las pruebas automatizadas se ejecutan correctamente.

## Diagnóstico final — Demuestra que comprendiste

### 1. Sigue una petición de principio a fin

Una persona abre `/pets/2`. Explica, en orden:

1. Qué envía el navegador y qué representa el número `2`.
2. Qué controlador y qué método atienden esa petición.
3. Cómo se recibe el identificador en Java.
4. Qué operación solicita el controlador al servicio.
5. Cómo busca el repositorio la mascota.
6. Qué sucede si la encuentra y qué sucede si no la encuentra.
7. Cómo llega el objeto a la plantilla.
8. Qué procesa Thymeleaf y qué recibe finalmente el navegador.

### 2. Compara dos fragmentos de Java

**Versión A:**

```java
InMemoryPetRepository repository = new InMemoryPetRepository();
PetService service = new PetService(repository);
```

**Versión B:**

```java
private final PetService petService;

public PetController(PetService petService) {
    this.petService = petService;
}
```

Explica quién crea los objetos en cada situación. ¿Por qué la segunda versión permite que Spring conecte los componentes sin escribir `new PetService(...)` dentro del controlador?

### 3. Localiza el error de responsabilidad

Una persona propone guardar una lista de mascotas directamente en `PetController` y generar una cadena HTML con `StringBuilder`. Otra propone colocar las reglas del catálogo en un archivo HTML de Thymeleaf.

Explica qué responsabilidad se está mezclando en cada caso y dónde ubicarías esas operaciones.

### 4. Predice el cambio

Si se agrega una quinta mascota exclusivamente en `InMemoryPetRepository`, ¿qué debería suceder al recargar el catálogo? ¿Qué archivos deberían permanecer sin cambios?

### 5. Explica términos sin memorizarlos

Define mediante el ejemplo construido: **paquete, clase, dependencia, Maven, starter, bean, inyección de dependencias, controlador, ruta, modelo de dominio, Model de Spring MVC, vista, plantilla, fragmento y recurso estático**.

## Cierre del laboratorio

Has pasado de una aplicación Java que muestra resultados en una terminal a una aplicación que atiende peticiones HTTP y produce páginas web. Las clases, los constructores y las colecciones siguen siendo los mismos conceptos de Java: Spring Boot añade infraestructura para iniciar la aplicación y administrar componentes; Spring MVC conecta rutas con controladores; Thymeleaf transforma los datos y las plantillas en HTML.

El aprendizaje central no consiste en memorizar anotaciones. Consiste en **entender el recorrido de la información y asignar cada responsabilidad al lugar adecuado**. Cuando puedas explicar por qué un cambio en la lista del repositorio aparece en la interfaz sin reescribir el controlador, habrás conectado los conceptos principales del laboratorio.

## Referencias oficiales

- [Spring Boot — Structuring Your Code](https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html): ubicación de la clase principal y organización de paquetes.
- [Spring Boot — Build Systems](https://docs.spring.io/spring-boot/reference/using/build-systems.html): dependencias, versiones administradas y starters.
- [Spring Boot — Servlet Web Applications](https://docs.spring.io/spring-boot/reference/web/servlet.html): aplicaciones web con Spring MVC, plantillas y recursos.
- [Spring Framework — Annotated Controllers](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller.html): `@Controller`, métodos de controlador y `Model`.
- [Spring Framework — Classpath Scanning and Managed Components](https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html): detección de `@Component`, `@Service` y `@Repository`.
- [Spring Framework — Using `@Autowired`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html): inyección de dependencias y constructores.
- [Thymeleaf 3.1 — Using Thymeleaf](https://www.thymeleaf.org/doc/tutorials/3.1/usingthymeleaf.html): expresiones, `th:each`, enlaces, condiciones y fragments.
- [Apache Maven — Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes): estructura básica y fases de construcción.
- [Java SE 21 — API de `Optional`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html): presencia o ausencia de valores.
- [MDN — HTML: HyperText Markup Language](https://developer.mozilla.org/en-US/docs/Web/HTML): HTML semántico y elementos básicos de una página.
