# Java — Día 1: primeros pasos con el lenguaje

Durante esta jornada vas a preparar el espacio de trabajo y a construir tus primeros programas en Java. El objetivo no es memorizar sintaxis: al terminar debes poder crear un archivo Java, ejecutarlo, trabajar con variables, recibir datos desde teclado y resolver cálculos sencillos.

**Tiempo de trabajo efectivo:** 4 horas y 30 minutos.

| Práctica | Tema | Tiempo aproximado |
|---|---|---:|
| 1.1 | Verificación del entorno | 30 min |
| 1.2 | Primer programa, variables y tipos de datos | 60 min |
| 1.3 | Operadores y expresiones | 60 min |
| 1.4 | Entrada de datos con `Scanner` | 60 min |
| 1.5 | Reto integrador: cotizador de compra | 60 min |

> Trabaja sobre los ejemplos, ejecútalos y modifícalos. No avances si el programa anterior todavía presenta errores que no comprendes.

---

## Práctica 1.1 — Verificación del entorno

### Objetivos

Al terminar esta práctica podrás:

- comprobar que el JDK está disponible;
- verificar que Visual Studio Code reconoce archivos Java;
- ejecutar un programa desde VS Code;
- compilar y ejecutar el mismo programa desde la terminal.

La mayor parte de los equipos del ambiente ya debería contar con las herramientas necesarias. Por eso, antes de instalar o modificar algo, verifica lo que ya está disponible.

### 1. Verificar Java

Abre Visual Studio Code y luego una terminal desde:

`Terminal > New Terminal`

También puedes usar:

`Ctrl + ``

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

No es necesario que el texto sea exactamente igual; puede cambiar según la distribución del JDK instalada.

Si ambos comandos funcionan, continúa con la práctica.

Si aparece otra versión, alguno de los comandos no existe o Java no está disponible, **no instales ni desinstales software por tu cuenta**. Informa al instructor para revisar el equipo.

### 2. Verificar el soporte de Java en VS Code

Abre la vista de extensiones con:

`Ctrl + Shift + X`

Busca estas extensiones:

- **Language Support for Java™ by Red Hat**
- **Debugger for Java**

Para las primeras prácticas son suficientes. La documentación oficial de Visual Studio Code indica que estas dos extensiones cubren lo necesario para su tutorial básico de Java.

Si ya aparecen instaladas, no debes hacer nada más.

Si falta alguna y VS Code permite instalarla normalmente, puedes hacerlo. Si la instalación está restringida en el equipo, continúa con la terminal e informa al instructor.

### 3. Crear el primer archivo

Crea una carpeta para el trabajo de la jornada. Puedes usar esta estructura:

```text
java-dia-01/
└── HolaJava.java
```

Abre la carpeta `java-dia-01` desde VS Code y crea el archivo `HolaJava.java`.

Escribe:

```java
public class HolaJava {

    public static void main(String[] args) {
        System.out.println("Java está funcionando.");
    }
}
```

Guarda el archivo.

Observa dos detalles:

- el archivo se llama `HolaJava.java`;
- la clase pública se llama `HolaJava`.

Por ahora conserva siempre esa correspondencia.

### 4. Ejecutar desde VS Code

Si las extensiones de Java están activas, sobre el método `main` aparecerán las opciones **Run** y **Debug**.

Selecciona **Run**.

En el panel inferior debe aparecer:

```text
Java está funcionando.
```

Durante estos laboratorios utilizaremos principalmente la **terminal integrada** para ver la salida y escribir datos cuando el programa los solicite.

### 5. Ejecutar desde la terminal

Aunque VS Code facilite la ejecución, conviene comprobar una vez qué ocurre detrás del botón **Run**.

Ubícate en la terminal dentro de la carpeta donde guardaste `HolaJava.java`.

Compila el archivo:

```bash
javac HolaJava.java
```

Si no hay errores, aparecerá un nuevo archivo:

```text
HolaJava.class
```

Ahora ejecútalo:

```bash
java HolaJava
```

Resultado:

```text
Java está funcionando.
```

El flujo básico es:

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

### Ejercicio

Modifica el programa para que muestre:

```text
=========================
     PRIMER PROGRAMA
=========================
Nombre: Laura
Programa: ADSO
Trimestre: 5

Entorno Java listo.
```

Puedes utilizar varias instrucciones `System.out.println(...)`.

### Reto

Crea un archivo llamado `Presentacion.java` y escribe un programa que muestre una presentación breve con al menos cuatro datos.

No copies el contenido de `HolaJava.java` sin revisarlo. Ajusta el nombre de la clase y el contenido de la salida.

### Resultado esperado

Al terminar debes poder ejecutar correctamente:

```bash
javac Presentacion.java
java Presentacion
```

Si tienes las extensiones instaladas, comprueba también que el botón **Run** produce el mismo resultado.

### Referencias oficiales

- [Getting Started with Java in VS Code](https://code.visualstudio.com/docs/java/java-tutorial)
- [Java in Visual Studio Code](https://code.visualstudio.com/docs/languages/java)
- [JDK 21 Tool Specifications](https://docs.oracle.com/en/java/javase/21/docs/specs/man/index.html)

---

## Práctica 1.2 — Variables y tipos de datos

### Objetivos

Al terminar esta práctica podrás:

- declarar variables;
- reconocer algunos de los tipos de datos más utilizados;
- modificar el valor de una variable;
- utilizar variables para construir una salida.

### 1. Guardar información

Un programa necesita almacenar información mientras se está ejecutando. En Java cada variable tiene un tipo definido.

Revisa el siguiente ejemplo:

```java
public class DatosBasicos {

    public static void main(String[] args) {
        String producto = "Teclado";
        int cantidad = 3;
        double precio = 85000.0;
        boolean disponible = true;

        System.out.println("Producto: " + producto);
        System.out.println("Cantidad: " + cantidad);
        System.out.println("Precio: $" + precio);
        System.out.println("Disponible: " + disponible);
    }
}
```

Crea el archivo `DatosBasicos.java`, ejecútalo y observa la salida.

### 2. Tipos que utilizaremos hoy

| Tipo | Ejemplo | Uso |
|---|---|---|
| `String` | `"Monitor"` | texto |
| `int` | `25` | números enteros |
| `double` | `19.95` | números con decimales |
| `boolean` | `true` | valores verdadero/falso |
| `char` | `'A'` | un carácter |

Java distingue mayúsculas y minúsculas. `String` y `string` no representan lo mismo.

### Ejercicio

Crea un archivo `PerfilAprendiz.java`.

Declara variables para almacenar:

- nombre;
- edad;
- trimestre;
- promedio;
- estado activo.

Muestra todos los datos en consola.

Una posible salida es:

```text
Nombre: Andrea
Edad: 20
Trimestre: 5
Promedio: 4.2
Activo: true
```

Después cambia los valores y vuelve a ejecutar el programa.

### Reto

Representa mediante variables los datos básicos de un producto:

- nombre;
- código;
- precio;
- cantidad disponible;
- estado activo.

Muestra una ficha sencilla:

```text
-------------------------
       PRODUCTO
-------------------------
Código: 105
Nombre: Mouse inalámbrico
Precio: $78000.0
Cantidad: 12
Activo: true
```

### Pistas

- El nombre del producto es texto.
- La cantidad disponible es un entero.
- El precio puede contener decimales.
- El estado puede representarse con `boolean`.

### Resultado esperado

El programa debe compilar sin errores y la salida debe construirse a partir de variables, no escribiendo todos los datos directamente dentro de `println`.

---

## Práctica 1.3 — Operadores y expresiones

### Objetivos

Al terminar esta práctica podrás:

- realizar cálculos con variables;
- utilizar los operadores aritméticos básicos;
- almacenar el resultado de una expresión;
- dividir un problema sencillo en cálculos intermedios.

### 1. Operaciones básicas

Java permite utilizar operadores aritméticos como:

| Operador | Operación |
|---|---|
| `+` | suma |
| `-` | resta |
| `*` | multiplicación |
| `/` | división |
| `%` | residuo |

Ejecuta este ejemplo:

```java
public class CalculoCompra {

    public static void main(String[] args) {
        double precio = 85000.0;
        int cantidad = 3;

        double subtotal = precio * cantidad;

        System.out.println("Precio unitario: $" + precio);
        System.out.println("Cantidad: " + cantidad);
        System.out.println("Subtotal: $" + subtotal);
    }
}
```

### 2. Construir el cálculo por etapas

Ahora agrega un descuento del 10 %.

```java
double porcentajeDescuento = 0.10;
double descuento = subtotal * porcentajeDescuento;
double total = subtotal - descuento;
```

Muestra por separado:

```text
Subtotal: ...
Descuento: ...
Total: ...
```

### Ejercicio

Crea un programa llamado `CalculoSalario.java`.

Utiliza variables para representar:

- nombre del trabajador;
- horas trabajadas;
- valor por hora.

Calcula:

```text
salario = horas trabajadas × valor por hora
```

Muestra un resumen similar a:

```text
Trabajador: Camila
Horas trabajadas: 40
Valor por hora: $18000.0
Pago: $720000.0
```

### Reto

Crea `FacturaSimple.java`.

El programa debe tener:

- nombre del producto;
- precio unitario;
- cantidad;
- porcentaje de descuento.

Debe calcular y mostrar:

1. subtotal;
2. valor del descuento;
3. total después del descuento.

### Pistas

No intentes resolver todo dentro de un único `println`.

Calcula cada valor y guárdalo:

```java
double subtotal = ...;
double descuento = ...;
double total = ...;
```

### Resultado esperado

Para un producto de $100000, cantidad 2 y descuento de 10 %, el resultado debe ser:

```text
Subtotal: $200000.0
Descuento: $20000.0
Total: $180000.0
```

### Reto adicional

Agrega una variable para representar un impuesto y calcula el valor final después del descuento y del impuesto.

---

## Práctica 1.4 — Entrada de datos con Scanner

Hasta ahora los datos han estado escritos directamente en el código. El siguiente paso es permitir que la persona que ejecuta el programa los ingrese desde la terminal.

### Objetivos

Al terminar esta práctica podrás:

- crear un objeto `Scanner`;
- leer texto y números desde teclado;
- utilizar los datos ingresados en un cálculo;
- construir un programa interactivo sencillo.

### 1. Leer datos desde la terminal

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

### 2. Leer valores numéricos

`Scanner` también permite leer números.

```java
System.out.print("Ingresa tu edad: ");
int edad = scanner.nextInt();

System.out.print("Ingresa tu estatura: ");
double estatura = scanner.nextDouble();
```

Para esta práctica solicita primero los textos y después los valores numéricos. Más adelante revisaremos con detalle la validación de entradas.

### Ejercicio

Crea `DatosUsuario.java`.

El programa debe solicitar:

1. nombre;
2. edad;
3. ciudad;
4. estatura.

Después muestra un resumen con los valores ingresados.

### Reto

Transforma el ejercicio de salario de la práctica anterior.

El nuevo programa debe preguntar:

```text
Nombre del trabajador:
Horas trabajadas:
Valor por hora:
```

y calcular el pago utilizando los valores escritos por el usuario.

Ejemplo:

```text
Nombre del trabajador: Sara
Horas trabajadas: 36
Valor por hora: 20000

-------------------------
RESUMEN DE PAGO
-------------------------
Trabajador: Sara
Horas: 36
Valor por hora: $20000.0
Pago total: $720000.0
```

### Pistas

Recuerda importar:

```java
import java.util.Scanner;
```

Crear el lector:

```java
Scanner scanner = new Scanner(System.in);
```

Y cerrarlo cuando ya no sea necesario:

```java
scanner.close();
```

### Resultado esperado

El resultado del programa debe cambiar cuando cambien los datos ingresados, sin necesidad de editar el código fuente.

---

## Práctica 1.5 — Reto integrador: cotizador de compra

Esta práctica reúne lo trabajado durante la jornada. No hay un programa completo para copiar. Parte de los ejemplos anteriores y construye la solución paso a paso.

### Objetivo

Crear una aplicación de consola que reciba los datos de una compra y genere un resumen con los cálculos correspondientes.

### Requisitos

Crea un archivo:

```text
CotizadorCompra.java
```

El programa debe solicitar:

- nombre del cliente;
- nombre del producto;
- precio unitario;
- cantidad;
- porcentaje de descuento.

El porcentaje se ingresará como un número entero. Por ejemplo:

```text
10
```

representa un descuento del 10 %.

El programa debe calcular:

```text
subtotal = precio × cantidad
porcentaje = descuento / 100
valor descuento = subtotal × porcentaje
total = subtotal - valor descuento
```

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

-----------------------------
RESUMEN DE COMPRA
-----------------------------
Cliente: Laura
Producto: Teclado mecánico
Precio unitario: $180000.0
Cantidad: 2
Subtotal: $360000.0
Descuento: $36000.0
Total: $324000.0
```

### Pistas

Necesitarás combinar:

- variables;
- `Scanner`;
- multiplicación;
- división;
- resta;
- concatenación de texto.

Para convertir un porcentaje entero a su valor decimal puedes dividir entre `100.0`:

```java
double porcentaje = descuentoIngresado / 100.0;
```

Utiliza variables intermedias para los cálculos. Esto facilita revisar el programa cuando algo no produce el resultado esperado.

### Comprobaciones

Antes de dar por terminado el reto, prueba al menos estos casos:

| Precio | Cantidad | Descuento | Total esperado |
|---:|---:|---:|---:|
| 100000 | 2 | 10 | 180000 |
| 50000 | 3 | 0 | 150000 |
| 80000 | 5 | 25 | 300000 |

Si alguno de los resultados no coincide, revisa primero las operaciones antes de cambiar varias líneas al mismo tiempo.

### Reto adicional

Agrega al programa:

- porcentaje de impuesto;
- valor del impuesto;
- total final.

Decide en qué momento debe aplicarse el impuesto y deja el cálculo separado en variables para que pueda revisarse con facilidad.

---

## Cierre de la jornada

Al terminar el día debes poder:

- reconocer la estructura básica de una clase Java con `main`;
- compilar y ejecutar un archivo Java;
- ejecutar el mismo archivo desde VS Code;
- declarar variables de diferentes tipos;
- realizar operaciones con esas variables;
- leer datos utilizando `Scanner`;
- construir un programa de consola que reciba datos y genere un resultado.

Conserva los archivos de las cinco prácticas. Los utilizaremos como referencia durante el desarrollo del curso.
