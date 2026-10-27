# Laboratorio 6 — Arquitectura Java: de la ejecución del programa a la memoria

En los [Laboratorios 1–4](../README.md) aprendiste a trabajar con variables, métodos, objetos y colecciones. También utilizaste `javac` para compilar y `java` para ejecutar programas. Pero ¿qué sucede entre guardar un archivo `.java` y ver una salida?, ¿qué ocurre cuando un método llama a otro?, ¿por qué dos variables pueden modificar el mismo objeto?, ¿y por qué reasignar un parámetro no cambia la variable del método que hizo la llamada?

En este laboratorio vas a estudiar la arquitectura de **Java 21** mediante programas que podrás ejecutar y depurar. Comenzarás por el JDK, el JRE y la JVM; continuarás con bytecode, llamadas a métodos, Stack y Heap; y terminarás con un **caso de estudio real de PetMatch Community**, una aplicación desarrollada con Spring Boot. Para realizar las prácticas **no necesitas ejecutar PetMatch ni instalar Spring Boot, Maven o una base de datos**.

| Práctica | Tema |
|---|---|
| 6.1 | JDK, JRE y JVM: herramientas y responsabilidades |
| 6.2 | Compilación, bytecode y ciclo de ejecución |
| 6.3 | Stack: llamadas a métodos, parámetros y variables locales |
| 6.4 | Heap: objetos, arreglos, identidad y referencias |
| 6.5 | Paso de parámetros y caso real de PetMatch |
| 6.6 | Reto integrador, depuración y diagnóstico |

> **Forma de trabajo:** lee cada explicación, **predice** el resultado, **ejecuta**, explica qué ocurrió y **modifica** el programa. No basta con copiar código y obtener una salida. Los diagramas de Stack y Heap son modelos conceptuales; no pretenden representar las direcciones físicas ni las optimizaciones concretas de todas las JVM.

## Preparación del trabajo

Utiliza **Java 21**, Visual Studio Code y una terminal. No agregues dependencias. Conserva los archivos de los laboratorios anteriores para relacionar los conceptos nuevos con programas que ya conoces.

Crea una carpeta para las prácticas:

~~~text
java-lab-06/
├── practice-6-1/
├── practice-6-2/
├── practice-6-3/
├── practice-6-4/
├── practice-6-5/
└── memory-challenge/
~~~

Cada ejemplo se crea en su carpeta correspondiente. Para compilar un archivo Java sencillo, sitúate en la carpeta que lo contiene y usa:

~~~bash
javac Example.java
java Example
~~~

Sustituye `Example` por el nombre real de la clase. No escribas `.java` al ejecutar una clase **ya compilada** con este procedimiento.

Si el JDK no está disponible o aparece una versión diferente, informa al instructor antes de instalar, desinstalar o modificar el ambiente.

### Antes de comenzar — Diagnóstico inicial

Responde sin buscar las definiciones:

1. ¿Qué diferencia hay entre JDK y JVM?
2. ¿Qué produce el compilador `javac`?
3. ¿Qué ocurre cuando un método llama a otro método?
4. Si dos variables apuntan al mismo objeto y una modifica su estado, ¿qué observará la otra?
5. ¿Java pasa los objetos «por referencia»? Escribe tu explicación inicial.

Estas respuestas servirán para comparar lo que piensas ahora con lo que podrás explicar al finalizar. No son preguntas de memorización.

---

## Práctica 6.1 — JDK, JRE y JVM: herramientas y responsabilidades

### Objetivos

Al finalizar esta práctica podrás:

- distinguir lenguaje Java, JDK, JRE y JVM;
- reconocer las funciones de `java`, `javac` y `javap`;
- comprobar la versión del entorno instalado;
- explicar qué necesitas para desarrollar y qué necesitas para ejecutar;
- comprender por qué un JRE independiente no es obligatorio al disponer de un JDK moderno.

### 1. ¿Todo lo que llamamos Java es lo mismo?

Cuando escribes `class`, `int`, `new`, `if` y métodos utilizas el **lenguaje de programación Java**. Sus reglas establecen cómo expresar un programa.

Para desarrollarlo y ejecutarlo intervienen varias herramientas y componentes:

| Concepto | Significado | Función |
|---|---|---|
| **JDK** | Java Development Kit | Herramientas para desarrollar, incluidas `javac`, `java` y `javap`, además del entorno necesario para ejecutar. |
| **JRE** | Java Runtime Environment | Entorno de ejecución que incluye una JVM y componentes necesarios para ejecutar aplicaciones Java. |
| **JVM** | Java Virtual Machine | Máquina virtual que ejecuta programas compilados en bytecode y administra áreas de memoria. |
| **Lenguaje Java** | Sintaxis y reglas de programación | Permite expresar clases, métodos, instrucciones y datos. |

**Importante:** un JDK de Java 21 contiene también las piezas necesarias para ejecutar aplicaciones. No es necesario instalar un JRE separado. El término **JRE** sigue siendo útil para distinguir el entorno de ejecución de las herramientas de desarrollo, aunque su distribución independiente haya cambiado según el proveedor y la versión.

Un JDK no es «una JVM mejor». El JDK proporciona herramientas, entre ellas un compilador; la JVM es uno de los componentes que intervienen en la ejecución.

### 2. Verificar el ambiente

Abre una terminal y ejecuta:

~~~bash
java --version
~~~

Este comando informa sobre el lanzador y el entorno Java disponible. Un ejemplo válido tendrá una línea que mencione Java **21**, aunque la distribución y la revisión de mantenimiento sean distintas.

Después ejecuta:

~~~bash
javac --version
~~~

`javac` es el **compilador**. Si `java` está disponible pero `javac` no, todavía no has comprobado que puedas compilar mediante esa terminal.

Finalmente ejecuta:

~~~bash
javap -version
~~~

`javap` permite examinar archivos de clase compilados y desensamblar su bytecode. Lo usarás en la práctica 6.2.

**Antes de continuar:** registra las versiones que aparecen. Si las herramientas pertenecen a instalaciones distintas, informa al instructor antes de cambiar configuraciones.

### 3. Relacionar las herramientas con su responsabilidad

Observa esta tabla:

| Necesidad | Herramienta |
|---|---|
| Compilar `ExecutionFlow.java` | `javac` |
| Iniciar el programa compilado | `java` |
| Inspeccionar el bytecode de la clase | `javap` |

El archivo que contiene código fuente utiliza la extensión `.java`. El compilador genera archivos `.class`, que contienen información e instrucciones en un formato destinado a la JVM.

### 4. ¿Por qué puede funcionar Java en diferentes sistemas operativos?

Una aplicación compilada para una versión compatible de la plataforma puede ejecutar su bytecode en implementaciones de JVM disponibles para distintos sistemas operativos.

Eso no garantiza compatibilidad absoluta con cualquier equipo: importan la versión de Java, las bibliotecas, las dependencias y los recursos externos.

La JVM tampoco tiene que interpretar indefinidamente cada instrucción. Implementaciones como HotSpot pueden interpretar bytecode y compilar partes a código máquina mediante **JIT (Just-In-Time)**. No todos los métodos tienen que compilarse mediante JIT.

### Ejercicio — Herramienta correcta

Responde:

1. ¿Qué comando utilizarías para saber si puedes **compilar**?
2. ¿Cuál sirve para iniciar un programa?
3. ¿Por qué disponer de `java` no garantiza disponer de `javac`?
4. Si ya recibes un `.class` compatible con las dependencias necesarias, ¿necesitas un compilador para iniciar esa clase?
5. ¿Qué herramienta permite examinar el bytecode?

### Reto — Explica el error

Un compañero afirma: «Instalé una JVM, por lo tanto puedo desarrollar y compilar cualquier proyecto Java».

Explica por qué esa conclusión no está justificada. Utiliza correctamente **JDK**, **compilador** y **JVM**.

### Comprobación

Sin leer la tabla anterior, explica qué hace cada herramienta y qué debes revisar para trabajar con Java 21.

---

## Práctica 6.2 — Compilación, bytecode y ciclo de ejecución

### Objetivos

Al finalizar esta práctica podrás:

- diferenciar los archivos `.java` y `.class`;
- compilar hacia una carpeta de salida separada;
- ejecutar una clase indicando su *class path*;
- inspeccionar bytecode con `javap -c`;
- reconocer los conceptos de carga de clases, interpretación y JIT;
- distinguir errores de compilación, inicio y lógica.

### 1. El recorrido del programa

Cuando compilas y ejecutas una clase, puedes representar el recorrido así:

~~~text
ExecutionFlow.java
       |
       | javac (compilador)
       v
ExecutionFlow.class
       |
       | java (lanzador)
       v
JVM: carga y verifica clases cuando corresponde
       |
       | Ejecuta bytecode; puede interpretar o usar código compilado por JIT
       v
Resultado en la terminal
~~~

El código fuente `.java` contiene instrucciones escritas por una persona. El archivo `.class` contiene bytecode e información de la clase. No es un ejecutable nativo de Windows o Linux.

La carga, la vinculación y la inicialización de clases tienen reglas específicas. Para este nivel es suficiente identificar que la JVM prepara las clases y ejecuta sus instrucciones. No hace falta memorizar todas las etapas internas.

### 2. Crear el programa

En `practice-6-2`, crea `ExecutionFlow.java`:

~~~java
public class ExecutionFlow {

    public static void main(String[] args) {
        int base = 7;
        int bonus = 5;

        int total = add(base, bonus);

        System.out.println("Total: " + total);
    }

    static int add(int first, int second) {
        return first + second;
    }
}
~~~

Observa:

- `public class ExecutionFlow` declara una clase pública. Su nombre coincide con el archivo.
- `main` es el punto de entrada que utilizaremos con el lanzador `java`.
- `base` y `bonus` son variables locales.
- `add(base, bonus)` llama a un método y entrega dos argumentos.
- `first` y `second` son los parámetros de `add`.
- `return first + second` devuelve un `int`.
- `System.out.println` imprime el resultado.

### Antes de ejecutar

Predice la salida y explica qué valores recibe `add`.

### 3. Compilar por separado

Desde la carpeta del archivo ejecuta:

~~~bash
javac -d out ExecutionFlow.java
~~~

`-d out` especifica la carpeta de salida. La estructura resultante debe contener:

~~~text
practice-6-2/
├── ExecutionFlow.java
└── out/
    └── ExecutionFlow.class
~~~

Si falta el `.class`, revisa los mensajes de compilación.

### 4. Ejecutar el bytecode

Ejecuta:

~~~bash
java -cp out ExecutionFlow
~~~

`-cp out` indica dónde debe buscar clases el lanzador. Se escribe el **nombre de la clase principal**, no el nombre del archivo con extensión.

Resultado esperado:

~~~text
Total: 12
~~~

Desde Java 11 también es posible iniciar directamente determinados archivos fuente con `java ExecutionFlow.java`. **No utilizaremos ese modo aquí**, porque queremos separar la compilación de la ejecución y observar el archivo `.class`.

### 5. Observar el bytecode

Ejecuta:

~~~bash
javap -classpath out -c -p ExecutionFlow
~~~

En el bytecode del método `add(int, int)` encontrarás instrucciones equivalentes a:

~~~text
iload_0
iload_1
iadd
ireturn
~~~

| Instrucción | Interpretación introductoria |
|---|---|
| `iload_0` | Carga un entero desde la primera posición local utilizada por este método estático. |
| `iload_1` | Carga el segundo entero. |
| `iadd` | Suma dos enteros de la pila de operandos. |
| `ireturn` | Devuelve un valor entero al método llamador. |

El bytecode utiliza una **pila de operandos** para operar sobre valores. Más adelante veremos también la **pila de llamadas**: no son dos nombres para exactamente lo mismo. Cada marco de método incluye su propia pila de operandos.

No memorices todas las instrucciones del desensamblado. Comprueba que la compilación produjo instrucciones de la JVM y no una copia del código Java original.

### 6. ¿Interpretación o JIT?

Una implementación de JVM puede ejecutar bytecode interpretándolo y, cuando resulta beneficioso, compilar determinados fragmentos a instrucciones de la máquina durante la ejecución (*JIT*). El comportamiento preciso depende de la implementación, las opciones y el programa.

No debemos afirmar que todos los métodos se compilan por JIT ni que existe un número universal de llamadas a partir del cual eso sucede.

### Ejercicio — ¿Por qué no cambió la salida?

1. Modifica `bonus` de `5` a `8` **sin recompilar**.
2. Ejecuta nuevamente `java -cp out ExecutionFlow`. ¿Qué observas?
3. Vuelve a ejecutar `javac -d out ExecutionFlow.java`.
4. Ejecuta la clase otra vez.
5. Explica por qué el resultado de la aplicación compilada cambia después de actualizar el `.class`.

### Laboratorio de errores

Trabaja con una copia del archivo para provocar y corregir un error a la vez.

**Error 1 — Sintaxis.** Elimina el punto y coma de `int base = 7;`. Intenta compilar. El error ocurre antes de iniciar tu aplicación.

**Error 2 — Clase no localizada.** Compila correctamente e intenta ejecutar `java -cp out MissingClass`. ¿Qué componente intenta encontrar esa clase?

**Error 3 — Nombre incorrecto.** Ejecuta `java -cp out ExecutionFlow.class`. ¿Por qué no se escribe la extensión al indicar una clase compilada?

**Error 4 — Lógica.** Cambia `return first + second;` por `return first - second;`. Compila y ejecuta: no hay un error sintáctico, pero el cálculo es distinto del requisito.

### Comprobación

Explica qué representan `.java`, `.class`, `javac`, `java`, `-cp`, `javap`, bytecode y JIT. Luego señala en qué etapa apareció cada error del ejercicio.

---

## Práctica 6.3 — Stack: métodos, parámetros y variables locales

### Objetivos

Al finalizar esta práctica podrás:

- identificar el orden de las llamadas y sus retornos;
- representar conceptualmente los marcos de ejecución (*stack frames*);
- diferenciar parámetros de variables locales;
- observar la pila de llamadas con el depurador;
- comprender por qué una llamada recursiva puede mantener varias invocaciones activas.

### 1. ¿Qué guarda la pila de una JVM?

Cada hilo de ejecución dispone de una **pila de la máquina virtual Java**. Al invocar un método, se crea un marco para esa invocación. El marco contiene información necesaria para ejecutar el método, incluidos parámetros, variables locales y una pila de operandos. Cuando el método termina normalmente, el control regresa a la llamada anterior.

Imagina este recorrido:

~~~text
Inicialmente:
[ main ]

main llama a calculate:
[ calculate ]
[ main      ]

calculate llama a doubleValue:
[ doubleValue ]
[ calculate   ]
[ main        ]

doubleValue retorna:
[ calculate ]
[ main      ]

calculate retorna:
[ main ]
~~~

La fila superior representa el método activo. **Es un modelo de llamadas**, no un plano físico de memoria. La implementación de una JVM puede optimizar internamente los marcos y otras operaciones sin alterar el comportamiento observable del lenguaje.

### 2. Seguir una cadena de métodos

Crea `MethodStack.java` dentro de `practice-6-3`:

~~~java
public class MethodStack {

    public static void main(String[] args) {
        int initialValue = 4;
        int result = calculate(initialValue);

        System.out.println("Final result: " + result);
    }

    static int calculate(int value) {
        int doubled = doubleValue(value);
        return doubled + 3;
    }

    static int doubleValue(int number) {
        return number * 2;
    }
}
~~~

Antes de ejecutar, identifica:

- `initialValue` y `result`: variables locales de `main`.
- `value`: parámetro de `calculate`.
- `doubled`: variable local de `calculate`.
- `number`: parámetro de `doubleValue`.

Son nombres que pertenecen a **invocaciones concretas**. Los valores pueden viajar entre métodos sin que sus variables locales se conviertan en una única variable compartida.

### Antes de ejecutar

Cuando se encuentra activo `doubleValue`, responde:

1. ¿Qué valor tiene `number`?
2. ¿Qué otros métodos todavía están esperando el retorno?
3. ¿Cuánto devuelve `doubleValue`?
4. ¿Qué se guardará finalmente en `result`?

Compila y ejecuta:

~~~bash
javac MethodStack.java
java MethodStack
~~~

Resultado esperado:

~~~text
Final result: 11
~~~

`main` entrega `4`, `doubleValue` retorna `8`, `calculate` devuelve `8 + 3` y `main` imprime `11`.

### 3. Seguir llamadas con el depurador

Abre `MethodStack.java` en VS Code:

1. Coloca un punto de interrupción en `return number * 2;`.
2. Selecciona **Debug**.
3. Observa el panel **Call Stack**: deben aparecer las llamadas activas que llevan a `doubleValue`.
4. En **Variables**, identifica `number`.
5. Utiliza **Step Out** para regresar a `calculate`.
6. Inspecciona ahora `doubled` y vuelve a `main`.

El aspecto concreto de los paneles depende de la versión de las extensiones. Lo fundamental es observar qué método está activo y cuáles esperan su resultado.

**Pregunta de análisis:** ¿ya terminó `calculate` mientras estás detenido dentro de `doubleValue`? ¿Por qué?

### 4. Parámetros con el mismo nombre

Estos dos métodos podrían existir en una misma clase:

~~~java
static void displayFirst(int number) {
    System.out.println(number);
}

static void displaySecond(int number) {
    System.out.println(number);
}
~~~

Los parámetros se llaman igual, pero no son una sola variable compartida. Cada llamada crea su propio contexto de ejecución.

### 5. Recursividad: llamadas al mismo método

Crea `SafeRecursion.java`:

~~~java
public class SafeRecursion {

    static void countDown(int value) {
        System.out.println("Entering: " + value);

        if (value == 0) {
            System.out.println("Base case");
            return;
        }

        countDown(value - 1);
        System.out.println("Returning: " + value);
    }

    public static void main(String[] args) {
        countDown(3);
    }
}
~~~

### Antes de ejecutar

Escribe las líneas de salida en orden. Fíjate que cada llamada tiene una instrucción pendiente **después** de llamar a `countDown(value - 1)`.

Resultado esperado:

~~~text
Entering: 3
Entering: 2
Entering: 1
Entering: 0
Base case
Returning: 1
Returning: 2
Returning: 3
~~~

Las llamadas se completan en orden inverso. Si una ejecución requiere demasiados marcos, podría producirse `StackOverflowError`. **No es necesario provocar un desbordamiento real**.

### Ejercicio — Un cambio en la pila

Modifica la llamada inicial a `countDown(2)`. Dibuja qué invocaciones están activas al llegar al caso base y predice todas las líneas.

### Depuración — Una variable calculada que se ignora

En una copia de `MethodStack.java`, cambia la última instrucción de `calculate` por:

~~~java
return value + 3;
~~~

El programa compila, pero ya no utiliza `doubled`. Localiza el problema con el depurador y explica la diferencia entre el valor calculado y el valor retornado.

### Comprobación

Debes poder dibujar la secuencia de invocaciones `main` → `calculate` → `doubleValue`, reconocer las variables locales de cada método y justificar por qué un método no puede continuar en una instrucción que depende de un retorno todavía pendiente.

---

## Práctica 6.4 — Heap: objetos, arreglos, identidad y referencias

### Objetivos

Al terminar esta práctica podrás:

- distinguir valores primitivos y valores de referencia;
- explicar qué ocurre conceptualmente al crear una instancia con `new`;
- identificar alias y objetos independientes;
- diferenciar identidad (`==`) de igualdad de contenido (`.equals()`);
- comprender el papel de los arreglos, `null` y la elegibilidad para Garbage Collection.

### 1. Un entero y una referencia no representan lo mismo

Compara:

~~~java
int age = 3;
~~~

con:

~~~java
Pet animal = new Pet("Luna");
~~~

La variable `age` contiene un valor primitivo de tipo `int`. La variable `animal` contiene un **valor de referencia** que permite acceder a una instancia de `Pet`.

En el modelo definido por la especificación de la JVM, las instancias de clases y los arreglos se asignan desde una región compartida llamada **Heap**. Para enseñar las referencias usaremos letras como **A** y **B**; **no son direcciones de memoria reales**.

### 2. Dos referencias pueden llegar al mismo objeto

Crea `ObjectReferences.java` en `practice-6-4`:

~~~java
public class ObjectReferences {

    static final class Pet {
        private String name;

        Pet(String name) {
            this.name = name;
        }

        String getName() {
            return name;
        }

        void rename(String newName) {
            name = newName;
        }
    }

    public static void main(String[] args) {
        Pet first = new Pet("Luna");
        Pet alias = first;
        Pet independent = new Pet("Luna");

        System.out.println("First and alias: " + (first == alias));
        System.out.println("First and independent: " + (first == independent));

        alias.rename("Sol");

        System.out.println("First name: " + first.getName());
        System.out.println("Independent name: " + independent.getName());
    }
}
~~~

`Pet` es una clase anidada, creada así para mantener el ejemplo en un solo archivo. `name` es un atributo privado; `getName()` lo consulta y `rename()` lo modifica. El constructor utiliza `this.name` para asignar el valor recibido.

### Antes de ejecutar

Dibuja:

~~~text
first       ----> Objeto A: Pet(name = "Luna")
alias       ----> Objeto A: Pet(name = "Luna")
independent ----> Objeto B: Pet(name = "Luna")
~~~

¿Cuáles variables están conectadas al mismo objeto? ¿Qué nombres tendrán los objetos después de `alias.rename("Sol")`?

Compila y ejecuta:

~~~bash
javac ObjectReferences.java
java ObjectReferences
~~~

Salida esperada:

~~~text
First and alias: true
First and independent: false
First name: Sol
Independent name: Luna
~~~

La asignación `Pet alias = first` **no crea otro objeto**. Copia el valor de referencia que ya estaba en `first`. Como ambos acceden a A, el cambio hecho mediante `alias` también es visible mediante `first`.

`independent`, en cambio, guarda una referencia al objeto B, creado mediante otro `new`.

### 3. ¿Qué significa `==` cuando comparamos referencias?

`==` entre dos referencias comprueba si contienen el mismo valor de referencia, es decir, si identifican la misma instancia; dos referencias `null` también se comparan como iguales.

No significa automáticamente que dos objetos tengan los mismos datos.

Para demostrar la diferencia con `String`, agrega este fragmento en un método `main` de una clase de prueba:

~~~java
String firstText = new String("Luna");
String secondText = new String("Luna");

System.out.println(firstText == secondText);
System.out.println(firstText.equals(secondText));
~~~

Salida esperada:

~~~text
false
true
~~~

Aquí utilizamos `new String(...)` **solo para demostrar identidad**. En programas cotidianos generalmente escribimos `"Luna"` directamente. `.equals()` consulta la igualdad de contenido según la implementación de la clase. Nuestra clase `Pet` **no redefine** `.equals()`, así que no debes suponer que dos mascotas con el mismo nombre serán iguales mediante ese método.

### 4. Los arreglos también son objetos

Prueba este fragmento dentro de un `main`:

~~~java
int[] firstNumbers = {2, 4, 6};
int[] sameArray = firstNumbers;
int[] otherArray = {2, 4, 6};

sameArray[0] = 99;

System.out.println(firstNumbers[0]);
System.out.println(otherArray[0]);
System.out.println(firstNumbers == sameArray);
System.out.println(firstNumbers == otherArray);
~~~

### Antes de ejecutar

Predice las cuatro líneas. Justifica por qué el cambio mediante `sameArray` afecta o no a `firstNumbers` y `otherArray`.

Un arreglo es un objeto aunque contenga valores primitivos. Asignar su referencia a otra variable **no** copia automáticamente todos sus elementos.

### 5. `null` no es un objeto vacío

Una variable de tipo referencia puede contener `null`, lo que significa que no refiere a ningún objeto:

~~~java
Pet missing = null;
~~~

Si invocas `missing.getName()` cuando sigue siendo `null`, ocurrirá un `NullPointerException` en tiempo de ejecución.

No escribas que `new Pet("Luna")` y `null` son dos formas de crear una mascota sin datos. Con `null` **no se crea ninguna instancia**.

### 6. ¿Cuándo desaparecen los objetos?

Observa:

~~~java
Pet current = new Pet("Luna");
current = new Pet("Sol");
~~~

Cuando `current` pasa a referirse a la segunda instancia, la primera ya no puede alcanzarse **mediante esa variable**. Si no existe ningún otro camino de referencias alcanzables hacia la primera instancia, puede quedar **elegible para Garbage Collection (GC)**.

El GC gestiona la recuperación de memoria de objetos que dejan de ser necesarios. **No se garantiza** que un objeto sea recolectado inmediatamente después de una asignación, ni podemos predecir un momento exacto de liberación.

### Ejercicio — Reasignar un alias

En una copia de `ObjectReferences.java`:

1. Agrega `Pet third = first;`.
2. Comprueba el nombre de `third`.
3. Ejecuta `alias = new Pet("Nova");`.
4. Después ejecuta `alias.rename("Max");`.
5. Consulta nuevamente `first`, `third`, `independent` y `alias`.

Dibuja las referencias antes y después de la reasignación. Explica por qué el nombre de A no se modifica con `alias.rename("Max")`.

### Depuración — Referencia nula

En una copia del ejemplo agrega dentro de `main`:

~~~java
Pet missing = null;
System.out.println(missing.getName());
~~~

Predice el tipo de error. Ejecútalo una vez, identifica la línea en el *stack trace* y después corrige el acceso. ¿Por qué compiló un programa que falló durante la ejecución?

### Comprobación

Explica sin confundir estos conceptos: valor primitivo, valor de referencia, objeto, instancia independiente, alias, igualdad de contenido, referencia `null` y elegibilidad para GC.

---

## Práctica 6.5 — Paso de parámetros y caso real de PetMatch

### Objetivos

Al finalizar esta práctica podrás:

- explicar por qué Java pasa **todos** sus argumentos por valor;
- comprobar qué ocurre cuando modificas un parámetro primitivo;
- distinguir modificar un objeto de reasignar una referencia local;
- analizar el comportamiento de un arreglo enviado a un método;
- explicar el ejemplo original `MemoryDemo.java` de PetMatch.

### 1. Argumento y parámetro: dos papeles diferentes

Considera:

~~~java
int quantity = 5;
increase(quantity);
~~~

`quantity` pertenece al método llamador. En `increase(quantity)`, el **argumento** aporta un valor a la llamada.

El método receptor puede declarar:

~~~java
static void increase(int value) {
    value = value + 10;
}
~~~

`value` es un **parámetro local** de la nueva invocación. Al comenzar contiene una copia del valor del argumento.

La regla de Java no consiste en «primitivos por valor, objetos por referencia». **Todos los argumentos se pasan por valor.** Si el argumento es de tipo referencia, lo copiado es precisamente su valor de referencia.

### 2. Parámetro primitivo: cambiar la copia

Crea `PrimitiveParameters.java` en `practice-6-5`:

~~~java
public class PrimitiveParameters {

    static void increase(int value) {
        value = value + 10;
        System.out.println("Inside: " + value);
    }

    public static void main(String[] args) {
        int quantity = 5;
        increase(quantity);
        System.out.println("Outside: " + quantity);
    }
}
~~~

### Antes de ejecutar

Predice el valor dentro de `increase` y después de retornar a `main`.

Compila y ejecuta:

~~~bash
javac PrimitiveParameters.java
java PrimitiveParameters
~~~

Salida esperada:

~~~text
Inside: 15
Outside: 5
~~~

Se modificó el parámetro `value`, no la variable `quantity` del llamador.

### 3. Arreglos: modificar el objeto frente a reasignar la referencia

Crea `ArrayReferences.java`:

~~~java
public class ArrayReferences {

    static void updateFirst(int[] values) {
        values[0] = 99;
    }

    static void replaceArray(int[] values) {
        values = new int[]{7, 8};
        System.out.println("Inside: " + values[0]);
    }

    public static void main(String[] args) {
        int[] numbers = {1, 2};

        updateFirst(numbers);
        replaceArray(numbers);

        System.out.println("Outside: " + numbers[0] + ", " + numbers[1]);
    }
}
~~~

### Antes de ejecutar

1. ¿`updateFirst` modifica el objeto arreglo que también utiliza `main`?
2. ¿`replaceArray` modifica la variable `numbers` del llamador?
3. ¿Cuántos arreglos hay después de crear el segundo arreglo?
4. ¿Qué se imprimirá dentro y fuera?

Compila y ejecuta:

~~~bash
javac ArrayReferences.java
java ArrayReferences
~~~

Salida esperada:

~~~text
Inside: 7
Outside: 99, 2
~~~

`updateFirst` utiliza una referencia compartida para modificar una posición del arreglo original.

`replaceArray` recibe la misma referencia por valor, pero después cambia **su parámetro local** para apuntar a un arreglo nuevo. La variable `numbers` conserva la referencia anterior.

### 4. Caso de estudio — El ejemplo original de PetMatch

En PetMatch Community existe un programa independiente que permite enseñar estas reglas. Su nombre es [`MemoryDemo.java`](https://github.com/ethan-fullstack/petmatch-community/blob/examples/java-jvm-memory/MemoryDemo.java) y está acompañado por un [README explicativo](https://github.com/ethan-fullstack/petmatch-community/blob/examples/java-jvm-memory/README.md).

El ejemplo **no es un controlador Spring**, no depende de JPA y no requiere conexión a la base de datos. Se conserva como un estudio de caso basado en el dominio de mascotas.

> El código que sigue reproduce el ejemplo de PetMatch. Conserva sus nombres y mensajes originales para poder comparar las salidas con el caso de estudio.

Crea `MemoryDemo.java` en `practice-6-5`:

~~~java
/** Java passes every argument by value, including reference values. */
public class MemoryDemo {
    static final class Pet {
        String name;

        Pet(String name) {
            this.name = name;
        }
    }

    static void change(Pet local) {
        local.name = "Sol"; // Same object, visible to caller.
        local = new Pet("Otra"); // Only the local reference value changes.
        System.out.println("Dentro: " + local.name);
    }

    public static void main(String[] args) {
        Pet original = new Pet("Luna");
        change(original);
        System.out.println("Fuera: " + original.name);
        if (!original.name.equals("Sol")) throw new AssertionError("Paso por valor");
    }
}
~~~

`static final class Pet` define una clase anidada sencilla. `String name` almacena el nombre, y `this.name = name` asigna el valor recibido por el constructor al atributo de la instancia.

La última instrucción comprueba una condición: si `original.name` no fuera `"Sol"`, lanzaría un `AssertionError`. No necesitas activar `-ea` porque se utiliza explícitamente `throw new AssertionError`, no la palabra clave `assert`.

### Antes de ejecutar

1. Dibuja la variable `original` y el objeto creado mediante `new Pet("Luna")`.
2. ¿Qué valor recibe el parámetro `local` al iniciar `change(original)`?
3. ¿Qué objeto modifica `local.name = "Sol"`?
4. ¿Qué significa `local = new Pet("Otra")`?
5. ¿Qué imprimirán las dos líneas de salida?
6. ¿Se lanzará el `AssertionError`?

Compila y ejecuta:

~~~bash
javac MemoryDemo.java
java MemoryDemo
~~~

Salida esperada:

~~~text
Dentro: Otra
Fuera: Sol
~~~

### 5. Explicación paso a paso de Stack y Heap

Usaremos las letras **A** y **B** únicamente para identificar objetos en el dibujo.

**Momento 1 — En `main`:**

~~~text
Stack (main)                Heap
original ------- A ------> Pet { name = "Luna" }
~~~

La variable local `original` contiene una referencia al objeto A.

**Momento 2 — Entrada a `change(original)`:**

~~~text
Stack (change)              Heap
local ---------- A ------> Pet { name = "Luna" }

Stack (main)
original ------- A ------> mismo objeto
~~~

`local` recibió una copia del **valor de referencia** de `original`. Son variables distintas que llegan al mismo objeto.

**Momento 3 — `local.name = "Sol"`:**

~~~text
local ---------- A ------> Pet { name = "Sol" }
original ------- A ------> mismo objeto
~~~

Se modifica el objeto A. Por eso el cambio también es visible desde `original`.

**Momento 4 — `local = new Pet("Otra")`:**

~~~text
local ---------- B ------> Pet { name = "Otra" }
original ------- A ------> Pet { name = "Sol"  }
~~~

Se creó otro objeto B. **Solamente la variable local `local`** cambia de referencia. `original` no ha sido reasignada.

**Momento 5 — Impresiones y retorno:**

| Instrucción | Objeto consultado | Salida |
|---|---|---|
| `System.out.println("Dentro: " + local.name)` | B | `Dentro: Otra` |
| `System.out.println("Fuera: " + original.name)` | A | `Fuera: Sol` |
| Comprobación con `AssertionError` | A | La condición es falsa; no se lanza el error. |

Después de retornar `change`, la variable local deja de estar activa. Si ninguna otra referencia alcanzable lleva a B, el objeto B podrá quedar **elegible para GC**, sin garantía de cuándo será recolectado.

**Conclusión:** el método utilizó una referencia para **modificar** A y después **reasignó** su propia variable para apuntar a B. Ambos comportamientos son compatibles con el **paso por valor** de Java.

### 6. Cuatro modificaciones controladas

Trabaja siempre con una copia y vuelve al original antes de empezar otra variante.

**Variante A — Cambiar el orden.** Coloca `local = new Pet("Otra");` antes de `local.name = "Sol";`. Predice qué se imprimirá y por qué.

**Variante B — Sin reasignación.** Elimina temporalmente `local = new Pet("Otra");`. Explica por qué ambas impresiones consultan A.

**Variante C — Retornar un objeto.** Crea un método nuevo que retorne `Pet`, construya un objeto nuevo y lo devuelva. Asigna el resultado explícitamente en `main`. Explica por qué el llamador **sí** puede elegir modificar su propia variable al recibir un valor retornado.

**Variante D — Agregar un alias.** Antes de llamar a `change`, agrega `Pet alias = original;`. Después imprime `alias.name`. ¿Cuántos objetos se crearon al declarar ese alias?

### Ejercicio — Reasignar un parámetro a `null`

Agrega este método a una clase de prueba:

~~~java
static void clearLocalReference(Object value) {
    value = null;
}
~~~

Crea un objeto en `main`, envíalo como argumento y comprueba si la variable del llamador queda en `null`. Explica la diferencia entre reasignar **el parámetro** y modificar **el estado de un objeto**.

### Depuración — Detecta la afirmación incorrecta

Analiza:

> «Como el método modificó el objeto original, Java pasa objetos por referencia».

Corrige esa afirmación utilizando estas expresiones: **copia del valor de referencia**, **mismo objeto**, **parámetro local** y **reasignación**.

### Comprobación

Explica, sin consultar la tabla de momentos, por qué el caso de PetMatch imprime `Dentro: Otra` y `Fuera: Sol`. Debes poder demostrar el resultado con un dibujo de dos objetos y dos referencias.

---

## Práctica 6.6 — Reto integrador, depuración y diagnóstico

### Objetivos

Al finalizar esta práctica podrás:

- analizar un programa formado por varias clases;
- predecir resultados usando referencias y llamadas a métodos;
- utilizar el depurador para seguir el flujo real;
- comprobar reglas mediante pequeñas verificaciones ejecutables;
- explicar con precisión qué significa el paso por valor.

### 1. Reto integrador — Seguimiento de mascotas

En `memory-challenge` crea dos archivos. Este escenario utiliza mascotas para mantener el vínculo con PetMatch, pero no modifica el proyecto Spring Boot.

Crea `PetProfile.java`:

~~~java
public class PetProfile {
    private final String name;
    private int visits;

    public PetProfile(String name) {
        this.name = name;
        this.visits = 0;
    }

    public void registerVisit() {
        visits++;
    }

    public String getName() {
        return name;
    }

    public int getVisits() {
        return visits;
    }
}
~~~

Observa que `name` es `private final`: no puede reasignarse después de su inicialización. `visits` es privado y se modifica mediante el método `registerVisit()`. Cada instancia conserva su contador de visitas.

Ahora crea `MemoryChallenge.java`:

~~~java
public class MemoryChallenge {

    private static void update(PetProfile current) {
        current.registerVisit();

        current = new PetProfile("Nova");
        current.registerVisit();

        System.out.println("Inside: " + current.getName()
                + " (" + current.getVisits() + ")");
    }

    private static PetProfile createReplacement(PetProfile previous) {
        return new PetProfile("Max");
    }

    public static void main(String[] args) {
        PetProfile pet = new PetProfile("Luna");
        PetProfile alias = pet;

        update(pet);

        System.out.println("Outside: " + pet.getName()
                + " (" + pet.getVisits() + ")");

        PetProfile replacement = createReplacement(pet);

        System.out.println("Same reference: " + (pet == alias));
        System.out.println("Replacement: " + replacement.getName()
                + " (" + replacement.getVisits() + ")");
    }
}
~~~

### 2. Antes de ejecutar — Predicción obligatoria

No presiones **Run** todavía.

1. ¿Cuántas instrucciones `new PetProfile(...)` se ejecutarán en total?
2. ¿Cuántos objetos diferentes se crearán?
3. ¿`pet` y `alias` apuntan inicialmente al mismo objeto?
4. ¿Qué objeto recibe el primer incremento de visitas?
5. ¿Qué objeto recibe el segundo incremento?
6. ¿`createReplacement(pet)` cambia a qué objeto apunta `pet`?
7. ¿Qué cuatro líneas se imprimirán?
8. Cuando termine `update`, ¿existe una referencia utilizable desde `main` al objeto `"Nova"`?
9. ¿Qué objetos podrían ser elegibles para GC una vez que se pierdan todas las referencias alcanzables hacia ellos?

Dibuja tres objetos identificados como A, B y C. Incluye `pet`, `alias`, `current` y `replacement` en el momento en que esas variables existen.

### 3. Compilar y comprobar

Desde `memory-challenge` ejecuta:

~~~bash
javac PetProfile.java MemoryChallenge.java
java MemoryChallenge
~~~

Compara el resultado con tu predicción. Si alguna línea no coincide, identifica qué **instrucción concreta** explica la diferencia antes de modificar el código.

### 4. Una comprobación sin bibliotecas externas

Crea `MemoryChallengeCheck.java` en la misma carpeta:

~~~java
public class MemoryChallengeCheck {

    public static void main(String[] args) {
        PetProfile original = new PetProfile("Luna");
        PetProfile alias = original;

        alias.registerVisit();

        if (original.getVisits() != 1) {
            throw new AssertionError("Shared object was not updated");
        }

        PetProfile independent = new PetProfile("Luna");

        if (original == independent) {
            throw new AssertionError("Distinct objects share an identity");
        }

        if (independent.getVisits() != 0) {
            throw new AssertionError("Independent state was unexpectedly modified");
        }

        System.out.println("All reference checks passed");
    }
}
~~~

Compila y ejecuta:

~~~bash
javac PetProfile.java MemoryChallengeCheck.java
java MemoryChallengeCheck
~~~

Salida esperada:

~~~text
All reference checks passed
~~~

Estas verificaciones comprueban algunas reglas, pero **no sustituyen una suite completa de pruebas**. Todavía no necesitamos JUnit o herramientas adicionales.

### 5. Inspeccionar las referencias en el depurador

Abre `MemoryChallenge.java` en VS Code.

1. Coloca un breakpoint en la primera instrucción `current.registerVisit();`.
2. Inicia **Debug** y observa las variables locales y el *Call Stack*.
3. Avanza hasta el primer incremento y consulta `current.getVisits()`.
4. Detente después de `current = new PetProfile("Nova");`.
5. Comprueba que el parámetro `current` ahora refiere a otra instancia.
6. Regresa a `main` e inspecciona `pet.getName()` y `pet.getVisits()`.
7. Repite el análisis después de recibir el resultado de `createReplacement`.

El depurador permite observar variables y llamadas, pero no convierte nuestros diagramas A/B/C en direcciones físicas reales del Heap.

### 6. Actividad de depuración — Tres errores

Trabaja con copias del programa.

**Error A — Referencia compartida accidental.** En la clase de comprobación, reemplaza la creación de `independent` por una asignación que reutilice `original`. Comprueba qué aserción falla y por qué.

**Error B — Ignorar el retorno.** Llama a `createReplacement(pet);` sin asignar su resultado. ¿Cambia `pet`? Explica qué ocurrió con el objeto retornado.

**Error C — Referencia nula.** En una copia declara `PetProfile missing = null;` e intenta `missing.registerVisit();`. Predice y observa el error, identifica la línea del stack trace y después corrige el acceso.

### 7. Reto de ampliación

Agrega dos métodos diferentes:

- `registerTwoVisits(PetProfile pet)`: modifica el mismo objeto recibido, registrando exactamente dos visitas.
- `createNewProfile(String name)`: crea una instancia y **la devuelve** al método que llama.

Comprueba ambos comportamientos desde `main`. Explica cuál método cambia el **estado de un objeto** y cuál entrega un nuevo **valor de referencia** para que el llamador pueda asignarlo.

No agregues repositorios, controladores, conexión a base de datos ni dependencias de Spring.

### 8. Diagnóstico final — Demostrar comprensión

Responde individualmente. No necesitas entregar un informe PDF; el instructor podrá evaluar el código y la explicación durante la sesión.

| N.º | Pregunta |
|---|---|
| 1 | ¿Qué contiene el JDK que permite compilar? |
| 2 | ¿Para qué sirve una JVM al ejecutar un archivo `.class`? |
| 3 | ¿Qué diferencia hay entre `java` y `javac`? |
| 4 | ¿Qué muestra `javap -c`? |
| 5 | ¿Qué es un *stack frame*? |
| 6 | ¿Qué diferencia hay entre pila de llamadas y pila de operandos? |
| 7 | ¿En qué área define la JVM la asignación de memoria de instancias y arreglos? |
| 8 | ¿Cuál es la diferencia entre una variable `int` y una variable `PetProfile`? |
| 9 | ¿Qué significa `first == second` entre referencias? |
| 10 | ¿Por qué un alias permite observar cambios en un mismo objeto? |
| 11 | ¿Por qué reasignar un parámetro no reasigna una variable del llamador? |
| 12 | ¿Cuál es la salida exacta de `MemoryDemo.java` y por qué? |
| 13 | ¿Qué significa que un objeto sea elegible para GC? |
| 14 | ¿Por qué una referencia `null` puede ocasionar un error en tiempo de ejecución? |
| 15 | ¿Cómo observarías con el depurador el orden de llamadas y retornos? |

#### Microejercicio — Predice la salida

Lee este fragmento:

~~~java
int[] values = {3, 6};
int[] other = values;
other[1] = 9;

System.out.println(values[1]);
System.out.println(values == other);
~~~

Predice ambas líneas y explica por qué se producen. Después ejecútalo dentro de un método `main` y verifica.

### 9. Comprobación oral y evidencia

Selecciona `MemoryDemo.java` o `MemoryChallenge.java`. Debes poder:

1. identificar los objetos creados;
2. dibujar las referencias antes y después de una llamada;
3. reconocer parámetros y variables locales;
4. predecir y explicar los resultados;
5. localizar las llamadas en el depurador;
6. corregir una afirmación incorrecta sobre el supuesto «paso por referencia».

Conserva tus archivos, predicciones y correcciones en tu espacio de trabajo o repositorio. Si el instructor utiliza Moodle para el diagnóstico, responde allí según sus indicaciones. **No** es obligatorio generar un informe PDF o modificar el aplicativo PetMatch.

---

## Cierre del laboratorio

Ahora puedes conectar los conceptos:

**Código Java** (`.java`) → **compilación con `javac`** → **archivo `.class`** → **ejecución por la JVM**.

Durante la ejecución, cada invocación tiene su contexto de variables locales, parámetros y operandos. Los métodos pueden crear y modificar objetos; las instancias y los arreglos se asignan desde el Heap según el modelo de la JVM; y varias variables pueden conservar referencias hacia una misma instancia.

La regla central del caso PetMatch es:

> **Java pasa todos los argumentos por valor.** Cuando ese valor es una referencia, un método puede utilizarla para modificar el objeto compartido, pero reasignar su parámetro no modifica la variable del método que llamó.

En los siguientes temas encontrarás las mismas reglas dentro de servicios, entidades y controladores de Spring Boot. El framework no cambia la semántica del lenguaje Java: añade nuevas responsabilidades y mecanismos sobre esa base.

### Referencias oficiales — Java 21

- [Java 21 — Herramienta `javac`](https://docs.oracle.com/en/java/javase/21/docs/specs/man/javac.html)
- [Java 21 — Herramienta `java`](https://docs.oracle.com/en/java/javase/21/docs/specs/man/java.html)
- [Java 21 — Herramienta `javap`](https://docs.oracle.com/en/java/javase/21/docs/specs/man/javap.html)
- [Java Virtual Machine Specification, Java SE 21 — Áreas de memoria y marcos de ejecución (§2.5–2.6)](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html)
- [Java Language Specification, Java SE 21 — Invocación de métodos y evaluación de argumentos (§15.12.4)](https://docs.oracle.com/javase/specs/jls/se21/html/jls-15.html#jls-15.12.4)
- [PetMatch Community — README del ejemplo de memoria](https://github.com/ethan-fullstack/petmatch-community/blob/examples/java-jvm-memory/README.md)
