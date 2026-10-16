# Laboratorio 5 — Simulación completa de un proyecto Java con Scrum y GitHub

Este laboratorio presenta **un proyecto ficticio de principio a fin**. En lugar de proponer una secuencia de ejercicios, muestra cómo un equipo transforma una idea en una aplicación Java, cómo toma decisiones durante tres sprints, cómo deja evidencia verificable en GitHub y cómo organiza la entrega final.

La parte central es **comprender Scrum en funcionamiento**: para qué sirve cada responsabilidad, evento, artefacto y compromiso; qué decisiones se toman; y qué ocurre cuando el trabajo no sale exactamente como se había planeado. El código Java acompaña la historia para mostrar que los registros del proceso corresponden a un producto real, no a reuniones y tarjetas desconectadas del desarrollo.

> **Naturaleza del caso.** Personas, reuniones, estados, números de Issues y Pull Requests, mensajes de commits y resultados que aparecen a continuación son **datos simulados**. No corresponden a un repositorio que se haya creado realmente. En un proyecto auténtico, GitHub asigna los números y registra automáticamente las fechas; esos registros no deben recrearse ni alterarse para aparentar trabajo anterior.

## Vista general del simulacro

| Elemento | Decisión del caso |
|---|---|
| Producto | **RecursoTrack**, sistema de consola para gestionar préstamos de recursos compartidos |
| Plataforma de desarrollo | Java 21, Visual Studio Code y terminal |
| Gestión | Scrum, GitHub Issues, Pull Requests y GitHub Projects |
| Equipo | Tres personas; **las tres desarrollan software** |
| Duración | **7 días exactos**, incluida la entrega y sustentación grupal |
| Sprint 1 | Días 1–2: registrar y consultar recursos |
| Sprint 2 | Días 3–4: gestionar préstamos y abordar devoluciones |
| Sprint 3 | Días 5–7: completar devoluciones, reportes, validaciones y cierre |
| Entrega | Repositorio, trazabilidad, informe PDF y demostración grupal el día 7 |

El caso se puede **leer y estudiar sin realizar siete jornadas**. Los días representan la secuencia de una actividad integradora real, no instrucciones para simular fechas ni para publicar registros ficticios como si hubieran ocurrido.

---

## 1. La idea del producto: antes de hablar de tareas

### El problema

Una organización comparte recursos —adaptadores, proyectores, calculadoras, herramientas y otros elementos—. La disponibilidad se controla de manera informal. A veces alguien solicita un recurso que ya está prestado o registra una devolución que excede la cantidad original.

El equipo decide resolver un problema limitado: **registrar recursos, consultar disponibilidad y controlar préstamos y devoluciones desde una aplicación de consola**. No incluye cuentas de usuario, base de datos, interfaz gráfica, servicios web ni persistencia. Los datos existirán únicamente mientras el programa esté abierto.

Esta delimitación es fundamental: siete días alcanzan para un producto pequeño, no para construir un sistema administrativo completo.

### El pitch del día 1

Antes de programar, el equipo expone una única presentación grupal de **60 segundos como máximo**:

| Tiempo | Contenido del ejemplo |
|---|---|
| 0–15 s | «Cuando varias personas comparten equipos, resulta difícil saber cuáles están disponibles». |
| 15–35 s | «Proponemos RecursoTrack, una aplicación Java que registra equipos y controla préstamos y devoluciones». |
| 35–50 s | «La persona responsable podrá consultar los recursos, evitar préstamos sin existencias y obtener un resumen». |
| 50–60 s | «Así se reducen inconsistencias y se conoce la disponibilidad sin hacer cálculos manuales». |

El pitch explica **problema, solución, usuario y valor**; no intenta enumerar clases Java ni demostrar Scrum. Es una actividad grupal de inicio y **no se transcribe en el informe PDF**.

### Product Goal: el resultado que se busca

> **Product Goal:** disponer de una herramienta de consola que permita conocer y controlar, de manera confiable, la cantidad disponible de recursos compartidos mediante registros, préstamos, devoluciones y consultas.

No dice «crear tres clases», «hacer veinte commits» o «usar un HashMap». Esos son medios de implementación. El Product Goal expresa el valor que el producto debe aportar.

---

## 2. Scrum: conceptos explicados a través del caso

### 2.1. Qué es Scrum y qué no es

Scrum es un **marco de trabajo ligero** para generar valor mediante soluciones adaptativas a problemas complejos. En este caso se utiliza para entregar versiones comprobables de RecursoTrack, inspeccionar los resultados y ajustar el plan cuando aparece información nueva.

Su lógica se apoya en **tres pilares del empirismo**:

| Pilar | Qué representa | Cómo se observará en RecursoTrack |
|---|---|---|
| **Transparencia** | El trabajo y su estado deben ser visibles y entendibles | Una devolución con un error no figurará como terminada |
| **Inspección** | Revisar frecuentemente el avance y el producto | Cada Sprint Review probará una versión ejecutable |
| **Adaptación** | Cambiar el plan cuando los hechos lo requieren | La devolución incompleta se reconsiderará para el Sprint 3 |

Los **cinco valores** ayudan a que esto ocurra: **compromiso** con el objetivo, **foco** en el trabajo importante, **apertura** para reconocer problemas, **respeto** entre integrantes y **coraje** para tomar decisiones difíciles. Ocultar una prueba fallida para poner un Issue en `Done` va en contra de esa forma de trabajar.

**Scrum no es un tablero de GitHub.** GitHub proporciona registros y herramientas, pero no decide los objetivos, inspecciona el incremento ni mejora la colaboración por sí solo.

### 2.2. Responsabilidades: todos construyen el producto

Para el caso se utiliza este equipo ficticio:

| Persona | Responsabilidad Scrum adicional | Participación en desarrollo |
|---|---|---|
| Alex | **Product Owner** | Implementación, revisión y pruebas |
| Dani | **Scrum Master** | Implementación, revisión y pruebas |
| Sam | Ninguna adicional | Implementación, revisión y pruebas |

**Las tres personas son Developers.** Es una regla explícita de este escenario: asumir también las responsabilidades de Product Owner o Scrum Master no excluye el trabajo técnico. Tampoco significa que cada integrante escriba cada línea, pero **cualquiera debe poder explicar el proyecto completo**.

Las responsabilidades no forman una jerarquía:

- **Product Owner:** procura maximizar el valor y gestiona de forma efectiva el Product Backlog. En la historia, Alex prioriza primero el registro y los préstamos, antes que los reportes.
- **Scrum Master:** contribuye a la correcta comprensión y aplicación de Scrum y a la efectividad del equipo. En la historia, Dani ayuda a hacer visible un bloqueo y a mejorar la revisión de cambios; no es un jefe que reparte órdenes.
- **Developers:** convierten elementos del Product Backlog en incrementos que cumplen la calidad acordada. Todos planifican cómo trabajar, programan, prueban, revisan y adaptan el Sprint Backlog.

La organización con responsabilidades superpuestas responde al **tamaño reducido del equipo de este caso**. No debe confundirse con una regla general de Scrum que obligue a todo Product Owner o Scrum Master a programar.

### 2.3. Eventos: cinco momentos con propósitos diferentes

El **Sprint** es el evento contenedor. En su interior ocurren otros cuatro eventos:

| Evento | Propósito en Scrum | Qué se verá en el caso |
|---|---|---|
| **Sprint** | Crear valor dentro de un periodo fijo | Tres periodos de 2, 2 y 3 días |
| **Sprint Planning** | Definir por qué importa el sprint, qué se puede hacer y cómo empezar | Selección de historias, Sprint Goal y plan de los Developers |
| **Daily Scrum** | Inspeccionar avance hacia el Sprint Goal y adaptar el plan diario | Seguimientos breves con decisiones, no reportes a una autoridad |
| **Sprint Review** | Inspeccionar el resultado del sprint con personas interesadas y adaptar el producto | Demostración de funciones y decisiones sobre el Product Backlog |
| **Sprint Retrospective** | Mejorar calidad y efectividad de la forma de trabajar | Una mejora concreta después de cada sprint |

Una **Review** pregunta principalmente *qué valor aporta hoy el producto y qué cambiaremos a continuación*. Una **Retrospective** pregunta principalmente *cómo podemos trabajar mejor*. No se sustituyen entre sí.

La Daily Scrum tampoco exige las tradicionales tres preguntas «qué hice, qué haré, qué me bloquea»; esa fórmula puede servir, pero su propósito es revisar el avance hacia el **Sprint Goal** y ajustar el plan.

Para estos sprints excepcionalmente cortos el equipo usa reuniones breves —por ejemplo, Planning de 15–20 minutos y seguimiento diario de 5 minutos—. **Son duraciones elegidas para el simulacro**, no tiempos obligatorios establecidos por Scrum.

### 2.4. Artefactos y compromisos: qué debe existir y para qué

| Artefacto oficial | Su compromiso oficial | Qué muestra en este caso |
|---|---|---|
| **Product Backlog** | **Product Goal** | Trabajo ordenado para lograr el control de recursos |
| **Sprint Backlog** | **Sprint Goal** | Objetivo del sprint, elementos seleccionados y plan de ejecución |
| **Increment** | **Definition of Done** | Software integrado, utilizable y con la calidad acordada |

**Historia de usuario, Issue, rama, commit, Pull Request y tablero no son artefactos oficiales de Scrum.** Son convenciones y herramientas que el equipo utiliza para implementar y dejar trazabilidad de los artefactos.

Un commit prueba que se registró un cambio, **no** que una historia esté terminada. Un incremento puede existir antes de que finalice el sprint, siempre que cumpla la Definition of Done y sea utilizable. No basta con «tener código» si todavía falla, no está integrado o no satisface los criterios acordados.

---

## 3. Del Product Goal a un trabajo visible en GitHub

### 3.1. Product Backlog del ejemplo

Alex, como Product Owner, mantiene una lista ordenada de seis historias. El equipo conversa sobre ellas para precisar su alcance y sus criterios de aceptación.

| ID lógico | Historia de usuario resumida | Criterio de aceptación destacado | Prioridad inicial |
|---|---|---|---:|
| **HU-01** | Como responsable quiero registrar recursos para conocer cuáles existen | Rechaza códigos repetidos y cantidades no positivas | 1 |
| **HU-02** | Quiero listar y buscar recursos para conocer su inventario | Informa cuando no hay registros o no existe el código | 2 |
| **HU-03** | Quiero prestar una unidad para actualizar la disponibilidad | Nunca permite prestar sin existencias | 3 |
| **HU-04** | Quiero devolver una unidad para recuperar su disponibilidad | Nunca permite sobrepasar la cantidad total | 4 |
| **HU-05** | Quiero ver categorías y totales para interpretar el inventario | Las categorías no se duplican y los totales son correctos | 5 |
| **HU-06** | Quiero continuar usando el programa tras una entrada equivocada | Un texto no numérico no cierra el menú | 6 |

Una historia describe **una necesidad y su valor**. En cambio, «implementar `buscarPorCodigo()`» o «crear `Recurso.java`» es una **tarea técnica**, útil para el plan de desarrollo, pero no equivale a una historia de usuario.

### 3.2. Cómo queda descrita una historia real

Ejemplo de **HU-03 — Prestar una unidad**:

```markdown
# HU-03 — Prestar una unidad

Como responsable de los recursos,
quiero registrar el préstamo de una unidad,
para conocer cuántas unidades siguen disponibles.

## Criterios de aceptación
- [ ] Con tres unidades disponibles, prestar una deja dos.
- [ ] Con cero disponibles, no se concede otro préstamo.
- [ ] Un código desconocido se informa sin modificar el inventario.
- [ ] Un préstamo rechazado no cambia la disponibilidad.

## Tareas técnicas
- Implementar prestarUno() en Recurso.
- Incorporar la operación desde Inventario/Main.
- Comprobar los casos normal, sin existencias y código inexistente.
```

El número **HU-03** es una identificación lógica definida por el equipo. GitHub asigna otro identificador numérico al Issue (por ejemplo, `#3`). No debe asumirse que ambos números siempre coinciden.

### 3.3. Definition of Done del caso

El equipo establece antes del desarrollo una **Definition of Done compartida**. Una historia queda `Done` solamente si:

1. Satisface todos sus criterios de aceptación.
2. El código Java compila y la función puede ejecutarse.
3. Se comprobaron los casos normales y los límites pertinentes.
4. Los cambios están revisados e integrados en `main`.
5. Los Issues y Pull Requests conservan referencias verificables.
6. El `README.md` sigue describiendo correctamente cómo ejecutar el proyecto.
7. El equipo puede explicar las decisiones y el código incorporado.

Esta lista es la **decisión de calidad del equipo para el simulacro**, no un texto universal que Scrum obligue a copiar. Su función es impedir que cada persona interprete «terminado» de una forma distinta.

### 3.4. Repositorio y tablero: herramientas de seguimiento

Se crea un repositorio llamado `recursotrack-demo`, con esta estructura prevista:

```text
recursotrack-demo/
├── README.md
├── Main.java
├── Recurso.java
├── Inventario.java
└── PRUEBAS.md
```

- `README.md`: objetivo del producto, integrantes, responsabilidades, instrucciones de compilación y enlaces.
- `Recurso.java`: datos y reglas que protegen las existencias de cada recurso.
- `Inventario.java`: colección de recursos, búsquedas y reportes.
- `Main.java`: menú, `Scanner`, interacción y mensajes.
- `PRUEBAS.md`: casos probados, resultados **observados** y referencias a historias.

En GitHub Projects se dispone una vista con estados:

```text
Backlog  →  Todo  →  In Progress  →  In Review  →  Done
```

Estas columnas **son una convención del equipo** y no una estructura exigida por la Guía Scrum. La columna muestra estado actual; el Sprint Goal y las decisiones se documentan por separado.

Cada historia tiene su **Issue** y se vincula al Project. Las subtareas pueden describirse dentro del Issue o como sub-issues, según el nivel de seguimiento necesario. Un campo `Sprint` o etiquetas `sprint-1`, `sprint-2`, `sprint-3` permiten identificar el ciclo. Si una historia pasa de un sprint a otro, se conserva en comentarios el resultado anterior; no se borra la historia de lo ocurrido.

### 3.5. Cómo se conectan planificación y código

En el caso, una historia sigue la cadena:

```text
Product Goal
    ↓
Product Backlog (historia + criterios)
    ↓
Sprint Backlog (Sprint Goal + trabajo seleccionado + plan)
    ↓
GitHub Issue (decisiones y tareas)
    ↓
Rama y commits (cambios de implementación)
    ↓
Pull Request (revisión y resultados de pruebas)
    ↓
Integración en main
    ↓
Incremento demostrado + pruebas verificadas
    ↓
Estado Done, si cumple la Definition of Done
```

El objetivo **no es producir muchos registros**: es que cada cambio importante pueda explicarse desde una necesidad hasta su funcionamiento. Una historia no se termina solamente porque fue movida de columna.

### 3.6. Cómo se refleja la configuración en la interfaz de GitHub

Aunque los menús de GitHub pueden cambiar con el tiempo, la secuencia de trabajo es reconocible. El equipo realiza las siguientes operaciones **antes de iniciar el desarrollo de las historias**:

| Momento | Acción habitual en GitHub | Resultado visible |
|---|---|---|
| Crear el repositorio | **New repository**, nombre y `README.md` inicial | Un espacio compartido con una rama principal |
| Registrar las historias | **Issues → New issue**, título, descripción y criterios de aceptación | HU-01 a HU-06 con números asignados por GitHub |
| Organizar el producto | **Projects → New project**, vista **Board** | Tablero que reúne las historias |
| Mostrar el flujo | Configurar el campo **Status** con `Backlog`, `Todo`, `In Progress`, `In Review` y `Done` | Estado visible de cada elemento |
| Identificar el sprint | Utilizar etiquetas `sprint-1`, `sprint-2`, `sprint-3` o un campo propio | Se distingue el trabajo seleccionado en cada Sprint Backlog |
| Registrar tareas técnicas | Casillas en el Issue o sub-issues enlazados, según el tamaño del trabajo | Plan técnico conectado con la historia |
| Revisar implementación | **Pull requests → New pull request** desde una rama de trabajo hacia `main` | Conversación sobre cambios, pruebas y revisión |
| Cerrar trabajo | Integrar solo cuando los criterios y la Definition of Done se cumplen | Código en `main`, estado `Done` y trazabilidad verificable |

El **Product Goal**, los **Sprint Goals** y la **Definition of Done** se conservan como texto claro y accesible, por ejemplo en el `README.md` y en los registros de Planning enlazados desde el Project. El tablero por sí solo no contiene automáticamente esas decisiones.

La herramienta tampoco crea la conversación de la Daily Scrum ni realiza la Review: el equipo lleva a cabo los eventos y deja **un resumen breve de las decisiones relevantes** en los Issues de seguimiento. Esto evita confundir «crear un Issue sobre una reunión» con haber celebrado realmente el evento.

---

## 4. Cronología de los siete días

| Día | Sprint | Decisión o hecho principal | Evidencia que debería quedar |
|---|---|---|---|
| **1** | 1 | Pitch, Product Goal, Product Backlog, roles, Planning e inicio del código | Historias, tablero, Sprint Goal, primeros cambios |
| **2** | 1 | Seguimiento, integración, Review y Retrospective | Versión 1, pruebas, PRs y registro de cierre |
| **3** | 2 | Planning de préstamos y devoluciones; comienzo de implementación | Sprint Backlog 2, tareas y cambios |
| **4** | 2 | Se detecta devolución incorrecta; Review y Retrospective | HU-03 terminada, HU-04 pendiente e impedimento visible |
| **5** | 3 | Replanificación de HU-04 y desarrollo de reportes | Sprint Backlog 3, nuevas tareas y revisiones |
| **6** | 3 | Manejo de errores, integración y pruebas de límite | HU-04, HU-05 y HU-06 verificables |
| **7** | 3 | Verificación final, Review, Retrospective, PDF y sustentación grupal | Versión final e informe entregados ese mismo día |

Las fechas reales deben quedar registradas automáticamente por las herramientas durante un proyecto verdadero. Este cuadro solo representa **días numerados del simulacro**, nunca registros históricos.

---

## 5. Sprint 1: el primer incremento (días 1 y 2)

### Día 1: Sprint Planning

**Sprint Goal:** «Permitir registrar y consultar los recursos compartidos desde un programa ejecutable».

¿Por qué este objetivo? Porque aún no tiene sentido desarrollar préstamos si el programa no puede crear y encontrar los recursos a los que se aplicarán.

Los Developers seleccionan **HU-01** y **HU-02** junto con el Product Owner. Convierten las historias en un plan de construcción:

| Historia | Trabajo técnico acordado | Comprobación necesaria |
|---|---|---|
| HU-01 | Crear `Recurso`, constructor con validaciones e incorporación a `Inventario` | Registro válido, código repetido, cantidad incorrecta |
| HU-02 | Crear listado, búsqueda por código y primeras opciones de `Main` | Inventario vacío, búsqueda exitosa y búsqueda sin resultado |

El **Sprint Backlog** es el objetivo, los elementos elegidos y ese plan. No es solamente la etiqueta `sprint-1`.

### Primer diseño de objetos

El equipo comienza separando dos responsabilidades. `Recurso` conoce los datos de cada elemento y su cantidad; `Inventario` conserva varios recursos y los busca.

Extracto de `Recurso.java`:

```java
public class Recurso {
    private final String codigo;
    private final String nombre;
    private final String categoria;
    private final int cantidadTotal;
    private int cantidadDisponible;

    public Recurso(String codigo, String nombre, String categoria, int cantidadTotal) {
        if (codigo == null || codigo.isBlank()
                || nombre == null || nombre.isBlank()
                || categoria == null || categoria.isBlank()
                || cantidadTotal <= 0) {
            throw new IllegalArgumentException("Datos del recurso inválidos");
        }
        this.codigo = codigo.trim();
        this.nombre = nombre.trim();
        this.categoria = categoria.trim();
        this.cantidadTotal = cantidadTotal;
        this.cantidadDisponible = cantidadTotal;
    }
    // Métodos de consulta y reglas de préstamo se completan por incrementos.
}
```

**Por qué está construido así:** los atributos `private` evitan accesos directos desde el menú; `final` protege los datos que no deben reasignarse; `cantidadDisponible` cambiará cuando se preste o devuelva una unidad; el constructor impide crear recursos con datos inválidos; `this` distingue el atributo del parámetro.

Extracto de `Inventario.java`:

```java
private final List<Recurso> recursos = new ArrayList<>();

public boolean agregar(Recurso nuevo) {
    if (nuevo == null || buscarPorCodigo(nuevo.getCodigo()) != null) {
        return false;
    }
    return recursos.add(nuevo);
}

public Recurso buscarPorCodigo(String codigo) {
    if (codigo == null) return null;
    for (Recurso recurso : recursos) {
        if (recurso.getCodigo().equalsIgnoreCase(codigo.trim())) {
            return recurso;
        }
    }
    return null;
}
```

**Por qué:** `List<Recurso>` mantiene múltiples objetos; `ArrayList` permite agregar y recorrer recursos; el `for-each` busca por código; `equalsIgnoreCase` compara contenido textual ignorando mayúsculas; `buscarPorCodigo()` devuelve `null` cuando no encuentra un elemento y quien lo invoque debe atender ese resultado.

`Main` incorpora `Scanner`, un menú con `switch` y un ciclo que permite repetir operaciones. En esta primera versión se verifican las funciones incluidas en HU-01 y HU-02; la recuperación completa ante entradas numéricas incorrectas todavía pertenece a HU-06 y se desarrollará posteriormente.

### Cómo queda la primera historia en GitHub

**Ejemplo totalmente ilustrativo:** supongamos que GitHub asigna el Issue `#1` a HU-01. La rama podría ser `feature/1-registro-recursos`. Los cambios se registran progresivamente, no en un único commit final.

```bash
git switch main
git pull
git switch -c feature/1-registro-recursos
# Después de modificar y probar los archivos pertinentes:
git add Recurso.java Inventario.java Main.java
git commit -m "feat: registrar recursos y validar cantidades (#1)"
git push -u origin feature/1-registro-recursos
```

Después se abre un **Pull Request** contra `main`. Su descripción señala la historia relacionada, explica el comportamiento agregado y consigna los casos de prueba. Si ya satisface los criterios y la Definition of Done, la descripción puede incluir `Closes #1`; al fusionarse con la rama predeterminada, GitHub puede cerrar automáticamente ese Issue.

En HU-02 se sigue un recorrido equivalente con otra rama y otro PR. Que ambos PRs estén integrados permite probar **un incremento conjunto** que registra y consulta recursos.

### Día 2: Daily Scrum y demostración

El equipo realiza una Daily Scrum breve. El registro del ejemplo dice:

> **Día 2 simulado:** HU-01 registra recursos y rechaza códigos repetidos. HU-02 lista recursos, pero faltaba comunicar que una búsqueda no produjo resultados. Se acuerda resolver ese caso antes de solicitar la revisión final.

El mensaje es útil porque registra **el estado respecto del Sprint Goal y la decisión tomada**, no porque todos hayan recitado un reporte personal.

En la **Sprint Review**, el equipo demuestra desde consola:

```text
1. Registrar recurso: R1 / Proyector / Equipos / 3  → aceptado
2. Registrar nuevamente el código R1                → rechazado
3. Listar recursos                                  → aparece R1
4. Buscar R9                                        → no encontrado
```

Supongamos que ambos Issues cumplen sus criterios y la Definition of Done. El **Incremento 1** se ejecuta desde `main` y sirve para registrar y consultar recursos. El interesado sugiere mostrar también disponibilidad durante la consulta; el Product Owner considera la sugerencia al ordenar las historias siguientes.

En la **Sprint Retrospective** el equipo reconoce que inicialmente dejó las pruebas para el último momento. Acuerda una mejora concreta: **comprobar los límites de una historia antes de enviar su PR a revisión**.

**Estado al terminar Sprint 1:** HU-01 y HU-02 `Done`. La versión integrada funciona en el alcance comprometido; los préstamos todavía no están disponibles.

---

## 6. Sprint 2: un resultado parcial que obliga a adaptar (días 3 y 4)

### Día 3: Sprint Planning

**Sprint Goal:** «Gestionar préstamos y devoluciones sin perder consistencia en las existencias».

Se seleccionan **HU-03** y **HU-04**. Los Developers acuerdan usar métodos de instancia en `Recurso`, porque ese objeto debe proteger la regla de que sus existencias no pueden ser negativas ni superar el total.

La operación de préstamo propuesta es:

```java
public boolean prestarUno() {
    if (cantidadDisponible == 0) {
        return false;
    }
    cantidadDisponible--;
    return true;
}
```

El retorno `boolean` permite a la capa de interacción informar éxito o rechazo. La regla se comprueba **dentro del objeto**, no exclusivamente en `Main`, de modo que otra parte del programa tampoco pueda dejar existencias negativas llamando a ese método.

### Un defecto de devolución que la primera revisión no detectó

Una versión inicial de HU-04 contiene este código incorrecto:

```java
// Versión defectuosa del Sprint 2: NO debe considerarse terminada.
public boolean devolverUno() {
    cantidadDisponible++;
    return true;
}
```

El defecto aparece con un recurso que tiene **2 unidades totales**:

| Operación | Disponibles con el código defectuoso | Resultado correcto esperado |
|---|---:|---:|
| Registrar el recurso | 2 | 2 |
| Prestar una unidad | 1 | 1 |
| Devolver una unidad | 2 | 2 |
| Intentar devolver otra unidad | **3** | **Rechazo, permanecen 2** |

El error no es cosmético: vulnera una **regla de negocio** y hace que la disponibilidad deje de representar las existencias reales.

### Día 4: inspección y decisión de no ocultar el problema

La Daily Scrum y las verificaciones muestran lo siguiente:

| Historia | Estado real | Decisión del equipo |
|---|---|---|
| HU-03 — Préstamo | Cumple criterios; código probado e integrado | Marcar `Done` |
| HU-04 — Devolución | No supera la prueba de límite y no cumple Definition of Done | Mantener incompleta |

El Sprint Goal apuntaba a préstamos **y** devoluciones consistentes. Por tanto, aunque HU-03 aporta un incremento utilizable, el equipo reconoce que **el Sprint Goal no se alcanzó completamente**.

En la **Sprint Review** demuestra el préstamo correcto y explica la devolución pendiente. El Product Owner mantiene la corrección de HU-04 como prioridad antes de aceptar reportes más elaborados. La historia sigue existiendo en el Product Backlog y podrá seleccionarse en el siguiente Sprint Planning: no se «traslada automáticamente» ni se declara terminada para embellecer el tablero.

La **Sprint Retrospective** identifica la causa de proceso: solo se había probado el camino normal de devolución. Se acuerda **definir pruebas de límite antes de completar una implementación y pedir revisión cruzada**.

**Estado al terminar Sprint 2:** HU-03 `Done`; HU-04 pendiente. El registro deja claro que existió trabajo sobre HU-04, pero no un incremento terminado correspondiente a ella.

Esta situación permite distinguir los tres pilares:

- **Transparencia:** se muestra el defecto y no se cierra el Issue.
- **Inspección:** se ejecuta una secuencia que revela que el total puede excederse.
- **Adaptación:** se revisa la prioridad y se mejora el proceso de pruebas.

---

## 7. Sprint 3: recuperar trabajo pendiente y cerrar el producto (días 5, 6 y 7)

### Día 5: nueva Sprint Planning

El Product Owner confirma que la devolución correcta sigue siendo prioritaria. Los Developers seleccionan **HU-04, HU-05 y HU-06**, considerando la capacidad disponible.

**Sprint Goal:** «Completar un control de existencias confiable y ofrecer consultas útiles sin interrumpirse ante errores de entrada».

HU-04 conserva en su Issue el comentario que explica **por qué no cumplió la Definition of Done en Sprint 2**. El nuevo Sprint Backlog la selecciona otra vez, pero no borra el registro del sprint anterior.

### Corregir HU-04

La regla correcta es comprobar el límite antes de incrementar:

```java
public boolean devolverUno() {
    if (cantidadDisponible >= cantidadTotal) {
        return false;
    }
    cantidadDisponible++;
    return true;
}
```

La prueba que fallaba ahora debe devolver `false` cuando se intenta exceder la cantidad total, sin cambiar el estado. Esa comprobación queda asociada al Issue y al Pull Request correspondiente.

### HU-05: colecciones elegidas por una necesidad

`Inventario` ya utiliza una `List<Recurso>` para conservar objetos. El reporte necesita dos vistas distintas:

**Categorías únicas con `Set`:**

```java
public Set<String> obtenerCategorias() {
    Set<String> categorias = new HashSet<>();
    for (Recurso recurso : recursos) {
        categorias.add(recurso.getCategoria());
    }
    return categorias;
}
```

`HashSet` evita elementos iguales; **no garantiza orden de presentación**. En esta versión las categorías se comparan como textos sensibles a mayúsculas; normalizar `Equipos` y `equipos` sería una mejora futura si el producto lo necesitara.

**Disponibilidad por categoría con `Map`:**

```java
public Map<String, Integer> disponiblesPorCategoria() {
    Map<String, Integer> totales = new HashMap<>();
    for (Recurso recurso : recursos) {
        String categoria = recurso.getCategoria();
        int anterior = totales.getOrDefault(categoria, 0);
        totales.put(categoria, anterior + recurso.getCantidadDisponible());
    }
    return totales;
}
```

`HashMap` relaciona cada categoría con una suma de unidades. La parte clave es `anterior + ...`: sin esa acumulación, registrar otro recurso de la misma categoría **sobrescribiría** el valor anterior. `HashMap` tampoco garantiza un orden al imprimir los resultados.

**Ejemplo comprobable:** con R1 en `Equipos`, total 3 y una unidad prestada, y R2 en `Equipos`, total 2 y ninguna prestada, la categoría debe mostrar **4 disponibles**, no 2 ni 5.

### HU-06: el programa no termina por escribir un valor equivocado

`Main` lee una opción como texto e intenta convertirla. Una entrada como `hola` produce un `NumberFormatException`; el programa debe informar el problema y solicitar otro valor.

```java
private int leerEntero(String mensaje) {
    while (true) {
        System.out.print(mensaje);
        String texto = scanner.nextLine();
        try {
            return Integer.parseInt(texto.trim());
        } catch (NumberFormatException e) {
            System.out.println("Introduce un número entero válido.");
        }
    }
}
```

El `while` tiene una función concreta: **repetir hasta obtener un entero válido**. El `do while` del menú, en cambio, servirá para mostrar al menos una vez las opciones y repetirlas hasta elegir salir. Las reglas de dominio, como cantidad positiva, siguen verificándose además en `Recurso`.

El proyecto utiliza un arreglo `String[]` para las opciones del menú y un `for` para imprimirlo. Así se incorpora un arreglo con una finalidad visible y se distingue de la `List` dinámica que almacena recursos.

### Día 6: integración y verificación

Con HU-04 corregida y las funciones de reportes y lectura segura integradas, el equipo ejecuta los casos de prueba. El archivo `PRUEBAS.md` registra **resultado esperado, resultado observado y referencia al Issue**.

| Caso | Entrada o acción | Resultado esperado | Resultado simulado |
|---|---|---|---|
| P-01 | Consultar inventario vacío | Mensaje claro | Correcto |
| P-02 | Registrar R1 con 3 unidades | Quedan 3 disponibles | Correcto |
| P-03 | Registrar de nuevo el código R1 | Rechazo | Correcto |
| P-04 | Prestar una unidad de R1 | Quedan 2 | Correcto |
| P-05 | Devolver esa unidad | Quedan 3 | Correcto |
| P-06 | Devolver de nuevo sin préstamo pendiente | Rechazo; quedan 3 | Correcto |
| P-07 | Intentar prestar más que las disponibles | Rechazo sin alterar estado | Correcto |
| P-08 | Escribir `hola` como opción | Mensaje y nueva lectura | Correcto |
| P-09 | Buscar código inexistente | Mensaje informativo | Correcto |
| P-10 | Dos recursos en la misma categoría | Una categoría en el `Set` | Correcto |
| P-11 | R1 con 2 disponibles y R2 con 2, ambos en Equipos | Reporte Equipos = 4 | Correcto |
| P-12 | Elegir opción 0 | Salida normal | Correcto |

La columna final expresa **el resultado asumido en esta simulación**, no pruebas que una persona haya ejecutado en un repositorio real. En un desarrollo real no se escribe «Correcto» hasta comprobarlo.

El equipo revisa el estado integrado de `main`, no solamente el código que funciona en una rama local. Cada Pull Request describe su cambio, evidencia de pruebas y relación con el Issue; después de verificar Definition of Done, se actualiza el estado a `Done`.

### Día 7: cierre completo, sin extender el calendario

El día 7 incluye cuatro momentos, en este orden:

1. **Verificación final del incremento:** se compila y se demuestra la aplicación integrada.
2. **Sprint Review:** se presentan registros, préstamos, devoluciones, categorías, reporte y manejo de errores; se inspecciona lo terminado frente al Product Goal.
3. **Sprint Retrospective:** se evalúa cómo mejoraron las pruebas de límite, la revisión cruzada y la coordinación, y se registra una mejora útil para trabajos posteriores.
4. **Entrega y sustentación grupal:** se entrega el PDF y se demuestra el producto; cualquier integrante puede explicar una historia, un archivo o un cambio.

No hay un «día 8 para terminar el informe» y no existe una presentación individual adicional.

### Resultado de los tres sprints

| Sprint | Días | Historias consideradas | Terminadas al cierre | Resultado verificable |
|---|---|---|---|---|
| **1** | 1–2 | HU-01, HU-02 | HU-01, HU-02 | Registro y consulta |
| **2** | 3–4 | HU-03, HU-04 | HU-03 | Préstamos correctos; devolución aún pendiente |
| **3** | 5–7 | HU-04, HU-05, HU-06 | HU-04, HU-05, HU-06 | Devoluciones, reportes y entradas seguras |

Al cierre hay **6 historias únicas terminadas**, no 7. HU-04 aparece en la planificación de dos sprints porque fue replanificada; **no se cuenta dos veces como funcionalidad terminada**.

---

## 8. Cómo se vería la trazabilidad completa en GitHub

### 8.1. Correspondencia entre hechos y registros

La siguiente tabla es un **ejemplo sintético**. Los números `#1` a `#6` para Issues y `#10` a `#15` para PRs se usan solo para visualizar las relaciones. En un repositorio auténtico serían los números que GitHub asignara.

| Historia | Issue simulado | Rama ilustrativa | PR simulado | Sprint de terminación | Resultado |
|---|---|---|---|---|---|
| HU-01 | `#1` | `feature/1-registro-recursos` | `#10` | 1 | Done |
| HU-02 | `#2` | `feature/2-consultas` | `#11` | 1 | Done |
| HU-03 | `#3` | `feature/3-prestamos` | `#12` | 2 | Done |
| HU-04 | `#4` | `feature/4-devoluciones` | `#13` | **3** | Pendiente en 2; Done en 3 |
| HU-05 | `#5` | `feature/5-reportes` | `#14` | 3 | Done |
| HU-06 | `#6` | `feature/6-entradas-seguras` | `#15` | 3 | Done |

**Importante:** una rama puede tener avances durante Sprint 2 y terminar integrándose en Sprint 3. La tabla registra el **sprint en el que la historia satisface Definition of Done**, no pretende decir que todo su código se escribió en ese mismo periodo.

### 8.2. Reporte de tareas: planificación frente a resultado

Para que el seguimiento no se reduzca a seis historias generales, el equipo identifica **doce tareas técnicas únicas**. Esta tabla representa los estados **al cierre de cada sprint**, no una lista de tareas que se hubieran creado en una fecha real.

| Tarea | Historia | Trabajo técnico | Plan inicial | Situación al cierre de ese sprint | Cierre final |
|---|---|---|---|---|---|
| T-01 | HU-01 | Modelar `Recurso` | Sprint 1 | Done | Sprint 1 |
| T-02 | HU-01 | Validar y registrar recursos | Sprint 1 | Done | Sprint 1 |
| T-03 | HU-02 | Listar recursos | Sprint 1 | Done | Sprint 1 |
| T-04 | HU-02 | Buscar recursos e integrar menú inicial | Sprint 1 | Done | Sprint 1 |
| T-05 | HU-03 | Implementar préstamo seguro | Sprint 2 | Done | Sprint 2 |
| T-06 | HU-03 | Probar límites del préstamo | Sprint 2 | Done | Sprint 2 |
| T-07 | HU-04 | Implementar devolución con límite | Sprint 2 | **Incompleta** | Sprint 3 |
| T-08 | HU-04 | Verificar casos límite de devolución | Sprint 2 | **Prueba fallida** | Sprint 3 |
| T-09 | HU-05 | Obtener categorías únicas con `Set` | Sprint 3 | Done | Sprint 3 |
| T-10 | HU-05 | Agrupar disponibilidades con `Map` | Sprint 3 | Done | Sprint 3 |
| T-11 | HU-06 | Repetir entrada numérica tras error | Sprint 3 | Done | Sprint 3 |
| T-12 | HU-06 | Integrar, verificar y documentar manejo de errores | Sprint 3 | Done | Sprint 3 |

El estado de T-07 y T-08 **no se reescribe como `Done` en Sprint 2**. Los registros del Sprint 2 conservan el defecto y la prueba fallida; en Sprint 3 se verifica su corrección y allí consta el cierre. Esto permite distinguir **esfuerzo realizado**, **trabajo terminado** y **trabajo replanificado**.

El resumen de tareas por sprint quedaría así:

| Sprint | Tareas contempladas | Terminadas durante el sprint | No terminadas al cierre |
|---|---:|---:|---:|
| 1 | 4 | 4 | 0 |
| 2 | 4 | 2 | 2 |
| 3 | 6 (incluye 2 pendientes del anterior) | 6 | 0 |

En consecuencia, hay **12 tareas únicas**, todas completadas al final. Las filas de trabajo planificado suman **14 apariciones**, porque dos tareas se consideraron en más de un sprint. No son catorce tareas diferentes.

### 8.3. Un Pull Request comprensible

Ejemplo de descripción para HU-04 una vez corregida:

```markdown
## Objetivo
Corregir la devolución para impedir que la cantidad disponible
supere la cantidad total del recurso.

## Historia relacionada
Closes #4

## Cambios
- Validación del límite en Recurso.devolverUno().
- Mensaje al rechazar la operación desde Main.
- Casos de prueba normal y devolución excesiva.

## Comprobación
- Con total=2 y disponibles=1, devolver deja disponibles=2.
- Volver a devolver retorna false y mantiene disponibles=2.
- El programa compila y sigue aceptando operaciones.

## Decisión
HU-04 no terminó en Sprint 2 por una prueba de límite fallida.
Se completa en Sprint 3 tras corregir y comprobar el defecto.
```

En GitHub, `Closes #4` puede cerrar el Issue al fusionarse el PR con la rama predeterminada. Por eso **no se usa como atajo para cerrar una historia que aún no cumple su Definition of Done**. Para relacionar trabajo parcial sin prometer cierre puede utilizarse `Refs #4`.

### 8.4. Un registro de seguimiento útil

El equipo conserva un Issue de seguimiento por sprint o comentarios vinculados, sin repetir la misma información en múltiples archivos. Este sería un comentario del cierre del Sprint 2:

```markdown
## Sprint 2 — Día 4 (SIMULADO)
Sprint Goal: gestionar préstamos y devoluciones sin inconsistencias.

Avance verificado:
- HU-03: criterios cumplidos; integrada.
- HU-04: el caso de devolución excesiva falla; NO terminada.

Decisión de la Review:
- Mantener HU-04 en Product Backlog como prioridad alta.
- Considerar HU-04 en la siguiente Planning.

Mejora de la Retrospective:
- Especificar primero casos de límite.
- Pedir revisión cruzada antes de fusionar el PR.
```

La trazabilidad no requiere documentar cada minuto del trabajo. Requiere poder distinguir **qué se planeó, qué sucedió, qué cambió y por qué**.

### 8.5. Qué sería una inconsistencia

Estas situaciones impedirían confiar en el seguimiento:

| Registro | Problema que revela |
|---|---|
| Issue en `Done`, pero la prueba falla | El estado contradice la Definition of Done |
| Informe dice «HU-04 terminada en Sprint 2» | Contradice el impedimento registrado durante la Review |
| PR sin referencia al Issue | Se dificulta conectar el cambio con la necesidad |
| Todo el trabajo aparece subido al final sin seguimiento previo | No permite verificar el proceso incremental declarado |
| El reporte de tareas muestra siete historias terminadas | Cuenta dos veces HU-04 o infla el resultado |
| Un integrante no sabe qué modificó un commit del equipo | Falta apropiación técnica compartida |

---

## 9. El informe PDF final: resumen del proceso, no duplicación de GitHub

En el simulacro, el informe final tiene una estructura breve. Para una actividad integradora con esta escala se puede tomar como referencia una extensión máxima de **seis páginas**.

| Parte | Información que contiene | De dónde se obtiene |
|---|---|---|
| 1. Identificación | Producto, equipo, responsabilidades, Product Goal, enlaces | `README.md` y Product Backlog |
| 2. Planificación | Historias, Sprint Goals y Definition of Done | Issues y registros de Planning |
| 3. Sprint 1 | Planificado, terminado, Review y Retrospective | Project, Issues, PRs y comentarios |
| 4. Sprint 2 | HU-03 terminada, HU-04 pendiente, defecto y decisiones | Pruebas, seguimiento e Issue de HU-04 |
| 5. Sprint 3 | HU-04 replanificada, HU-05 y HU-06, pruebas y cierre | PRs, tablero y Review |
| 6. Consolidado | Métricas reales, dificultades, mejoras y conclusiones | Registros anteriores |

### Extracto de reporte consolidado (simulado)

| Indicador | Valor correcto para este caso |
|---|---:|
| Historias únicas del Product Backlog | 6 |
| Historias únicas terminadas al día 7 | 6 |
| Historias sin terminar al cierre | 0 |
| Historias replanificadas entre sprints | 1 (HU-04) |
| Tareas técnicas únicas | 12 |
| Tareas terminadas al día 7 | 12 |
| Tareas pendientes al cierre del Sprint 2 | 2 (T-07 y T-08) |
| Sprints realizados | 3 |
| Duración total | 7 días |
| Casos de prueba del ejemplo | 12 |
| Casos de prueba satisfactorios en el ejemplo | 12 |

Las **doce tareas** proceden del reporte sintético de la sección 8.2. En un proyecto real, las cantidades se obtienen de los Issues o del Project, y deben coincidir con las tareas efectivamente registradas; **no se reemplazan por los valores de este ejemplo**.

El PDF resume el proceso y enlaza la fuente de evidencia. No hace falta volver a copiar todos los comentarios, cada commit o cada captura del tablero. El **pitch grupal no se incluye** en ese documento.

---

## 10. Sustentación grupal y comprensión del trabajo

La sustentación del **día 7** no consiste en leer el informe. Es una **demostración conjunta** de que el producto funciona y de que todos comprenden el proceso y las decisiones que lo produjeron.

Una posible revisión del caso recorrería estos puntos:

| Elemento que se selecciona | Explicación técnica o de proceso esperada |
|---|---|
| `Recurso.java` | Por qué protege el estado y por qué las reglas no viven solo en el menú |
| `Inventario.java` | Por qué utiliza `List`, `Set` y `Map` para necesidades distintas |
| `cantidadDisponible` | Por qué cambia y por qué nunca debe superar `cantidadTotal` |
| `devolverUno()` | Cómo evita el defecto descubierto en Sprint 2 |
| `leerEntero()` | Por qué el `catch` evita que la lectura errónea interrumpa el menú |
| HU-04 | Qué criterio falló, por qué no estaba Done y cómo se replanificó |
| PR de HU-04 | Qué cambió, cómo se comprobó y por qué se integró en Sprint 3 |
| Sprint Review 2 | Qué se inspeccionó y qué decisión de producto se tomó |
| Sprint Retrospective 2 | Qué mejora de proceso se propuso y cómo se aplicó |
| Informe PDF | De qué Issues, PRs y pruebas salieron sus cifras |

### Si se usa inteligencia artificial

La inteligencia artificial puede ayudar a entender un concepto, proponer alternativas o encontrar defectos. **No convierte automáticamente sus respuestas en decisiones justificadas ni en código comprobado.**

El criterio de apropiación es el mismo para **cada archivo, variable, método, estructura, historia, tarea, commit, Pull Request y decisión**: cualquier integrante debe explicar **qué es, por qué está ahí, cómo funciona, qué alternativas había y qué ocurriría si se cambiara**. También debe poder demostrarlo con el programa o los registros.

No se exige que cada persona haya escrito toda la aplicación, sino que **nadie dependa de respuestas como «esa parte la hizo otra persona» o «la IA lo generó»**. La sustentación sigue siendo grupal; no se crea una exposición individual adicional.

---

## 11. Guía para reproducir este flujo en otro proyecto

RecursoTrack es un ejemplo, no una plantilla temática obligatoria. La misma lógica puede aplicarse a ventas, inventario, reservas, registros o seguimiento de tareas, siempre que el producto sea pequeño y se pueda completar dentro del plazo.

La secuencia de trabajo reproducible es:

**Primero, definir el valor.** Formular el problema, presentar una idea mediante un pitch breve y establecer un Product Goal que describa qué mejora tendrá el usuario. Evitar que la primera decisión sea escoger nombres de clases.

**Después, organizar el trabajo.** Construir un Product Backlog ordenado, escribir criterios de aceptación observables, fijar la Definition of Done, acordar las responsabilidades y crear los Issues y el Project. Todos los integrantes participan como Developers en el contexto de esta actividad.

**Durante cada sprint, trabajar con evidencia.** Establecer el Sprint Goal y el Sprint Backlog, desarrollar en ramas, registrar commits comprensibles, revisar cambios mediante Pull Requests, comprobar comportamientos y actualizar los estados de acuerdo con la realidad. Daily Scrum permite ajustar el plan; Sprint Review inspecciona el producto y Sprint Retrospective mejora el proceso.

**Si algo no termina, mostrarlo.** Una historia inconclusa conserva su estado real y el motivo. El Product Owner considera su nueva prioridad y los Developers deciden en la siguiente Planning si cabe seleccionarla. No se reescribe la historia de un sprint anterior.

**Finalmente, demostrar coherencia.** El incremento que funciona, los Issues, el tablero, los PRs, las pruebas y el resumen del PDF deben contar la misma historia. La entrega y la sustentación del proyecto integrador ocurren **dentro del día 7**.

### Relación con los cuatro laboratorios de Java

| Tema previamente estudiado | Lugar visible en el caso |
|---|---|
| Variables, tipos de datos, constantes y operadores | Cantidades de recursos, comparaciones y modificaciones de existencias |
| `Scanner`, condiciones y `switch` | Lectura de datos, mensajes, validaciones y opciones del menú |
| `for`, `while` y `do while` | Impresión del menú, lectura segura y permanencia en ejecución |
| Métodos con parámetros y valores de retorno | Operaciones como `buscarPorCodigo()`, `prestarUno()` y `leerEntero()` |
| Clases, objetos, constructores, `this` y encapsulamiento | `Recurso`, `Inventario` y sus reglas de negocio |
| Arreglos y colecciones | `String[]` del menú, `List`, `ArrayList`, `Set`, `HashSet`, `Map` y `HashMap` |
| Excepciones | `IllegalArgumentException`, `NumberFormatException`, `try/catch` y cierre de recursos |

---

## Anexo A. Versión final coherente del producto simulado

Los siguientes tres archivos muestran **cómo podría verse el incremento final del día 7**. Se incluyen como referencia técnica para entender la evolución narrada: **no significa que este código hubiera estado disponible o terminado durante el Sprint 1 o 2**.

No necesita Maven, base de datos ni bibliotecas externas. Usa Java 21 y la biblioteca estándar.

### `Recurso.java`

```java
public class Recurso {
    private final String codigo;
    private final String nombre;
    private final String categoria;
    private final int cantidadTotal;
    private int cantidadDisponible;

    public Recurso(String codigo, String nombre, String categoria, int cantidadTotal) {
        if (codigo == null || codigo.isBlank()) {
            throw new IllegalArgumentException("El código es obligatorio.");
        }
        if (nombre == null || nombre.isBlank()) {
            throw new IllegalArgumentException("El nombre es obligatorio.");
        }
        if (categoria == null || categoria.isBlank()) {
            throw new IllegalArgumentException("La categoría es obligatoria.");
        }
        if (cantidadTotal <= 0) {
            throw new IllegalArgumentException("La cantidad debe ser positiva.");
        }
        this.codigo = codigo.trim();
        this.nombre = nombre.trim();
        this.categoria = categoria.trim();
        this.cantidadTotal = cantidadTotal;
        this.cantidadDisponible = cantidadTotal;
    }

    public String getCodigo() { return codigo; }
    public String getNombre() { return nombre; }
    public String getCategoria() { return categoria; }
    public int getCantidadTotal() { return cantidadTotal; }
    public int getCantidadDisponible() { return cantidadDisponible; }

    public boolean prestarUno() {
        if (cantidadDisponible == 0) return false;
        cantidadDisponible--;
        return true;
    }

    public boolean devolverUno() {
        if (cantidadDisponible >= cantidadTotal) return false;
        cantidadDisponible++;
        return true;
    }
}
```

### `Inventario.java`

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

public class Inventario {
    private final List<Recurso> recursos = new ArrayList<>();

    public boolean agregar(Recurso nuevo) {
        if (nuevo == null || buscarPorCodigo(nuevo.getCodigo()) != null) {
            return false;
        }
        return recursos.add(nuevo);
    }

    public Recurso buscarPorCodigo(String codigo) {
        if (codigo == null) return null;
        for (Recurso recurso : recursos) {
            if (recurso.getCodigo().equalsIgnoreCase(codigo.trim())) {
                return recurso;
            }
        }
        return null;
    }

    public List<Recurso> listar() {
        return new ArrayList<>(recursos);
    }

    public Set<String> obtenerCategorias() {
        Set<String> categorias = new HashSet<>();
        for (Recurso recurso : recursos) {
            categorias.add(recurso.getCategoria());
        }
        return categorias;
    }

    public Map<String, Integer> disponiblesPorCategoria() {
        Map<String, Integer> totales = new HashMap<>();
        for (Recurso recurso : recursos) {
            String categoria = recurso.getCategoria();
            int anterior = totales.getOrDefault(categoria, 0);
            totales.put(categoria, anterior + recurso.getCantidadDisponible());
        }
        return totales;
    }
}
```

### `Main.java`

```java
import java.util.List;
import java.util.Map;
import java.util.Scanner;

public class Main {
    private final Inventario inventario = new Inventario();
    private final Scanner scanner = new Scanner(System.in);

    public static void main(String[] args) {
        new Main().ejecutar();
    }

    private void ejecutar() {
        try {
            int opcion;
            do {
                mostrarMenu();
                opcion = leerEntero("Opción: ");
                switch (opcion) {
                    case 1 -> registrar();
                    case 2 -> listar();
                    case 3 -> buscar();
                    case 4 -> prestar();
                    case 5 -> devolver();
                    case 6 -> reporte();
                    case 0 -> System.out.println("Hasta pronto.");
                    default -> System.out.println("Opción no reconocida.");
                }
            } while (opcion != 0);
        } finally {
            scanner.close();
        }
    }

    private void mostrarMenu() {
        String[] opciones = {
            "1. Registrar recurso",
            "2. Listar recursos",
            "3. Buscar por código",
            "4. Prestar una unidad",
            "5. Devolver una unidad",
            "6. Categorías y reporte",
            "0. Salir"
        };
        System.out.println("\n=== RECURSOTRACK ===");
        for (int i = 0; i < opciones.length; i++) {
            System.out.println(opciones[i]);
        }
    }

    private int leerEntero(String mensaje) {
        while (true) {
            System.out.print(mensaje);
            String texto = scanner.nextLine();
            try {
                return Integer.parseInt(texto.trim());
            } catch (NumberFormatException e) {
                System.out.println("Introduce un número entero válido.");
            }
        }
    }

    private String leerTexto(String mensaje) {
        System.out.print(mensaje);
        return scanner.nextLine().trim();
    }

    private void registrar() {
        String codigo = leerTexto("Código: ");
        String nombre = leerTexto("Nombre: ");
        String categoria = leerTexto("Categoría: ");
        int cantidad = leerEntero("Cantidad total: ");
        try {
            Recurso recurso = new Recurso(codigo, nombre, categoria, cantidad);
            System.out.println(inventario.agregar(recurso)
                    ? "Recurso registrado." : "Ya existe ese código.");
        } catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }

    private void listar() {
        List<Recurso> recursos = inventario.listar();
        if (recursos.isEmpty()) {
            System.out.println("No hay recursos registrados.");
            return;
        }
        for (Recurso recurso : recursos) {
            System.out.printf("%s | %s | %s | disponibles: %d/%d%n",
                    recurso.getCodigo(), recurso.getNombre(), recurso.getCategoria(),
                    recurso.getCantidadDisponible(), recurso.getCantidadTotal());
        }
    }

    private void buscar() {
        Recurso recurso = inventario.buscarPorCodigo(leerTexto("Código: "));
        if (recurso == null) {
            System.out.println("Recurso no encontrado.");
        } else {
            System.out.printf("%s: %d de %d disponibles%n", recurso.getNombre(),
                    recurso.getCantidadDisponible(), recurso.getCantidadTotal());
        }
    }

    private void prestar() {
        Recurso recurso = inventario.buscarPorCodigo(leerTexto("Código: "));
        if (recurso == null) {
            System.out.println("Recurso no encontrado.");
        } else {
            System.out.println(recurso.prestarUno()
                    ? "Préstamo registrado." : "Sin unidades disponibles.");
        }
    }

    private void devolver() {
        Recurso recurso = inventario.buscarPorCodigo(leerTexto("Código: "));
        if (recurso == null) {
            System.out.println("Recurso no encontrado.");
        } else {
            System.out.println(recurso.devolverUno()
                    ? "Devolución registrada." : "No hay unidades pendientes de devolver.");
        }
    }

    private void reporte() {
        System.out.println("Categorías: " + inventario.obtenerCategorias());
        for (Map.Entry<String, Integer> entrada
                : inventario.disponiblesPorCategoria().entrySet()) {
            System.out.printf("%s: %d disponibles%n",
                    entrada.getKey(), entrada.getValue());
        }
    }
}
```

### Compilación y ejecución

En una misma carpeta que contenga los tres archivos:

```bash
javac Recurso.java Inventario.java Main.java
java Main
```

**Decisión técnica a destacar:** `Inventario.listar()` devuelve una **nueva lista**, por lo que el código externo no puede agregar o eliminar elementos de la colección interna a través de ella. Sin embargo, los objetos `Recurso` de esa copia son los mismos objetos y sus operaciones públicas siguen siendo accesibles. La integridad de las cantidades depende de las validaciones de `Recurso`. Es importante comprender esta diferencia en lugar de afirmar que se construyó una «copia profunda».

El `finally` de `Main` cierra el `Scanner` cuando termina el programa. En esta aplicación cerrarlo al finalizar es razonable, pues ya no se volverá a leer la entrada estándar.

---

## Cierre: qué enseña realmente la simulación

El punto decisivo del caso no es que RecursoTrack termine con seis historias. Es que **se puede reconstruir cómo llegó a ese resultado**:

**Idea → Product Goal → Product Backlog → Sprint Goal y Sprint Backlog → desarrollo y seguimiento → incremento verificable → Sprint Review → Sprint Retrospective → adaptación → entrega y sustentación grupal.**

HU-04 recuerda una regla esencial: **una historia puede tener código, commits y horas de trabajo sin estar terminada**. La calidad acordada, las pruebas y el estado real del incremento determinan si cumple la Definition of Done.

GitHub hace visible ese recorrido; Scrum aporta la lógica para decidir, inspeccionar y adaptar. El informe resume lo ocurrido, pero no debe reemplazar los registros. Y todos los integrantes, incluso si comparten responsabilidades de Product Owner o Scrum Master, participan en el desarrollo y **deben poder explicar con criterio cualquier elemento del producto y de su historia de construcción**, haya sido escrito directamente o apoyado con inteligencia artificial.

### Referencias

- [Scrum Guide 2020 — guía oficial](https://scrumguides.org/scrum-guide.html)
- [Scrum Guide 2020 — traducción latinoamericana](https://scrumguides.org/docs/scrumguide/v2020/2020-Scrum-Guide-Spanish-Latin-South-American.pdf)
- [GitHub Docs — Acerca de los Issues](https://docs.github.com/es/issues/tracking-your-work-with-issues/learning-about-issues/about-issues)
- [GitHub Docs — Acerca de Projects](https://docs.github.com/es/issues/planning-and-tracking-with-projects/learning-about-projects/about-projects)
- [GitHub Docs — Vincular un Pull Request a un Issue](https://docs.github.com/es/issues/tracking-your-work-with-issues/using-issues/linking-a-pull-request-to-an-issue)
- [Laboratorios Java 1–4 — repositorio de referencia](https://github.com/ethan-fullstack/java_explorer_integrator_innovator)
