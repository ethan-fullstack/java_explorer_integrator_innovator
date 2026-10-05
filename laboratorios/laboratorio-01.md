# Laboratorio 1 — Primeros pasos con Java

En este laboratorio vas a comprobar que el entorno está listo y a construir tus primeros programas en Java. La meta es que puedas crear un archivo, ejecutarlo, trabajar con datos, hacer cálculos y recibir información desde la terminal.

| Práctica | Tema |
|---|---|
| 1.1 | Verificación del entorno y primera ejecución |
| 1.2 | Variables, tipos de datos y constantes |
| 1.3 | Operadores y expresiones |
| 1.4 | Entrada de datos con `Scanner` |
| 1.5 | Reto integrador: cotizador de compra |

> Ejecuta cada ejemplo antes de modificarlo. Cuando algo falle, revisa el mensaje de error e intenta identificar la causa antes de cambiar varias líneas a la vez.

---

## Práctica 1.1 — Verificación del entorno y primera ejecución

### Objetivos

Al finalizar esta práctica podrás:

- comprobar que el JDK está disponible;
- verificar el soporte de Java en Visual Studio Code;
- crear y ejecutar una clase Java;
- compilar y ejecutar un programa desde la terminal;
- reconocer algunos errores básicos de compilación.

La mayor parte de los equipos del ambiente ya debería contar con las herramientas necesarias. Antes de instalar o modificar algo, verifica lo que ya está disponible.

### 1. Verificar Java

Abre Visual Studio Code y luego una terminal desde:

`Terminal > New Terminal`

También puedes abrirla con el atajo **Ctrl + `**.

Ejecuta:

```bash
java --version
```

Luego:

```bash
javac --version
```

En los equipos preparados para el curso se espera encontrar **Java 21**.

Un resultado válido será similar a:

```text
openjdk 21...
```

y:

```text
javac 21...
```

El texto puede variar según la distribución del JDK instalada.

Si ambos comandos funcionan, continúa.

Si aparece otra versión, alguno de los comandos no existe o Java no está disponible, **no instales ni desinstales software por tu cuenta**. Informa al instructor para revisar el equipo.

### 2. Verificar el soporte de Java en VS Code

Abre la vista de extensiones con:

**Ctrl + Shift + X**

Busca:

- **Language Support for Java™ by Red Hat**
- **Debugger for Java**

Si ya aparecen instaladas, continúa.

Si falta alguna y VS Code permite instalarla normalmente, puedes hacerlo. Si la instalación está restringida, continúa trabajando desde la terminal e informa al instructor.

### 3. Crear el primer programa

Crea una carpeta para el laboratorio:

```text
java-laboratorio-01/
```

Ábrela desde VS Code y crea el archivo:

```text
HolaJava.java
```

Escribe:

```java
public class HolaJava {

    public static void main(String[] args) {
        System.out.println("Java está funcionando.");
    }
}
```

Guarda el archivo.

Observa que:

- el archivo se llama `HolaJava.java`;
- la clase pública se llama `HolaJava`;
- la ejecución comienza en el método `main`.

Por ahora conserva siempre la correspondencia entre el nombre del archivo y el nombre de la clase pública.

### 4. Ejecutar desde VS Code

Si las extensiones de Java están activas, sobre el método `main` aparecerán las opciones **Run** y **Debug**.

Selecciona **Run**.

La salida debe incluir:

```text
Java está funcionando.
```

### 5. Compilar y ejecutar desde la terminal

Aunque VS Code facilite la ejecución, conviene reconocer el proceso que ocurre detrás del botón **Run**.

En la terminal, ubícate en la carpeta donde guardaste `HolaJava.java`.

Compila:

```bash
javac HolaJava.java
```

Si el código no tiene errores aparecerá:

```text
HolaJava.class
```

Ejecuta:

```bash
java HolaJava
```

Resultado:

```text
Java está funcionando.
```

El recorrido básico es:

```text
HolaJava.java
      |
      | javac
      v
HolaJava.class
      |
      | java
      v
     JVM
      |
      v
salida en consola
```

### Ejercicio — Modificar la salida

Modifica `HolaJava.java` para obtener una salida semejante a esta:

```text
=========================
     PRIMER PROGRAMA
=========================
Nombre: Laura
Programa: ADSO
Trimestre: 5

Entorno Java listo.
```

Utiliza varias instrucciones `System.out.println(...)`.

Compila y ejecuta nuevamente.

### Laboratorio de errores

Los errores de compilación forman parte del trabajo cotidiano. En esta actividad vas a provocarlos intencionalmente, uno a la vez.

#### Error 1 — Falta un punto y coma

Cambia temporalmente:

```java
System.out.println("Java está funcionando.");
```

por:

```java
System.out.println("Java está funcionando.")
```

Compila y lee el mensaje que aparece.

Después corrige el código.

#### Error 2 — Mayúsculas y minúsculas

Prueba:

```java
system.out.println("Java está funcionando.");
```

Compila, observa el error y corrígelo.

#### Error 3 — Archivo y clase con nombres diferentes

Crea un archivo `Prueba.java` con:

```java
public class OtraClase {

    public static void main(String[] args) {
        System.out.println("Prueba");
    }
}
```

Compila el archivo y observa el mensaje.

Después haz coincidir el nombre del archivo con el de la clase.

### Reto

Crea `Presentacion.java`.

El programa debe mostrar una presentación breve con al menos cuatro datos. Diseña tú mismo el formato de salida.

Comprueba las dos formas de ejecución:

```bash
javac Presentacion.java
java Presentacion
```

y, si el soporte Java está disponible en VS Code, mediante **Run**.

### Comprobación

Antes de continuar verifica que puedes responder con tus propias palabras:

- ¿Qué archivo escribes tú: `.java` o `.class`?
- ¿Qué comando compila?
- ¿Qué comando ejecuta?
- ¿Dónde comienza la ejecución del programa?
- ¿Por qué importa el uso correcto de mayúsculas y minúsculas?

### Referencias oficiales

- [Getting Started with Java in VS Code](https://code.visualstudio.com/docs/java/java-tutorial)
- [Java in Visual Studio Code](https://code.visualstudio.com/docs/languages/java)
- [JDK 21 Tool Specifications](https://docs.oracle.com/en/java/javase/21/docs/specs/man/index.html)

---

## Práctica 1.2 — Variables, tipos de datos y constantes

### Objetivos

Al finalizar esta práctica podrás:

- declarar e inicializar variables;
- reconocer algunos tipos de datos de uso frecuente;
- modificar el valor almacenado en una variable;
- declarar valores que no deben cambiar;
- utilizar variables para construir una salida.

### 1. Guardar información

Un programa necesita almacenar datos mientras se ejecuta. En Java cada variable tiene un tipo definido.

Crea `DatosBasicos.java`:

```java
public class DatosBasicos {

    public static void main(String[] args) {
        String producto = "Teclado";
        int cantidad = 3;
        double precio = 85000.0;
        boolean disponible = true;
        char categoria = 'A';

        System.out.println("Producto: " + producto);
        System.out.println("Cantidad: " + cantidad);
        System.out.println("Precio: $" + precio);
        System.out.println("Disponible: " + disponible);
        System.out.println("Categoría: " + categoria);
    }
}
```

Ejecuta el programa y relaciona cada dato con su tipo.

### 2. Tipos que utilizaremos

| Tipo | Ejemplo | Uso |
|---|---|---|
| `String` | `"Monitor"` | texto |
| `int` | `25` | números enteros |
| `double` | `19.95` | números con decimales |
| `boolean` | `true` | verdadero o falso |
| `char` | `'A'` | un carácter |

Observa la diferencia:

```java
String letraComoTexto = "A";
char letraComoCaracter = 'A';
```

Las cadenas utilizan comillas dobles. Los caracteres utilizan comillas simples.

Java también distingue mayúsculas y minúsculas. `String` y `string` no representan lo mismo.

### 3. Declarar, inicializar y reasignar

Una variable puede declararse y recibir un valor:

```java
int cantidad = 10;
```

Luego puede cambiar:

```java
System.out.println(cantidad);

cantidad = 15;

System.out.println(cantidad);
```

Ejecuta el código.

Responde antes de continuar:

- ¿qué valor se muestra primero?
- ¿qué valor se muestra después?
- ¿cambió el tipo de la variable o solamente su contenido?

### Ejercicio — Estado de un inventario

Crea `Inventario.java`.

Declara:

```java
String producto = "Monitor";
int unidades = 8;
double precio = 720000.0;
boolean disponible = true;
```

Muestra los datos.

Luego simula una actualización:

```java
unidades = 5;
precio = 699000.0;
```

Vuelve a mostrar los valores.

La salida debe permitir reconocer el estado inicial y el estado actualizado.

### 4. Valores que no deberían cambiar

Algunos datos se mantienen iguales durante una ejecución. Java permite marcar una variable con `final` para evitar que reciba otro valor.

Ejemplo:

```java
final double IVA = 0.19;
```

Prueba:

```java
IVA = 0.20;
```

Observa qué indica el compilador y luego elimina esa línea.

En estas prácticas usaremos nombres en mayúscula para reconocer fácilmente las constantes:

```java
final double IVA = 0.19;
final int MESES_ANIO = 12;
```

### 5. Leer código antes de ejecutarlo

Observa:

```java
public class LecturaVariables {

    public static void main(String[] args) {
        String nombre = "Ana";
        int edad = 19;

        System.out.println(nombre);
        System.out.println(edad);

        edad = 20;

        System.out.println(edad);
    }
}
```

Sin ejecutarlo todavía, escribe las tres líneas que esperas ver.

Después ejecútalo y compara tu predicción.

### Ejercicio — Perfil de aprendiz

Crea `PerfilAprendiz.java`.

Declara variables para:

- nombre;
- edad;
- trimestre;
- promedio;
- estado activo.

Muestra una ficha completa en consola.

Después modifica al menos dos valores y vuelve a ejecutar.

### Depuración — Corrige el programa

Copia este código en `ErrorVariables.java`:

```java
public class ErrorVariables {

    public static void main(String[] args) {
        int edad = "20";
        double precio = 85000.0
        boolean activo = "true";

        System.out.println(edad);
        System.out.println(precio);
        System.out.println(activo);
    }
}
```

El programa contiene varios errores.

Corrígelos uno a uno. Cada vez que hagas una corrección, vuelve a compilar y revisa si queda algún mensaje pendiente.

No reemplaces todo el código por otro programa.

### Reto

Representa mediante variables los datos básicos de un producto:

- nombre;
- código;
- precio;
- cantidad disponible;
- categoría;
- estado activo.

Incluye además una constante para representar el porcentaje de impuesto que se utilizará más adelante.

Muestra una ficha semejante a:

```text
-------------------------
       PRODUCTO
-------------------------
Código: 105
Nombre: Mouse inalámbrico
Categoría: A
Precio: $78000.0
Cantidad: 12
Activo: true
Impuesto: 0.19
```

### Comprobación

Antes de continuar asegúrate de distinguir:

- declaración;
- inicialización;
- reasignación;
- tipo de dato;
- constante.

---

## Práctica 1.3 — Operadores y expresiones

### Objetivos

Al finalizar esta práctica podrás:

- realizar operaciones con valores numéricos;
- utilizar suma, resta, multiplicación, división y residuo;
- reconocer la diferencia entre división entera y decimal;
- controlar el orden de una expresión con paréntesis;
- almacenar resultados intermedios para revisar los cálculos.

### 1. Operaciones básicas

| Operador | Operación |
|---|---|
| `+` | suma |
| `-` | resta |
| `*` | multiplicación |
| `/` | división |
| `%` | residuo |

Crea `Operadores.java`:

```java
public class Operadores {

    public static void main(String[] args) {
        int a = 10;
        int b = 3;

        System.out.println(a + b);
        System.out.println(a - b);
        System.out.println(a * b);
        System.out.println(a / b);
        System.out.println(a % b);
    }
}
```

### Antes de ejecutar

Escribe primero qué resultado esperas en cada línea.

Luego ejecuta y compara.

Presta especial atención a:

```java
a / b
```

y:

```java
a % b
```

### 2. División entera y división decimal

Ejecuta:

```java
int resultadoEntero = 5 / 2;
double resultadoDecimal = 5.0 / 2;

System.out.println(resultadoEntero);
System.out.println(resultadoDecimal);
```

Explica con una frase por qué los resultados son diferentes.

Prueba también:

```java
double otroResultado = 5 / 2;
System.out.println(otroResultado);
```

¿El resultado es el que esperabas?

### 3. El residuo `%`

El operador `%` devuelve lo que sobra de una división entera.

Ejemplo:

```java
int totalMinutos = 135;
int horas = totalMinutos / 60;
int minutos = totalMinutos % 60;

System.out.println("Horas: " + horas);
System.out.println("Minutos restantes: " + minutos);
```

Resultado:

```text
Horas: 2
Minutos restantes: 15
```

### Ejercicio — Conversión de tiempo

Crea `ConversionTiempo.java`.

Usa un total de **367 minutos** y calcula:

- horas completas;
- minutos restantes.

Después cambia el total por otros tres valores y comprueba el resultado.

### 4. Orden de las operaciones

Antes de ejecutar, predice:

```java
int resultado1 = 10 + 5 * 2;
int resultado2 = (10 + 5) * 2;

System.out.println(resultado1);
System.out.println(resultado2);
```

Ejecuta el programa.

Los paréntesis permiten hacer explícito qué parte de una expresión debe resolverse primero.

### Ejercicio — Promedio

Crea `CalculoPromedio.java`.

Declara tres calificaciones y calcula su promedio.

Prueba primero:

```java
int nota1 = 4;
int nota2 = 5;
int nota3 = 3;
```

Decide qué tipo debe tener la variable que almacena el promedio y revisa que la división no pierda la parte decimal.

### 5. Construir un cálculo por etapas

Crea `CalculoCompra.java`:

```java
public class CalculoCompra {

    public static void main(String[] args) {
        double precio = 85000.0;
        int cantidad = 3;
        double porcentajeDescuento = 0.10;

        double subtotal = precio * cantidad;
        double descuento = subtotal * porcentajeDescuento;
        double total = subtotal - descuento;

        System.out.println("Precio unitario: $" + precio);
        System.out.println("Cantidad: " + cantidad);
        System.out.println("Subtotal: $" + subtotal);
        System.out.println("Descuento: $" + descuento);
        System.out.println("Total: $" + total);
    }
}
```

Modifica:

- precio;
- cantidad;
- porcentaje de descuento.

Comprueba cómo cambia cada resultado.

### Ejercicio — Cálculo de salario

Crea `CalculoSalario.java`.

Utiliza variables para:

- nombre del trabajador;
- horas trabajadas;
- valor por hora.

Calcula el pago y muestra un resumen.

Luego agrega una constante:

```java
final double APORTE = 0.04;
```

Calcula cuánto representa el 4 % del pago.

No necesitas aplicar todavía reglas laborales reales; el ejercicio busca practicar operaciones.

### Depuración — ¿Qué está calculando?

Analiza este código antes de ejecutarlo:

```java
int unidades = 4;
double precio = 25000.0;
double descuento = 0.20;

double subtotal = unidades * precio;
double valorDescuento = subtotal * descuento;
double total = subtotal - valorDescuento;

System.out.println(total);
```

Responde:

1. ¿cuál es el subtotal?
2. ¿cuál es el valor del descuento?
3. ¿qué imprime la última línea?

Después ejecuta para comprobar.

### Reto — Factura simple

Crea `FacturaSimple.java`.

Define:

- nombre del producto;
- precio unitario;
- cantidad;
- porcentaje de descuento.

El programa debe mostrar:

1. subtotal;
2. valor del descuento;
3. total después del descuento.

No hagas todos los cálculos dentro de `println`. Guarda cada resultado en una variable.

Prueba como mínimo estos valores:

```text
Precio: 100000
Cantidad: 2
Descuento: 0.10
```

Resultado esperado:

```text
Subtotal: $200000.0
Descuento: $20000.0
Total: $180000.0
```

### Reto adicional

Agrega un impuesto y calcula:

- base después del descuento;
- valor del impuesto;
- total final.

Mantén cada cálculo en una variable diferente.

---

## Práctica 1.4 — Entrada de datos con Scanner

Hasta ahora los valores han estado escritos directamente en el código. A partir de esta práctica los programas recibirán información desde la terminal.

### Objetivos

Al finalizar esta práctica podrás:

- crear un objeto `Scanner`;
- leer texto y números;
- utilizar los datos ingresados en expresiones;
- reconocer un comportamiento frecuente al combinar lecturas numéricas y texto;
- convertir ejercicios anteriores en programas interactivos.

### 1. Leer texto

Crea `Saludo.java`:

```java
import java.util.Scanner;

public class Saludo {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Escribe tu nombre: ");
        String nombre = scanner.nextLine();

        System.out.println("Hola, " + nombre);

        scanner.close();
    }
}
```

Ejecuta el programa.

La terminal quedará esperando:

```text
Escribe tu nombre:
```

Escribe un nombre y presiona **Enter**.

Ejecuta nuevamente y utiliza otro nombre. El código no debe cambiar; solamente cambia la entrada.

### 2. Leer números

`Scanner` ofrece métodos diferentes según el tipo de dato:

```java
System.out.print("Edad: ");
int edad = scanner.nextInt();

System.out.print("Cantidad: ");
int cantidad = scanner.nextInt();

System.out.print("Precio: ");
double precio = scanner.nextDouble();
```

Crea `EntradaNumeros.java` y prueba las tres lecturas.

Si tu configuración regional utiliza coma como separador decimal, `nextDouble()` puede esperar un valor como `19,5` en lugar de `19.5`. Si ocurre, consulta al instructor antes de modificar la configuración del equipo.

### Ejercicio — Suma interactiva

Crea `SumaInteractiva.java`.

Solicita dos números enteros y muestra:

- suma;
- resta;
- multiplicación;
- división;
- residuo.

Antes de ejecutar con nuevos valores, intenta predecir la salida.

### 3. Cuando `nextLine()` parece saltarse una lectura

Crea `LecturasMixtas.java`:

```java
import java.util.Scanner;

public class LecturasMixtas {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Edad: ");
        int edad = scanner.nextInt();

        System.out.print("Nombre: ");
        String nombre = scanner.nextLine();

        System.out.println("Nombre: " + nombre);
        System.out.println("Edad: " + edad);

        scanner.close();
    }
}
```

Ejecuta el programa.

Es posible que después de escribir la edad parezca que el programa no permite ingresar el nombre.

Esto sucede porque `nextInt()` lee el número, pero queda pendiente el salto de línea que se produjo al presionar **Enter**. El siguiente `nextLine()` consume ese salto.

Una forma sencilla de resolverlo es:

```java
System.out.print("Edad: ");
int edad = scanner.nextInt();
scanner.nextLine();

System.out.print("Nombre: ");
String nombre = scanner.nextLine();
```

Corrige el programa y ejecútalo otra vez.

### Ejercicio — Datos de usuario

Crea `DatosUsuario.java`.

Solicita:

1. nombre;
2. ciudad;
3. edad;
4. cantidad de cursos terminados.

Después muestra una ficha con todos los datos ingresados.

### 4. Utilizar la entrada en un cálculo

Crea `ConversorTemperatura.java`.

Solicita una temperatura en grados Celsius y calcula Fahrenheit con:

```text
°F = (°C × 9 / 5) + 32
```

Ejemplo de ejecución:

```text
Temperatura en °C: 25
25.0 °C equivalen a 77.0 °F
```

Prueba al menos:

- 0 °C;
- 25 °C;
- 100 °C.

### Ejercicio — Salario interactivo

Transforma `CalculoSalario.java`.

El nuevo programa debe solicitar:

```text
Nombre del trabajador:
Horas trabajadas:
Valor por hora:
```

y producir un resumen como:

```text
-------------------------
RESUMEN DE PAGO
-------------------------
Trabajador: Sara
Horas: 36
Valor por hora: $20000.0
Pago total: $720000.0
```

### Depuración — Programa incompleto

Completa las partes faltantes:

```java
import java.util.Scanner;

public class CompraInteractiva {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Producto: ");
        String producto = ______________________;

        System.out.print("Precio: ");
        double precio = ______________________;

        System.out.print("Cantidad: ");
        int cantidad = ______________________;

        double subtotal = ______________________;

        System.out.println("Producto: " + producto);
        System.out.println("Subtotal: $" + subtotal);

        scanner.close();
    }
}
```

No consultes el ejemplo anterior hasta haber intentado completar todas las líneas.

### Comprobación

Antes de continuar asegúrate de poder explicar:

- para qué sirve `Scanner`;
- qué diferencia hay entre `nextLine()`, `nextInt()` y `nextDouble()`;
- por qué a veces se usa un `nextLine()` adicional después de leer un número;
- qué ventaja tiene pedir los datos al usuario en lugar de escribirlos directamente en el código.

---

## Práctica 1.5 — Reto integrador: cotizador de compra

En esta práctica vas a reunir lo trabajado en el laboratorio. No hay una solución completa para copiar.

### Objetivo

Construir una aplicación de consola que solicite los datos de una compra, realice los cálculos y presente un resumen ordenado.

### Requisitos

Crea:

```text
CotizadorCompra.java
```

El programa debe solicitar:

- nombre del cliente;
- nombre del producto;
- precio unitario;
- cantidad;
- porcentaje de descuento;
- porcentaje de impuesto.

Los porcentajes se ingresarán como números enteros. Por ejemplo:

```text
10
```

representa 10 %.

A partir de esos datos debes obtener:

- subtotal;
- valor del descuento;
- base después del descuento;
- valor del impuesto;
- total final.

No escribas valores calculados directamente en la salida. Cada resultado debe almacenarse primero en una variable.

### Ejemplo de ejecución

```text
=============================
       COTIZADOR JAVA
=============================

Nombre del cliente: Laura
Producto: Teclado mecánico
Precio unitario: 180000
Cantidad: 2
Descuento (%): 10
Impuesto (%): 19

-----------------------------
RESUMEN DE COMPRA
-----------------------------
Cliente: Laura
Producto: Teclado mecánico
Precio unitario: $180000.0
Cantidad: 2
Subtotal: $360000.0
Descuento: $36000.0
Base: $324000.0
Impuesto: $61560.0
TOTAL: $385560.0
```

El impuesto de este ejercicio se aplica sobre el valor obtenido después del descuento. Se utiliza únicamente como práctica de expresiones aritméticas; no representa una regla tributaria o comercial.

### Antes de programar

Escribe en comentarios, con tus propias palabras, qué cálculos necesitas realizar y en qué orden.

Ejemplo:

```java
// 1. Calcular ...
// 2. Calcular ...
// 3. ...
```

No escribas todavía las expresiones. Primero organiza el problema.

### Construcción

Puedes apoyarte en lo trabajado durante las prácticas anteriores, pero evita copiar un programa completo y luego cambiar nombres.

Construye y prueba por etapas:

1. crea el `Scanner`;
2. solicita y muestra los datos sin hacer cálculos;
3. calcula únicamente el subtotal y compruébalo;
4. agrega el descuento;
5. agrega el impuesto;
6. organiza la salida final.

Ejecuta el programa después de cada etapa.

### Pistas

Si necesitas convertir un porcentaje entero a una proporción decimal:

```java
double porcentaje = valorIngresado / 100.0;
```

Conviene utilizar variables intermedias con nombres que expliquen qué representan:

```java
double subtotal;
double valorDescuento;
double base;
double valorImpuesto;
double total;
```

Si el resultado no coincide con lo esperado, imprime temporalmente los valores intermedios y localiza en qué cálculo aparece la diferencia.

### Pruebas

Comprueba el programa con distintos escenarios.

#### Caso 1

```text
Precio: 100000
Cantidad: 2
Descuento: 10
Impuesto: 0
```

Resultado final esperado:

```text
180000.0
```

#### Caso 2

```text
Precio: 50000
Cantidad: 3
Descuento: 0
Impuesto: 0
```

Resultado final esperado:

```text
150000.0
```

#### Caso 3

```text
Precio: 80000
Cantidad: 5
Descuento: 25
Impuesto: 0
```

Resultado final esperado:

```text
300000.0
```

#### Caso 4

Utiliza:

```text
Precio: 100000
Cantidad: 1
Descuento: 0
Impuesto: 19
```

Calcula primero el resultado manualmente. Luego ejecuta el programa y comprueba si coincide.

### Revisión del código

Antes de terminar revisa:

- ¿los nombres de las variables permiten entender qué almacenan?
- ¿cada dato tiene un tipo adecuado?
- ¿los cálculos están separados en variables?
- ¿el usuario puede cambiar todos los datos sin editar el código?
- ¿la salida permite entender cómo se obtuvo el total?
- ¿el programa compila sin advertencias o errores que no comprendas?

### Reto de ampliación

Agrega al cotizador:

- costo de envío;
- nombre del vendedor;
- código de la cotización.

El costo de envío debe sumarse después de calcular el impuesto.

Muestra todos los datos en el resumen.

### Reto de depuración

Crea una copia del programa y provoca intencionalmente tres errores diferentes:

1. uno de sintaxis;
2. uno relacionado con un tipo de dato;
3. uno en una expresión aritmética que compile pero produzca un resultado incorrecto.

Corrige los tres y explica, en comentarios, qué ocurría en cada caso.

---

## Cierre del laboratorio

Al finalizar deberías poder:

- reconocer la estructura básica de una clase Java con `main`;
- compilar y ejecutar un archivo Java;
- utilizar VS Code y la terminal para trabajar con Java;
- declarar, inicializar y modificar variables;
- utilizar tipos de datos básicos y constantes;
- construir expresiones aritméticas;
- reconocer la división entera y utilizar el operador residuo;
- leer datos con `Scanner`;
- identificar y corregir errores sencillos;
- construir un programa de consola que reciba datos, procese información y presente un resultado.

Conserva los archivos creados durante las prácticas. Servirán como referencia en los siguientes laboratorios.
