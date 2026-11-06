# Laboratorio 11 — Registro, autenticación y autorización con Spring Security

Una aplicación puede gestionar mascotas, solicitudes de apoyo y estados persistentes y, sin embargo, tener un problema decisivo: **no saber quién está utilizando sus funciones**. Si cualquier visitante puede completar o cancelar solicitudes, modificar mascotas y eliminar registros, las reglas del negocio no bastan para proteger los datos. Antes de procesar una operación, la aplicación debe responder tres preguntas diferentes: quién intenta acceder, cómo demuestra su identidad y qué acciones tiene permitidas.

En este laboratorio ampliarás **PetMatch Mini**, la aplicación construida en los laboratorios anteriores, para incorporar **registro de usuarios, inicio y cierre de sesión, contraseñas almacenadas mediante funciones de resumen especializadas, roles, permisos de rutas y protección CSRF**. Seguirás trabajando con **Java 21, Spring Boot 4.1.1, Spring Security 7.1.2, Spring MVC, Thymeleaf, Spring Data JPA y MariaDB**. No crearás una API REST ni utilizarás JWT: en este laboratorio la interfaz es web y la autenticación utiliza **sesiones**.

La aplicación conservará `Pet`, `SupportRequest`, `SupportRequestService`, `PetService`, las relaciones JPA, las operaciones transaccionales y los formularios existentes. La seguridad se incorporará como una responsabilidad adicional, no como una reescritura del proyecto.

| Práctica | Aprendizaje principal | Resultado comprobable |
|---|---|---|
| 11.1 | Identidad, autenticación, autorización y amenazas básicas | Distinguir quién es un usuario de qué acciones puede ejecutar |
| 11.2 | Persistencia de cuentas, contraseñas y registro | Registrar cuentas `USER` sin almacenar contraseñas en texto plano |
| 11.3 | Spring Security, `UserDetailsService` y sesiones | Iniciar y cerrar sesión con usuarios almacenados en MariaDB |
| 11.4 | Reglas por rol y protección de operaciones existentes | Limitar cambios de mascotas y solicitudes a `ADMIN` |
| 11.5 | Thymeleaf, mensajes, CSRF y navegación | Utilizar formularios de seguridad y adaptar la interfaz |
| 11.6 | Pruebas, depuración y reto integrador | Demostrar respuestas `302`, `403`, autenticación y control CSRF |

El documento diferencia expresamente **archivos completos**, que deben crearse o sustituirse tal como se presentan, de **fragmentos**, que sirven para explicar una parte del código o modificar un archivo existente. Los nombres de paquetes, archivos, identificadores y comentarios de implementación están en inglés. Las explicaciones se presentan en español.

## Preparación — Identificar el punto de partida

### 1. Revisar las funcionalidades existentes

Comprueba que la aplicación permite:

- listar y consultar mascotas mediante `PetController`;
- crear, editar y eliminar mascotas mediante `PetService` y `PetRepository`;
- consultar y registrar solicitudes de apoyo con `SupportRequestController`;
- completar, cancelar y reemplazar solicitudes abiertas mediante `SupportRequestService`;
- persistir mascotas y solicitudes en las tablas `pets` y `support_requests`;
- ejecutar las pruebas disponibles sobre el dominio, los repositorios, los servicios y MVC.

El **Laboratorio 10** utiliza específicamente los nombres `SupportRequest`, `SupportRequestForm`, `SupportRequestRepository`, `SupportRequestController`, `SupportRequestService`, `SupportType` y `RequestStatus`. **Conserva esos nombres**: no los sustituyas por otras clases con responsabilidades parecidas.

### 2. Reconocer las rutas que vamos a proteger

El proyecto anterior ya tiene operaciones similares a las siguientes:

| Ruta | Método HTTP | Operación |
|---|---|---|
| `/pets` | GET | Consultar mascotas |
| `/pets/new` | GET | Mostrar formulario de creación de mascota |
| `/pets` | POST | Crear mascota |
| `/pets/{id}/edit` | GET | Mostrar edición de mascota |
| `/pets/{id}` | POST | Actualizar mascota |
| `/pets/{id}/delete` | POST | Eliminar mascota |
| `/pets/{petId}/requests` | GET | Consultar solicitudes de una mascota |
| `/pets/{petId}/requests/new` | GET | Mostrar formulario de solicitud |
| `/pets/{petId}/requests` | POST | Registrar solicitud |
| `/pets/{petId}/requests/{requestId}/complete` | POST | Completar solicitud |
| `/pets/{petId}/requests/{requestId}/cancel` | POST | Cancelar solicitud |
| `/pets/{petId}/requests/{requestId}/replace` | GET y POST | Reemplazar solicitud abierta |

Las rutas anteriores mantienen su comportamiento de negocio. Lo nuevo es que Spring Security decide **si una petición puede llegar a los controladores**.

### 3. Verificar configuración y dependencias

Conserva la configuración principal de MariaDB en `src/main/resources/application.yaml`:

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

server:
  port: 8080
```

Este es un ejemplo de las propiedades relevantes; conserva otras propiedades existentes si las utilizas. **No almacenes las credenciales en el repositorio ni cambies `ddl-auto` a `create-drop` en MariaDB**. La opción `update` facilita este ejercicio, pero no sustituye una herramienta de migración como Flyway o Liquibase en un proyecto mantenido a largo plazo.

Comprueba también que `src/test/resources/application.yaml` continúa utilizando **H2**, no la conexión de desarrollo a MariaDB. El aislamiento de las pruebas sigue siendo necesario al añadir cuentas de usuario.

### Diagnóstico inicial

Antes de continuar, explica: ¿qué diferencia hay entre estar registrado y haber iniciado sesión? ¿Una contraseña debe aparecer en la tabla `users` como texto legible? ¿Una persona autenticada puede eliminar automáticamente cualquier mascota? ¿Por qué ocultar un botón no impide enviar una petición HTTP? ¿Qué beneficio tiene mantener la regla de completar solicitudes dentro de `SupportRequestService` aunque exista Spring Security?

---

## Práctica 11.1 — Identificación, autenticación y autorización

### Objetivos

Al completar esta práctica podrás:

- distinguir identificación, autenticación, autorización y registro;
- explicar de manera inicial las funciones de las sesiones y las cookies;
- diferenciar una denegación por falta de autenticación de una denegación por permisos;
- describir por qué proteger botones no basta para proteger operaciones;
- proponer una política de permisos para las rutas ya construidas.

### 1. Cuatro conceptos que no deben confundirse

**Identificación** responde a la pregunta: *¿quién dices ser?* En este proyecto, la persona proporciona un correo electrónico, como `ana@example.com`.

**Autenticación** responde: *¿puedes demostrar que eres esa cuenta?* Para iniciar sesión, Spring Security verifica el correo y la contraseña con la cuenta almacenada.

**Autorización** responde: *¿qué funciones tienes permitido utilizar?* Una cuenta puede iniciar sesión correctamente y, aun así, no tener permiso para eliminar una mascota.

**Registro** es el proceso de crear una cuenta. Registrar una cuenta no implica que el navegador quede autenticado, a menos que la aplicación implemente expresamente esa operación adicional. En este laboratorio, después del registro se redirigirá al formulario de inicio de sesión.

```text
REGISTRO
  correo + contraseña + nombre
  ↓
  validar datos
  ↓
  guardar cuenta con password hash
  ↓
  mostrar acceso al inicio de sesión

INICIO DE SESIÓN
  correo + contraseña
  ↓
  Spring Security verifica credenciales
  ↓
  establece contexto autenticado en una sesión
  ↓
  el navegador puede utilizar rutas permitidas

AUTORIZACIÓN
  solicitud HTTP
  ↓
  ¿requiere autenticación?
  ↓
  ¿requiere algún rol concreto?
  ↓
  permitir o rechazar antes del controlador
```

### 2. ¿Qué es una sesión?

HTTP es un protocolo de intercambio de peticiones y respuestas. Cada petición contiene método, ruta, cabeceras y, en algunos casos, cuerpo. HTTP por sí solo no identifica automáticamente que dos peticiones provienen de una misma sesión autenticada.

En una aplicación web tradicional, tras un inicio de sesión correcto, el servidor puede asociar la autenticación a una **sesión HTTP**. El navegador conserva una cookie de sesión, frecuentemente llamada `JSESSIONID`, y la envía en peticiones posteriores al mismo sitio según sus atributos.

```text
Navegador                         Aplicación
   |                                  |
   | POST /login                      |
   |--------------------------------->|
   |             credenciales válidas |
   |<---------------------------------|
   |      cookie asociada a la sesión |
   |                                  |
   | GET /pets                        |
   |  cookie de sesión                |
   |--------------------------------->|
   |   Spring recupera autenticación  |
   |<---------------------------------|
   |        página autorizada         |
```

**La cookie no debe contener la contraseña.** Normalmente identifica una sesión mantenida en el servidor. Proteger esa cookie y la conexión es importante: por eso un despliegue real debe utilizar **HTTPS** y una configuración adecuada de cookies, expiración de sesión y proxy inverso.

### 3. ¿Qué diferencia existe entre `401`, `403` y una redirección?

- `401 Unauthorized` expresa que se requiere autenticación válida; es frecuente en APIs.
- `403 Forbidden` expresa que la solicitud está prohibida con los permisos actuales o que falla una protección como CSRF.
- `302 Found` es una redirección. En una web con formulario de acceso, un visitante que pide una ruta protegida generalmente será redirigido a `/login`.

No debes asumir que cada petición de una web con sesiones responde `401` cuando el usuario no ha iniciado sesión. **El comportamiento depende del mecanismo de entrada configurado**.

### 4. ¿Qué es un rol?

Representaremos dos roles:

| Rol | Responsabilidad en PetMatch Mini |
|---|---|
| `USER` | Consultar mascotas y solicitudes, y registrar solicitudes de apoyo |
| `ADMIN` | Además de consultar y registrar solicitudes, crear, editar y eliminar mascotas, y completar, cancelar o reemplazar solicitudes |

En Spring Security, `hasRole("ADMIN")` comprueba habitualmente una autoridad llamada **`ROLE_ADMIN`**. La cuenta almacena `ADMIN` como valor del enum; al autenticar se transforma en la autoridad `ROLE_ADMIN`.

La diferencia entre **rol** y **propiedad de un recurso** es esencial. Un usuario podría tener acceso a una ruta para consultar solicitudes, pero eso no prueba que deba acceder a información privada de cualquier otra persona. La aplicación actual **todavía no registra quién creó cada solicitud**, por lo que este laboratorio se limita de manera explícita a **permisos por rol**, sin afirmar que haya resuelto la propiedad (*ownership*) de los datos.

### 5. ¿Dónde debe realizarse el control de permisos?

No basta con escribir:

```html
<!-- Example only: hiding a button does not authorize an operation. -->
<button th:if="${isAdmin}">Eliminar</button>
```

Un cliente podría construir manualmente la petición `POST` aunque el botón esté oculto. Por eso protegeremos la **ruta en el servidor** mediante un `SecurityFilterChain`. Ocultar o mostrar botones es una mejora de experiencia, no un reemplazo del control de acceso.

Las reglas de negocio anteriores se conservan. `ADMIN` no puede completar una solicitud que ya está cancelada: la autorización puede permitirle llamar al servicio, pero el servicio todavía debe aplicar las reglas de estado.

### 6. Un primer análisis de amenazas

Piensa en cuatro errores comunes:

1. **Contraseñas visibles:** la base almacena la contraseña tal cual la escribió la persona.
2. **Elevación de privilegios:** el formulario de registro incluye un selector para elegir `ADMIN`.
3. **Acceso directo:** se oculta un botón, pero la ruta que ejecuta la acción no tiene protección.
4. **Petición forzada:** una página externa consigue que el navegador de alguien autenticado envíe una acción no deseada aprovechando su cookie de sesión.

Los tres primeros se abordarán con un diseño de cuentas, contraseñas y roles. El cuarto conduce a la protección **CSRF**, que estudiaremos en la práctica 11.5.

### Ejercicio — Diseña la política

Antes de programar, completa mentalmente la siguiente matriz:

| Operación | Visitante | `USER` | `ADMIN` |
|---|---|---|---|
| Abrir `/login` | Sí | Sí | Sí |
| Abrir `/register` | Sí | Sí | Sí |
| Abrir `/pets` | Redirección al login | Sí | Sí |
| Registrar mascota | No | No | Sí |
| Registrar solicitud de apoyo | No | Sí | Sí |
| Completar solicitud | No | No | Sí |
| Abrir `/admin` | No | No | Sí |

**Comprobación:** puedes separar registro, autenticación, autorización y regla de negocio sin utilizar las palabras como sinónimos.

---

## Práctica 11.2 — Crear cuentas y registrar usuarios de forma segura

### Objetivos

- agregar Spring Security sin eliminar dependencias existentes;
- modelar una cuenta de usuario como entidad JPA;
- validar formularios de registro y normalizar correos;
- almacenar un resumen de contraseña mediante `PasswordEncoder`;
- evitar que el registro público conceda privilegios administrativos.

### 1. Agregar dependencias a Maven

En `pom.xml`, dentro de `<dependencies>`, incorpora estos dos elementos **conservando todas las dependencias anteriores**:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security-test</artifactId>
    <scope>test</scope>
</dependency>
```

Los *starters* anteriores están disponibles en Spring Boot 4.1.1, que administra Spring Security 7.1.2. El segundo añade soporte para pruebas de autenticación, roles y CSRF. No es necesario escribir versiones explícitas cuando el `spring-boot-starter-parent` administra las dependencias.

**Comprueba que siguen presentes** `spring-boot-starter-webmvc`, `spring-boot-starter-thymeleaf`, `spring-boot-starter-validation`, `spring-boot-starter-data-jpa`, `mariadb-java-client`, `spring-boot-starter-webmvc-test`, `spring-boot-starter-data-jpa-test` y H2 en alcance de prueba.

> **Cambio importante al arrancar:** al colocar Spring Security en el classpath, Spring Boot protege las rutas por defecto. Durante la construcción de este laboratorio algunas páginas pueden redirigir a un login predeterminado hasta completar nuestra propia configuración. No desactives la seguridad para evitar esa situación.

### 2. Crear el rol como un tipo enumerado

Crea **`src/main/java/com/example/petmatchlab/model/UserRole.java`** — archivo completo:

```java
package com.example.petmatchlab.model;

public enum UserRole {
    USER,
    ADMIN
}
```

Una constante del enum representa un valor permitido en el dominio. No incluimos `ROLE_` dentro de la entidad; ese prefijo pertenece a la representación de autoridades que utiliza Spring Security.

### 3. Crear la entidad de cuenta

Crea **`src/main/java/com/example/petmatchlab/model/AppUser.java`** — archivo completo:

```java
package com.example.petmatchlab.model;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.EnumType;
import jakarta.persistence.Enumerated;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity
@Table(name = "app_users")
public class AppUser {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String displayName;

    @Column(nullable = false, unique = true, length = 190)
    private String email;

    @Column(nullable = false, length = 255)
    private String passwordHash;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private UserRole role;

    @Column(nullable = false)
    private boolean enabled;

    protected AppUser() {
    }

    public AppUser(String displayName, String email, String passwordHash) {
        this.displayName = displayName;
        this.email = email;
        this.passwordHash = passwordHash;
        this.role = UserRole.USER;
        this.enabled = true;
    }

    public Long getId() {
        return id;
    }

    public String getDisplayName() {
        return displayName;
    }

    public String getEmail() {
        return email;
    }

    public String getPasswordHash() {
        return passwordHash;
    }

    public UserRole getRole() {
        return role;
    }

    public boolean isEnabled() {
        return enabled;
    }
}
```

¿Por qué **`passwordHash`** en lugar de `password`? Porque el valor almacenado no será la contraseña original. Una función especializada produce una representación no reversible destinada a **verificar** contraseñas, no a recuperarlas.

¿Por qué `email` tiene `unique = true`? Porque el correo identifica la cuenta en el inicio de sesión. La aplicación debe impedir que dos cuentas se registren con el mismo correo normalizado. Además de una comprobación en el servicio, la base necesita imponer la unicidad frente a solicitudes concurrentes.

¿Por qué el constructor siempre asigna `UserRole.USER`? Porque el rol no debe recibirse de un formulario público. Un visitante puede elegir su nombre y contraseña, **no los privilegios de su cuenta**.

¿Por qué el campo `enabled`? Permite representar una cuenta deshabilitada. El adaptador de autenticación usará ese valor para impedir el acceso cuando corresponda. Este laboratorio no construye todavía una interfaz de administración de cuentas.

### 4. Crear el repositorio de usuarios

Crea **`src/main/java/com/example/petmatchlab/repository/AppUserRepository.java`** — archivo completo:

```java
package com.example.petmatchlab.repository;

import java.util.Optional;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.petmatchlab.model.AppUser;

public interface AppUserRepository extends JpaRepository<AppUser, Long> {

    Optional<AppUser> findByEmail(String email);

    boolean existsByEmail(String email);
}
```

Spring Data JPA implementa estas consultas a partir de las propiedades de `AppUser`. No necesitas escribir un DAO con JDBC ni crear `AppUserRepositoryImpl`.

### 5. Definir un formulario que no sea una entidad

Crea **`src/main/java/com/example/petmatchlab/dto/RegistrationForm.java`** — archivo completo:

```java
package com.example.petmatchlab.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public class RegistrationForm {

    @NotBlank(message = "A name is required")
    @Size(max = 100, message = "The name must have at most 100 characters")
    private String displayName;

    @NotBlank(message = "An email is required")
    @Email(message = "Enter a valid email address")
    @Size(max = 190, message = "The email must have at most 190 characters")
    private String email;

    @NotBlank(message = "A password is required")
    @Size(min = 12, max = 64,
          message = "The password must have between 12 and 64 characters")
    private String password;

    @NotBlank(message = "Confirm the password")
    private String confirmPassword;

    public String getDisplayName() {
        return displayName;
    }

    public void setDisplayName(String displayName) {
        this.displayName = displayName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getConfirmPassword() {
        return confirmPassword;
    }

    public void setConfirmPassword(String confirmPassword) {
        this.confirmPassword = confirmPassword;
    }
}
```

`@Email` hace una comprobación de formato, pero **no demuestra que el buzón exista ni que su propietario haya confirmado el correo**. La verificación por correo queda fuera de este laboratorio.

El límite de caracteres es una simplificación pedagógica. Las implementaciones de BCrypt tienen límites en **bytes** de contraseña y otras particularidades Unicode. En una aplicación real debes adoptar una política de contraseñas revisada, permitir gestores de contraseñas y evitar truncamientos silenciosos. Aquí utilizaremos el `PasswordEncoder` delegado de Spring Security, que permite evolucionar el algoritmo.

### 6. Crear una excepción específica

Crea **`src/main/java/com/example/petmatchlab/exception/DuplicateEmailException.java`** — archivo completo:

```java
package com.example.petmatchlab.exception;

public class DuplicateEmailException extends RuntimeException {

    public DuplicateEmailException() {
        super("An account with this email already exists");
    }
}
```

El controlador podrá convertirla en un mensaje de validación. No capturaremos cualquier excepción de base de datos como si se tratara necesariamente de un correo duplicado.

### 7. Comprender `PasswordEncoder`

No debes usar:

```java
// Incorrect: never store the submitted password directly.
String storedPassword = form.getPassword();
```

Tampoco utilices `Base64`, `SHA-256` sin un diseño especializado para contraseñas, ni un algoritmo casero como reemplazo de un codificador de contraseñas. La codificación Base64 **puede revertirse**. Una función general de hash rápida tampoco es una opción apropiada por sí sola para almacenar contraseñas.

Spring Security ofrece una interfaz `PasswordEncoder` con dos operaciones principales:

```java
String encoded = passwordEncoder.encode(rawPassword);
boolean matches = passwordEncoder.matches(rawPassword, encoded);
```

`encode` transforma la contraseña para almacenarla. `matches` verifica un intento de inicio de sesión. **No se descifra** el valor almacenado.

Utilizaremos `PasswordEncoderFactories.createDelegatingPasswordEncoder()`, que produce hashes con un prefijo de algoritmo, por ejemplo `{bcrypt}`. Ese prefijo permite identificar qué codificador corresponde a cada hash y facilita una futura migración de algoritmos.

### 8. Crear el servicio de registro

Crea **`src/main/java/com/example/petmatchlab/service/RegistrationService.java`** — archivo completo:

```java
package com.example.petmatchlab.service;

import java.util.Locale;
import java.util.Objects;

import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.petmatchlab.dto.RegistrationForm;
import com.example.petmatchlab.exception.DuplicateEmailException;
import com.example.petmatchlab.model.AppUser;
import com.example.petmatchlab.repository.AppUserRepository;

@Service
public class RegistrationService {

    private final AppUserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public RegistrationService(
        AppUserRepository userRepository,
        PasswordEncoder passwordEncoder
    ) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    @Transactional
    public AppUser register(RegistrationForm form) {
        validateForm(form);
        String email = normalizeEmail(form.getEmail());

        if (userRepository.existsByEmail(email)) {
            throw new DuplicateEmailException();
        }

        AppUser user = new AppUser(
            form.getDisplayName().strip(),
            email,
            passwordEncoder.encode(form.getPassword())
        );

        return userRepository.save(user);
    }

    private void validateForm(RegistrationForm form) {
        if (form == null || invalidText(form.getDisplayName(), 100)
            || invalidText(form.getEmail(), 190)
            || form.getPassword() == null
            || form.getPassword().length() < 12
            || form.getPassword().length() > 64
            || !Objects.equals(
                form.getPassword(), form.getConfirmPassword())) {
            throw new IllegalArgumentException("Invalid registration data");
        }
    }

    private boolean invalidText(String text, int maxLength) {
        return text == null || text.isBlank() || text.strip().length() > maxLength;
    }

    private String normalizeEmail(String email) {
        return email.strip().toLowerCase(Locale.ROOT);
    }
}
```

La clase mantiene separadas tres responsabilidades: el controlador validará entradas web y mostrará errores; `RegistrationService` comprobará el contrato del caso de uso y coordinará el registro; `AppUserRepository` almacenará la cuenta.

**La verificación `existsByEmail` no elimina por sí sola las carreras concurrentes.** Dos registros simultáneos podrían superar esa consulta. La restricción `UNIQUE` de MariaDB es la defensa definitiva contra duplicados. En una aplicación real, también se debe traducir cuidadosamente la violación de unicidad a un resultado de usuario y definir una política de errores que no oculte fallos de infraestructura.

### 9. Crear el controlador de registro

Crea **`src/main/java/com/example/petmatchlab/controller/RegistrationController.java`** — archivo completo:

```java
package com.example.petmatchlab.controller;

import jakarta.validation.Valid;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;

import com.example.petmatchlab.dto.RegistrationForm;
import com.example.petmatchlab.exception.DuplicateEmailException;
import com.example.petmatchlab.service.RegistrationService;

@Controller
public class RegistrationController {

    private final RegistrationService registrationService;

    public RegistrationController(RegistrationService registrationService) {
        this.registrationService = registrationService;
    }

    @GetMapping("/register")
    public String showForm(Model model) {
        model.addAttribute("registrationForm", new RegistrationForm());
        return "auth/register";
    }

    @PostMapping("/register")
    public String register(
        @Valid @ModelAttribute("registrationForm") RegistrationForm form,
        BindingResult bindingResult
    ) {
        if (form.getPassword() != null
            && !form.getPassword().equals(form.getConfirmPassword())) {
            bindingResult.rejectValue(
                "confirmPassword", "password.mismatch", "Passwords must match"
            );
        }

        if (bindingResult.hasErrors()) {
            return "auth/register";
        }

        try {
            registrationService.register(form);
        } catch (DuplicateEmailException exception) {
            bindingResult.rejectValue("email", "email.duplicate", exception.getMessage());
            return "auth/register";
        }

        return "redirect:/login?registered";
    }
}
```

En el caso de contraseña repetida de forma incorrecta, el controlador añade un error a `BindingResult` y conserva los datos del formulario. La contraseña **no se almacena hasta que todas las verificaciones necesarias hayan terminado**.

No incluimos `@PostMapping("/login")` aquí. El registro crea una cuenta, mientras que el procesamiento de las credenciales pertenece a Spring Security.

### 10. Inspeccionar los datos almacenados

Después de incorporar la configuración de seguridad y registrar una cuenta desde la interfaz, ejecuta:

```sql
SELECT id, display_name, email, role, enabled
FROM app_users
ORDER BY id;
```

Para comprobar que se almacenó un hash sin exponerlo completo, puedes consultar una representación parcial:

```sql
SELECT email, LEFT(password_hash, 8) AS hash_prefix,
       CHAR_LENGTH(password_hash) AS hash_length
FROM app_users;
```

El `password_hash` no debe coincidir con la contraseña escrita en el formulario. En el uso habitual del `DelegatingPasswordEncoder`, el prefijo puede ser `{bcrypt}`. **No imprimas contraseñas ni hashes completos en los logs del servidor**.

**Ejercicio:** explica por qué el rol se asigna dentro del constructor de `AppUser` y no se recibe desde el navegador. ¿Por qué necesitamos tanto `existsByEmail` como una restricción de unicidad en la base?

### Depuración — Errores de registro

- **`Not a managed type: AppUser`:** comprueba `@Entity` y el paquete `com.example.petmatchlab.model`.
- **La tabla `app_users` no aparece:** revisa la conexión activa, el mapeo y la configuración de inicialización de esquema.
- **`Duplicate entry` en MariaDB:** ya existe una fila con el mismo correo normalizado o hubo una carrera concurrente.
- **La contraseña parece legible en la base:** la aplicación no está utilizando correctamente `PasswordEncoder.encode(...)` antes de persistir.
- **Se genera una cuenta `ADMIN` desde el formulario público:** existe una elevación de privilegios y debes eliminar esa asignación.

**Comprobación:** puedes explicar cómo un formulario se transforma en una cuenta persistente `USER` sin guardar la contraseña original.

---
## Práctica 11.3 — Autenticación con Spring Security y sesiones

### Objetivos

Al completar esta práctica podrás:

- explicar por qué el formulario de acceso no debe comprobar contraseñas manualmente;
- adaptar las cuentas JPA al contrato `UserDetailsService`;
- configurar `SecurityFilterChain` y un formulario de inicio de sesión;
- reconocer dónde se establece la sesión y quién procesa `POST /login`;
- realizar el cierre de sesión utilizando el soporte de Spring Security.

### 1. Spring Security funciona antes del controlador MVC

Hasta ahora, una petición seguía un recorrido simplificado:

```text
Navegador → DispatcherServlet → Controller → Service → Repository
```

Con Spring Security, los filtros intervienen **antes** de que el controlador procese la petición:

```text
Navegador
   |
   v
Cadena de filtros de Spring Security
   |-- Protección CSRF
   |-- Autenticación
   |-- Recuperación del contexto de seguridad
   |-- Autorización de la ruta
   |
   v
DispatcherServlet
   |
   v
Controller → Service → Repository
```

No todos los filtros realizan las operaciones en el orden exacto de este diagrama conceptual; representa las **responsabilidades** principales, no la lista interna definitiva de filtros de la versión utilizada.

Si un visitante solicita una ruta protegida, Spring Security puede iniciar el proceso de autenticación sin que `PetController` tenga que consultar manualmente una cookie. Si alguien sin el rol requerido intenta ejecutar una operación, la cadena puede rechazarla antes de invocar el servicio.

### 2. ¿Quién verifica el correo y la contraseña?

Spring Security dispone de componentes para autenticar con usuario y contraseña. Uno de ellos, `DaoAuthenticationProvider`, trabaja con un `UserDetailsService` y un `PasswordEncoder`.

```text
POST /login
   ↓
Filtro de autenticación de formularios
   ↓
AuthenticationManager
   ↓
Proveedor de autenticación
   ├── UserDetailsService: encuentra la cuenta
   └── PasswordEncoder: comprueba la contraseña
   ↓
Authentication resultante
   ↓
Contexto de seguridad y sesión
```

La clase JPA `AppUser` no tiene que implementar `UserDetails`. Conservaremos separadas la entidad de persistencia y la representación utilizada por Spring Security.

### 3. Crear el adaptador `UserDetailsService`

Crea **`src/main/java/com/example/petmatchlab/security/DatabaseUserDetailsService.java`** — archivo completo:

```java
package com.example.petmatchlab.security;

import java.util.Locale;

import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import com.example.petmatchlab.model.AppUser;
import com.example.petmatchlab.repository.AppUserRepository;

@Service
public class DatabaseUserDetailsService implements UserDetailsService {

    private final AppUserRepository userRepository;

    public DatabaseUserDetailsService(AppUserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) {
        String email = username.strip().toLowerCase(Locale.ROOT);

        AppUser user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException(
                "User not found"
            ));

        return User.builder()
            .username(user.getEmail())
            .password(user.getPasswordHash())
            .authorities("ROLE_" + user.getRole().name())
            .disabled(!user.isEnabled())
            .build();
    }
}
```

El método se llama `loadUserByUsername` porque así se define el contrato de Spring Security. **No obliga a utilizar un nombre de usuario diferente del correo electrónico**: aquí el parámetro `username` contiene un email.

Observa que devuelve un `UserDetails` construido a partir de la entidad. Incluye la contraseña codificada, la autoridad del rol y el estado habilitado. **No devuelve el objeto `AppUser` directamente**, ni copia el formulario de registro a la sesión.

El import `org.springframework.security.core.userdetails.User` corresponde a una clase de Spring Security, no a la entidad `AppUser`. Son tipos distintos, con propósitos diferentes.

### 4. Configurar el codificador y la cadena de seguridad

Crea **`src/main/java/com/example/petmatchlab/security/SecurityConfig.java`** — archivo completo:

```java
package com.example.petmatchlab.security;

import jakarta.servlet.DispatcherType;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
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
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
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
                    HttpMethod.POST, "/pets/{petId}/requests"
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

**Lee el archivo en tres partes:**

1. `passwordEncoder()` proporciona una implementación de `PasswordEncoder` tanto a `RegistrationService` como al mecanismo de autenticación.
2. `authorizeHttpRequests(...)` describe la política de acceso a rutas y métodos HTTP.
3. `formLogin(...)` y `logout(...)` configuran los procesos de entrada y salida.

En esta configuración **no aparece `.csrf(csrf -> csrf.disable())`**. Eso es intencional. Conservamos la protección CSRF de la aplicación con sesiones.

Tampoco configuramos `httpBasic`: este laboratorio se concentra en navegación web con formularios. La autenticación de una futura API REST requiere discutir por separado sus mecanismos y las políticas aplicables; no debe improvisarse reutilizando sin más la configuración de esta página.

### 5. Interpretar las reglas y su orden

Las reglas se examinan según su orden de declaración. Las más específicas se escriben antes que el caso general:

```java
.requestMatchers("/admin", "/admin/**").hasRole("ADMIN")
```

La ruta de administración requiere el rol `ADMIN`. Después, algunas rutas `GET` que muestran formularios de edición y algunas rutas `POST` que modifican datos requieren igualmente `ADMIN`.

La creación de solicitudes queda disponible a `USER` y `ADMIN`:

```java
.requestMatchers(
    HttpMethod.POST, "/pets/{petId}/requests"
).hasAnyRole("USER", "ADMIN")
```

Finalmente:

```java
.anyRequest().authenticated()
```

exige autenticación para las demás rutas no incluidas en la lista pública. Eso comprende, por ejemplo, el listado de mascotas y la consulta de solicitudes.

**Importante:** esta política protege las rutas que existen hoy. Al introducir nuevos controladores o métodos `POST`, `PUT`, `PATCH` o `DELETE`, revisa siempre si deben restringirse a un rol. El `anyRequest().authenticated()` final no convierte automáticamente a cada futura operación de modificación en una operación solo para administradores.

### 6. Crear el controlador del formulario de acceso

Crea **`src/main/java/com/example/petmatchlab/controller/LoginController.java`** — archivo completo:

```java
package com.example.petmatchlab.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class LoginController {

    @GetMapping("/login")
    public String loginPage() {
        return "auth/login";
    }
}
```

Este controlador solo sirve **`GET /login`**. No contiene una consulta a `AppUserRepository`, no compara contraseñas ni escribe una cookie de sesión.

**¿Quién procesa `POST /login`?** El filtro de autenticación de Spring Security configurado en `SecurityFilterChain`. Crear otro `@PostMapping("/login")` para comprobar contraseñas manualmente duplicaría una responsabilidad sensible y podría introducir errores de seguridad.

### 7. Crear la página de inicio de sesión

Crea **`src/main/resources/templates/auth/login.html`** — archivo completo:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Iniciar sesión | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <section class="content-panel">
            <h1>Iniciar sesión</h1>

            <p th:if="${param.registered}" role="status">
                La cuenta fue registrada. Ahora puedes iniciar sesión.
            </p>
            <p th:if="${param.error}" role="alert">
                No se pudo iniciar sesión con esas credenciales.
            </p>
            <p th:if="${param.logout}" role="status">
                La sesión se cerró correctamente.
            </p>

            <form th:action="@{/login}" method="post">
                <div>
                    <label for="email">Correo electrónico</label>
                    <input id="email" name="email" type="email"
                           autocomplete="username" required>
                </div>
                <div>
                    <label for="password">Contraseña</label>
                    <input id="password" name="password" type="password"
                           autocomplete="current-password" required>
                </div>
                <button type="submit">Entrar</button>
            </form>

            <p>
                ¿No tienes una cuenta?
                <a th:href="@{/register}">Registrarse</a>
            </p>
        </section>
    </main>
</body>
</html>
```

El atributo HTML `name="email"` debe coincidir con:

```java
.usernameParameter("email")
```

Si usas `name="username"` en el HTML sin cambiar la configuración, Spring Security no recibirá el identificador esperado. Lo mismo sucede con `name="password"`.

El parámetro `?error` no explica si un correo existe o no; presenta un mensaje genérico de credenciales incorrectas. Esa decisión evita revelar innecesariamente información sobre cuentas a través de la respuesta de login.

### 8. Probar un recorrido de autenticación

Una vez creada la página de registro y terminada la configuración:

1. Abre `/pets` sin iniciar sesión. Debes llegar al formulario de acceso.
2. Abre `/register`, crea una cuenta y comprueba el mensaje de registro.
3. Vuelve a `/login` y escribe el correo y la contraseña registrados.
4. Después de autenticarte debes poder consultar `/pets`.
5. Intenta abrir `/admin` con la cuenta nueva. El servidor debe rechazar el acceso, porque todavía tiene rol `USER`.
6. Cierra sesión e intenta abrir `/pets` otra vez. Debe solicitar autenticación.

### 9. ¿Qué significa una autenticación exitosa?

Después de validar las credenciales, Spring Security crea un objeto `Authentication` que identifica a la cuenta y sus autoridades. El estado de seguridad se conserva a través de la sesión de acuerdo con la infraestructura configurada.

No significa que todas las reglas se hayan superado de una vez para siempre. En cada petición que llega a una ruta protegida se aplica la autorización correspondiente. Además, los servicios continúan verificando las reglas de negocio.

### 10. El cierre de sesión no es un enlace GET

La salida se realizará mediante `POST /logout`. Spring Security se ocupa de invalidar la sesión y limpiar el contexto de seguridad. No implementes un controlador que haga únicamente `return "redirect:/login"` y suponga que eso equivale a cerrar sesión.

Más adelante añadiremos un formulario de salida con `method="post"`. Cuando CSRF está activado, también debe enviar un token válido.

### Ejercicio — Sigue una petición protegida

Una persona anónima intenta abrir `/pets/7/requests`. Describe qué componente revisa primero su autorización, qué respuesta recibe y por qué `SupportRequestController.list(...)` no debe implementar un nuevo formulario de login.

Después la misma persona inicia sesión como `USER` y envía `POST /pets/7/requests/9/complete`. ¿Por qué debería recibir `403` sin que la transición de estado llegue a ejecutarse?

### Depuración — Login no funciona

- **La página `/login` redirige a sí misma:** comprueba `permitAll()` y la existencia de `GET /login`.
- **La cuenta existe, pero el login falla:** revisa el email normalizado, `UserDetailsService`, el prefijo del hash y los nombres de campos HTML.
- **La sesión parece desaparecer:** revisa cookies del navegador, dominio, puerto y configuración del servidor; evita mezclar direcciones distintas durante las pruebas.
- **Los usuarios deshabilitados pueden entrar:** comprueba la transformación `disabled(!user.isEnabled())`.
- **Aparece la contraseña generada de Spring Boot:** puede faltar tu `UserDetailsService` o no estar siendo detectado como componente.

**Comprobación:** el formulario de login no valida credenciales manualmente y Spring Security autentica mediante las cuentas de MariaDB.

---

## Práctica 11.4 — Proteger las operaciones del dominio mediante roles

### Objetivos

- diferenciar la restricción de la ruta de la validación de negocio;
- comprobar que `ADMIN` puede ejecutar operaciones que `USER` no puede;
- incorporar una página de administración mínima;
- preparar una cuenta `ADMIN` sin permitir que el registro público asigne ese rol;
- reconocer la limitación de una política basada solo en roles.

### 1. Conservar los servicios existentes

No reescribas `PetService` ni `SupportRequestService`. Ambos deben seguir aplicando las reglas de validación, integridad y transiciones del laboratorio anterior. La cadena de seguridad decide si se **permite llamar al controlador**; los servicios deciden si la **operación tiene sentido para el estado y las relaciones del dominio**.

Ejemplo:

```text
POST /pets/7/requests/9/complete
  ↓
Spring Security
  ├── USER: 403; no llega al controlador
  └── ADMIN: puede continuar
           ↓
       SupportRequestController
           ↓
       SupportRequestService
           ├── OPEN: completar
           └── CANCELLED: rechazar por regla de negocio
```

El rol `ADMIN` no convierte una transición inválida en válida.

### 2. Crear una página administrativa mínima

Crea **`src/main/java/com/example/petmatchlab/controller/AdminController.java`** — archivo completo:

```java
package com.example.petmatchlab.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class AdminController {

    @GetMapping("/admin")
    public String home() {
        return "admin/home";
    }
}
```

Crea **`src/main/resources/templates/admin/home.html`** — archivo completo:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Administración | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>
    <main class="page-container">
        <section class="content-panel">
            <h1>Administración</h1>
            <p>Esta sección requiere el rol de administrador.</p>
            <a th:href="@{/pets/new}">Registrar mascota</a>
            <a th:href="@{/pets}">Consultar mascotas</a>
        </section>
    </main>
</body>
</html>
```

No guardamos información sensible en esta vista. Su propósito es comprobar de forma visible una restricción de rutas con `hasRole("ADMIN")`.

### 3. Preparar una cuenta administrativa de desarrollo

**No añadas un campo de rol al registro público** y no crees una contraseña administrativa fija en un archivo Java, SQL de inicialización o `application.yaml`.

Para probar permisos, crea una segunda cuenta utilizando el formulario de registro. A continuación, **solo en la base de desarrollo y con acceso directo controlado a MariaDB**, asígnale el rol `ADMIN`:

```sql
SELECT id, email, role
FROM app_users
ORDER BY id;

UPDATE app_users
SET role = 'ADMIN'
WHERE email = 'admin@example.com'
  AND role = 'USER';
```

Antes de ejecutar `UPDATE`, sustituye `admin@example.com` por el **correo exacto de tu cuenta de prueba**, verifica que la fila correcta exista y que no pertenezca a otra persona. Puedes comprobar el resultado con:

```sql
SELECT id, email, role
FROM app_users
WHERE email = 'admin@example.com';
```

Este procedimiento es **solo una técnica de preparación de datos para experimentar localmente**; no es una propuesta de administración de privilegios para producción. En un sistema real se requiere un proceso de aprovisionamiento y autorización auditado, con controles de acceso a la base y revisión de las operaciones administrativas.

**Cierra sesión y vuelve a entrar** con la cuenta modificada. La sesión existente puede conservar autoridades anteriores hasta que ocurra una nueva autenticación; cambiar un campo en la base no implica que todas las sesiones ya abiertas actualicen sus permisos de inmediato.

### 4. Comprobar la política con dos cuentas

| Acción | Cuenta `USER` | Cuenta `ADMIN` |
|---|---|---|
| Consultar `/pets` | Permitida | Permitida |
| Consultar `/pets/{id}/requests` | Permitida | Permitida |
| Abrir `/pets/new` | `403` | Permitida |
| Crear mascota mediante POST | `403` | Permitida |
| Crear una solicitud mediante POST | Permitida | Permitida |
| Completar solicitud mediante POST | `403` | Permitida si el estado lo permite |
| Reemplazar solicitud abierta | `403` | Permitida si cumple las reglas |
| Abrir `/admin` | `403` | Permitida |

Un `USER` no obtiene privilegios administrativos cambiando la URL ni construyendo una petición manual. Las restricciones se aplican del lado del servidor.

### 5. ¿Qué ocurre con los recursos de otras personas?

En este momento el dominio todavía no vincula `Pet` ni `SupportRequest` con la identidad de la cuenta que los creó. Eso significa que **la autenticación y los roles no implementan propiedad sobre recursos**.

No escribas frases como «cada usuario solo puede ver sus solicitudes», porque el código construido todavía no garantiza esa condición. De hecho, el diseño actual permite a todos los usuarios autenticados consultar las solicitudes registradas y a los administradores operar sobre ellas de acuerdo con las reglas existentes.

Cuando necesites privacidad o control por propietario, debes introducir una relación explícita entre el recurso y la cuenta (por ejemplo, `SupportRequest.createdBy`), establecer una regla de acceso en servicio y repositorio, y probar que modificar identificadores en la URL no permite acceder a registros ajenos. La autorización por rol sería entonces **una capa de defensa**, no el único mecanismo.

### 6. ¿Qué pasaría si protegemos solo el formulario GET?

Supón que escribes una regla para bloquear `/pets/new`, pero olvidas proteger `POST /pets`. Un usuario puede enviar un POST con herramientas HTTP sin abrir el formulario. El servidor debe proteger **la operación que realiza el cambio**, no solo la página que muestra el botón.

Por eso `SecurityConfig` distingue explícitamente las rutas de creación y edición y los métodos HTTP utilizados.

### Ejercicio — Cambia una regla y comprueba su efecto

En una rama o copia de experimentación, cambia temporalmente:

```java
.requestMatchers("/admin", "/admin/**").hasRole("ADMIN")
```

por una regla que exija solamente `authenticated()`. Comprueba que una cuenta `USER` puede abrir `/admin`, identifica por qué se vulneró la matriz de permisos y **restaura inmediatamente** la regla original.

### Depuración — Un rol no funciona

- **Un administrador recibe `403`:** comprueba `AppUser.role`, la conversión a `ROLE_ADMIN` y si inició sesión después del cambio.
- **Un usuario normal puede completar una solicitud:** comprueba la coincidencia de la ruta y el método `POST` en `SecurityConfig`, y el orden de reglas.
- **Una página administrativa tiene acceso público:** puede existir un matcher `permitAll()` demasiado amplio antes de la regla administrativa.
- **La petición supera Spring Security pero falla con `409`:** la autorización fue aceptada; ahora el servicio está aplicando una regla de negocio. No elimines esa regla para «arreglar» el error.

**Comprobación:** puedes demostrar la diferencia entre `403 Forbidden`, una redirección por falta de autenticación y `409 Conflict` por transición inválida.

---
## Práctica 11.5 — Formularios, navegación y protección CSRF

### Objetivos

- completar el formulario de registro con Thymeleaf y Bean Validation;
- explicar qué problema soluciona un token CSRF;
- enviar el cierre de sesión como formulario POST;
- adaptar la navegación a usuarios anónimos, normales y administradores;
- revisar formularios existentes para garantizar que funcionan con CSRF habilitado.

### 1. Crear el formulario de registro

Crea **`src/main/resources/templates/auth/register.html`** — archivo completo:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<head th:replace="~{fragments/head :: head('Registro | PetMatch Mini')}"></head>
<body>
    <nav th:replace="~{fragments/navigation :: navigation}"></nav>

    <main class="page-container">
        <section class="content-panel">
            <h1>Crear cuenta</h1>
            <p>El registro crea una cuenta con permisos de usuario.</p>

            <form th:action="@{/register}"
                  th:object="${registrationForm}"
                  method="post">
                <div>
                    <label for="displayName">Nombre</label>
                    <input id="displayName" type="text"
                           th:field="*{displayName}"
                           maxlength="100" autocomplete="name" required>
                    <p th:if="${#fields.hasErrors('displayName')}"
                       th:errors="*{displayName}" role="alert">
                        Error en el nombre
                    </p>
                </div>

                <div>
                    <label for="email">Correo electrónico</label>
                    <input id="email" type="email"
                           th:field="*{email}"
                           maxlength="190" autocomplete="email" required>
                    <p th:if="${#fields.hasErrors('email')}"
                       th:errors="*{email}" role="alert">
                        Error en el correo
                    </p>
                </div>

                <div>
                    <label for="password">Contraseña</label>
                    <input id="password" name="password" type="password"
                           minlength="12" maxlength="64"
                           autocomplete="new-password" required>
                    <p th:if="${#fields.hasErrors('password')}"
                       th:errors="*{password}" role="alert">
                        Error en la contraseña
                    </p>
                </div>

                <div>
                    <label for="confirmPassword">Confirmar contraseña</label>
                    <input id="confirmPassword" name="confirmPassword"
                           type="password" autocomplete="new-password" required>
                    <p th:if="${#fields.hasErrors('confirmPassword')}"
                       th:errors="*{confirmPassword}" role="alert">
                        Error en la confirmación
                    </p>
                </div>

                <button type="submit">Registrar cuenta</button>
            </form>

            <p>
                <a th:href="@{/login}">Ya tengo una cuenta</a>
            </p>
        </section>
    </main>
</body>
</html>
```

Observa que **los campos de contraseña no utilizan `th:field` para reconstruir su valor**. Tienen `name="password"` y `name="confirmPassword"`, por lo que siguen siendo vinculados por Spring MVC al DTO, pero no se envían los valores de vuelta al HTML si aparece un error. En esa situación el usuario deberá volver a escribirlos.

La aplicación valida en el servidor incluso si el navegador permite omitir `required`, `minlength` u otras restricciones HTML. Un cliente puede enviar peticiones sin ejecutar la validación del navegador.

### 2. CSRF: una petición que aprovecha una sesión ajena

Imagina que una persona ha iniciado sesión en PetMatch Mini y que, sin cerrar esa sesión, visita una página maliciosa en otro sitio. Esa página intenta inducir al navegador a enviar un formulario `POST` a una ruta de PetMatch. Dependiendo de las políticas del navegador, las cookies y el contexto de la petición, podría aprovecharse la autenticación ya existente.

**Cross-Site Request Forgery (CSRF)** es el nombre de esta clase de ataque: una petición no deseada que intenta valerse de la sesión autenticada de otra persona.

El token CSRF permite que el servidor compruebe que una petición que cambia estado incluye un valor asociado al contexto legítimo de la aplicación. Un atacante que construye un formulario externo normalmente no puede obtener ese token válido mediante las políticas del navegador.

```text
Formulario legítimo de PetMatch Mini
  ├── POST /pets/7/requests
  ├── campos de negocio
  └── token CSRF correcto
          ↓
    Spring Security permite continuar

Formulario externo sin token válido
  ├── POST /pets/7/requests
  └── token ausente o incorrecto
          ↓
    Spring Security rechaza la petición
```

**CSRF no es lo mismo que XSS, autenticación o autorización.** La presencia de un token no demuestra que la persona tenga el rol necesario, y no reemplaza la validación de entradas ni la protección contra scripts maliciosos.

### 3. ¿Cómo llega el token al formulario Thymeleaf?

Cuando Spring Security está integrado con Spring MVC y Thymeleaf, los formularios que utilizan `th:action` con un método que cambia estado pueden recibir automáticamente un campo oculto CSRF mediante el `RequestDataValueProcessor`.

Por ejemplo, al renderizar:

```html
<form th:action="@{/logout}" method="post">
    <button type="submit">Cerrar sesión</button>
</form>
```

el HTML enviado al navegador puede contener un campo equivalente a:

```html
<input type="hidden" name="_csrf" value="generated-token-value">
```

**Este segundo bloque es ilustrativo**: no escribas un token fijo en el código. Su valor real lo genera y gestiona Spring Security y no debe copiarse entre sesiones.

Para comprobarlo, abre las herramientas de desarrollador del navegador, inspecciona un formulario renderizado y busca el campo oculto `_csrf`. Si no aparece, comprueba que estás utilizando una plantilla procesada por Thymeleaf con `th:action`, que la configuración de seguridad está activa y que el formulario se envía desde el contexto esperado.

### 4. Adaptar la navegación a la sesión

En el Laboratorio 7 se creó `templates/fragments/navigation.html`. Actualmente ofrece enlaces estáticos. Vamos a mantener el fragmento, pero mostraremos acciones distintas según la autenticación y el rol.

Primero crea **`src/main/java/com/example/petmatchlab/controller/NavigationModelAdvice.java`** — archivo completo:

```java
package com.example.petmatchlab.controller;

import org.springframework.security.authentication.AnonymousAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ModelAttribute;

@ControllerAdvice
public class NavigationModelAdvice {

    @ModelAttribute("currentEmail")
    public String currentEmail(Authentication authentication) {
        if (!isLoggedIn(authentication)) {
            return null;
        }
        return authentication.getName();
    }

    @ModelAttribute("isAdmin")
    public boolean isAdmin(Authentication authentication) {
        if (!isLoggedIn(authentication)) {
            return false;
        }
        return authentication.getAuthorities().stream()
            .anyMatch(authority ->
                authority.getAuthority().equals("ROLE_ADMIN")
            );
    }

    private boolean isLoggedIn(Authentication authentication) {
        return authentication != null
            && authentication.isAuthenticated()
            && !(authentication instanceof AnonymousAuthenticationToken);
    }
}
```

`@ControllerAdvice` permite ofrecer atributos de modelo a las vistas de varios controladores sin copiar el mismo código. El correo autenticado procede de `Authentication`, no de un parámetro manipulable del navegador.

`isAdmin` se utiliza para la presentación. **No sustituye** la comprobación real `hasRole("ADMIN")` de `SecurityConfig`. Aunque el navegador ocultase el enlace de administración, el servidor seguiría teniendo que rechazar una petición no autorizada.

Sustituye el contenido de **`src/main/resources/templates/fragments/navigation.html`** por este archivo completo:

```html
<!DOCTYPE html>
<html lang="es" xmlns:th="http://www.thymeleaf.org">
<body>
    <nav th:fragment="navigation" class="site-navigation"
         aria-label="Navegación principal">
        <a class="brand" th:href="@{/}">PetMatch Mini</a>

        <div class="navigation-links">
            <a th:href="@{/}">Inicio</a>
            <a th:href="@{/about}">Acerca de</a>
            <a th:if="${currentEmail != null}"
               th:href="@{/pets}">Mascotas</a>
            <a th:if="${isAdmin}"
               th:href="@{/admin}">Administración</a>
            <a th:if="${currentEmail == null}"
               th:href="@{/login}">Iniciar sesión</a>
            <a th:if="${currentEmail == null}"
               th:href="@{/register}">Registrarse</a>

            <span th:if="${currentEmail != null}"
                  th:text="${currentEmail}">user@example.com</span>

            <form th:if="${currentEmail != null}"
                  th:action="@{/logout}" method="post">
                <button type="submit">Cerrar sesión</button>
            </form>
        </div>
    </nav>
</body>
</html>
```

El fragmento conserva `th:fragment="navigation"`, por lo que las vistas anteriores que utilizan `th:replace` siguen encontrándolo en la misma ruta.

**Comprueba tres escenarios:** visitante sin sesión, cuenta `USER` y cuenta `ADMIN`. La navegación debería reflejar cada caso, pero recuerda que la comprobación decisiva permanece en la cadena de seguridad.

### 5. Revisar los formularios del CRUD ya existente

Antes de introducir Spring Security, las operaciones POST de la aplicación podían ejecutarse sin un token CSRF. Ahora, los siguientes formularios deben incluirlo automáticamente al renderizarse desde Thymeleaf:

- creación de mascota (`POST /pets`);
- actualización de mascota (`POST /pets/{id}`);
- eliminación de mascota (`POST /pets/{id}/delete`);
- creación de solicitud (`POST /pets/{petId}/requests`);
- completar y cancelar solicitudes;
- reemplazar una solicitud abierta;
- registro y login;
- cierre de sesión.

Revisa los archivos de `templates/pets/` y `templates/requests/`. Para cada formulario que envía POST, utiliza una acción procesada por Thymeleaf:

```html
<form th:action="@{/pets/{id}/delete(id=${pet.id})}"
      method="post">
    <button type="submit">Eliminar</button>
</form>
```

**Fragmento de ejemplo:** no crees una segunda copia del botón si ya existe. Comprueba que el archivo real utilice `th:action` y mantenga su estructura anterior.

Si un formulario se construye dinámicamente mediante JavaScript y no pasa por la integración habitual con Thymeleaf, tendrás que transportar el token de una manera adecuada. **No desactives CSRF** para que un formulario defectuoso funcione.

### 6. Ocultar controles administrativos sin retirar la protección del servidor

En los listados y detalles existentes, algunos enlaces o botones solo tienen sentido para un administrador. Puedes agregar `th:if="${isAdmin}"` a la etiqueta correspondiente.

Ejemplo de enlace al formulario de edición:

```html
<a th:if="${isAdmin}"
   th:href="@{/pets/{id}/edit(id=${pet.id})}">
    Editar mascota
</a>
```

Ejemplo de formulario para completar una solicitud:

```html
<form th:if="${isAdmin and request.isOpen()}"
      th:action="@{/pets/{petId}/requests/{requestId}/complete(
          petId=${pet.id}, requestId=${request.id})}"
      method="post">
    <button type="submit">Completar solicitud</button>
</form>
```

Estos son **fragmentos de integración**: debes adaptarlos al contexto exacto de la plantilla existente. No dupliques el `th:if` si el botón ya tiene una condición de estado; combina ambos requisitos en una sola expresión.

Los botones que realizan cambios no deben aparecer para `USER` cuando la ruta requiere `ADMIN`. Un usuario normal puede seguir viendo las solicitudes, pero no se le deben ofrecer acciones que el servidor rechazará con seguridad.

### 7. Comprobar con las herramientas del navegador

Abre las herramientas de desarrollo y revisa:

1. **Network:** al enviar el formulario de login, observa la petición `POST /login` y su respuesta de redirección.
2. **Storage / Application:** identifica la cookie de sesión. No copies ni compartas su valor.
3. **Elements:** comprueba que los formularios POST contienen el campo CSRF generado.
4. **Network:** después de cerrar sesión, intenta abrir `/pets`; debe producirse una nueva redirección a login.
5. **Network:** como `USER`, intenta una operación administrativa. El servidor debe rechazarla aunque la petición se fabrique sin utilizar el botón.

No publiques capturas que muestren cookies, tokens activos o credenciales personales. Utiliza cuentas de prueba y datos ficticios.

### 8. `GET` no debería modificar datos

CSRF se centra en métodos HTTP que modifican estado. Es importante que el diseño respete esa intención:

```text
GET    → consultar o mostrar formularios
POST   → crear o ejecutar cambios desde HTML
```

Si un controlador usa `@GetMapping("/delete")` para eliminar registros, **cámbialo**: un enlace visitado automáticamente o una solicitud generada sin intención del usuario podría provocar cambios. En PetMatch Mini las operaciones de modificación ya se expresan con POST, por lo que mantendremos esa decisión.

### Ejercicio — Analiza tres respuestas

- **Caso A:** `POST /pets/7/delete` como `ADMIN`, pero sin token CSRF. ¿Se debe permitir?
- **Caso B:** la misma operación con token válido y rol `USER`. ¿Se debe permitir?
- **Caso C:** la misma operación con token válido y rol `ADMIN`, pero la mascota tiene solicitudes asociadas. ¿Qué regla se aplica después de superar la seguridad?

Respuestas esperadas: A se rechaza por CSRF, B por autorización, y C llega a la regla de negocio que protege las relaciones. **Los tres controles resuelven problemas diferentes**.

### Depuración — El formulario devuelve `403`

1. Comprueba si la cuenta está autenticada.
2. Comprueba si tiene el rol que exige la ruta.
3. Comprueba si el método es POST y envía un token CSRF válido.
4. Comprueba que `th:action` se procesa por Thymeleaf.
5. Revisa si otra regla de `authorizeHttpRequests` coincide antes que la esperada.

No resuelvas un `403` quitando indiscriminadamente la seguridad. Comprende cuál de los controles lo produjo.

**Comprobación:** puedes iniciar y cerrar sesión mediante formularios, comprobar CSRF y distinguir elementos de navegación de permisos efectivos.

---
## Práctica 11.6 — Pruebas de seguridad, diagnóstico y reto integrador

### Objetivos

Al completar esta práctica podrás:

- probar el registro y la codificación de contraseñas sin levantar el servidor;
- comprobar con MockMvc el comportamiento de rutas públicas y protegidas;
- distinguir en pruebas fallos de autenticación, autorización y CSRF;
- actualizar las pruebas MVC de los laboratorios anteriores;
- identificar limitaciones del ejemplo antes de utilizarlo en un contexto real.

### 1. ¿Qué debemos comprobar?

El registro y el login involucran responsabilidades diferentes. Por eso no basta con una única prueba que verifique que se muestra `/login`.

| Requisito | Prueba que aporta evidencia |
|---|---|
| Contraseña no almacenada en texto plano | Servicio de registro y `PasswordEncoder` |
| Correo normalizado | Servicio de registro |
| No aceptar correos duplicados | Servicio y restricción única en la base |
| Rol inicial siempre `USER` | Entidad y registro |
| Cuenta encontrada por email | `DatabaseUserDetailsService` |
| Un visitante no abre rutas protegidas | MockMvc con filtros de seguridad |
| `USER` no puede realizar acciones administrativas | MockMvc con rol `USER` |
| `ADMIN` puede abrir `/admin` | MockMvc con rol `ADMIN` |
| Un POST sin token es rechazado | MockMvc sin CSRF |
| Login correcto e incorrecto | MockMvc y cuentas de prueba persistidas |
| Las reglas de negocio continúan operativas | Pruebas anteriores de servicios y dominio |

Estas pruebas se complementan. Un MockMvc con un usuario simulado comprueba una regla de acceso, pero **no demuestra que la contraseña de una cuenta real se valide correctamente**. Por eso incluiremos también una prueba de login con una cuenta almacenada en la base de pruebas.

### 2. Probar el registro sin ejecutar HTTP

Crea **`src/test/java/com/example/petmatchlab/service/RegistrationServiceTests.java`** — archivo completo:

```java
package com.example.petmatchlab.service;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.verifyNoInteractions;
import static org.mockito.Mockito.when;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;
import org.springframework.security.crypto.password.PasswordEncoder;

import com.example.petmatchlab.dto.RegistrationForm;
import com.example.petmatchlab.exception.DuplicateEmailException;
import com.example.petmatchlab.model.AppUser;
import com.example.petmatchlab.model.UserRole;
import com.example.petmatchlab.repository.AppUserRepository;

@ExtendWith(MockitoExtension.class)
class RegistrationServiceTests {

    private static final String PASSWORD = "ExamplePassword123!";

    @Mock
    private AppUserRepository repository;

    private PasswordEncoder passwordEncoder;
    private RegistrationService service;

    @BeforeEach
    void setUp() {
        passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
        service = new RegistrationService(repository, passwordEncoder);
    }

    @Test
    void shouldNormalizeEmailHashPasswordAndAssignUserRole() {
        RegistrationForm form = form("  ANA@EXAMPLE.COM  ", PASSWORD);
        when(repository.save(any(AppUser.class)))
            .thenAnswer(invocation -> invocation.getArgument(0));

        AppUser created = service.register(form);

        assertEquals("ana@example.com", created.getEmail());
        assertEquals("Ana", created.getDisplayName());
        assertEquals(UserRole.USER, created.getRole());
        assertTrue(created.isEnabled());
        assertFalse(PASSWORD.equals(created.getPasswordHash()));
        assertTrue(passwordEncoder.matches(PASSWORD, created.getPasswordHash()));
        verify(repository).existsByEmail("ana@example.com");
    }

    @Test
    void shouldRejectDuplicateEmailBeforeSaving() {
        RegistrationForm form = form("ana@example.com", PASSWORD);
        when(repository.existsByEmail("ana@example.com")).thenReturn(true);

        assertThrows(DuplicateEmailException.class,
            () -> service.register(form));

        verify(repository, never()).save(any(AppUser.class));
    }

    @Test
    void shouldRejectDifferentConfirmationPassword() {
        RegistrationForm form = form("ana@example.com", PASSWORD);
        form.setConfirmPassword("DifferentPassword123!");

        assertThrows(IllegalArgumentException.class,
            () -> service.register(form));

        verifyNoInteractions(repository);
    }

    @Test
    void shouldRejectShortPassword() {
        RegistrationForm form = form("ana@example.com", "short");

        assertThrows(IllegalArgumentException.class,
            () -> service.register(form));

        verifyNoInteractions(repository);
    }

    private RegistrationForm form(String email, String password) {
        RegistrationForm form = new RegistrationForm();
        form.setDisplayName("  Ana  ");
        form.setEmail(email);
        form.setPassword(password);
        form.setConfirmPassword(password);
        return form;
    }
}
```

**Lee la primera prueba.** No compara el hash con un texto fijo, porque los algoritmos modernos de almacenamiento de contraseñas incorporan valores aleatorios y pueden producir diferentes hashes para la misma contraseña. Comprueba la propiedad relevante: que el valor almacenado **no es la contraseña** y que el codificador puede verificarla mediante `matches`.

La contraseña de prueba es ficticia y solo pertenece a la suite de pruebas; **no la utilices como contraseña de una cuenta real ni la copies como credencial administrativa**.

### 3. Probar el adaptador de autenticación

Crea **`src/test/java/com/example/petmatchlab/security/DatabaseUserDetailsServiceTests.java`** — archivo completo:

```java
package com.example.petmatchlab.security;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.mockito.Mockito.when;

import java.util.Optional;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import com.example.petmatchlab.model.AppUser;
import com.example.petmatchlab.repository.AppUserRepository;

@ExtendWith(MockitoExtension.class)
class DatabaseUserDetailsServiceTests {

    @Mock
    private AppUserRepository repository;

    private DatabaseUserDetailsService service;

    @BeforeEach
    void setUp() {
        service = new DatabaseUserDetailsService(repository);
    }

    @Test
    void shouldLoadNormalizedEmailAndRole() {
        AppUser user = new AppUser(
            "Ana", "ana@example.com", "{bcrypt}example-hash"
        );
        when(repository.findByEmail("ana@example.com"))
            .thenReturn(Optional.of(user));

        UserDetails details = service.loadUserByUsername(" ANA@EXAMPLE.COM ");

        assertEquals("ana@example.com", details.getUsername());
        assertTrue(details.getAuthorities().stream().anyMatch(
            authority -> authority.getAuthority().equals("ROLE_USER")
        ));
        assertTrue(details.isEnabled());
    }

    @Test
    void shouldRejectUnknownEmail() {
        when(repository.findByEmail("missing@example.com"))
            .thenReturn(Optional.empty());

        assertThrows(UsernameNotFoundException.class,
            () -> service.loadUserByUsername("missing@example.com"));
    }
}
```

El valor de hash ficticio del primer test **no se utiliza para autenticar**; sirve únicamente para comprobar la adaptación de la entidad a `UserDetails`. La autenticación real con `PasswordEncoder` se verificará con una prueba integrada separada.

### 4. Probar la entidad y el repositorio de usuarios

Crea **`src/test/java/com/example/petmatchlab/repository/AppUserRepositoryTests.java`** — archivo completo:

```java
package com.example.petmatchlab.repository;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.data.jpa.test.autoconfigure.DataJpaTest;

import com.example.petmatchlab.model.AppUser;
import com.example.petmatchlab.model.UserRole;

@DataJpaTest
class AppUserRepositoryTests {

    @Autowired
    private AppUserRepository repository;

    @Test
    void shouldFindPersistedUserByEmail() {
        AppUser saved = repository.saveAndFlush(
            new AppUser("Ana", "ana@example.com", "{bcrypt}test-value")
        );

        var result = repository.findByEmail("ana@example.com");

        assertTrue(result.isPresent());
        assertEquals(saved.getId(), result.orElseThrow().getId());
        assertEquals(UserRole.USER, result.orElseThrow().getRole());
    }
}
```

Esta prueba utiliza H2 y una transacción de test aislada; **no guarda una cuenta en tu MariaDB de desarrollo**. Comprueba que Spring Data JPA puede almacenar y recuperar la entidad. La cadena `{bcrypt}test-value` es un valor ficticio para probar el repositorio; no sirve como contraseña verificable.

### 5. Probar los permisos y CSRF con MockMvc

Crea **`src/test/java/com/example/petmatchlab/security/SecurityRoutesIntegrationTests.java`** — archivo completo:

```java
package com.example.petmatchlab.security;

import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.csrf;
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.user;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.redirectedUrlPattern;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest
@AutoConfigureMockMvc
class SecurityRoutesIntegrationTests {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldAllowAnonymousAccessToLogin() throws Exception {
        mockMvc.perform(get("/login"))
            .andExpect(status().isOk());
    }

    @Test
    void shouldRedirectAnonymousVisitorToLogin() throws Exception {
        mockMvc.perform(get("/pets"))
            .andExpect(status().is3xxRedirection())
            .andExpect(redirectedUrlPattern("**/login"));
    }

    @Test
    void shouldForbidRegularUserFromAdminPage() throws Exception {
        mockMvc.perform(get("/admin")
                .with(user("ana@example.com").roles("USER")))
            .andExpect(status().isForbidden());
    }

    @Test
    void shouldAllowAdminToOpenAdminPage() throws Exception {
        mockMvc.perform(get("/admin")
                .with(user("admin@example.com").roles("ADMIN")))
            .andExpect(status().isOk());
    }

    @Test
    void shouldForbidRegularUserFromCreatingPet() throws Exception {
        mockMvc.perform(post("/pets")
                .with(user("ana@example.com").roles("USER"))
                .with(csrf()))
            .andExpect(status().isForbidden());
    }

    @Test
    void shouldForbidRegularUserFromCompletingRequest() throws Exception {
        mockMvc.perform(post("/pets/7/requests/9/complete")
                .with(user("ana@example.com").roles("USER"))
                .with(csrf()))
            .andExpect(status().isForbidden());
    }

    @Test
    void shouldRejectAdminPostWithoutCsrfToken() throws Exception {
        mockMvc.perform(post("/pets")
                .with(user("admin@example.com").roles("ADMIN")))
            .andExpect(status().isForbidden());
    }

    @Test
    void shouldRejectInvalidCsrfToken() throws Exception {
        mockMvc.perform(post("/pets")
                .with(user("admin@example.com").roles("ADMIN"))
                .with(csrf().useInvalidToken()))
            .andExpect(status().isForbidden());
    }
}
```

Las pruebas que utilizan `.with(user(...))` **simulan el estado de una cuenta autenticada** para verificar reglas de rutas. No necesitan crear esas cuentas en H2. Esta técnica ayuda a aislar la política de autorización.

El `post("/pets")` sin token es rechazado **antes** de que el controlador procese los campos de la mascota. No es necesario enviar nombre, especie o edad para probar que el filtro CSRF funciona.

**Observa la diferencia entre estas dos pruebas:**

```java
post("/pets")
    .with(user("ana@example.com").roles("USER"))
    .with(csrf())
```

se rechaza por falta de permiso, mientras que:

```java
post("/pets")
    .with(user("admin@example.com").roles("ADMIN"))
```

se rechaza por falta de token CSRF. Ambas pueden responder `403`, pero **las causas y las correcciones son diferentes**.

### 6. Probar un inicio de sesión contra una cuenta persistida

Crea **`src/test/java/com/example/petmatchlab/security/LoginIntegrationTests.java`** — archivo completo:

```java
package com.example.petmatchlab.security;

import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.csrf;
import static org.springframework.security.test.web.servlet.response.SecurityMockMvcResultMatchers.authenticated;
import static org.springframework.security.test.web.servlet.response.SecurityMockMvcResultMatchers.unauthenticated;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.test.web.servlet.MockMvc;

import com.example.petmatchlab.model.AppUser;
import com.example.petmatchlab.repository.AppUserRepository;

@SpringBootTest
@AutoConfigureMockMvc
class LoginIntegrationTests {

    private static final String EMAIL = "login-test@example.com";
    private static final String PASSWORD = "ExamplePassword123!";

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private AppUserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @BeforeEach
    void prepareUser() {
        userRepository.findByEmail(EMAIL)
            .ifPresent(userRepository::delete);

        userRepository.saveAndFlush(new AppUser(
            "Test User", EMAIL, passwordEncoder.encode(PASSWORD)
        ));
    }

    @Test
    void shouldAuthenticateRegisteredUserWithCorrectPassword() throws Exception {
        mockMvc.perform(post("/login")
                .with(csrf())
                .param("email", EMAIL)
                .param("password", PASSWORD))
            .andExpect(status().is3xxRedirection())
            .andExpect(authenticated().withUsername(EMAIL));
    }

    @Test
    void shouldRejectIncorrectPassword() throws Exception {
        mockMvc.perform(post("/login")
                .with(csrf())
                .param("email", EMAIL)
                .param("password", "WrongPassword123!"))
            .andExpect(status().is3xxRedirection())
            .andExpect(unauthenticated());
    }
}
```

**Esta prueba es diferente de la anterior.** El usuario de prueba sí está almacenado en la base H2 y el login ejecuta el mecanismo de autenticación real. La contraseña utilizada para preparar el hash debe coincidir exactamente con la enviada al formulario.

En caso de usar un perfil especial para pruebas, comprueba que `src/test/resources/application.yaml` continúa seleccionando H2. Nunca apuntes estas pruebas de creación y borrado de cuentas a la base MariaDB que contiene información que deseas conservar.

### 7. Comprobar el registro mediante HTTP

Crea **`src/test/java/com/example/petmatchlab/controller/RegistrationControllerIntegrationTests.java`** — archivo completo:

```java
package com.example.petmatchlab.controller;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.csrf;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.redirectedUrl;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.webmvc.test.autoconfigure.AutoConfigureMockMvc;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.test.web.servlet.MockMvc;

import com.example.petmatchlab.model.UserRole;
import com.example.petmatchlab.repository.AppUserRepository;

@SpringBootTest
@AutoConfigureMockMvc
class RegistrationControllerIntegrationTests {

    private static final String EMAIL = "registration-test@example.com";
    private static final String PASSWORD = "ExamplePassword123!";

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private AppUserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Test
    void shouldRegisterUserWithEncodedPasswordAndDefaultRole() throws Exception {
        userRepository.findByEmail(EMAIL).ifPresent(userRepository::delete);

        mockMvc.perform(post("/register")
                .with(csrf())
                .param("displayName", "Ana")
                .param("email", EMAIL)
                .param("password", PASSWORD)
                .param("confirmPassword", PASSWORD))
            .andExpect(status().is3xxRedirection())
            .andExpect(redirectedUrl("/login?registered"));

        var stored = userRepository.findByEmail(EMAIL).orElseThrow();
        assertEquals(UserRole.USER, stored.getRole());
        assertFalse(PASSWORD.equals(stored.getPasswordHash()));
        assertTrue(passwordEncoder.matches(PASSWORD, stored.getPasswordHash()));
    }

    @Test
    void shouldRejectRegistrationWithoutCsrfToken() throws Exception {
        mockMvc.perform(post("/register")
                .param("displayName", "Ana")
                .param("email", "other-registration@example.com")
                .param("password", PASSWORD)
                .param("confirmPassword", PASSWORD))
            .andExpect(status().isForbidden());
    }
}
```

Esta prueba verifica la entrada HTTP, la creación real de una cuenta en la base de pruebas y el rol inicial. La eliminación previa de una cuenta con el mismo correo es una precaución de aislamiento de test. **No utilices este patrón indiscriminadamente contra registros reales**, porque eliminar cuentas existentes puede afectar otros datos.

### 8. Actualizar las pruebas MVC de los laboratorios anteriores

Al incorporar Spring Security, las pruebas de controladores existentes pueden fallar aunque sus reglas de negocio continúen funcionando. Por ejemplo:

```java
mockMvc.perform(get("/pets"))
    .andExpect(status().isOk());
```

Ahora se espera una redirección si la petición no está autenticada. Si lo que quieres comprobar es **el listado para una persona autenticada**, agrega el usuario simulado:

```java
mockMvc.perform(get("/pets")
        .with(user("ana@example.com").roles("USER")))
    .andExpect(status().isOk());
```

En una prueba que verifica una operación administrativa por POST, utiliza un rol adecuado y un token válido:

```java
mockMvc.perform(post("/pets")
        .with(user("admin@example.com").roles("ADMIN"))
        .with(csrf())
        .param("name", "Luna")
        .param("species", "Dog")
        .param("age", "3"))
    .andExpect(status().is3xxRedirection());
```

Para estos fragmentos, agrega los imports estáticos de:

```java
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.csrf;
import static org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors.user;
```

**No sustituyas las aserciones de negocio** simplemente por aserciones de seguridad. Las pruebas antiguas siguen siendo valiosas. Deben adaptarse para que el flujo llegue al controlador bajo las nuevas condiciones de acceso.

Para `SupportRequestControllerIntegrationTests` del Laboratorio 10, revisa especialmente:

- consultas `GET` de solicitudes inexistentes: ahora requieren usuario autenticado para llegar al servicio y producir `404`;
- creación de solicitudes: requiere un `USER` o `ADMIN` con CSRF válido;
- completar, cancelar y reemplazar: requieren `ADMIN` y CSRF válido;
- pruebas de error `409`: necesitan superar primero la autorización para verificar la regla de estado;
- formulario de reemplazo `GET`: también requiere `ADMIN`.

Si una prueba que antes esperaba `404` ahora obtiene `302`, **no significa que desapareció el manejo de recursos inexistentes**: significa que no se ha autenticado y la cadena de seguridad ha detenido la petición antes de llegar a esa comprobación.

### 9. Ejecutar la suite

Desde la raíz del proyecto, en Linux o macOS:

```bash
./mvnw test
```

En Windows PowerShell:

```powershell
.\mvnw.cmd test
```

Si una prueba falla, lee el primer error causal relevante y clasifícalo:

| Síntoma | Pregunta inicial |
|---|---|
| Error de compilación | ¿El nombre de la clase, import o dependencia coincide con Spring Boot 4? |
| `NoSuchBeanDefinitionException` | ¿Existe el componente y se encuentra en un paquete escaneado? |
| `302` inesperado | ¿La petición simulada necesita autenticación? |
| `403` inesperado en POST | ¿Falta el rol, el token CSRF o ambos? |
| `404` | ¿Existe el recurso y coincide la ruta con el controlador? |
| `409` | ¿Se cumplió la autorización, pero falló una regla del dominio? |
| `Table not found: APP_USERS` | ¿Se configuró correctamente H2 y el mapeo de la entidad? |
| Login redirige a `?error` | ¿Coinciden email, contraseña, codificador y estado `enabled`? |
| Prueba JPA funciona y MariaDB no | ¿La base de pruebas y MariaDB están configuradas de manera diferente? |

Evita un arreglo que solo cambie las aserciones para que todas las pruebas queden verdes. La suite debe demostrar el comportamiento **esperado**, no aceptar cualquier resultado.

### 10. Laboratorio de errores — Investiga la causa

**Caso A: contraseña almacenada en texto plano.** Alguien reemplaza `passwordEncoder.encode(form.getPassword())` por `form.getPassword()`. ¿Qué información se expone si se obtiene una copia de la base? ¿Por qué la comprobación de login puede fallar al esperar un hash con prefijo de algoritmo?

**Caso B: rol editable en el registro.** Se agrega un campo `role` al formulario y se copia directamente a la entidad. ¿Por qué esto constituye una elevación de privilegios? ¿Qué cambio elimina la posibilidad en el flujo público?

**Caso C: `POST /login` devuelve `403`.** ¿Falta el CSRF? ¿Se utilizó `th:action`? ¿El test añadió `.with(csrf())`? Antes de revisar las credenciales, determina si la petición fue aceptada por la protección CSRF.

**Caso D: rol `ADMIN` en MariaDB, pero `403` en navegador.** ¿La sesión empezó antes del cambio de rol? ¿La autoridad construida por `DatabaseUserDetailsService` se llama `ROLE_ADMIN`?

**Caso E: `USER` puede consultar mascotas, pero no completar solicitudes.** ¿Se trata de un fallo o de la política esperada? Identifica la regla exacta de `SecurityConfig` que produce ese resultado.

**Caso F: el botón desaparece, pero la petición POST todavía funciona.** El desarrollador añadió `th:if="${isAdmin}"` y olvidó la regla de autorización. Explica por qué la interfaz no constituye una barrera de seguridad.

**Caso G: al activar Spring Security todas las pruebas de controladores fallan.** Distingue cuáles necesitan usuario simulado, cuáles necesitan CSRF y cuáles deben seguir probando el comportamiento de un visitante sin sesión.

**Caso H: se intenta acceder a solicitudes ajenas.** ¿Qué relación entre `SupportRequest` y `AppUser` falta para aplicar propiedad sobre datos? ¿Por qué la existencia del rol `USER` no resuelve esa necesidad?

### 11. Reto integrador — Cuenta y permisos comprobables

Amplía PetMatch Mini con una página `/profile` que muestre únicamente información no sensible de la cuenta autenticada. Como mínimo, debe presentar el correo electrónico, el nombre visible y el rol, sin incluir `passwordHash`.

Para resolverlo, analiza primero:

1. ¿De dónde obtiene Spring Security el correo del usuario autenticado?
2. ¿Qué servicio debe recuperar la entidad `AppUser` correspondiente?
3. ¿Por qué el controlador no debe aceptar un `userId` libre para decidir qué perfil mostrar?
4. ¿Qué ocurre si una sesión contiene una identidad cuyo registro fue eliminado después de iniciar sesión?
5. ¿Qué información debe pasarse a Thymeleaf y cuál debe permanecer exclusivamente en el servidor?
6. ¿Cómo comprobarías con MockMvc que un visitante recibe una redirección a login?
7. ¿Cómo demostrarías que un `USER` autenticado no obtiene automáticamente el rol de administrador?

**Restricciones del reto:** no muestres el hash de contraseña, no crees un campo para elegir el rol, no expongas una ruta que permita consultar arbitrariamente todas las cuentas y no desactives CSRF. Puedes comenzar con un DTO de lectura sencillo y un servicio con una consulta `findByEmail(...)`.

**Criterios de comprobación:** la página se limita al usuario autenticado, una cuenta inexistente produce un error controlado, la vista no expone secretos y las reglas de negocio sobre mascotas y solicitudes siguen funcionando.

### 12. Ampliación opcional — Cuenta deshabilitada

El atributo `enabled` permite practicar una condición adicional de autenticación. Diseña una prueba que obtenga una cuenta con `enabled = false` y compruebe que `UserDetails.isEnabled()` devuelve `false`.

No agregues una pantalla de desactivación de cuentas que cualquier usuario pueda utilizar sobre un identificador arbitrario. Para una operación administrativa real, harían falta autorización, auditoría, reglas sobre sesiones existentes y manejo cuidadoso de estados.

### 13. Ampliación opcional — Una política de contraseña más completa

La validación de longitud mínima y máxima de este ejercicio es un punto de partida, no una política exhaustiva. Investiga recomendaciones modernas de contraseñas, gestores de contraseñas, límites de bytes, caracteres Unicode y mecanismos de recuperación de cuenta. La recuperación por correo, los tokens de un solo uso y la verificación de correo no están implementados en el laboratorio.

Una aplicación expuesta en internet también necesita controles contra intentos repetidos de login y registro, monitoreo de eventos de seguridad, configuración segura de cookies, HTTPS y procedimientos para recuperar cuentas. No afirmes que el ejemplo es un sistema de identidad listo para producción por el solo hecho de almacenar hashes.

### Comprobación final

Antes de terminar, explica sin consultar el código:

- ¿Por qué `RegistrationController` no implementa el inicio de sesión?
- ¿Qué contrato implementa `DatabaseUserDetailsService` y qué datos devuelve?
- ¿Qué diferencia hay entre `PasswordEncoder.encode` y `matches`?
- ¿Por qué un rol en la base no debe entregarse mediante un campo del registro público?
- ¿Qué cambia al activar CSRF y conservar formularios `POST` de Thymeleaf?
- ¿Por qué una sesión autenticada no autoriza automáticamente a eliminar cualquier mascota?
- ¿Cómo distingues en una prueba un `302`, un `403` y un `409`?
- ¿Qué política sigue faltando si deseas que cada usuario acceda solo a sus propios registros?

---

## Mapa final del proyecto

Los archivos nuevos se suman a los de los laboratorios anteriores. No elimines los modelos, servicios y controladores de mascotas y solicitudes.

```text
petmatch-lab/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/com/example/petmatchlab/
│   │   │   ├── controller/
│   │   │   │   ├── HomeController.java
│   │   │   │   ├── PetController.java
│   │   │   │   ├── SupportRequestController.java
│   │   │   │   ├── RegistrationController.java
│   │   │   │   ├── LoginController.java
│   │   │   │   ├── AdminController.java
│   │   │   │   ├── NavigationModelAdvice.java
│   │   │   │   └── PageErrorHandler.java
│   │   │   ├── dto/
│   │   │   │   ├── PetForm.java
│   │   │   │   ├── SupportRequestForm.java
│   │   │   │   └── RegistrationForm.java
│   │   │   ├── exception/
│   │   │   │   ├── BusinessRuleException.java
│   │   │   │   ├── ResourceNotFoundException.java
│   │   │   │   └── DuplicateEmailException.java
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
│   │   │   └── service/
│   │   │       ├── PetService.java
│   │   │       ├── SupportRequestService.java
│   │   │       └── RegistrationService.java
│   │   └── resources/
│   │       ├── application.yaml
│   │       └── templates/
│   │           ├── fragments/
│   │           │   └── navigation.html
│   │           ├── auth/
│   │           │   ├── login.html
│   │           │   └── register.html
│   │           ├── admin/
│   │           │   └── home.html
│   │           ├── pets/
│   │           ├── requests/
│   │           └── errors/
│   └── test/
│       ├── java/com/example/petmatchlab/
│       │   ├── controller/
│       │   │   └── RegistrationControllerIntegrationTests.java
│       │   ├── repository/
│       │   │   └── AppUserRepositoryTests.java
│       │   ├── security/
│       │   │   ├── DatabaseUserDetailsServiceTests.java
│       │   │   ├── SecurityRoutesIntegrationTests.java
│       │   │   └── LoginIntegrationTests.java
│       │   └── service/
│       │       └── RegistrationServiceTests.java
│       └── resources/application.yaml
└── mvnw
```

El mapa destaca los archivos nuevos; omite algunos archivos anteriores de plantillas, pruebas y recursos para facilitar la lectura. **No borres un archivo existente porque no aparezca en este esquema resumido**.

## Revisión del funcionamiento

| Situación | Resultado esperado |
|---|---|
| Abrir `/register` sin sesión | Muestra formulario de registro |
| Registrar una cuenta válida | Persiste `AppUser` con rol `USER` y hash de contraseña |
| Registrar dos veces el mismo correo | Se rechaza la segunda cuenta |
| Registrar contraseñas diferentes | Se muestra error del formulario |
| Abrir `/pets` sin iniciar sesión | Redirige a `/login` |
| Iniciar sesión con contraseña válida | Autentica y redirige a `/pets` |
| Iniciar sesión con contraseña incorrecta | Regresa a `/login?error` |
| Abrir `/admin` como `USER` | `403 Forbidden` |
| Abrir `/admin` como `ADMIN` | Muestra la página administrativa |
| Crear mascota como `USER` | `403 Forbidden` |
| Crear mascota como `ADMIN` con CSRF válido | Llega al controlador y aplica reglas de negocio |
| Crear solicitud como `USER` con CSRF válido | Puede registrarla si las reglas lo permiten |
| Completar solicitud como `USER` | `403 Forbidden` |
| Completar solicitud como `ADMIN` | Permitido solo si el estado del dominio lo permite |
| Enviar un POST sin CSRF válido | `403 Forbidden` |
| Cerrar sesión mediante POST con CSRF | Invalida sesión y redirige al login |
| Abrir `/pets` después del logout | Solicita iniciar sesión de nuevo |
| Ejecutar pruebas sin MariaDB | La suite usa H2 y dobles cuando corresponde |

## Errores frecuentes y cómo interpretarlos

### «No puedo ver ninguna página después de agregar Spring Security»

Comprueba si la página requiere autenticación y si existe una ruta `/login` pública. No retires las dependencias de seguridad para evitar la redirección: configura el flujo esperado.

### «Los datos del registro se guardan, pero el login siempre falla»

Comprueba que `DatabaseUserDetailsService` consulta el correo normalizado, que el hash está almacenado con un algoritmo compatible y que los campos de login se llaman `email` y `password`. Verifica además si la cuenta tiene `enabled = true`.

### «La prueba esperaba 404 y ahora recibe 302»

El recurso no está siendo consultado porque el visitante todavía no se ha autenticado. Agrega un usuario de prueba cuando quieras comprobar el comportamiento del controlador para alguien con acceso.

### «La prueba de POST recibe 403 con un administrador»

Comprueba primero si falta el token CSRF. Una autoridad `ROLE_ADMIN` no reemplaza ese control.

### «Ya cambié el rol en MariaDB, pero sigo sin permiso»

Cierra sesión y autentícate de nuevo. El contexto de seguridad de una sesión puede conservar las autoridades obtenidas cuando se inició.

### «El formulario se envía, pero aparece un error de negocio»

Comprueba que la autorización fue superada y revisa las reglas de `PetService` o `SupportRequestService`. Tener permiso para intentar una acción no garantiza que el estado de la aplicación la permita.

### «El usuario ve los registros de otra cuenta»

La aplicación **todavía no implementa propiedad de recursos por usuario**. Para conseguirla se debe modelar esa relación y autorizar según el usuario autenticado y el registro consultado. Los roles por sí solos no bastan.

### «Se puede crear un administrador desde un formulario público»

No se debe aceptar un rol elegido por el navegador. Verifica que `RegistrationForm` no tenga un campo de rol y que `AppUser` asigne `USER` en el constructor utilizado por el registro.

## Cierre del laboratorio

La aplicación ya no depende únicamente de las reglas internas de cada operación. Ahora sabe **qué cuenta se ha autenticado y qué rutas puede utilizar según sus permisos**. Has incorporado un modelo de cuentas persistido en MariaDB, un servicio de registro, codificación segura de contraseñas, integración con `UserDetailsService`, sesiones web, formularios de login y logout, una política de autorización y pruebas para verificar que esa política funciona.

La separación de responsabilidades es la idea más importante: **Spring Security gestiona la autenticación, la sesión y el acceso a las rutas; los controladores reciben peticiones válidas; los servicios aplican reglas de negocio; y los repositorios persisten datos**. Ninguno de esos mecanismos sustituye automáticamente a los demás.

La seguridad no se completa con un login. En un sistema real siguen siendo necesarios HTTPS, protección de sesiones, límites a intentos repetidos, recuperación de cuentas, verificación de correo, permisos sobre recursos concretos, auditoría, migraciones de esquema y revisión de dependencias. Este laboratorio construye una base verificable para comprender esos temas sin presentarlos como funcionalidades que ya están implementadas.

## Referencias oficiales

- [Spring Boot 4.1.1 — Security](https://docs.spring.io/spring-boot/reference/web/spring-security.html): configuración por defecto y personalización de la seguridad web.
- [Spring Boot 4.1.1 — Build Systems](https://docs.spring.io/spring-boot/reference/using/build-systems.html): dependencias y starters de seguridad y pruebas en Spring Boot 4.
- [Spring Boot 4.1.1 — Managed Dependency Versions](https://docs.spring.io/spring-boot/appendix/dependency-versions/coordinates.html): versiones administradas de Spring Security.
- [Spring Security 7.1 — Form Login](https://docs.spring.io/spring-security/reference/7.1/servlet/authentication/passwords/form.html): procesamiento de formularios y páginas de autenticación.
- [Spring Security 7.1 — DaoAuthenticationProvider](https://docs.spring.io/spring-security/reference/7.1/servlet/authentication/passwords/dao-authentication-provider.html): colaboración entre `UserDetailsService` y `PasswordEncoder`.
- [Spring Security 7.1 — Password Storage](https://docs.spring.io/spring-security/reference/7.1/features/authentication/password-storage.html): hashes de contraseña y codificador delegado.
- [Spring Security 7.1 — Authorize HTTP Requests](https://docs.spring.io/spring-security/reference/7.1/servlet/authorization/authorize-http-requests.html): reglas por método HTTP, rol y orden de autorización.
- [Spring Security 7.1 — CSRF](https://docs.spring.io/spring-security/reference/7.1/servlet/exploits/csrf.html): protección de peticiones que modifican estado e integración con Thymeleaf.
- [Spring Security 7.1 — Logout](https://docs.spring.io/spring-security/reference/7.1/servlet/authentication/logout.html): cierre de sesión e invalidación del contexto.
- [Spring Security 7.1 — MockMvc](https://docs.spring.io/spring-security/reference/7.1/servlet/test/mockmvc/setup.html): integración de los filtros con pruebas HTTP simuladas.
- [Spring Security 7.1 — Testing with CSRF](https://docs.spring.io/spring-security/reference/7.1/servlet/test/mockmvc/csrf.html): tokens válidos e inválidos en MockMvc.
- [Spring Security 7.1 — Running as a User](https://docs.spring.io/spring-security/reference/7.1/servlet/test/mockmvc/authentication.html): usuarios y autoridades simulados en pruebas.
- [Spring Data JPA — Repositories](https://docs.spring.io/spring-data/jpa/reference/repositories/core-concepts.html): repositorios, consultas y persistencia de entidades.
- [Thymeleaf 3.1 — Using Thymeleaf](https://www.thymeleaf.org/doc/tutorials/3.1/usingthymeleaf.html): formularios, expresiones y fragmentos.
- [Jakarta Bean Validation 3.1](https://jakarta.ee/specifications/bean-validation/3.1/): validación declarativa de datos de entrada.
- [Java SE 21 — Security APIs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/security/package-summary.html): conceptos generales de seguridad y criptografía de Java.
