# Laboratorio 9 — Persistencia de datos con MariaDB, JPA, Hibernate y Spring Data JPA

Una aplicación puede permitir registrar, consultar, editar y eliminar información y, aun así, tener una limitación importante: **perder todos los cambios cuando deja de ejecutarse**. Eso ocurre si los datos únicamente viven en una colección Java. Una colección resulta útil para practicar la lógica de un programa, pero no es un almacenamiento permanente.

En este laboratorio transformarás **PetMatch Mini** para que las mascotas se guarden en **MariaDB**, una base de datos relacional. Aprenderás a reconocer tablas y filas, ejecutar consultas SQL, configurar la conexión desde Spring Boot, convertir una clase Java en una entidad JPA y reemplazar el repositorio en memoria por uno administrado por Spring Data JPA. Al final comprobarás que los datos permanecen después de detener y volver a iniciar la aplicación.

El objetivo no es aprender anotaciones de memoria. Es poder explicar **qué información se guarda, dónde se guarda, cómo llega hasta allí y cómo vuelve a convertirse en objetos Java**.

Partiremos de la aplicación construida anteriormente: `PetController`, `PetService`, `PetForm`, la clase `Pet`, las vistas Thymeleaf y las operaciones CRUD ya existen. No será necesario rehacer los formularios ni crear otro proyecto.

| Práctica | Aprendizaje principal | Resultado comprobable |
|---|---|---|
| 9.1 | Persistencia, datos relacionales y SQL básico | Consultar y describir una tabla sin confundirla con una colección Java |
| 9.2 | MariaDB, JDBC, Maven y `application.yaml` | Preparar una base de datos y configurar una conexión sin guardar contraseñas en el código |
| 9.3 | ORM, Jakarta Persistence, Hibernate y mapeo de entidades | Convertir `Pet` en una entidad persistente con identificador generado |
| 9.4 | Spring Data JPA y repositorios | Reemplazar `InMemoryPetRepository` por una interfaz `JpaRepository` |
| 9.5 | Persistir las operaciones CRUD y comprobar SQL | Registrar, consultar, modificar y eliminar datos en MariaDB desde la interfaz existente |
| 9.6 | Pruebas, depuración y reto integrador | Verificar persistencia, errores y separación de responsabilidades |

> **Forma de trabajo:** antes de ejecutar un comando, anticipa su efecto. Después comprueba el resultado tanto en la aplicación como en la base de datos. Distingue siempre entre un ejemplo **conceptual**, que explica una idea, y un **archivo completo**, que debes incorporar al proyecto. No ejecutes instrucciones que eliminen bases de datos o tablas para experimentar: utiliza consultas y registros de prueba que puedas identificar.

## Preparación — Reconocer el punto de partida

Necesitas:

- **JDK 21**, para mantener la versión de Java empleada por la aplicación.
- El proyecto `petmatch-lab` con **Spring Boot 4.1.1**, Maven Wrapper, Spring MVC, Thymeleaf y Bean Validation.
- El CRUD funcional desarrollado anteriormente, con `PetController`, `PetService`, `PetForm` e `InMemoryPetRepository`.
- **MariaDB Server** instalado y ejecutándose, o una instancia de MariaDB a la que puedas conectarte con permisos para crear una base de datos de práctica.
- Un cliente de MariaDB para ejecutar SQL desde la terminal. También puedes utilizar un cliente gráfico compatible; los comandos SQL serán los mismos.

La estructura inicial se parece a esta:

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
    │   │   ├── dto/PetForm.java
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
    │               ├── detail.html
    │               └── form.html
    └── test/java/com/example/petmatchlab/
        ├── controller/PetControllerIntegrationTests.java
        ├── repository/InMemoryPetRepositoryTests.java
        └── service/PetServiceTests.java
```

Si los nombres o datos de tus mascotas son diferentes, no necesitas restaurarlos. Lo importante es conservar los contratos de las operaciones y comprender las modificaciones. En los ejemplos aparecerán nombres como **Luna**, **Milo** y **Kira** únicamente para facilitar las comprobaciones.

### Antes de comenzar — Diagnóstico

Responde sin buscar definiciones:

1. ¿Dónde guarda los objetos la clase `InMemoryPetRepository`?
2. ¿Qué sucede con esos objetos cuando el proceso Java termina?
3. ¿Por qué un formulario HTML no debería conocer la contraseña de una base de datos?
4. Si `PetService` consulta un repositorio, ¿debería importar si los datos están en un `Map` o en MariaDB?
5. ¿Qué diferencia supones que existe entre un objeto Java con atributos y una fila de una tabla?
6. ¿Cómo demostrarías que un dato sigue existiendo después de reiniciar una aplicación?

Al terminar el laboratorio deberías poder responder cada pregunta con un ejemplo de código y una comprobación observable.

---

## Práctica 9.1 — De la memoria de Java a una base de datos relacional

### Objetivos

Al finalizar esta práctica podrás:

- explicar el significado de **persistencia**;
- diferenciar un proceso Java, una base de datos y un servidor de base de datos;
- reconocer bases de datos, tablas, columnas, filas y claves primarias;
- ejecutar consultas `SELECT` con filtros y ordenación;
- relacionar operaciones CRUD con instrucciones SQL;
- describir la diferencia entre `NULL` y una cadena vacía.

### 1. El problema: los datos solo existen mientras Java se ejecuta

En el repositorio anterior se utilizaba una estructura parecida a esta:

```java
private final Map<Long, Pet> pets = new LinkedHashMap<>();
```

Un `Map` almacena parejas de clave y valor **en la memoria del proceso Java**. Por ejemplo:

```text
Clave     Objeto
-----     -----------------------------
1         Pet(id=1, name="Luna", ...)
2         Pet(id=2, name="Milo", ...)
3         Pet(id=3, name="Nube", ...)
```

Al registrar una mascota, la aplicación añadía otro objeto al `Map`. Al editarla, reemplazaba un valor. Al eliminarla, quitaba una entrada.

El inconveniente aparece al cerrar el proceso:

```text
Aplicación encendida
    └── memoria del proceso Java
          └── Map<Long, Pet>

Aplicación detenida
    └── el proceso ya no existe
          └── los objetos del Map tampoco
```

Si el constructor del repositorio tenía tres mascotas iniciales, volver a ejecutar el programa recreaba esas tres mascotas. Eso **no** significaba que los registros nuevos se hubieran guardado.

**Persistir** significa conservar información en un almacenamiento que permite recuperarla después de que termine la ejecución que la creó. Una base de datos relacional es una manera habitual de conseguirlo.

### 2. El servidor de aplicación y el servidor de base de datos son distintos

Observa la siguiente representación:

```text
Navegador
    |
    | HTTP
    v
Spring Boot (proceso Java)
    |
    | consultas y operaciones de datos
    v
MariaDB Server
    |
    | almacenamiento administrado por el motor
    v
Tablas y registros persistentes
```

Los dos servidores pueden estar instalados en el mismo equipo, pero **son programas con responsabilidades diferentes**:

| Componente | Responsabilidad |
|---|---|
| Navegador | Presentar HTML y enviar formularios |
| Spring Boot | Recibir peticiones, aplicar reglas y preparar respuestas |
| MariaDB Server | Almacenar y consultar registros de manera persistente |
| Cliente MariaDB | Permitir ejecutar SQL directamente contra MariaDB |

Cerrar Spring Boot no significa necesariamente cerrar MariaDB. Si la base de datos continúa disponible, los registros que se hayan guardado correctamente seguirán allí.

**Predice:** si apagas la aplicación, consultas una tabla desde el cliente MariaDB y todavía encuentras las mascotas, ¿qué demuestra eso sobre el lugar donde viven sus datos?

### 3. ¿Qué es una base de datos relacional?

Una **base de datos** organiza información. En una base de datos *relacional*, una forma principal de organizarla es mediante **tablas**.

Imagina la tabla `pets`:

```text
pets
+----+------+---------+-----+--------------------------+
| id | name | species | age | description              |
+----+------+---------+-----+--------------------------+
|  1 | Luna | Dog     |   3 | Enjoys long walks.       |
|  2 | Milo | Cat     |   2 | Prefers quiet places.    |
|  3 | Kira | Dog     |   4 | Very playful.            |
+----+------+---------+-----+--------------------------+
```

Las palabras `id`, `name`, `species`, `age` y `description` son los **nombres de las columnas**. Cada fila contiene los valores correspondientes a una mascota.

| Concepto | Explicación | Ejemplo |
|---|---|---|
| Tabla | Conjunto estructurado de registros | `pets` |
| Columna | Dato definido para cada registro | `species` |
| Fila | Registro individual | Una mascota llamada Luna |
| Tipo SQL | Tipo permitido para los valores | `VARCHAR`, `INT`, `BIGINT` |
| Clave primaria | Identificador que distingue una fila | `id` |
| Restricción | Condición que la base de datos exige | `NOT NULL` |

Una tabla **no es** una lista Java: se consulta con SQL, tiene tipos y restricciones propios y el motor administra su almacenamiento.

### 4. ¿Por qué necesitamos una clave primaria?

Dos mascotas pueden llamarse `Luna`. Por tanto, el nombre no es un identificador confiable.

```text
id=1, name="Luna", species="Dog"
id=8, name="Luna", species="Cat"
```

La columna `id` permite distinguir ambas filas. Cuando `id` es una **clave primaria** (*primary key*), la base de datos exige que sus valores identifiquen inequívocamente las filas.

El identificador sirve para las operaciones que ya conoces:

```text
GET /pets/8
POST /pets/8
POST /pets/8/delete
```

El navegador envía el identificador de la mascota que interesa. El servidor utiliza ese dato para buscar la fila correspondiente. **El navegador no debería elegir el identificador de una mascota nueva**: dejaremos que MariaDB lo genere.

### 5. SQL: un lenguaje para trabajar con datos

**SQL**, de *Structured Query Language*, permite describir qué información se quiere consultar o modificar en una base de datos relacional.

Lee esta consulta como una frase:

```sql
SELECT id, name, species, age
FROM pets
WHERE species = 'Dog'
ORDER BY name;
```

- `SELECT` indica las columnas que queremos consultar.
- `FROM pets` señala la tabla.
- `WHERE` limita las filas según una condición.
- `ORDER BY` establece el orden del resultado.
- El punto y coma `;` termina la instrucción en el cliente SQL.

Esta consulta expresa: «Muestra el identificador, el nombre, la especie y la edad de las mascotas cuya especie sea Dog, ordenadas por nombre».

Todavía no ejecutes la consulta: primero crearemos la base de datos y la tabla. Si la tabla no existe, MariaDB no podrá resolver `FROM pets`.

### 6. CRUD y SQL: cuatro familias de operaciones

Ya conoces el acrónimo **CRUD** por el formulario web:

| Operación de la aplicación | Instrucción SQL típica | Intención |
|---|---|---|
| Create | `INSERT` | Crear una fila |
| Read | `SELECT` | Consultar filas |
| Update | `UPDATE` | Cambiar valores de filas existentes |
| Delete | `DELETE` | Eliminar filas |

Estas instrucciones ilustran la idea, **no son todavía pasos para ejecutar**:

```sql
-- Conceptual examples: execute only after creating the table.
INSERT INTO pets (name, species, age, description)
VALUES ('Kira', 'Dog', 4, 'Very playful.');

SELECT id, name, species, age, description
FROM pets;

UPDATE pets
SET age = 5
WHERE id = 3;

DELETE FROM pets
WHERE id = 3;
```

Fíjate en `WHERE id = 3` en las dos últimas instrucciones. Sin el filtro `WHERE`, una sentencia `UPDATE` o `DELETE` podría afectar **todas las filas** de la tabla. No utilices estos ejemplos para modificar datos que quieras conservar.

En el proyecto no escribiremos manualmente cada `INSERT` o `UPDATE` dentro del controlador. Más adelante verás cómo una biblioteca convierte operaciones Java en SQL.

### 7. Tipos y valores ausentes

Los tipos de una tabla y los tipos Java se parecen, pero no son idénticos:

| Java | SQL habitual | Observación |
|---|---|---|
| `Long` | `BIGINT` | Entero suficientemente grande para identificadores |
| `String` | `VARCHAR(n)` | Texto de longitud limitada |
| `int` | `INT` | Número entero |
| `String` opcional | `VARCHAR(n)` que acepta `NULL` | Un valor puede estar ausente |

`NULL` en SQL significa que no hay un valor almacenado para esa columna. No es lo mismo que una cadena vacía `''` ni que un texto formado por espacios.

Esto conecta con una regla del servicio existente: una descripción que el usuario deja vacía puede normalizarse a `null` antes de guardarla.

### 8. ¿Qué son las relaciones?

Una aplicación puede tener varias tablas relacionadas. Por ejemplo, una tabla de propietarios y otra de mascotas podría conectarse mediante una columna `owner_id`. Esa columna haría referencia al identificador del propietario.

```text
owners                         pets
+----+--------+                 +----+------+----------+
| id | name   |                 | id | name | owner_id |
+----+--------+                 +----+------+----------+
|  1 | Ana    | <-------------  | 10 | Luna |    1     |
+----+--------+                 +----+------+----------+
```

La referencia entre tablas se puede proteger mediante una **clave foránea** (*foreign key*). Por ahora trabajaremos con una única tabla `pets`, de manera que sea posible comprender primero la persistencia de una entidad simple. Lo importante es reconocer qué problema resuelven las relaciones sin tener que introducirlas en todo el código.

### Ejercicio — Traduce una consulta

Explica en lenguaje cotidiano qué harían estas instrucciones si la tabla existiera:

```sql
SELECT name, age FROM pets WHERE age >= 3 ORDER BY age DESC;
```

```sql
SELECT COUNT(*) FROM pets;
```

Después responde:

1. ¿La primera consulta modifica una fila?
2. ¿Qué indica `COUNT(*)`?
3. ¿Qué columna necesitarías para buscar una mascota específica por su identificador?
4. Si dos filas tienen el mismo `name`, ¿es necesariamente un error?

### Depuración — Identifica el tipo de problema

| Síntoma | Pregunta útil |
|---|---|
| Una consulta dice que la tabla no existe | ¿Se creó la tabla en la base de datos correcta? |
| Aparecen filas de otra aplicación | ¿Qué base de datos está seleccionada? |
| Una consulta devuelve cero filas | ¿No hay datos o el filtro `WHERE` excluye todos? |
| Un identificador parece duplicarse | ¿La columna está definida realmente como clave primaria? |

### Comprobación

Debes poder diferenciar memoria y persistencia, explicar una fila y una clave primaria y traducir las operaciones CRUD a su intención SQL sin depender de un controlador ni de una plantilla Thymeleaf.

---

## Práctica 9.2 — Preparar MariaDB y conectar Spring Boot

### Objetivos

Al finalizar podrás:

- distinguir MariaDB Server del cliente MariaDB;
- crear una base de datos independiente para la aplicación;
- utilizar un usuario de base de datos específico para la práctica;
- explicar para qué sirve un controlador JDBC (*driver*);
- añadir dependencias Maven compatibles;
- configurar `application.yaml` con variables de entorno;
- diagnosticar errores básicos de conexión.

### 1. Comprobar que MariaDB está disponible

MariaDB puede instalarse de distintas maneras según el sistema operativo. Si ya tienes una instancia en ejecución, no hace falta reinstalarla.

Desde una terminal con el cliente instalado, comprueba:

```bash
mariadb --version
```

Esto informa la versión del **cliente**; no confirma por sí solo que el servidor esté encendido o accesible. Para comprobar una conexión, utiliza una cuenta administrativa de tu instalación:

```bash
mariadb -u root -p
```

`-u root` solicita el usuario `root` de MariaDB y `-p` hace que el cliente solicite la contraseña sin escribirla directamente en el comando. Según cómo se haya instalado MariaDB, la cuenta administrativa puede utilizar autenticación del sistema operativo; en ese caso, usa el procedimiento de conexión configurado para tu instancia.

**No confundas** el usuario `root` de MariaDB con la cuenta de tu sistema operativo. Además, **no** utilizaremos la cuenta administrativa para conectar la aplicación.

Una vez dentro del cliente, comprueba:

```sql
SELECT VERSION();
```

Para salir:

```sql
EXIT;
```

Si falla la conexión, revisa el mensaje exacto. «Comando no encontrado» indica un problema distinto de «Access denied» o «Connection refused».

### 2. Crear una base de datos para PetMatch Mini

Entra al cliente MariaDB con una cuenta que tenga permisos para crear bases de datos y usuarios. Ejecuta:

```sql
CREATE DATABASE IF NOT EXISTS petmatch_lab
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;
```

Descompongamos la instrucción:

- `CREATE DATABASE` crea una base de datos.
- `IF NOT EXISTS` evita un error si ya existe otra base con ese nombre; no borra la existente.
- `petmatch_lab` es el nombre elegido.
- `utf8mb4` permite almacenar caracteres Unicode, incluidos símbolos que necesitan varios bytes.
- `COLLATE` establece reglas de comparación y ordenación de texto.

Comprueba el resultado:

```sql
SHOW DATABASES;
```

Selecciona la base de datos:

```sql
USE petmatch_lab;
```

Consulta las tablas:

```sql
SHOW TABLES;
```

Si está recién creada, no debería existir todavía la tabla `pets`. Esa ausencia es correcta: **crear una base de datos no implica crear automáticamente sus tablas**.

### 3. Crear un usuario de aplicación

Una aplicación web no necesita conectarse como administrador global de MariaDB. Utiliza una cuenta dedicada y limita sus permisos a su propia base de datos.

Desde una sesión administrativa, reemplaza la contraseña del ejemplo por una contraseña local propia y suficientemente robusta:

```sql
CREATE USER IF NOT EXISTS 'petmatch_user'@'127.0.0.1'
    IDENTIFIED BY 'replace_with_a_unique_local_password';

GRANT ALL PRIVILEGES ON petmatch_lab.*
    TO 'petmatch_user'@'127.0.0.1';
```

**Lee la instrucción antes de ejecutarla:**

- `'petmatch_user'` es la cuenta de MariaDB que utilizará el programa.
- `'127.0.0.1'` limita el origen de conexión a esa dirección local, que también usaremos en la URL JDBC.
- `petmatch_lab.*` limita los privilegios a los objetos de esta base de datos, no a todas las bases del servidor.
- `ALL PRIVILEGES` en esa base se utiliza aquí porque Hibernate necesitará crear y ajustar el esquema durante las prácticas. **No es una recomendación para una cuenta de producción**: allí se separan los permisos de migración y los permisos mínimos de ejecución.

> **Importante:** `CREATE USER IF NOT EXISTS` no cambia la contraseña de una cuenta que ya existe. Si utilizas una cuenta creada anteriormente, debes conocer sus credenciales válidas o actualizarla conscientemente con los permisos administrativos correspondientes; no sigas suponiendo que se aplicó la contraseña del ejemplo.

Comprueba los permisos:

```sql
SHOW GRANTS FOR 'petmatch_user'@'127.0.0.1';
```

Sal de la sesión administrativa y prueba la conexión de la nueva cuenta desde una terminal:

```bash
mariadb --protocol=TCP -h 127.0.0.1 -P 3306 -u petmatch_user -p petmatch_lab
```

El argumento `--protocol=TCP` hace explícito que deseas conectarte por red local y no por un archivo de socket. Esto ayuda a distinguir las cuentas de MariaDB asociadas a `localhost` de las asociadas a `127.0.0.1`.

Dentro de esa conexión ejecuta:

```sql
SELECT DATABASE();
SHOW TABLES;
```

La primera instrucción debería indicar `petmatch_lab`. Si no puedes conectarte con esta cuenta, resuelve ese problema antes de introducir Spring Boot: **una configuración Java no arregla credenciales SQL incorrectas**.

### 4. ¿Cómo puede Java hablar con MariaDB?

Un navegador utiliza HTTP para comunicarse con Spring Boot. Spring Boot necesita otro mecanismo para comunicarse con MariaDB.

**JDBC**, *Java Database Connectivity*, proporciona interfaces estándar de Java para trabajar con conexiones y operaciones de bases de datos relacionales.

Un **driver JDBC** es una biblioteca que implementa esa comunicación para un motor concreto. Utilizaremos **MariaDB Connector/J**.

```text
Código Java
    |
    | APIs de persistencia
    v
JPA / Hibernate / Spring Data
    |
    | JDBC
    v
MariaDB Connector/J
    |
    | protocolo de base de datos
    v
MariaDB Server
```

En la aplicación no será necesario escribir `DriverManager.getConnection(...)` ni manejar manualmente un `ResultSet` por cada consulta. Pero comprender **JDBC** evita la idea equivocada de que Hibernate establece una conexión mágica sin ninguna infraestructura subyacente.

### 5. Añadir las dependencias al `pom.xml`

Abre el `pom.xml` existente y conserva sus dependencias de Spring MVC, Thymeleaf, Validation y pruebas web. Agrega **dentro de `<dependencies>`**:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.mariadb.jdbc</groupId>
    <artifactId>mariadb-java-client</artifactId>
    <scope>runtime</scope>
</dependency>
```

La primera dependencia incorpora las bibliotecas necesarias para trabajar con **Spring Data JPA** y **Hibernate**. La segunda proporciona el controlador JDBC para MariaDB.

**¿Por qué aparece `<scope>runtime</scope>`?** Porque nuestro código Java no invocará directamente clases del driver. Necesitamos que el driver esté disponible cuando la aplicación establezca la conexión. Maven administra esas diferencias entre lo necesario para compilar y lo necesario para ejecutar.

El `pom.xml` completo, conservando los elementos ya utilizados, debería quedar así:

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

Más adelante añadiremos dependencias **de prueba** para comprobar los repositorios sin modificar los datos de la base de desarrollo. Por ahora, el fragmento anterior basta para arrancar la aplicación conectada a MariaDB una vez implementada la entidad y el repositorio.

### 6. Configurar la conexión en `application.yaml`

La aplicación ya contiene `src/main/resources/application.yaml`. Configura allí la conexión:

```yaml
spring:
  application:
    name: petmatch-lab
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: update
    open-in-view: false
    show-sql: true
    properties:
      hibernate:
        format_sql: true

server:
  port: 8080
```

**No copies la contraseña en `application.yaml`.** Los nombres `${DB_URL}`, `${DB_USERNAME}` y `${DB_PASSWORD}` indican que Spring debe obtener los valores de variables de entorno.

| Propiedad | ¿Para qué sirve? |
|---|---|
| `spring.datasource.url` | Dirección JDBC y nombre de la base de datos |
| `spring.datasource.username` | Cuenta con la que se conecta la aplicación |
| `spring.datasource.password` | Contraseña de esa cuenta |
| `spring.jpa.hibernate.ddl-auto` | Indica qué hará Hibernate con el esquema de las tablas |
| `spring.jpa.open-in-view` | Controla si la sesión JPA permanece disponible durante el renderizado web |
| `spring.jpa.show-sql` | Muestra sentencias SQL generadas en la salida de la aplicación |
| `hibernate.format_sql` | Presenta ese SQL de forma más legible |

**La indentación importa.** YAML organiza propiedades mediante espacios. `password` debe quedar al mismo nivel que `url` y `username`, y las propiedades de JPA deben quedar dentro de `spring`.

La URL tendrá esta forma:

```text
jdbc:mariadb://127.0.0.1:3306/petmatch_lab
```

Descompón cada parte:

| Parte | Significado |
|---|---|
| `jdbc:mariadb:` | Tipo de conexión y driver compatible |
| `127.0.0.1` | Equipo donde se encuentra MariaDB |
| `3306` | Puerto de la instancia; es el habitual, pero puede ser distinto |
| `petmatch_lab` | Base de datos a utilizar |

**No uses `http://`**: la base de datos no está recibiendo esta conexión mediante HTTP.

### 7. Definir las variables de entorno

Abre una terminal desde la carpeta raíz del proyecto. Utiliza la misma contraseña que configuraste al crear el usuario de MariaDB.

**Linux o macOS:**

```bash
export DB_URL='jdbc:mariadb://127.0.0.1:3306/petmatch_lab'
export DB_USERNAME='petmatch_user'
export DB_PASSWORD='replace_with_your_local_password'
```

**Windows PowerShell:**

```powershell
$env:DB_URL = 'jdbc:mariadb://127.0.0.1:3306/petmatch_lab'
$env:DB_USERNAME = 'petmatch_user'
$env:DB_PASSWORD = 'replace_with_your_local_password'
```

**Importante:** reemplaza el texto de ejemplo por la contraseña real. Estos comandos establecen variables en el entorno de la terminal desde la que iniciarás la aplicación. Si abres una terminal nueva, puede ser necesario configurarlas de nuevo.

No publiques contraseñas, archivos con credenciales ni capturas que las muestren. Para una aplicación desplegada o compartida deben utilizarse mecanismos adecuados de gestión de secretos y privilegios mínimos.

### 8. ¿Qué ocurre al iniciar Spring Boot?

Spring Boot lee las propiedades, detecta las dependencias y configura un objeto **DataSource**, que permite obtener conexiones a MariaDB. Habitualmente utiliza un **pool de conexiones**, un conjunto administrado de conexiones reutilizables.

Esta configuración no crea automáticamente una cuenta de MariaDB ni resuelve una base de datos que no existe. Necesita:

```text
MariaDB accesible
        +
base de datos existente
        +
usuario autorizado
        +
URL y credenciales correctas
        +
driver JDBC en las dependencias
```

**Todavía no ejecutes el proyecto modificado:** el servicio sigue esperando `InMemoryPetRepository` y no hemos preparado la entidad que JPA necesita. En las prácticas siguientes haremos los cambios de código correspondientes y luego iniciaremos la aplicación completa.

### 9. Entender `ddl-auto: update` sin confundirlo con una migración

La propiedad:

```yaml
spring:
  jpa:
    hibernate:
      ddl-auto: update
```

permite que Hibernate intente ajustar el **esquema** de la base de datos a partir de las entidades que encuentre. Por ejemplo, puede crear la tabla `pets` cuando todavía no existe.

Reconoce los valores principales:

| Valor | Comportamiento general |
|---|---|
| `none` | No realiza acciones automáticas de generación o actualización del esquema |
| `validate` | Comprueba que el esquema existente corresponde al mapeo; no lo crea |
| `update` | Intenta actualizar el esquema existente |
| `create` | Recrea el esquema administrado, con riesgo de perder datos previos |
| `create-drop` | Crea el esquema y lo elimina al cerrar el contexto |

Usaremos `update` **exclusivamente como apoyo en un entorno local de aprendizaje**. No equivale a llevar un historial controlado de cambios ni garantiza resolver toda modificación compleja del modelo. En aplicaciones que deben conservar datos importantes, los cambios de esquema se administran mediante migraciones versionadas y procedimientos de respaldo.

**No cambies a `create` o `create-drop` para solucionar errores de manera impulsiva.** Esas opciones pueden provocar pérdida de datos.

### Ejercicio — Diagnóstico antes de iniciar

Clasifica estos problemas según ocurran en Maven, configuración, autenticación o conectividad:

1. `mariadb-java-client` no puede descargarse.
2. `DB_PASSWORD` no está definida en el entorno.
3. El servidor escucha en otro puerto.
4. El usuario existe, pero no tiene permisos sobre `petmatch_lab`.
5. El nombre de la base de datos en la URL contiene un error tipográfico.
6. `spring.datasource.username` está mal indentada en YAML.

**Antes de seguir:** comprueba desde el cliente MariaDB que puedes conectarte como `petmatch_user`, consultar `SELECT DATABASE()` y ver `petmatch_lab` como base seleccionada.

### Comprobación

Debes poder explicar la diferencia entre servidor, driver, URL y cuenta, y demostrar que la contraseña no está escrita en `pom.xml` ni en `application.yaml`. También debes comprender por qué `ddl-auto: update` es una comodidad local, no una estrategia general de evolución de bases de datos.

---
## Práctica 9.3 — Convertir `Pet` en una entidad JPA

### Objetivos

Al finalizar podrás:

- distinguir una clase de dominio, una **entidad JPA** y una tabla SQL;
- explicar qué significa **ORM**;
- diferenciar **Jakarta Persistence**, **Hibernate** y **Spring Data JPA**;
- utilizar `@Entity`, `@Table`, `@Id`, `@GeneratedValue` y `@Column`;
- comprender por qué una entidad necesita un constructor sin argumentos;
- explicar por qué `Long` es apropiado para un identificador generado;
- reconocer el papel de las restricciones en la base de datos.

### 1. El problema: dos representaciones de los mismos datos

En Java representamos una mascota mediante un objeto:

```java
Pet pet = new Pet("Kira", "Dog", 4, "Very playful.");
```

En MariaDB queremos representar esa mascota mediante una fila:

```text
pets
+----+------+---------+-----+---------------+
| id | name | species | age | description   |
+----+------+---------+-----+---------------+
|  7 | Kira | Dog     |   4 | Very playful. |
+----+------+---------+-----+---------------+
```

No son la misma estructura:

- Java trabaja con **clases, constructores, objetos, referencias y métodos**.
- SQL trabaja con **tablas, columnas, tipos de datos, claves y restricciones**.

Si escribiéramos toda la conversión manualmente, tendríamos que ejecutar SQL, leer cada columna del resultado y construir objetos Java a partir de esas columnas.

### 2. ORM: relacionar objetos con tablas

**ORM** significa *Object-Relational Mapping*, o mapeo objeto-relacional.

Su idea central es describir una correspondencia entre ambas representaciones:

```text
Java                       MariaDB
--------------------       -------------------
Clase Pet                  Tabla pets
Objeto Pet                 Fila de pets
Campo name                 Columna name
Campo age                  Columna age
Campo id                   Clave primaria id
```

Esa correspondencia permite trabajar con objetos Java mientras una biblioteca realiza las operaciones de persistencia correspondientes.

**ORM no elimina las bases de datos.** Tampoco significa que SQL deje de existir: las operaciones reales siguen llegando al motor de la base de datos.

### 3. JPA, Hibernate y Spring Data JPA no son sinónimos

Estos tres nombres describen responsabilidades diferentes:

| Tecnología | Qué es | Responsabilidad |
|---|---|---|
| Jakarta Persistence (JPA) | Una especificación estándar de persistencia en Java | Define contratos y anotaciones como `@Entity` y `@Id` |
| Hibernate ORM | Una implementación de persistencia utilizada por Spring Boot | Interpreta mapeos, gestiona entidades y genera/ejecuta SQL |
| Spring Data JPA | Una biblioteca de Spring Data | Simplifica la creación y uso de repositorios basados en JPA |

Representación simplificada:

```text
PetService
    |
    v
PetRepository (Spring Data JPA)
    |
    v
Jakarta Persistence / Hibernate
    |
    v
JDBC / MariaDB Connector/J
    |
    v
MariaDB Server
```

**Predice:** si en una clase colocas `@Entity`, ¿significa que Spring Data JPA ya sabe qué formulario HTML mostrar? No: el mapeo de una entidad y la presentación de una vista resuelven problemas distintos.

### 4. ¿Qué cambiará en `Pet`?

La clase `Pet` anterior tenía atributos `final` y un constructor que recibía incluso el identificador. Eso era apropiado para los objetos creados en memoria y definidos por nuestro propio código.

Con JPA necesitamos algunos cambios:

1. Representar la clase como entidad persistente.
2. Permitir que Hibernate construya instancias cuando recupera filas.
3. Permitir que el motor asigne un identificador después de insertar un registro.
4. Permitir cambios controlados en los atributos que pueden editarse.
5. Conservar los **getters** que utilizan el controlador, el servicio y Thymeleaf.

Para mantener el ejemplo fácil de seguir, sustituiremos el archivo completo en vez de introducir fragmentos que dependan de constructores anteriores.

### 5. Reemplazar `model/Pet.java`

Sustituye por completo el contenido de `src/main/java/com/example/petmatchlab/model/Pet.java`:

```java
package com.example.petmatchlab.model;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "pets")
public class Pet {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, length = 80)
    private String species;

    @Column(nullable = false)
    private int age;

    @Column(length = 1000)
    private String description;

    protected Pet() {
    }

    public Pet(String name, String species, int age, String description) {
        this.name = name;
        this.species = species;
        this.age = age;
        this.description = description;
    }

    public void updateDetails(
        String name,
        String species,
        int age,
        String description
    ) {
        this.name = name;
        this.species = species;
        this.age = age;
        this.description = description;
    }

    public Long getId() {
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

Este es un **archivo completo**. Cada elemento tiene una función que merece entenderse antes de continuar.

### 6. Leer `@Entity` y `@Table`

```java
@Entity
@Table(name = "pets")
public class Pet {
    // ...
}
```

- `@Entity` indica que la clase es una entidad administrada mediante Jakarta Persistence.
- `@Table(name = "pets")` indica el nombre de la tabla utilizada para persistir sus instancias.
- El nombre de la clase puede ser `Pet` y el nombre de la tabla `pets`; no tienen por qué escribirse exactamente igual.

`@Entity` **no convierte a `Pet` en un controlador de Spring MVC**. Una entidad es un objeto del dominio que participa en la persistencia; no procesa rutas HTTP.

El comentario `// ...` de este fragmento significa que se omitió el resto de la clase para explicar la anotación. No debes copiar este fragmento por encima del archivo completo.

### 7. Leer `@Id` y `@GeneratedValue`

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

- `@Id` identifica la propiedad utilizada como **clave primaria**.
- `@GeneratedValue` indica que su valor será generado como parte de la operación de persistencia.
- `GenerationType.IDENTITY` utiliza una estrategia de identificadores respaldada por una columna de identidad del motor, como `AUTO_INCREMENT` en MariaDB.

La entidad nueva se construye sin pasar un identificador:

```java
Pet pet = new Pet("Kira", "Dog", 4, "Very playful.");
```

En ese instante, antes de persistirla:

```text
pet.getId() → null
```

Después de guardarla correctamente, Hibernate puede recuperar el identificador generado por MariaDB y asignarlo a la entidad.

**¿Por qué `Long` y no `long`?**

- `long` es un tipo primitivo; su valor inicial es `0` cuando forma parte de un objeto sin asignación explícita.
- `Long` es un tipo de referencia y puede contener `null`.
- En este diseño, `null` expresa con claridad que el objeto todavía **no tiene un identificador persistente**.

`0` y `null` no significan lo mismo. No asignes manualmente `1L`, `2L` o `3L` a mascotas nuevas: el control del identificador pasa a la base de datos.

### 8. Leer `@Column`

```java
@Column(nullable = false, length = 100)
private String name;
```

La anotación describe propiedades del mapeo a la columna:

- `nullable = false` exige que el valor persistido en la columna no sea SQL `NULL`.
- `length = 100` indica una longitud prevista de hasta 100 caracteres para un texto de longitud variable.

El código también contiene:

```java
@Column(nullable = false)
private int age;
```

Al ser `int` un tipo primitivo, el atributo siempre tiene un valor numérico en un objeto Java, pero ese valor podría ser negativo. Por eso el formulario ya contiene `@Min(0)` y el servicio también verifica la regla.

**Importante: una restricción de base de datos no reemplaza la validación de entrada.**

| Mecanismo | Ejemplo | Qué protege |
|---|---|---|
| Validación del formulario | `@NotBlank` | La entrada recibida desde un formulario |
| Regla del servicio | Rechazar una edad negativa | La operación aun si se invoca sin formulario |
| Restricción de columna | `nullable = false` | La integridad de un dato almacenado |

`nullable = false` por sí solo no rechaza cadenas que contienen espacios ni garantiza una edad no negativa. Son mecanismos complementarios.

### 9. ¿Por qué el constructor vacío es `protected`?

```java
protected Pet() {
}
```

JPA necesita poder construir objetos al convertir filas recuperadas en entidades. Por eso exige un constructor sin argumentos que sea `public` o `protected`.

El constructor con parámetros sirve para el código de nuestra aplicación:

```java
public Pet(String name, String species, int age, String description) {
    this.name = name;
    this.species = species;
    this.age = age;
    this.description = description;
}
```

La visibilidad `protected` evita que cualquier clase externa invoque el constructor vacío como operación normal de creación. No significa que la clase herede de otra clase escrita por nosotros.

**Predice:** ¿qué error de diseño introducirías si todas las partes del programa crearan mascotas con `new Pet()` y dejaran el nombre, la especie y la edad sin establecer?

### 10. ¿Por qué el método `updateDetails(...)`?

En el modelo anterior, para actualizar una mascota el repositorio construía **otro** objeto con el mismo identificador y reemplazaba la entrada del `Map`.

En el modelo persistente queremos representar la modificación de una entidad:

```java
public void updateDetails(
    String name,
    String species,
    int age,
    String description
) {
    this.name = name;
    this.species = species;
    this.age = age;
    this.description = description;
}
```

Este método permite cambiar únicamente los datos editables. **No modifica el identificador**.

No es necesario crear un `setId(...)`. La identidad del registro no es un campo del formulario y no debe cambiarse por una edición normal.

El método no decide si los valores son válidos: esa responsabilidad sigue estando en la validación y en el servicio. Separar esas tareas hace que la entidad sea fácil de comprender.

### 11. El mapeo es una descripción, no una operación CRUD

Después de escribir `@Entity`, `@Id` y `@Column`, Java tiene una definición para interpretar la clase como entidad. Pero las anotaciones no ejecutan automáticamente:

```text
registrar una mascota
buscar todas las mascotas
modificar una mascota
eliminar una mascota
```

Para hacer esas operaciones necesitamos llamar a un repositorio. Aprenderás a hacerlo en la práctica siguiente.

### Ejercicio — Reconoce la responsabilidad

Indica en qué parte del archivo `Pet.java` encontrarías:

1. El nombre de la tabla SQL.
2. La propiedad que identifica una fila.
3. La estrategia de generación del identificador.
4. La posibilidad de que `description` no tenga valor.
5. La restricción de longitud del nombre.
6. El constructor utilizado al registrar una mascota.
7. El método que modifica sus datos editables.

Después responde: ¿qué ocurriría con las expresiones Thymeleaf `${pet.name}` y `${pet.id}` si se eliminaran sus respectivos getters?

### Laboratorio de errores

En una copia de trabajo que puedas restaurar, experimenta de forma controlada y **sin ejecutar modificaciones destructivas en la base de datos**:

- Quita temporalmente `@Entity` y observa, al iniciar la aplicación completa más adelante, cómo reacciona JPA cuando el repositorio intenta utilizar una clase no administrada.
- Cambia temporalmente `Long id` por `String id` sin modificar las anotaciones. ¿Por qué deja de existir una correspondencia clara con la estrategia de identidad elegida?
- Elimina temporalmente el constructor sin argumentos. ¿Qué requisito de JPA estarías incumpliendo?

Restaura el archivo completo antes de continuar. La finalidad es relacionar cada error con una responsabilidad, no memorizar mensajes de excepción concretos.

### Comprobación

Puedes explicar por qué `Pet` ahora es una entidad, por qué su identificador comienza en `null`, por qué existe un constructor `protected` sin argumentos y por qué una columna SQL no es simplemente un atributo Java.

---

## Práctica 9.4 — Sustituir el repositorio en memoria por Spring Data JPA

### Objetivos

Al finalizar podrás:

- describir qué aporta un repositorio a la arquitectura en capas;
- distinguir una clase implementada manualmente de una interfaz de Spring Data;
- crear un repositorio que extienda `JpaRepository<Pet, Long>`;
- reconocer métodos CRUD heredados;
- comprender el uso de `Optional<Pet>`;
- explicar por qué no hace falta escribir una implementación SQL propia;
- conservar un controlador que no sabe cómo se almacenan los datos.

### 1. Antes: un repositorio construido por nosotros

En el proyecto anterior el repositorio tenía una estructura similar a esta:

```java
@Repository
public class InMemoryPetRepository {
    private final Map<Long, Pet> pets = new LinkedHashMap<>();
    // ...
}
```

**Nosotros** decidíamos qué estructura de datos utilizar, cómo asignar el siguiente identificador y cómo reemplazar o borrar entradas.

Al cambiar a una base de datos, la responsabilidad sigue siendo la misma —acceder a los datos—, pero el mecanismo de almacenamiento es distinto.

### 2. Después: una interfaz de Spring Data JPA

Elimina `src/main/java/com/example/petmatchlab/repository/InMemoryPetRepository.java` de este proyecto, después de comprobar que conservas una copia del trabajo previo mediante Git.

Crea `src/main/java/com/example/petmatchlab/repository/PetRepository.java`:

```java
package com.example.petmatchlab.repository;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.petmatchlab.model.Pet;

public interface PetRepository extends JpaRepository<Pet, Long> {
}
```

Este es el **archivo completo**. Observa cuánto código desaparece en comparación con el `Map`.

**Preguntas importantes:**

- ¿Dónde está la clase que implementa los métodos de esta interfaz?
- ¿Por qué no hemos escrito `SELECT` o `INSERT` en Java?
- ¿Quién se encarga ahora de obtener el siguiente identificador?

Spring Data JPA reconoce la interfaz de repositorio y genera la implementación que necesita la aplicación. Esa implementación delega en JPA/Hibernate las operaciones de persistencia.

No necesitamos crear un archivo llamado `PetRepositoryImpl.java` para las operaciones CRUD habituales. Tampoco necesitamos incluir `@Repository` en esta interfaz para que Spring Data la detecte cuando se encuentra bajo los paquetes escaneados correctamente.

### 3. Entender los tipos genéricos

Lee:

```java
JpaRepository<Pet, Long>
```

Son dos parámetros de tipo:

| Parámetro | Valor | Significado |
|---|---|---|
| Primer tipo | `Pet` | Tipo de entidad que manejará el repositorio |
| Segundo tipo | `Long` | Tipo de la propiedad identificadora de esa entidad |

Ya utilizaste genéricos con `List<Pet>`, `Map<Long, Pet>` y `Optional<Pet>`. La idea es la misma: indicar al compilador con qué tipos trabajará la estructura o interfaz.

Si escribieras `JpaRepository<Pet, String>` mientras `Pet.id` es `Long`, introducirías una contradicción en el contrato del repositorio.

### 4. Los métodos ya disponibles

La interfaz hereda métodos para las operaciones habituales:

```java
petRepository.findAll();
petRepository.findById(7L);
petRepository.save(pet);
petRepository.delete(pet);
petRepository.existsById(7L);
petRepository.count();
```

Aquí `petRepository` representa un objeto que el servicio recibirá mediante inyección de dependencias. El fragmento anterior es **ilustrativo**: no debes pegar esas seis líneas en una clase sin definir antes el objeto y la operación en la que se usarán.

| Método | Resultado o efecto |
|---|---|
| `findAll()` | Obtiene entidades persistidas |
| `findById(id)` | Devuelve `Optional<Pet>` |
| `save(pet)` | Guarda una entidad nueva o actualiza una existente, según su estado |
| `delete(pet)` | Solicita eliminar la entidad |
| `existsById(id)` | Comprueba existencia mediante un booleano |
| `count()` | Obtiene el número de registros |

`findAll()` no garantiza por sí solo el orden de las filas. Si la interfaz debe mostrar un orden específico, el programa debe solicitarlo explícitamente. Lo haremos ordenando por `id`.

### 5. `Optional<Pet>` sigue siendo útil

```java
Optional<Pet> result = petRepository.findById(7L);
```

El tipo representa dos posibilidades:

```text
Hay una fila con id=7
    → Optional contiene un Pet

No hay una fila con id=7
    → Optional está vacío
```

El repositorio **no debería fabricar una mascota ficticia** cuando el identificador no existe. La capa superior decide qué hacer con la ausencia.

El controlador ya aprendió a convertir ese resultado en una respuesta HTTP `404 Not Found`. No necesitas cambiar esa responsabilidad al introducir MariaDB.

### 6. ¿Qué significa realmente `save(...)`?

En el repositorio en memoria existían dos métodos diferentes:

```text
create(...)
update(id, ...)
```

En Spring Data JPA utilizaremos `save(...)` tanto al crear como al actualizar, pero el objeto llega en estados diferentes.

**Caso A: entidad nueva**

```java
Pet newPet = new Pet("Kira", "Dog", 4, null);
Pet savedPet = petRepository.save(newPet);
```

El objeto no tiene un identificador persistente. Spring Data JPA lo trata como una entidad nueva y solicita a JPA que la persista. MariaDB genera el identificador.

**Caso B: entidad existente**

```java
Pet existingPet = petRepository.findById(7L).orElseThrow();
existingPet.updateDetails("Kira", "Dog", 5, null);
Pet savedPet = petRepository.save(existingPet);
```

El objeto procede de una consulta de persistencia y representa un registro existente. La operación `save(...)` permite que JPA guarde los cambios correspondientes.

Estos fragmentos **no** son métodos completos del servicio; son ejemplos para comparar estados. No necesitas copiarlos todavía. En la práctica siguiente escribirás `PetService` completo.

### 7. Una introducción a los estados de una entidad

Para interpretar el comportamiento de JPA, reconoce estas tres situaciones:

| Estado conceptual | Idea principal |
|---|---|
| Nueva (*transient*) | El objeto Java se creó, pero todavía no se ha guardado |
| Administrada (*managed*) | Un contexto de persistencia de JPA está siguiendo su estado |
| Separada (*detached*) | El objeto representa datos persistidos, pero ya no pertenece a un contexto de persistencia activo |

No confundas «entidad existente» con «entidad siempre administrada»: una misma entidad puede pasar por distintos estados durante el procesamiento.

Un **contexto de persistencia** es el mecanismo mediante el cual JPA mantiene y sigue entidades durante una unidad de trabajo. Hibernate puede detectar cambios de entidades administradas, pero no debes concluir que modificar cualquier objeto Java ejecuta SQL automáticamente en cualquier lugar del programa.

Por eso el servicio que construiremos llamará a `save(...)` de forma explícita después de modificar una mascota. Más adelante será posible profundizar en los límites transaccionales y en el seguimiento automático de cambios.

### 8. Mantener la dirección de las dependencias

La arquitectura quedará así:

```text
PetController
    |
    v
PetService
    |
    v
PetRepository (interfaz)
    |
    v
Spring Data JPA / Hibernate
    |
    v
MariaDB
```

**El controlador no necesita importar `PetRepository`.** Ya depende de `PetService` y debe continuar haciéndolo.

Tampoco hace falta que `PetForm` conozca JPA. Un formulario describe los datos de entrada; una entidad describe el estado que se almacenará. Aunque tengan propiedades parecidas, cumplen funciones diferentes.

### 9. ¿Qué cambia y qué no cambia?

| Archivo | Cambio previsto | Motivo |
|---|---|---|
| `Pet.java` | Sí | Mapear el objeto como entidad JPA |
| `PetRepository.java` | Nuevo | Sustituir el acceso en memoria por persistencia |
| `InMemoryPetRepository.java` | Se elimina | Ya no se necesita el `Map` |
| `PetService.java` | Sí | Llamar a los métodos de Spring Data JPA |
| `PetController.java` | No | Ya utiliza operaciones de `PetService` |
| `PetForm.java` | No | La entrada de datos no depende del almacenamiento |
| `pets/form.html` | No | El usuario sigue enviando los mismos datos |
| `pets/list.html` | No | Thymeleaf sigue recibiendo una lista de mascotas |
| `pets/detail.html` | No | Los getters de `Pet` continúan disponibles |

**Esta es la ventaja concreta de haber separado responsabilidades:** al sustituir el mecanismo de acceso a datos, no es necesario reconstruir toda la aplicación web.

### Ejercicio — Identifica el origen de cada comportamiento

Relaciona cada afirmación con `PetRepository`, `PetService`, `PetController` o MariaDB:

1. Decide mostrar la página `pets/form` si hay errores de validación.
2. Ofrece el método heredado `findById(...)`.
3. Genera el valor de una columna configurada con `AUTO_INCREMENT`.
4. Elimina espacios al inicio y final del nombre antes de persistirlo.
5. Convierte la ausencia de un registro en un error HTTP `404`.
6. Ejecuta finalmente la operación de almacenamiento solicitada por Hibernate.

### Depuración — Cambios incompletos

Si sustituyes `InMemoryPetRepository` por `PetRepository` pero **no actualizas** `PetService`, el proyecto puede fallar al compilar porque importa una clase que ya no existe. No es un fallo de MariaDB: es un **cambio incompleto en las referencias Java**.

Si conservas el viejo repositorio en memoria y creas el nuevo, puede que la aplicación compile, pero seguirá existiendo código innecesario y será difícil interpretar los resultados. Mantén un solo mecanismo activo para el catálogo.

### Comprobación

Puedes explicar por qué Spring Data JPA implementa una interfaz que no escribiste manualmente, de dónde provienen `findById`, `save` y `delete`, y por qué las vistas y los controladores no tienen que conocer el motor de almacenamiento.

---
## Práctica 9.5 — Persistir el CRUD y comprobar los cambios en MariaDB

### Objetivos

Al finalizar podrás:

- adaptar el servicio para utilizar `PetRepository`;
- conservar la normalización y las reglas de entrada existentes;
- registrar entidades nuevas con identificadores generados;
- modificar y eliminar registros persistidos;
- obtener listas ordenadas desde el repositorio;
- comprobar cada operación mediante SQL;
- demostrar que los datos sobreviven a un reinicio.

### 1. Revisar el contrato del servicio

El controlador que construiste anteriormente utiliza estas operaciones:

```text
PetService.findAll()
PetService.findById(id)
PetService.create(form)
PetService.update(id, form)
PetService.delete(id)
```

Para conservar la aplicación web, mantendremos exactamente esos métodos y sus tipos de resultado:

| Operación | Resultado Java |
|---|---|
| `findAll()` | `List<Pet>` |
| `findById(long id)` | `Optional<Pet>` |
| `create(PetForm form)` | `Pet` |
| `update(long id, PetForm form)` | `Optional<Pet>` |
| `delete(long id)` | `boolean` |

Esto permite que el controlador siga gestionando las respuestas HTTP de la misma manera.

**El cambio no consiste en poner consultas SQL dentro de `PetController`.** Consiste en adaptar la implementación del servicio para delegar las operaciones de datos a Spring Data JPA.

### 2. Reemplazar `service/PetService.java`

Sustituye por completo `src/main/java/com/example/petmatchlab/service/PetService.java`:

```java
package com.example.petmatchlab.service;

import java.util.List;
import java.util.Optional;

import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;

import com.example.petmatchlab.dto.PetForm;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.repository.PetRepository;

@Service
public class PetService {

    private final PetRepository petRepository;

    public PetService(PetRepository petRepository) {
        this.petRepository = petRepository;
    }

    public List<Pet> findAll() {
        return petRepository.findAll(Sort.by("id").ascending());
    }

    public Optional<Pet> findById(long id) {
        return petRepository.findById(id);
    }

    public Pet create(PetForm form) {
        validateForm(form);

        Pet pet = new Pet(
            form.getName().strip(),
            form.getSpecies().strip(),
            form.getAge(),
            normalizeDescription(form.getDescription())
        );

        return petRepository.save(pet);
    }

    public Optional<Pet> update(long id, PetForm form) {
        validateForm(form);

        return petRepository.findById(id).map(pet -> {
            pet.updateDetails(
                form.getName().strip(),
                form.getSpecies().strip(),
                form.getAge(),
                normalizeDescription(form.getDescription())
            );

            return petRepository.save(pet);
        });
    }

    public boolean delete(long id) {
        return petRepository.findById(id).map(pet -> {
            petRepository.delete(pet);
            return true;
        }).orElse(false);
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

Es un **archivo completo**. Comparándolo con el servicio anterior, observa que se conservaron las reglas de validación y normalización. Lo que cambió fue la dependencia y la manera de guardar y recuperar las entidades.

### 3. Seguir una consulta

```java
public List<Pet> findAll() {
    return petRepository.findAll(Sort.by("id").ascending());
}
```

El servicio pide una lista de entidades, ordenadas por su identificador.

`Sort.by("id").ascending()` expresa un criterio de ordenación. En una consulta SQL equivalente esperarías algo como:

```sql
SELECT id, name, species, age, description
FROM pets
ORDER BY id ASC;
```

La sentencia que Hibernate genere puede tener otra presentación, otros alias o un orden de columnas diferente. **No es necesario que el texto SQL sea idéntico para que el comportamiento sea correcto.**

Cuando el controlador ejecuta:

```java
model.addAttribute("pets", petService.findAll());
```

Thymeleaf recibe una lista de objetos `Pet`, igual que antes. **No recibe filas SQL ni un `ResultSet`.**

### 4. Seguir una creación

El servicio crea un objeto Java:

```java
Pet pet = new Pet(
    form.getName().strip(),
    form.getSpecies().strip(),
    form.getAge(),
    normalizeDescription(form.getDescription())
);
```

En ese instante el identificador todavía es `null`. Después solicita:

```java
return petRepository.save(pet);
```

La operación se traduce, mediante JPA/Hibernate, a una inserción en la base de datos. MariaDB asigna el identificador, y el objeto devuelto por `save(...)` permite al controlador construir la redirección:

```text
redirect:/pets/{id}
```

El usuario no escribió el identificador en el formulario. El código Java tampoco mantuvo una variable `nextId`.

**Predice:** si eliminas una mascota con id `7` y registras otra, ¿deberías suponer que MariaDB reutilizará automáticamente el `7`? No. Los identificadores generados no deben utilizarse como contadores de elementos ni como una secuencia sin saltos.

### 5. Seguir una actualización

El método primero busca el registro:

```java
return petRepository.findById(id).map(pet -> {
    // ...
});
```

`map(...)` de `Optional` ejecuta la función solamente si contiene un valor. Si el identificador no existe, devuelve un `Optional` vacío.

Cuando sí existe una mascota, modifica sus atributos editables:

```java
pet.updateDetails(
    form.getName().strip(),
    form.getSpecies().strip(),
    form.getAge(),
    normalizeDescription(form.getDescription())
);
```

Después guarda el resultado:

```java
return petRepository.save(pet);
```

¿Por qué hacemos una llamada explícita a `save(...)`? En esta implementación el método de servicio no define una transacción que abarque toda la lectura y modificación. `save(...)` entrega los cambios a la operación de persistencia del repositorio. **No debes depender de que modificar un objeto Java aislado siempre actualice una fila.**

La distinción entre una entidad administrada y otra separada del contexto ayuda a interpretar este comportamiento.

### 6. Seguir una eliminación

```java
public boolean delete(long id) {
    return petRepository.findById(id).map(pet -> {
        petRepository.delete(pet);
        return true;
    }).orElse(false);
}
```

Este código distingue dos situaciones:

```text
id existente
    → encontrar entidad
    → eliminar entidad
    → devolver true

id inexistente
    → Optional vacío
    → devolver false
```

El controlador ya convierte `false` en una respuesta HTTP `404` y, cuando la eliminación tiene éxito, redirige al catálogo.

**Límite del ejemplo:** aquí se presenta una operación simple de una entidad. Si en una aplicación hay varias escrituras dependientes, o modificaciones simultáneas que deben coordinarse, hacen falta reglas y límites transaccionales explícitos. No confundas el CRUD introductorio con una solución completa a todos los problemas de concurrencia.

### 7. ¿Qué sucede con el controlador y los formularios?

Abre `PetController.java` y revisa sus imports. Debe importar `PetService`, `PetForm` y `Pet`, pero no `InMemoryPetRepository`.

Si conservaste el controlador completo de la práctica anterior, **no necesitas cambiarlo**. Continúan existiendo las mismas rutas:

| Método | Ruta | Acción |
|---|---|---|
| GET | `/pets` | Listar |
| GET | `/pets/new` | Mostrar formulario de creación |
| POST | `/pets` | Registrar |
| GET | `/pets/{id}` | Ver detalle |
| GET | `/pets/{id}/edit` | Mostrar formulario de edición |
| POST | `/pets/{id}` | Actualizar |
| POST | `/pets/{id}/delete` | Eliminar |

Tampoco necesitas modificar `PetForm.java`: sus anotaciones de validación continúan protegiendo las entradas que llegan mediante formularios.

La principal comprobación arquitectónica es esta: **el catálogo cambia de almacenamiento sin que el navegador tenga que aprender nuevas rutas**.

### 8. Iniciar por primera vez la aplicación conectada a MariaDB

Antes de ejecutar, verifica lo siguiente:

- MariaDB está encendido y acepta conexiones de `petmatch_user`.
- Existe la base de datos `petmatch_lab`.
- El `pom.xml` contiene `spring-boot-starter-data-jpa` y `mariadb-java-client`.
- Las variables `DB_URL`, `DB_USERNAME` y `DB_PASSWORD` están definidas en la terminal donde ejecutarás Java.
- `Pet.java` es la entidad JPA completa.
- `PetRepository.java` existe y `PetService.java` utiliza esa interfaz.
- El código principal ya no hace referencia a `InMemoryPetRepository`.

Desde la raíz del proyecto inicia la aplicación.

**Linux o macOS:**

```bash
./mvnw spring-boot:run
```

**Windows PowerShell:**

```powershell
.\mvnw.cmd spring-boot:run
```

En el primer arranque Maven podría necesitar obtener bibliotecas nuevas. Spring Boot intentará configurar la conexión y Hibernate inspeccionará las entidades.

Si la base de datos está vacía y `ddl-auto: update` está activado, Hibernate normalmente creará la tabla correspondiente al mapeo de `Pet`. No interpretes un arranque fallido como una razón para borrar toda la base: **lee primero la causa del error**.

### 9. Comprobar que la tabla existe

Sin detener la aplicación, abre otra terminal y conecta el cliente MariaDB como el usuario de la aplicación:

```bash
mariadb --protocol=TCP -h 127.0.0.1 -P 3306 -u petmatch_user -p petmatch_lab
```

Dentro del cliente:

```sql
SHOW TABLES;
```

Busca `pets` entre los resultados. Examina las columnas:

```sql
DESCRIBE pets;
```

También puedes consultar la definición de la tabla:

```sql
SHOW CREATE TABLE pets;
```

Localiza los nombres de columna, tipos, clave primaria y la estrategia de generación automática del identificador.

**No necesitas obtener exactamente la misma salida textual que aparece en otro computador:** la representación del DDL puede variar entre versiones del motor, dialectos y detalles de configuración. Comprueba los conceptos relevantes.

### 10. ¿Por qué el catálogo aparece inicialmente vacío?

En el repositorio en memoria tenías algo parecido a:

```java
pets.put(1L, new Pet(...));
```

Esos objetos se reconstruían al iniciar Java. **No fueron migrados a MariaDB automáticamente.**

La nueva tabla empieza sin registros si acabas de crearla. Por eso:

```sql
SELECT id, name, species, age, description
FROM pets
ORDER BY id;
```

puede devolver cero filas. Eso es correcto.

No recrees el antiguo constructor con `pets.put(...)` dentro de la entidad ni del repositorio JPA. Para observar la persistencia, empezaremos creando mascotas **desde el formulario web existente**.

### 11. Prueba de creación y verificación SQL

Abre:

```text
http://localhost:8080/pets/new
```

Registra una mascota de prueba con estos valores ilustrativos:

```text
Name: Kira
Species: Dog
Age: 4
Description: Very playful.
```

Los nombres visibles de los campos pueden estar en español; lo importante es que correspondan a las propiedades `name`, `species`, `age` y `description`.

Después del envío válido, la aplicación debe redirigir al detalle de la mascota recién creada. **Anota el identificador real que aparece en la dirección. No supongas que será `1`.**

En el cliente MariaDB ejecuta:

```sql
SELECT id, name, species, age, description
FROM pets
ORDER BY id;
```

Busca una fila con los valores introducidos. Comprueba que el `id` corresponde a la dirección del detalle.

**¿Qué has verificado?** No solo que el navegador muestra una página, sino que el dato quedó registrado en el sistema de almacenamiento.

### 12. Prueba de edición y verificación SQL

Desde el detalle, abre el formulario de edición de Kira. Cambia su edad a `5` y guarda.

La dirección del detalle debe conservar el mismo identificador. En MariaDB consulta:

```sql
SELECT id, name, age
FROM pets
WHERE name = 'Kira'
ORDER BY id;
```

Localiza la fila que corresponde al identificador anotado y comprueba la nueva edad. Si existen varias mascotas llamadas Kira, el **identificador** permite distinguir el registro correcto.

También puedes realizar una consulta dirigida sustituyendo `<pet_id>` por el número que observaste (sin escribir los símbolos `<` y `>`):

```sql
SELECT id, name, species, age
FROM pets
WHERE id = 7;
```

En esta última instrucción, `7` es un **ejemplo**; reemplázalo por el valor real. La sentencia solo consulta: no modifica registros.

### 13. Prueba de eliminación y verificación SQL

Registra una **segunda mascota de prueba** y anota su identificador. Después elimínala desde el botón correspondiente en la interfaz.

Consulta:

```sql
SELECT id, name, species, age
FROM pets
ORDER BY id;
```

La segunda mascota ya no debe aparecer. La primera, que no fue eliminada, debe mantenerse.

Prueba también:

```text
http://localhost:8080/pets/{id-eliminado}
```

Sustituye el marcador por el número real. La aplicación debe responder con un estado HTTP `404`, porque ese recurso ya no existe.

Esto demuestra que la operación de eliminación no se limitó a ocultar un elemento de Thymeleaf.

### 14. La prueba fundamental: detener y reiniciar

Ahora detén Spring Boot mediante **Ctrl+C**. No detengas MariaDB ni elimines su base de datos.

Con el proceso Java detenido, puedes ejecutar desde el cliente MariaDB:

```sql
SELECT id, name, species, age, description
FROM pets
ORDER BY id;
```

La mascota que no eliminaste debería seguir apareciendo.

Inicia de nuevo Spring Boot desde la terminal que tiene definidas las variables de entorno:

**Linux o macOS:**

```bash
./mvnw spring-boot:run
```

**Windows PowerShell:**

```powershell
.\mvnw.cmd spring-boot:run
```

Visita el catálogo y el detalle de la mascota conservada.

**Resultado esperado:** sus datos siguen presentes y el identificador sigue siendo el mismo. Esta es la diferencia observable entre un repositorio en memoria y un repositorio persistente.

### 15. Leer el SQL generado por Hibernate

El archivo `application.yaml` activa:

```yaml
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```

Cuando registras, consultas, modificas o eliminas una mascota, revisa la salida de Spring Boot. Podrás encontrar sentencias conceptualmente equivalentes a:

```sql
INSERT INTO pets (name, species, age, description) VALUES (?, ?, ?, ?);
SELECT id, name, species, age, description FROM pets WHERE id = ?;
UPDATE pets SET name = ?, species = ?, age = ?, description = ? WHERE id = ?;
DELETE FROM pets WHERE id = ?;
```

Los signos `?` representan parámetros de la consulta preparada. No significa que se insertará literalmente un signo de interrogación en la tabla.

**Advertencia:** esas cuatro sentencias son ejemplos simplificados, no una transcripción garantizada del SQL exacto generado por Hibernate. El ORM puede decidir el orden de columnas, utilizar alias y añadir condiciones según el mapeo y la versión utilizada.

No confundas `show-sql` con un mecanismo de auditoría. En un entorno real, el registro de consultas debe configurarse con cuidado para no exponer datos sensibles ni generar salidas innecesarias.

### 16. Consultar sin modificar: otras instrucciones SQL útiles

Una vez tengas varios registros de prueba, ejecuta:

```sql
SELECT COUNT(*) AS total_pets
FROM pets;
```

Esto devuelve el número de filas registradas.

Para consultar las especies disponibles:

```sql
SELECT DISTINCT species
FROM pets
ORDER BY species;
```

Para obtener mascotas de al menos tres años:

```sql
SELECT id, name, age
FROM pets
WHERE age >= 3
ORDER BY age DESC, name ASC;
```

Para buscar un texto en el nombre:

```sql
SELECT id, name, species
FROM pets
WHERE name LIKE '%a%'
ORDER BY name;
```

La interpretación de mayúsculas y minúsculas en algunas comparaciones depende de la colación configurada en MariaDB. No supongas que todas las instalaciones comparan texto exactamente igual.

### 17. Comparar la consulta SQL con el catálogo

El navegador obtiene `List<Pet>` desde el servicio, y la consulta SQL obtiene filas desde MariaDB.

Si los resultados no coinciden, investiga **dónde** aparece la diferencia:

```text
MariaDB contiene el registro
    └── ¿PetRepository lo recupera?
          └── ¿PetService lo devuelve?
                └── ¿PetController lo añade al Model?
                      └── ¿Thymeleaf lo presenta?
```

Esta secuencia evita modificar archivos al azar. Una fila que no existe en la base de datos no se arregla cambiando CSS. Una fila existente que no aparece en la vista requiere seguir el recorrido de lectura.

### Ejercicio — El identificador no es un contador

Registra tres mascotas y anota los identificadores asignados. Elimina una de ellas y registra otra.

Responde:

1. ¿Se conservan las mascotas que no eliminaste?
2. ¿El identificador nuevo tiene que ser el siguiente número sin saltos?
3. ¿Por qué no conviene mostrar «total de mascotas» utilizando el identificador más alto?
4. ¿Qué consulta SQL permite contar correctamente las filas actuales?
5. ¿Dónde debe decidirse qué mascota eliminar, en la vista o en el servicio y el repositorio?

### Reto — Incorporar una consulta derivada por especie

Spring Data JPA también puede crear consultas a partir del **nombre de un método**. Como ampliación, añade a `PetRepository`:

```java
List<Pet> findBySpeciesIgnoreCaseOrderByNameAsc(String species);
```

Para que la interfaz compile, importa `java.util.List` además de `JpaRepository`.

La expresión puede leerse en partes:

```text
findBy                 → buscar por
Species                → propiedad species
IgnoreCase             → ignorar diferencia entre mayúsculas/minúsculas
OrderByNameAsc         → ordenar por name ascendente
```

Esta declaración se basa en propiedades **reales** de la entidad Java. No depende de inventar manualmente una consulta SQL. La comparación de texto se delega al mecanismo generado y a la base de datos.

Agrega un método de servicio que llame al repositorio y ensaya su resultado con una prueba. Antes de ampliar la interfaz HTML, comprueba que el método devuelve únicamente los registros de la especie solicitada.

**Este reto es opcional.** El CRUD principal ya funciona con los métodos heredados de `JpaRepository`.

### Depuración — ¿En qué capa está el fallo?

| Síntoma | Qué revisar primero |
|---|---|
| El proyecto no compila porque falta `InMemoryPetRepository` | Imports y pruebas antiguas que todavía lo utilizan |
| Spring Boot no inicia por una excepción JDBC | Servidor, URL, puerto, cuenta, contraseña o permisos |
| Se inicia pero no existe la tabla `pets` | Anotaciones JPA, paquete de la entidad y configuración del esquema |
| El formulario muestra errores de edad | Validación y conversión de entrada; no necesariamente la base de datos |
| El catálogo está vacío al conectarse por primera vez | La base nueva todavía no contiene filas |
| Los datos desaparecen al reiniciar | Confirma que estás usando el repositorio JPA y la misma base de datos |
| Hay datos SQL pero no aparecen en HTML | Consulta del servicio, `Model` y renderizado Thymeleaf |
| Una edición no se guarda | Comprueba que se recuperó la entidad y se llamó a `save(...)` |
| La tabla contiene datos inesperados | Verifica `SELECT DATABASE()` y la URL del `DataSource` |

### Comprobación

Demuestra cuatro comportamientos desde el navegador y mediante SQL: **crear, consultar, editar y eliminar**. Después detén y reinicia Spring Boot y comprueba que los registros conservados permanecen en MariaDB. Explica qué clases cambiaron y por qué el controlador y las plantillas siguieron funcionando.

---
## Práctica 9.6 — Pruebas, depuración y reto integrador

### Objetivos

Al finalizar podrás:

- explicar por qué las pruebas anteriores del repositorio en memoria ya no representan el código actual;
- comprobar las operaciones JPA con una base de datos de prueba independiente;
- conservar las pruebas MVC sin alterar los datos de MariaDB utilizados durante el desarrollo;
- comprobar una regla del servicio mediante un repositorio simulado;
- distinguir una prueba unitaria de una prueba de integración;
- diagnosticar fallos de configuración, mapeo y persistencia;
- demostrar que la aplicación guarda información de forma permanente.

### 1. Las pruebas también deben seguir la arquitectura

El cambio de almacenamiento afecta a los archivos de prueba. Antes existían clases como:

```text
InMemoryPetRepositoryTests.java
PetServiceTests.java
PetControllerIntegrationTests.java
```

La primera clase construía un `InMemoryPetRepository` y comprobaba el comportamiento de su `Map`. Pero ese archivo de implementación ya no existe en el proyecto actual. Si dejas la prueba sin cambios, el compilador intentará importar una clase eliminada.

La segunda prueba también construía un servicio con el repositorio antiguo:

```java
new PetService(new InMemoryPetRepository());
```

Esa instrucción debe reemplazarse por una forma adecuada de aislar o preparar `PetRepository`.

La tercera clase utiliza `@SpringBootTest`, que crea un contexto de Spring. Ahora ese contexto incluye la configuración JPA y necesita una base de datos accesible, incluso cuando el objetivo de la prueba es una ruta HTML.

**No solucionaremos estos problemas eliminando todas las pruebas.** Los resolveremos adaptando el entorno de pruebas a la nueva arquitectura.

### 2. Dos preguntas diferentes exigen dos clases de prueba

Compara:

**Prueba unitaria:**

> Si el servicio recibe un formulario válido, ¿normaliza correctamente el nombre y llama al repositorio para guardar la entidad?

Podemos comprobarlo con un repositorio simulado, sin iniciar Spring Boot ni MariaDB.

**Prueba de integración JPA:**

> Si guardamos un `Pet` mediante `JpaRepository`, ¿podemos recuperarlo por su identificador utilizando una base de datos de prueba?

Aquí necesitamos infraestructura real de persistencia, aunque utilicemos una base temporal y aislada en vez de la base de desarrollo.

| Tipo | Qué intenta verificar | Qué necesita |
|---|---|---|
| Unitaria del servicio | Reglas y colaboración del servicio | `PetService` real y repositorio simulado |
| Integración JPA | Entidad, repositorio, ORM y base de datos | Contexto JPA y motor de base de datos de prueba |
| Integración MVC | Rutas, validación y respuestas | Infraestructura MVC y dependencias configuradas |
| Comprobación manual con MariaDB | Persistencia real entre ejecuciones | La aplicación y la base de datos MariaDB de desarrollo |

Ninguna prueba aislada demuestra absolutamente todo. Cada una ofrece evidencia sobre un límite distinto.

### 3. Añadir dependencias de prueba

Dentro de `<dependencies>` de `pom.xml`, **conservando todas las dependencias anteriores**, añade:

```xml
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

La primera proporciona el soporte de pruebas para Spring Data JPA. La segunda incorpora **H2**, una base de datos que puede ejecutarse dentro del proceso de prueba.

**¿Por qué utilizar H2 si el proyecto almacena datos en MariaDB?**

- Las pruebas del repositorio no deben borrar ni alterar accidentalmente las mascotas que registraste para comprobar la aplicación.
- Una base aislada permite preparar y descartar datos de prueba.
- Es útil para verificar el mapeo y las operaciones básicas sin exigir que MariaDB esté encendido durante cada ejecución de la suite.

**Límite importante:** H2 no es MariaDB. Su modo de compatibilidad ayuda a aproximar ciertos comportamientos, pero no reproduce todos los tipos, reglas SQL, dialectos ni particularidades del motor real. Por eso **se mantienen las comprobaciones manuales de persistencia con MariaDB**. Para una verificación más exigente se utilizan pruebas con el mismo motor de base de datos que la aplicación real.

`<scope>test</scope>` evita introducir H2 como dependencia del funcionamiento habitual de la aplicación. El programa continuará utilizando MariaDB al arrancar normalmente.

### 4. Crear una configuración separada para las pruebas

Crea la carpeta `src/test/resources` y dentro un archivo `application.yaml`:

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:petmatch_test;MODE=MariaDB;DB_CLOSE_DELAY=-1
    driver-class-name: org.h2.Driver
    username: sa
    password: ""
  jpa:
    hibernate:
      ddl-auto: create-drop
    open-in-view: false
    show-sql: false
```

Este archivo contiene configuración **exclusiva para las pruebas**, no para la aplicación que ejecutas normalmente.

**Compara los dos archivos:**

| Archivo | Base de datos | Comportamiento previsto |
|---|---|---|
| `src/main/resources/application.yaml` | MariaDB mediante variables de entorno | Conserva los registros |
| `src/test/resources/application.yaml` | H2 en memoria | Prepara tablas temporales para las pruebas |

Las herramientas de construcción habituales sitúan los recursos de prueba antes que los de la aplicación en el classpath durante la ejecución de pruebas. Así, el archivo de prueba proporciona los valores de conexión correspondientes.

¿Por qué aquí sí utilizamos `create-drop`? Porque la base H2 **es temporal y destinada únicamente a pruebas**. No debe utilizarse esa configuración contra la MariaDB que contiene datos que quieras conservar.

**No intercambies las rutas de estos dos archivos.** Si copias `create-drop` al archivo principal, estarás cambiando el comportamiento de la base de datos persistente.

### 5. Reemplazar la prueba del repositorio

Elimina `src/test/java/com/example/petmatchlab/repository/InMemoryPetRepositoryTests.java` después de comprobar que el nuevo repositorio está implementado.

Crea `src/test/java/com/example/petmatchlab/repository/PetRepositoryTests.java`:

```java
package com.example.petmatchlab.repository;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.data.jpa.test.autoconfigure.DataJpaTest;

import com.example.petmatchlab.model.Pet;

import jakarta.persistence.EntityManager;

@DataJpaTest
class PetRepositoryTests {

    @Autowired
    private PetRepository repository;

    @Autowired
    private EntityManager entityManager;

    @Test
    void shouldGenerateIdentifierWhenSavingNewPet() {
        Pet pet = new Pet("Kira", "Dog", 4, "Very playful.");

        Pet saved = repository.saveAndFlush(pet);

        assertNotNull(saved.getId());
        assertEquals("Kira", saved.getName());
    }

    @Test
    void shouldFindPersistedPetByIdentifier() {
        Pet saved = repository.saveAndFlush(
            new Pet("Luna", "Dog", 3, "Enjoys walks.")
        );
        Long id = saved.getId();
        entityManager.clear();

        Pet found = repository.findById(id).orElseThrow();

        assertEquals("Luna", found.getName());
        assertEquals(3, found.getAge());
    }

    @Test
    void shouldPersistChangesToExistingPet() {
        Pet saved = repository.saveAndFlush(
            new Pet("Milo", "Cat", 2, null)
        );
        Long id = saved.getId();

        saved.updateDetails("Milo", "Cat", 5, null);
        repository.saveAndFlush(saved);
        entityManager.clear();

        Pet found = repository.findById(id).orElseThrow();

        assertEquals(5, found.getAge());
    }

    @Test
    void shouldDeletePersistedPet() {
        Pet saved = repository.saveAndFlush(
            new Pet("Nube", "Rabbit", 1, null)
        );
        Long id = saved.getId();

        repository.delete(saved);
        repository.flush();
        entityManager.clear();

        assertFalse(repository.findById(id).isPresent());
    }

    @Test
    void shouldReturnEmptyWhenIdentifierDoesNotExist() {
        assertTrue(repository.findById(Long.MAX_VALUE).isEmpty());
    }
}
```

**Recorre una prueba paso a paso.**

```java
Pet saved = repository.saveAndFlush(
    new Pet("Luna", "Dog", 3, "Enjoys walks.")
);
Long id = saved.getId();
entityManager.clear();
Pet found = repository.findById(id).orElseThrow();
```

- `saveAndFlush(...)` guarda la entidad y solicita sincronizar los cambios pendientes con la base de datos de prueba.
- `getId()` recupera el identificador generado.
- `entityManager.clear()` separa las entidades administradas del contexto actual para que la siguiente recuperación no reutilice simplemente el mismo objeto administrado.
- `findById(id)` solicita buscar la entidad por su identificador.
- La aserción final comprueba un dato recuperado.

`@DataJpaTest` prepara una porción del contexto enfocada en JPA. Sus pruebas se ejecutan normalmente dentro de transacciones que **se revierten al terminar cada método**, de forma que cada caso pueda trabajar sin conservar accidentalmente sus cambios.

**Predice:** ¿por qué una prueba que guarda una mascota en H2 no demuestra por sí sola que esa mascota quedará registrada en tu MariaDB después de reiniciar Spring Boot?

### 6. Reemplazar las pruebas del servicio

No conserves una prueba que siga construyendo `new InMemoryPetRepository()`. En lugar de ese repositorio, utilizaremos **Mockito** para simular el comportamiento de `PetRepository`.

Reemplaza por completo `src/test/java/com/example/petmatchlab/service/PetServiceTests.java`:

```java
package com.example.petmatchlab.service;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;
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

import com.example.petmatchlab.dto.PetForm;
import com.example.petmatchlab.model.Pet;
import com.example.petmatchlab.repository.PetRepository;

@ExtendWith(MockitoExtension.class)
class PetServiceTests {

    @Mock
    private PetRepository repository;

    private PetService service;

    @BeforeEach
    void setUp() {
        service = new PetService(repository);
    }

    @Test
    void shouldNormalizeFieldsBeforeSaving() {
        PetForm form = newForm("  Kira  ", "  Dog  ", 4);
        form.setDescription("  Very playful.  ");
        when(repository.save(any(Pet.class)))
            .thenAnswer(invocation -> invocation.getArgument(0));

        Pet created = service.create(form);

        assertEquals("Kira", created.getName());
        assertEquals("Dog", created.getSpecies());
        assertEquals("Very playful.", created.getDescription());
        verify(repository).save(any(Pet.class));
    }

    @Test
    void shouldRejectInvalidAgeBeforeCallingRepository() {
        PetForm form = newForm("Kira", "Dog", -1);

        assertThrows(IllegalArgumentException.class, () -> service.create(form));

        verify(repository, never()).save(any(Pet.class));
    }

    @Test
    void shouldReturnEmptyWhenUpdatingMissingPet() {
        PetForm form = newForm("Kira", "Dog", 5);
        when(repository.findById(999L)).thenReturn(Optional.empty());

        assertTrue(service.update(999L, form).isEmpty());
        verify(repository, never()).save(any(Pet.class));
    }

    @Test
    void shouldUpdateExistingPetWithoutCreatingAnotherOne() {
        PetForm form = newForm("Milo", "Cat", 5);
        Pet existing = new Pet("Milo", "Cat", 2, null);
        when(repository.findById(7L)).thenReturn(Optional.of(existing));
        when(repository.save(any(Pet.class)))
            .thenAnswer(invocation -> invocation.getArgument(0));

        Pet updated = service.update(7L, form).orElseThrow();

        assertEquals(5, updated.getAge());
        verify(repository).save(existing);
    }

    @Test
    void shouldReturnFalseWhenDeletingMissingPet() {
        when(repository.findById(999L)).thenReturn(Optional.empty());

        assertFalse(service.delete(999L));
        verify(repository, never()).delete(any(Pet.class));
    }

    @Test
    void shouldDeleteExistingPet() {
        Pet existing = new Pet("Luna", "Dog", 3, null);
        when(repository.findById(8L)).thenReturn(Optional.of(existing));

        assertTrue(service.delete(8L));
        verify(repository).delete(existing);
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

El repositorio de estas pruebas es un **mock**, no una base de datos real. Permite indicar qué devolverá ante una búsqueda y verificar que el servicio realizó la operación esperada.

La diferencia conceptual es clara:

```text
Prueba unitaria de PetService
    PetService real
         |
         v
    PetRepository simulado

Prueba JPA de PetRepository
    PetRepository real
         |
         v
    Hibernate + H2
```

El método `thenAnswer(...)` del mock devuelve el mismo objeto `Pet` que recibió `save(...)`. Esto sirve para comprobar la normalización realizada por el servicio sin exigir que la base de datos asigne identificadores. **No reproduce todas las reglas de JPA ni reemplaza las pruebas del repositorio.**

### 7. Conservar las pruebas del controlador

El archivo `PetControllerIntegrationTests.java` construido anteriormente no necesita volver a importar `InMemoryPetRepository`. Como las rutas y contratos de `PetService` se conservaron, sus comprobaciones de formularios, errores y redirecciones siguen siendo pertinentes.

Revisa que la clase use:

```java
@SpringBootTest
@AutoConfigureMockMvc
class PetControllerIntegrationTests {
    // ...
}
```

Conserva el import correspondiente a Spring Boot 4:

```java
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
```

Ahora esa prueba utiliza la configuración de H2 definida en `src/test/resources/application.yaml`, de manera que las creaciones realizadas por los métodos de prueba no necesiten modificar la base MariaDB que utilizas durante el desarrollo.

**Importante:** las pruebas MVC con `@SpringBootTest` pueden compartir un contexto de Spring y, en consecuencia, datos temporales dentro de ese contexto. Evita escribir aserciones frágiles que supongan que el siguiente identificador será siempre `1L` o que la tabla estará vacía si otra prueba pudo insertar registros.

### 8. Ejecutar las pruebas

Desde la raíz del proyecto:

**Linux o macOS:**

```bash
./mvnw test
```

**Windows PowerShell:**

```powershell
.\mvnw.cmd test
```

La suite debería ejecutar las pruebas unitarias del servicio, las pruebas JPA del repositorio y las pruebas MVC existentes.

Si la suite falla:

1. Identifica el **nombre de la clase de prueba** que falló.
2. Distingue si fue un **error de compilación**, **error de creación del contexto** o una **aserción incorrecta**.
3. Verifica que no haya archivos de prueba que todavía importen `InMemoryPetRepository`.
4. Comprueba que `src/test/resources/application.yaml` existe en la ruta correcta.
5. Confirma las dependencias `spring-boot-starter-data-jpa-test` y `h2`.
6. Revisa el primer error significativo de la traza, no solamente la última línea.

**No utilices `-DskipTests` para ocultar un error.** Saltar las pruebas puede ser útil en circunstancias específicas de construcción, pero no demuestra que la aplicación funcione.

### 9. Entender qué demuestra cada comprobación

Supón que se presentan estas tres evidencias:

- `PetServiceTests` pasa con un repositorio simulado.
- `PetRepositoryTests` pasa utilizando Hibernate y H2.
- Los datos creados en MariaDB siguen allí al detener y reiniciar Spring Boot.

Estas evidencias responden preguntas distintas:

| Evidencia | Qué permite afirmar |
|---|---|
| Prueba unitaria | El servicio aplica las reglas comprobadas y utiliza correctamente el repositorio simulado |
| Prueba JPA | Los mapeos y operaciones comprobados funcionan en la base de prueba utilizada |
| Reinicio con MariaDB | Los registros verificados permanecen en el almacenamiento real utilizado por la aplicación |

No concluyas que toda la persistencia es infalible solo porque una consulta funcionó. Una aplicación real también debe considerar fallos de red, permisos, volumen de información, operaciones concurrentes y cambios de esquema.

### 10. Laboratorio de errores — Localiza la causa

Realiza estos diagnósticos **sobre configuraciones o archivos que puedas restaurar**, sin modificar ni eliminar datos importantes.

**Caso A — Variable ausente.**

Inicia la aplicación desde una terminal donde `DB_PASSWORD` no esté definida. Observa qué propiedad no puede resolverse y qué diferencia hay entre un valor ausente y una contraseña incorrecta.

**Caso B — Puerto equivocado.**

Configura una URL que señale un puerto en el que MariaDB no esté escuchando. Identifica el error de conexión y explica por qué no es un error de Thymeleaf.

**Caso C — Entidad sin identificador.**

En una copia de trabajo, elimina temporalmente `@Id` de `Pet.java`. ¿Por qué JPA no puede administrar correctamente una entidad sin identificar su clave primaria?

**Caso D — Una prueba vieja.**

Deja temporalmente un import hacia `InMemoryPetRepository` en una prueba. Ejecuta la compilación y explica por qué falla incluso si MariaDB está funcionando.

**Caso E — Dato inválido.**

Envía una mascota sin nombre o con edad negativa. ¿Debería llamarse al repositorio? ¿Cómo puedes comprobar desde SQL que no se creó ningún registro adicional?

**Caso F — Registro inexistente.**

Intenta consultar `/pets/99999999`. ¿Por qué debe producir una respuesta 404 en lugar de fabricar una entidad vacía?

**Caso G — Base de datos distinta.**

Si SQL muestra un registro que no aparece en la aplicación, comprueba que el cliente y `DB_URL` apuntan a la misma base de datos. ¿Por qué dos bases llamadas de forma similar pueden causar confusión aunque las dos funcionen correctamente?

Restaura las configuraciones al terminar los experimentos.

### 11. Reto integrador — Persistencia comprobable

Completa una secuencia de operaciones utilizando el navegador y verificando cada resultado mediante SQL:

1. Registrar **dos mascotas diferentes**.
2. Anotar sus identificadores reales.
3. Consultarlas desde sus direcciones de detalle.
4. Editar la edad de una de ellas.
5. Confirmar con SQL que cambió la fila correspondiente y no la otra.
6. Eliminar únicamente la segunda mascota.
7. Comprobar que el identificador eliminado devuelve un error HTTP 404.
8. Detener la aplicación sin detener MariaDB.
9. Consultar la tabla desde el cliente MariaDB.
10. Reiniciar la aplicación y verificar que la primera mascota conserva su identificador y sus datos modificados.
11. Ejecutar las pruebas y distinguir cuáles utilizan H2 y cuáles utilizan mocks.

No es suficiente enseñar la tarjeta de una mascota en HTML: **debe poder demostrarse el recorrido de la información hasta la base de datos y su recuperación posterior**.

### 12. Ampliación opcional — Buscar por especie

Si completaste el método derivado `findBySpeciesIgnoreCaseOrderByNameAsc(...)`, escribe una prueba de repositorio que:

1. Guarde una mascota con especie `Dog` y otra con especie `Cat`.
2. Consulte `Dog` mediante el método derivado.
3. Compruebe que solo se recuperan las mascotas de la especie solicitada.
4. Compruebe que la consulta respeta el criterio de ordenación.

No necesitas modificar la página de inicio ni crear nuevos controladores para demostrar que el repositorio conoce el nuevo método. Primero comprueba el comportamiento de persistencia y después considera si deseas ofrecerlo mediante la interfaz.

### 13. Diagnóstico final — Explica el recorrido completo

Responde mediante el código implementado:

**Situación A — Registrar.** Una persona completa el formulario de mascota y presiona Guardar.

- ¿Qué método HTTP envía el navegador?
- ¿Qué objeto recibe Spring MVC?
- ¿Dónde se validan los datos de entrada?
- ¿Dónde se eliminan los espacios innecesarios?
- ¿Dónde se construye `new Pet(...)`?
- ¿Quién llama a `save(...)`?
- ¿Quién genera el identificador?
- ¿Por qué el controlador puede redirigir al detalle sin escribir SQL?

**Situación B — Consultar.** Alguien visita `/pets/7`.

- ¿Qué clase recibe la ruta?
- ¿Qué método recupera el identificador desde la dirección?
- ¿Qué representa `Optional<Pet>`?
- ¿De dónde salen los atributos del objeto que recibe Thymeleaf?
- ¿Qué ocurre si el identificador no existe?

**Situación C — Editar.** Se modifica la edad de una mascota existente.

- ¿Por qué no se crea un objeto nuevo con un identificador arbitrario?
- ¿Qué hace `updateDetails(...)`?
- ¿Por qué el servicio llama a `save(...)`?
- ¿Cómo se verifica que la actualización llegó a MariaDB?

**Situación D — Reiniciar.** Se detiene y vuelve a iniciar la aplicación.

- ¿Qué objetos de Java dejaron de existir al detenerse el proceso?
- ¿Qué información permaneció en la base de datos?
- ¿Qué hace `findAll()` para reconstruir la lista que necesita la vista?
- ¿Por qué los identificadores no tienen que empezar de nuevo en `1`?

### Comprobación

Deberías poder explicar sin consultar una lista de anotaciones: **persistencia, tabla, fila, columna, clave primaria, SQL, JDBC, driver, DataSource, ORM, JPA, Hibernate, entidad, repositorio, `JpaRepository`, identificador generado y contexto de persistencia**.

Además, debes demostrar que el CRUD funciona sobre MariaDB y que los datos se conservan después de reiniciar Spring Boot.

---

## Mapa final del proyecto

Después de completar el laboratorio, los archivos principales quedarán organizados así:

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
    │   │   ├── dto/
    │   │   │   └── PetForm.java
    │   │   ├── model/
    │   │   │   └── Pet.java
    │   │   ├── repository/
    │   │   │   └── PetRepository.java
    │   │   └── service/
    │   │       └── PetService.java
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
    │               ├── detail.html
    │               └── form.html
    └── test/
        ├── java/com/example/petmatchlab/
        │   ├── controller/
        │   │   └── PetControllerIntegrationTests.java
        │   ├── repository/
        │   │   └── PetRepositoryTests.java
        │   └── service/
        │       └── PetServiceTests.java
        └── resources/
            └── application.yaml
```

La clase `InMemoryPetRepository` y su prueba correspondiente ya no aparecen. Se reemplazaron por `PetRepository` y `PetRepositoryTests`.

### Recorrido de una creación

```text
Navegador
    |
    | POST /pets
    v
PetController
    |
    | PetForm + @Valid + BindingResult
    v
PetService
    |
    | validar y normalizar
    | new Pet(name, species, age, description)
    v
PetRepository.save(pet)
    |
    v
Spring Data JPA / Hibernate
    |
    | JDBC + MariaDB Connector/J
    v
MariaDB
    |
    | INSERT INTO pets (...)
    | identificador generado
    v
PetController
    |
    | redirect:/pets/{id}
    v
Navegador
```

### Recorrido de una consulta

```text
Navegador
    |
    | GET /pets
    v
PetController
    |
    v
PetService.findAll()
    |
    v
PetRepository.findAll(Sort.by("id").ascending())
    |
    v
Hibernate + MariaDB
    |
    | resultados de SQL
    v
List<Pet>
    |
    v
Model de Spring MVC
    |
    v
Thymeleaf: pets/list.html
    |
    v
HTML presentado por el navegador
```

## Revisión del funcionamiento

Comprueba los resultados sin suponer que los identificadores comienzan en un número fijo:

| Comprobación | Resultado esperado |
|---|---|
| Conectar con el cliente MariaDB | La cuenta `petmatch_user` accede a `petmatch_lab` |
| Iniciar Spring Boot con variables de entorno | La aplicación establece una conexión válida |
| Consultar `SHOW TABLES` | Existe `pets` |
| Consultar `DESCRIBE pets` | Aparecen columnas correspondientes a la entidad |
| Abrir `/pets` con la base vacía | Se muestra correctamente el estado vacío |
| Registrar una mascota | Aparece una fila nueva en `pets` |
| Ver su detalle | Los datos coinciden con la fila consultada |
| Editarla | La fila conserva su identificador y cambia sus valores |
| Registrar con datos inválidos | Aparecen errores y no se inserta una fila |
| Eliminar una mascota | Su fila desaparece y la ruta del detalle responde 404 |
| Detener Spring Boot | MariaDB sigue conservando las filas no eliminadas |
| Reiniciar Spring Boot | El catálogo recupera las mascotas persistidas |
| Ejecutar la suite de pruebas | Las pruebas utilizan el entorno temporal y terminan correctamente |

## Errores frecuentes y cómo interpretarlos

### «No se puede conectar a la base de datos»

No significa automáticamente que Hibernate está mal implementado. Confirma primero que MariaDB está encendido, que la URL es correcta y que el cliente puede conectarse con la misma cuenta.

### «El usuario no tiene acceso»

Verifica el nombre de la cuenta, el host configurado, la contraseña y los permisos específicos sobre la base. Una cuenta `'petmatch_user'@'localhost'` no es necesariamente intercambiable en todos los entornos con `'petmatch_user'@'127.0.0.1'`.

### «No se puede resolver `${DB_PASSWORD}`»

Comprueba que la variable de entorno existe en la terminal o proceso desde el que se inicia Spring Boot. No soluciones el error copiando la contraseña al repositorio.

### «Not a managed type» o entidad no administrada

Revisa que `Pet` tenga `@Entity`, que las dependencias JPA estén disponibles y que la clase se encuentre dentro de los paquetes escaneados por la aplicación.

### «No identifier specified for entity»

Comprueba la presencia de `@Id` y su import desde `jakarta.persistence`. La clave primaria permite que JPA distinga entidades y filas.

### «Table 'pets' doesn't exist»

Revisa la base seleccionada, el mapeo `@Table(name = "pets")` y las propiedades de inicialización del esquema. No borres la base para intentar resolverlo sin entender la causa.

### «El programa inicia, pero faltan mascotas»

Recuerda que los tres objetos creados por el antiguo repositorio en memoria no se copiaron por sí solos a MariaDB. Comprueba qué registros están realmente almacenados en `pets`.

### «Después de crear, el id es nulo»

Si observas el objeto **antes** de llamar a `save(...)`, `null` es un resultado normal para una entidad nueva con identificador generado. Si continúa siendo nulo después de una operación de persistencia que se supone exitosa, revisa la entidad, el repositorio y la configuración del identificador.

### «Cambiar una propiedad no actualizó la fila»

Diferencia el estado de un objeto Java del estado guardado en MariaDB. En el servicio construido, la operación de actualización termina llamando expresamente a `petRepository.save(pet)`.

### «Las pruebas funcionan, pero la conexión real falla»

Las pruebas JPA de este laboratorio utilizan H2 y las pruebas unitarias utilizan mocks. Por tanto, una suite exitosa no garantiza que el usuario de MariaDB tenga permisos o que el puerto configurado sea correcto. Comprueba también el funcionamiento real.

## Cierre del laboratorio

Has transformado un CRUD que almacenaba objetos en memoria en una aplicación capaz de **persistir información en una base de datos relacional**. La interfaz se mantiene: el navegador utiliza las mismas rutas y formularios. Lo que cambió fue la representación de los datos y la manera de guardarlos y recuperarlos.

La idea central es entender el recorrido completo: **un formulario produce datos Java; el servicio valida y coordina; el repositorio utiliza JPA; Hibernate trabaja mediante JDBC; MariaDB almacena registros que pueden recuperarse después de reiniciar la aplicación**.

También has comprobado una propiedad importante de la separación de responsabilidades: cambiar el almacenamiento no obliga a reescribir cada controlador y cada plantilla HTML. Lo fundamental es mantener contratos claros entre componentes y verificar el comportamiento en cada límite.

## Referencias oficiales

- [Spring Boot — SQL Databases](https://docs.spring.io/spring-boot/reference/data/sql.html): configuración de `DataSource`, integración de JPA, generación de esquema y `open-in-view`.
- [Spring Boot — Database Initialization](https://docs.spring.io/spring-boot/how-to/data-initialization.html): valores de `spring.jpa.hibernate.ddl-auto` y precauciones con el esquema.
- [Spring Boot — Build Systems](https://docs.spring.io/spring-boot/reference/using/build-systems.html): dependencias y *starters*, incluidos los de JPA y pruebas.
- [Spring Data JPA — Core Concepts](https://docs.spring.io/spring-data/jpa/reference/repositories/core-concepts.html): operaciones heredadas por los repositorios.
- [Spring Data JPA — Persisting Entities](https://docs.spring.io/spring-data/jpa/reference/jpa/entity-persistence.html): estados de una entidad y funcionamiento de `save(...)`.
- [Spring Data JPA — Query Methods](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html): consultas derivadas a partir de nombres de métodos.
- [Spring Boot — Testing Spring Boot Applications](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html): `@SpringBootTest`, `@DataJpaTest` y bases de datos para pruebas.
- [Jakarta Persistence — Specification](https://jakarta.ee/specifications/persistence/): contratos de entidades, identificadores y persistencia.
- [Hibernate ORM — Documentation](https://hibernate.org/orm/documentation/): mapeo objeto-relacional y administración de entidades.
- [MariaDB — CREATE DATABASE](https://mariadb.com/docs/server/reference/sql-statements/data-definition/create/create-database): creación de bases de datos.
- [MariaDB — CREATE USER](https://mariadb.com/docs/server/reference/sql-statements/account-management-sql-statements/create-user): cuentas de base de datos.
- [MariaDB — GRANT](https://mariadb.com/docs/server/reference/sql-statements/account-management-sql-statements/grant): permisos sobre bases de datos y tablas.
- [MariaDB Connector/J — Official Guide](https://mariadb.com/docs/connectors/mariadb-connector-j/about-mariadb-connector-j): controlador JDBC y formato de URL para MariaDB.
- [MariaDB — SELECT](https://mariadb.com/docs/server/reference/sql-statements/data-manipulation/selecting-data/select): consultas con filtros y ordenación.
- [Java SE 21 — JDBC API](https://docs.oracle.com/en/java/javase/21/docs/api/java.sql/module-summary.html): interfaces estándar de acceso a bases de datos desde Java.
