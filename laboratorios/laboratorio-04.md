# Laboratorio 4 — Colecciones y manejo de excepciones en Java

En el [Laboratorio 3](laboratorio-03.md) construiste una billetera orientada a objetos. Cada instancia conservaba su saldo y controlaba las operaciones de ingreso y gasto. Sin embargo, la aplicación no recordaba cada operación individual: conocía los totales, pero no tenía un historial que pudiera recorrerse o consultarse.

En este laboratorio aprenderás a trabajar con varios datos y objetos mediante arreglos, listas, conjuntos y mapas. También manejarás errores de entrada para evitar que un valor inesperado interrumpa el programa.

El reto final será ampliar la billetera para registrar cada movimiento válido, consultar su historial y generar reportes por categoría.

| Práctica | Tema |
|---|---|
| 4.1 | Arreglos, índices y recorridos |
| 4.2 | `List`, `ArrayList` y listas de objetos |
| 4.3 | `Set`, `HashSet`, `Map` y `HashMap` |
| 4.4 | Manejo de excepciones y entradas seguras |
| 4.5 | Reto integrador: billetera con historial y reportes |

> Lee, predice, ejecuta y modifica los ejemplos antes de resolver los ejercicios. En las actividades de depuración algunos fragmentos tienen errores intencionales. No los confundas con los ejemplos que deben ejecutarse correctamente.

## Preparación del trabajo

Conserva los archivos del Laboratorio 3, especialmente `Billetera.java`, `Main.java` y, si lo construiste, `PruebaBilleteras.java`. No los reemplaces; la nueva versión deberá poder compararse con la anterior.

Crea una carpeta con esta organización:

~~~text
java-laboratorio-04/
├── practica-4-1/
├── practica-4-2/
├── practica-4-3/
├── practica-4-4/
└── MiBilleteraJava/
~~~

Trabaja con Java 21 desde VS Code o desde la terminal. Cada práctica debe tener sus propios archivos para evitar conflictos entre clases con el mismo nombre.

Para compilar y ejecutar un programa sencillo:

~~~bash
javac ArreglosBasicos.java
java ArreglosBasicos
~~~

Cuando el ejemplo tenga varias clases en archivos diferentes, compila los archivos necesarios desde la carpeta correspondiente. En el reto final utilizaremos:

~~~bash
javac Movimiento.java Billetera.java Main.java
java Main
~~~

No se necesitan Maven, Spring Boot, bases de datos ni bibliotecas externas. Si el equipo no permite ejecutar Java localmente, utiliza el entorno alternativo autorizado por el instructor sin instalar o actualizar software por tu cuenta.

---

## Práctica 4.1 — Arreglos, índices y recorridos

### Objetivos

Al terminar esta práctica podrás:

- crear arreglos de valores;
- consultar y modificar elementos mediante índices;
- utilizar `length` para conocer su tamaño;
- recorrer un arreglo con `for` y `for-each`;
- distinguir una colección de tamaño fijo de una lista que puede crecer;
- identificar un acceso fuera de los límites del arreglo.

### 1. ¿Por qué necesitamos varios valores?

En los laboratorios anteriores podíamos declarar variables independientes:

~~~java
double gasto1 = 30000;
double gasto2 = 45000;
double gasto3 = 20000;
~~~

Pero si hay muchos gastos, resultaría incómodo crear una nueva variable por cada uno.

Un arreglo permite guardar varios valores del mismo tipo:

~~~java
double[] gastos = {30000, 45000, 20000};
~~~

En este caso tenemos tres posiciones, numeradas a partir de cero.

### 2. Crear y consultar un arreglo

Crea `ArreglosBasicos.java`:

~~~java
public class ArreglosBasicos {

    public static void main(String[] args) {
        String[] productos = {"Teclado", "Mouse", "Monitor"};

        System.out.println(productos[0]);
        System.out.println(productos[1]);
        System.out.println(productos[2]);
        System.out.println("Cantidad: " + productos.length);
    }
}
~~~

**Antes de ejecutar:** escribe las cuatro líneas de salida que esperas.

Después ejecuta y confirma.

Observa que `productos.length` devuelve la cantidad de elementos. No es un método: no lleva paréntesis.

### 3. Modificar un elemento

Cambia el segundo producto:

~~~java
productos[1] = "Mouse inalámbrico";
~~~

Agrega esa instrucción dentro de `main`, después de crear el arreglo, y muestra nuevamente sus elementos.

El arreglo conserva tres posiciones. Se puede reemplazar el valor de una posición, pero no se puede ampliar su longitud después de crearlo.

### 4. Crear un arreglo con tamaño definido

Prueba:

~~~java
int[] cantidades = new int[4];

cantidades[0] = 5;
cantidades[1] = 8;
cantidades[2] = 2;
cantidades[3] = 10;

System.out.println(cantidades[2]);
System.out.println(cantidades.length);
~~~

Este fragmento debe agregarse dentro de un método `main` existente o colocarse en una clase propia.

Cuando creas un arreglo de `int` con `new int[4]`, sus elementos comienzan con el valor cero.

### Ejercicio — Ventas registradas

Crea `RegistroVentas.java`.

Declara un arreglo con los valores:

~~~java
double[] ventas = {120000, 85000, 95000, 150000, 60000};
~~~

Muestra:

- la primera venta;
- la última venta;
- la cantidad de ventas;
- la tercera venta después de modificar su valor a 100000.

No escribas cinco variables independientes.

### 5. Recorrido con `for`

Crea `RecorrerArreglo.java`:

~~~java
public class RecorrerArreglo {

    public static void main(String[] args) {
        String[] categorias = {"Transporte", "Alimentación", "Salud"};

        for (int i = 0; i < categorias.length; i++) {
            System.out.println(i + " -> " + categorias[i]);
        }
    }
}
~~~

Resultado esperado:

~~~text
0 -> Transporte
1 -> Alimentación
2 -> Salud
~~~

El ciclo utiliza `i < categorias.length`. Si escribes `i <= categorias.length`, en algún momento intentarás acceder a una posición inexistente.

### 6. Recorrido con `for-each`

Cuando solo necesitamos leer los elementos, Java ofrece otra forma:

~~~java
String[] categorias = {"Transporte", "Alimentación", "Salud"};

for (String categoria : categorias) {
    System.out.println(categoria);
}
~~~

Este fragmento también debe ir dentro de `main`.

Compara los dos recorridos. Con `for` tienes el índice; con `for-each` trabajas directamente con el valor de cada elemento.

### Ejercicio — Total y promedio

Crea `PromedioGastos.java`.

Utiliza:

~~~java
double[] gastos = {30000, 45000, 20000, 10000};
~~~

Calcula:

- suma de los gastos;
- cantidad de registros;
- promedio;
- cuántos gastos fueron superiores a 25000.

Resuelve primero la suma con un ciclo `for`. Después intenta una versión con `for-each`.

Resultados esperados:

~~~text
Total: 105000.0
Cantidad: 4
Promedio: 26250.0
Mayores que 25000: 2
~~~

### 7. Una lista de opciones conocidas

Un arreglo también puede conservar valores que no necesitamos agregar ni eliminar durante la ejecución:

~~~java
String[] tipos = {"INGRESO", "GASTO"};

for (String tipo : tipos) {
    System.out.println(tipo);
}
~~~

Más adelante, cuando el número de movimientos sea variable, utilizaremos `ArrayList`.

### Depuración — Índice incorrecto

Crea `ErrorIndice.java`:

~~~java
public class ErrorIndice {

    public static void main(String[] args) {
        int[] valores = {10, 20, 30};

        System.out.println(valores[3]);
    }
}
~~~

Ejecuta y observa la excepción. ¿Cuál es la última posición válida? Corrige la instrucción.

Después prueba modificar un ciclo para que utilice `i <= valores.length` y comprueba por qué también falla.

### Reto — Registro de calificaciones

Crea `NotasArreglo.java`.

Solicita mediante `Scanner` cinco calificaciones, guárdalas en un arreglo y muestra:

- todas las notas con su posición;
- promedio;
- nota mayor;
- nota menor;
- cantidad de notas aprobadas, considerando aprobación desde 3.0.

Primero resuelve el almacenamiento y el recorrido. Luego agrega los cálculos.

### Comprobación

Antes de continuar explica qué diferencia existe entre `length` e índice, por qué el primer elemento está en la posición cero y por qué un arreglo no permite agregar libremente una sexta posición si fue creado con cinco.

---

## Práctica 4.2 — `List`, `ArrayList` y listas de objetos

### Objetivos

Al terminar esta práctica podrás:

- crear una lista con `ArrayList`;
- comprender la utilidad de `List<T>` y los tipos genéricos;
- agregar, consultar, modificar y eliminar elementos;
- recorrer listas con `for` y `for-each`;
- almacenar objetos propios dentro de una colección;
- devolver una copia de una lista sin exponer directamente su estructura interna.

### 1. ¿Cuándo usar una lista?

La billetera puede registrar dos movimientos o doscientos. No conocemos la cantidad cuando se inicia el programa.

A diferencia de un arreglo, una lista puede crecer y reducirse a medida que agregamos o eliminamos elementos.

Crea `ListaProductos.java`:

~~~java
import java.util.ArrayList;
import java.util.List;

public class ListaProductos {

    public static void main(String[] args) {
        List<String> productos = new ArrayList<>();

        productos.add("Teclado");
        productos.add("Mouse");
        productos.add("Monitor");

        System.out.println(productos);
        System.out.println("Cantidad: " + productos.size());
    }
}
~~~

Resultado:

~~~text
[Teclado, Mouse, Monitor]
Cantidad: 3
~~~

En:

~~~java
List<String> productos = new ArrayList<>();
~~~

- `List` es la interfaz que expresa las operaciones principales de una lista;
- `String` es el tipo de elementos;
- `ArrayList` es una implementación de esa interfaz;
- `<>` permite inferir el tipo indicado a la izquierda.

Para las próximas prácticas emplearemos habitualmente `List<T>` en las variables y `new ArrayList<>()` para crear la colección.

### 2. Operaciones fundamentales

| Operación | Ejemplo | Resultado |
|---|---|---|
| Agregar | `add("Mouse")` | añade un elemento |
| Consultar | `get(0)` | devuelve el elemento de un índice |
| Reemplazar | `set(0, "Teclado")` | cambia un elemento |
| Eliminar por índice | `remove(0)` | elimina la posición indicada |
| Eliminar por valor | `remove("Mouse")` | elimina la primera coincidencia |
| Cantidad | `size()` | devuelve el número de elementos |
| Comprobar | `contains("Mouse")` | devuelve `true` o `false` |
| Lista vacía | `isEmpty()` | comprueba si no hay elementos |

Agrega dentro de `main`, después de crear la lista del ejemplo:

~~~java
productos.set(1, "Mouse inalámbrico");

System.out.println("Primero: " + productos.get(0));
System.out.println("¿Hay monitor? " + productos.contains("Monitor"));

productos.remove("Monitor");

System.out.println("Después: " + productos);
~~~

### Antes de ejecutar — Predice

Con la lista:

~~~java
List<String> nombres = new ArrayList<>();

nombres.add("Ana");
nombres.add("Luis");
nombres.add("Ana");
nombres.remove("Ana");

System.out.println(nombres);
~~~

¿Desaparecen las dos apariciones de `Ana`? Escribe tu predicción y comprueba.

### 3. Índices y cantidad

En los arreglos utilizábamos `length`. En las listas usamos `size()`:

~~~java
for (int i = 0; i < productos.size(); i++) {
    System.out.println((i + 1) + ". " + productos.get(i));
}
~~~

Con `for-each`:

~~~java
for (String producto : productos) {
    System.out.println(producto);
}
~~~

Estos fragmentos se agregan dentro de un método `main` donde ya exista la lista `productos`.

### Ejercicio — Lista de tareas

Crea `ListaTareas.java`.

Comienza con una lista vacía y permite introducir cuatro tareas mediante `Scanner`. Después:

- muestra las tareas numeradas;
- consulta la cantidad;
- pregunta por una tarea y comprueba si existe;
- elimina una tarea por su nombre si está registrada;
- muestra la lista actualizada.

Prueba la búsqueda de una tarea que no existe antes de eliminar nada.

### 4. Genéricos y tipos de elementos

`List<String>` indica una lista de cadenas. Para números enteros se utiliza `List<Integer>`, no `List<int>`:

~~~java
List<Integer> cantidades = new ArrayList<>();

cantidades.add(5);
cantidades.add(8);

System.out.println(cantidades.get(0));
~~~

Los genéricos trabajan con tipos de referencia. `Integer` es el tipo contenedor correspondiente a `int`.

### 5. Una lista de objetos propios

El verdadero objetivo de esta práctica es guardar movimientos de una billetera.

Crea en una subcarpeta de `practica-4-2` dos archivos: `Movimiento.java` e `HistorialSimple.java`.

En `Movimiento.java`:

~~~java
public class Movimiento {

    private final String tipo;
    private final String categoria;
    private final double valor;

    public Movimiento(String tipo, String categoria, double valor) {
        this.tipo = tipo;
        this.categoria = categoria;
        this.valor = valor;
    }

    public String getTipo() {
        return tipo;
    }

    public String getCategoria() {
        return categoria;
    }

    public double getValor() {
        return valor;
    }
}
~~~

Los atributos son `final` y no hay métodos que los modifiquen. Esta primera clase permite leer los datos del movimiento; en el reto integrador añadirás las validaciones de creación que correspondan.

En `HistorialSimple.java`:

~~~java
import java.util.ArrayList;
import java.util.List;

public class HistorialSimple {

    public static void main(String[] args) {
        List<Movimiento> movimientos = new ArrayList<>();

        movimientos.add(new Movimiento("INGRESO", "Salario", 1500000));
        movimientos.add(new Movimiento("GASTO", "Transporte", 30000));
        movimientos.add(new Movimiento("GASTO", "Alimentación", 45000));

        for (Movimiento movimiento : movimientos) {
            System.out.println(
                movimiento.getTipo() + " | "
                + movimiento.getCategoria() + " | $"
                + movimiento.getValor()
            );
        }

        System.out.println("Movimientos: " + movimientos.size());
    }
}
~~~

Compila y ejecuta desde esa carpeta:

~~~bash
javac Movimiento.java HistorialSimple.java
java HistorialSimple
~~~

Observa que la lista guarda referencias a objetos `Movimiento`. Cada uno conserva su propio tipo, categoría y valor.

### Ejercicio — Total de gastos

Agrega dentro de `main` de `HistorialSimple.java` un recorrido que acumule el valor de los movimientos cuyo tipo sea `"GASTO"`.

Utiliza:

~~~java
movimiento.getTipo().equals("GASTO")
~~~

Resultado esperado para los datos anteriores:

~~~text
Total de gastos: 75000.0
~~~

Después agrega otro movimiento de gasto y comprueba que el total cambie.

### 6. Consultar una lista sin exponer la colección interna

Imagina que `Billetera` conserva:

~~~java
private List<Movimiento> movimientos = new ArrayList<>();
~~~

Si un método entrega directamente esa lista, otra clase podría agregar movimientos sin pasar por las validaciones de ingresos y gastos.

Una opción sencilla consiste en devolver una copia:

~~~java
public List<Movimiento> getMovimientos() {
    return new ArrayList<>(movimientos);
}
~~~

Este fragmento pertenece a una clase que ya tenga declarado el atributo `movimientos`.

La copia protege la **estructura de la lista**: quien la recibe no puede agregar ni eliminar elementos de la lista original. Es una copia superficial, por lo que sigue apuntando a los mismos objetos `Movimiento`. En este ejercicio esos objetos no tienen setters y sus atributos son finales.

### Ejercicio — Copia del historial

En `HistorialSimple.java`:

1. crea una copia de la lista mediante `new ArrayList<>(movimientos)`;
2. elimina el primer elemento solamente de la copia;
3. imprime la cantidad de la lista original y la cantidad de la copia.

Comprueba que el historial original conserva sus tres movimientos.

### Depuración — Tipo incorrecto

Prueba, dentro de un método, estas instrucciones:

~~~java
List<String> nombres = new ArrayList<>();
nombres.add("Laura");
nombres.add(25);
~~~

Lee el error de compilación. ¿Qué ventaja aporta declarar el tipo de los elementos?

Después corrige el ejemplo para que la lista contenga únicamente cadenas.

### Depuración — Modificación durante el recorrido

No elimines elementos de un `ArrayList` directamente con `lista.remove(elemento)` mientras lo recorres con `for-each`: ese patrón puede provocar errores o comportamientos inesperados.

Para este laboratorio, primero localiza el elemento y elimínalo después del recorrido, o utiliza un método como `removeIf(...)` en ejercicios de ampliación. No necesitas aprender todavía iteradores explícitos.

### Reto — Historial de una caja

Crea `RegistroCaja.java` que mantenga una `List<Movimiento>`.

Debe:

- crear cuatro movimientos con tipos de ingreso y gasto;
- imprimirlos numerados en el orden de registro;
- calcular el total de ingresos;
- calcular el total de gastos;
- mostrar la cantidad de movimientos.

Todavía no necesitas un menú. El propósito es dominar la colección de objetos y sus recorridos.

### Comprobación

Antes de continuar explica cuándo prefieres un arreglo y cuándo una lista, qué representa `List<Movimiento>` y por qué no conviene devolver directamente la lista interna de una billetera.

---

## Práctica 4.3 — `Set`, `HashSet`, `Map` y `HashMap`

### Objetivos

Al terminar esta práctica podrás:

- utilizar un `Set` para conservar valores únicos;
- conocer la diferencia entre una lista y un conjunto;
- crear un `Map` de pares clave–valor;
- agregar, consultar y actualizar valores de un mapa;
- agrupar importes por categoría;
- elegir una estructura adecuada según el problema.

### 1. Categorías sin repetir

En un historial pueden existir varios gastos con categoría `Transporte`. Pero al consultar qué categorías se han utilizado, nos interesa mostrarlas una sola vez.

Crea `CategoriasUnicas.java`:

~~~java
import java.util.HashSet;
import java.util.Set;

public class CategoriasUnicas {

    public static void main(String[] args) {
        Set<String> categorias = new HashSet<>();

        categorias.add("Transporte");
        categorias.add("Alimentación");
        categorias.add("Transporte");
        categorias.add("Salud");

        System.out.println("Cantidad: " + categorias.size());
        System.out.println("¿Hay transporte? " + categorias.contains("Transporte"));

        for (String categoria : categorias) {
            System.out.println(categoria);
        }
    }
}
~~~

El conjunto tendrá **tres categorías distintas**. `HashSet` no garantiza un orden de recorrido. No construyas una prueba que dependa de cómo aparecen ordenadas.

### 2. Diferencias entre `List` y `Set`

| Característica | `List` / `ArrayList` | `Set` / `HashSet` |
|---|---|---|
| Elementos repetidos | Permitidos | No |
| Consulta por índice | Sí | No |
| Orden de inserción | `ArrayList` lo conserva | `HashSet` no lo garantiza |
| Uso en la billetera | Historial de movimientos | Categorías distintas |

Cuando necesitas mantener el orden del historial utiliza `ArrayList`. Cuando solo necesitas valores únicos, `HashSet` es apropiado.

### Ejercicio — Categorías de un historial

Copia `Movimiento.java` e `HistorialSimple.java` de la práctica 4.2 a una carpeta nueva.

A partir de la lista de movimientos construye:

~~~java
Set<String> categorias = new HashSet<>();
~~~

Recorre la lista y agrega cada categoría de gasto al conjunto. Muestra cuántas categorías diferentes se utilizaron.

Prueba dos gastos en `Transporte` y uno en `Salud`. El resultado debe indicar dos categorías diferentes.

### 3. Un mapa de clave y valor

Si necesitas conocer el total gastado en cada categoría, puedes representar la información así:

~~~text
Transporte -> 75000
Alimentación -> 120000
Salud -> 40000
~~~

Cada categoría funciona como clave y el importe acumulado como valor.

Crea `PrimerMapa.java`:

~~~java
import java.util.HashMap;
import java.util.Map;

public class PrimerMapa {

    public static void main(String[] args) {
        Map<String, Double> gastos = new HashMap<>();

        gastos.put("Transporte", 30000.0);
        gastos.put("Alimentación", 45000.0);

        System.out.println(gastos.get("Transporte"));
        System.out.println(gastos.containsKey("Salud"));

        gastos.put("Transporte", 50000.0);

        System.out.println(gastos.get("Transporte"));
        System.out.println("Categorías: " + gastos.size());
    }
}
~~~

`put()` agrega un par clave–valor o reemplaza el valor de una clave existente. En este ejemplo, el segundo `put("Transporte", ...)` sustituye el importe anterior; no lo suma.

### 4. Acumular importes en un mapa

Para sumar varias operaciones de la misma categoría, utiliza `getOrDefault()`.

Crea `AcumuladosMapa.java`:

~~~java
import java.util.HashMap;
import java.util.Map;

public class AcumuladosMapa {

    public static void main(String[] args) {
        Map<String, Double> totales = new HashMap<>();

        String categoria = "Transporte";
        double valor = 30000;

        double anterior = totales.getOrDefault(categoria, 0.0);
        totales.put(categoria, anterior + valor);

        valor = 45000;
        anterior = totales.getOrDefault(categoria, 0.0);
        totales.put(categoria, anterior + valor);

        System.out.println("Transporte: $" + totales.get("Transporte"));
    }
}
~~~

Resultado:

~~~text
Transporte: $75000.0
~~~

La expresión:

~~~java
totales.getOrDefault(categoria, 0.0)
~~~

devuelve el acumulado anterior o cero si la categoría todavía no existe.

### 5. Recorrer las entradas del mapa

Agrega dentro de `main` del ejemplo anterior:

~~~java
for (Map.Entry<String, Double> entrada : totales.entrySet()) {
    System.out.println(entrada.getKey() + ": $" + entrada.getValue());
}
~~~

`entrySet()` permite recorrer pares de clave y valor. Con `HashMap` tampoco debe suponerse un orden particular de las claves.

### Antes de ejecutar — Predice

¿Qué valor final tendrá cada categoría?

~~~java
Map<String, Double> valores = new HashMap<>();

valores.put("Salud", 10000.0);
valores.put("Salud", 20000.0);
valores.put("Transporte", 5000.0);

System.out.println(valores.get("Salud"));
System.out.println(valores.size());
~~~

Ejecuta y comprueba. Explica por qué `put()` no conserva los dos importes anteriores bajo una misma clave.

### Ejercicio — Gastos agrupados

Crea `GastosPorCategoria.java`.

Utiliza estas operaciones:

| Tipo | Categoría | Valor |
|---|---|---:|
| GASTO | Transporte | 30000 |
| GASTO | Alimentación | 45000 |
| GASTO | Transporte | 20000 |
| INGRESO | Salario | 1500000 |
| GASTO | Salud | 40000 |

Representa las operaciones como objetos `Movimiento` y guárdalas en una lista.

Después:

1. crea un `Map<String, Double>`;
2. recorre solamente los movimientos de tipo `GASTO`;
3. acumula los valores por categoría;
4. muestra los pares del mapa.

Resultados esperados, sin importar el orden de presentación:

~~~text
Transporte: $50000.0
Alimentación: $45000.0
Salud: $40000.0
~~~

El ingreso por salario **no** debe formar parte del mapa de gastos.

### 6. Eliminar y consultar

En un `Set` y en un `Map` existen operaciones de búsqueda y eliminación:

~~~java
categorias.remove("Salud");
~~~

~~~java
totales.remove("Transporte");
~~~

También puedes consultar:

~~~java
categorias.contains("Transporte");
totales.containsKey("Alimentación");
~~~

Estos fragmentos pertenecen a los programas donde ya declaraste las variables respectivas.

### Depuración — Clave inexistente

Observa:

~~~java
Map<String, Double> totales = new HashMap<>();
double salud = totales.get("Salud");
~~~

¿Por qué puede fallar esta segunda línea durante la ejecución?

Cuando `get()` no encuentra una clave, devuelve `null`. Aquí se intenta convertir ese valor en un `double` primitivo.

Corrige utilizando:

~~~java
double salud = totales.getOrDefault("Salud", 0.0);
~~~

### Reto — Reporte financiero

Crea `ReporteFinanciero.java`.

Construye una lista con al menos seis movimientos, de los cuales:

- dos sean ingresos;
- cuatro sean gastos;
- exista una categoría de gasto repetida.

Obtén y muestra:

- todas las categorías utilizadas, sin repetir;
- únicamente las categorías de gastos, sin repetir;
- importe total de gastos por categoría;
- cantidad de categorías de gastos distintas.

### Comprobación

Antes de continuar, explica por qué el historial debe seguir siendo una lista y no un conjunto, y por qué para acumular gastos por categoría es más conveniente un mapa que varias variables independientes.

---

## Práctica 4.4 — Manejo de excepciones y entradas seguras

### Objetivos

Al terminar esta práctica podrás:

- reconocer una excepción en un programa;
- utilizar `try` y `catch`;
- convertir entradas numéricas con `Integer.parseInt()` y `Double.parseDouble()`;
- atender `NumberFormatException`;
- reconocer `IllegalArgumentException` para rechazar argumentos inválidos;
- comprender cuándo se ejecuta `finally`;
- mantener activo el menú después de una entrada incorrecta.

### 1. Un programa que se interrumpe

Crea `ConversionDirecta.java`:

~~~java
import java.util.Scanner;

public class ConversionDirecta {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Cantidad: ");
        String entrada = scanner.nextLine();

        int cantidad = Integer.parseInt(entrada);

        System.out.println("Cantidad registrada: " + cantidad);

        scanner.close();
    }
}
~~~

Ejecuta con `5` y después con `cinco`.

Al escribir texto que no representa un entero aparece una excepción `NumberFormatException`.

No es un error de compilación: ocurre cuando el programa se ejecuta y encuentra una entrada que no puede convertirse.

### 2. Atender el error con `try` y `catch`

Crea `ConversionSegura.java`:

~~~java
import java.util.Scanner;

public class ConversionSegura {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Cantidad: ");
        String entrada = scanner.nextLine();

        try {
            int cantidad = Integer.parseInt(entrada);
            System.out.println("Cantidad registrada: " + cantidad);
        } catch (NumberFormatException e) {
            System.out.println("Debes escribir un número entero.");
        }

        System.out.println("Programa finalizado.");
        scanner.close();
    }
}
~~~

Prueba `10`, `-3` y `abc`.

Observa que `try/catch` atiende la conversión inválida, pero **no decide** si la cantidad cumple las reglas del negocio. Una cantidad negativa sí puede convertirse a entero; para rechazarla necesitamos una condición.

### 3. Repetir hasta recibir una entrada válida

Crea `LeerEntero.java`:

~~~java
import java.util.Scanner;

public class LeerEntero {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        int cantidad;
        boolean valida = false;

        do {
            System.out.print("Cantidad no negativa: ");
            String entrada = scanner.nextLine();

            try {
                cantidad = Integer.parseInt(entrada);

                if (cantidad >= 0) {
                    System.out.println("Cantidad aceptada: " + cantidad);
                    valida = true;
                } else {
                    System.out.println("No puede ser negativa.");
                }
            } catch (NumberFormatException e) {
                System.out.println("Debes escribir un entero válido.");
            }

        } while (!valida);

        scanner.close();
    }
}
~~~

Prueba esta secuencia:

~~~text
abc
-2
5
~~~

El programa debe volver a pedir el valor hasta recibir `5`.

### Ejercicio — Leer una opción segura

Crea `MenuSeguro.java`.

Muestra:

~~~text
1. Consultar
2. Registrar
0. Salir
~~~

Solicita una opción desde `nextLine()`, conviértela con `Integer.parseInt()` y utiliza `switch`.

Si el usuario escribe `hola`, el programa debe informar que la opción no es numérica y volver al menú. Si escribe `9`, debe informar que la opción no existe. La opción `0` termina la ejecución.

### 4. Valores decimales

Con `Double.parseDouble()` puedes convertir un texto a `double`:

~~~java
String entrada = "25000.50";
double precio = Double.parseDouble(entrada);

System.out.println(precio);
~~~

Agrega este fragmento dentro de `main` de una clase propia.

Usa punto como separador decimal cuando trabajes con `Double.parseDouble()`, por ejemplo `25000.50`.

**Atención a los valores especiales:** `Double.parseDouble()` también puede aceptar textos como `NaN` e `Infinity`. No son importes válidos para nuestra billetera.

Comprueba que el valor sea finito:

~~~java
double valor = Double.parseDouble(entrada);

if (!Double.isFinite(valor) || valor <= 0) {
    System.out.println("Debes escribir un monto positivo y finito.");
}
~~~

Una comprobación como `valor <= 0` no detecta `NaN`. Por eso es importante usar también `Double.isFinite(valor)`.

### Ejercicio — Monto seguro

Crea `MontoSeguro.java`.

Solicita un monto y:

- rechaza texto que no represente un número;
- rechaza cero;
- rechaza números negativos;
- rechaza `NaN` e `Infinity`;
- repite la pregunta hasta recibir un monto positivo y finito.

Prueba:

~~~text
abc
0
-500
NaN
Infinity
30000
~~~

Solamente el último valor debe aceptarse.

### 5. `IllegalArgumentException` y las reglas de una clase

Una clase puede indicar que recibió un argumento inválido.

Crea `ProductoValidado.java`:

~~~java
public class ProductoValidado {

    private final String nombre;
    private final double precio;

    public ProductoValidado(String nombre, double precio) {
        if (nombre == null || nombre.isBlank()) {
            throw new IllegalArgumentException("Nombre obligatorio.");
        }

        if (!Double.isFinite(precio) || precio <= 0) {
            throw new IllegalArgumentException("Precio inválido.");
        }

        this.nombre = nombre;
        this.precio = precio;
    }

    public String getNombre() {
        return nombre;
    }

    public double getPrecio() {
        return precio;
    }
}
~~~

En `PruebaProducto.java`, en la misma carpeta:

~~~java
public class PruebaProducto {

    public static void main(String[] args) {
        try {
            ProductoValidado producto = new ProductoValidado("Teclado", -500);
            System.out.println(producto.getNombre());
        } catch (IllegalArgumentException e) {
            System.out.println("No se pudo crear: " + e.getMessage());
        }
    }
}
~~~

Compila:

~~~bash
javac ProductoValidado.java PruebaProducto.java
java PruebaProducto
~~~

Aquí `throw` comunica que los datos no cumplen las reglas de creación.

En la billetera, para ingresos y gastos seguiremos utilizando métodos que devuelvan `boolean`, como en el Laboratorio 3. No es necesario convertir todos los rechazos normales del negocio en excepciones.

### 6. `finally`

Un bloque `finally` se ejecuta normalmente después de `try` y `catch`, exista o no una excepción.

Crea `EjemploFinally.java`:

~~~java
public class EjemploFinally {

    public static void main(String[] args) {
        try {
            int numero = Integer.parseInt("abc");
            System.out.println(numero);
        } catch (NumberFormatException e) {
            System.out.println("No se pudo convertir.");
        } finally {
            System.out.println("Terminó el intento de conversión.");
        }
    }
}
~~~

Resultado:

~~~text
No se pudo convertir.
Terminó el intento de conversión.
~~~

`finally` puede servir para tareas de cierre y limpieza. No es obligatorio agregarlo a todos los bloques `try`.

### 7. Diferencia entre excepciones

| Situación | Qué ocurre |
|---|---|
| Escribir `abc` para un entero | `NumberFormatException` |
| Acceder a un índice inexistente | `ArrayIndexOutOfBoundsException` o `IndexOutOfBoundsException`, según la colección |
| Crear un producto con precio inválido, cuando la clase lo prohíbe | `IllegalArgumentException` |
| Ingresar un gasto que supera el saldo | Operación rechazada por una regla de negocio; no requiere lanzar una excepción |
| Buscar en un `Map` una clave ausente | `get()` devuelve `null`; no lanza automáticamente una excepción |

No uses un `catch (Exception e)` general para ocultar problemas que deberías resolver en el código. Captura el tipo de error que esperas y sabes cómo manejar.

### Depuración — Excepción incorrecta

Analiza:

~~~java
try {
    int opcion = Integer.parseInt("hola");
    System.out.println(opcion);
} catch (ArithmeticException e) {
    System.out.println("Entrada incorrecta.");
}
~~~

¿Por qué ese `catch` no atiende el error? Cambia el tipo de excepción y vuelve a probar.

### Reto — Mini calculadora resistente a entradas incorrectas

Crea `CalculadoraSegura.java`.

El programa debe:

1. mostrar un menú con suma, resta, multiplicación y división;
2. leer una opción válida sin interrumpirse por texto;
3. solicitar dos números;
4. atender entradas no numéricas;
5. impedir división entre cero;
6. permitir volver al menú hasta seleccionar salir.

Para esta práctica basta con usar `nextLine()` y convertir los valores mediante `parseInt` o `parseDouble`. Mantén separados los mensajes de entrada inválida y los de operación no permitida.

### Comprobación

Antes de continuar asegúrate de distinguir un error de conversión de una operación inválida por reglas del programa. Explica también qué casos resuelve `try/catch` y por qué los valores `NaN` e `Infinity` deben rechazarse al tratar importes monetarios.

---

## Práctica 4.5 — Reto integrador: Mi Billetera Java 2.0

Vas a ampliar la billetera orientada a objetos que construiste en el Laboratorio 3.

La aplicación debe seguir permitiendo registrar ingresos y gastos con sus validaciones, pero ahora **cada operación válida generará un movimiento**. El historial permitirá consultar registros, filtrar por tipo y generar informes por categoría.

No habrá archivos ni base de datos. Los movimientos existirán solamente mientras la aplicación esté en ejecución.

### Objetivo

Construir una billetera orientada a objetos que utilice `ArrayList<Movimiento>` para el historial, `Set` para las categorías únicas, `Map` para los totales agrupados y `try/catch` para las entradas incorrectas.

### 1. Archivos del proyecto

En `MiBilleteraJava` crea o actualiza:

~~~text
MiBilleteraJava/
├── Main.java
├── Billetera.java
├── Movimiento.java
└── README.md
~~~

Puedes añadir `PruebaBilletera.java` para comprobar operaciones sin utilizar el menú.

No es necesario utilizar paquetes ni un gestor de dependencias.

### 2. Responsabilidad de las clases

| Archivo | Responsabilidad |
|---|---|
| `Movimiento.java` | Representar un registro de ingreso o gasto |
| `Billetera.java` | Mantener saldo, historial y reglas de las operaciones |
| `Main.java` | Leer datos, mostrar menú y presentar resultados |
| `README.md` | Explicar la aplicación y cómo comprobarla |

`Movimiento` y `Billetera` no deben utilizar `Scanner` ni imprimir el menú. Las decisiones sobre el saldo y el historial pertenecen a `Billetera`.

### 3. Clase `Movimiento`

Cada movimiento debe conservar:

- tipo: `"INGRESO"` o `"GASTO"`;
- categoría: por ejemplo `"Salario"`, `"Transporte"` o `"Alimentación"`;
- valor: monto de la operación.

Los atributos serán `private final` y se inicializarán mediante el constructor.

Incluye métodos públicos para consultar los tres datos.

Puedes partir de este esqueleto:

~~~java
public class Movimiento {

    private final String tipo;
    private final String categoria;
    private final double valor;

    public Movimiento(String tipo, String categoria, double valor) {
        // Valida los datos antes de guardarlos.
        this.tipo = tipo;
        this.categoria = categoria;
        this.valor = valor;
    }

    public String getTipo() {
        return tipo;
    }

    public String getCategoria() {
        return categoria;
    }

    public double getValor() {
        return valor;
    }
}
~~~

**Reglas de creación:**

- el tipo debe ser `INGRESO` o `GASTO`;
- la categoría no puede estar vacía ni contener solamente espacios;
- el valor debe ser positivo y finito;
- un movimiento creado no debe cambiar sus datos.

Puedes utilizar `IllegalArgumentException` para rechazar la construcción con valores incorrectos.

### 4. Ampliar `Billetera`

Conserva los datos y operaciones del Laboratorio 3:

- titular;
- saldo;
- cantidad y total de ingresos;
- cantidad y total de gastos;
- registro y validación de ingresos;
- registro y validación de gastos.

Agrega:

~~~java
private List<Movimiento> movimientos;
~~~

Recuerda importar:

~~~java
import java.util.ArrayList;
import java.util.List;
~~~

Dentro del constructor de `Billetera` inicializa:

~~~java
this.movimientos = new ArrayList<>();
~~~

Esta lista pertenece a **cada objeto** `Billetera`, no a la clase de manera compartida.

### 5. Registrar operaciones con categorías

En el Laboratorio 3, los métodos recibían únicamente el monto. Ahora deben incorporar la categoría:

~~~java
public boolean registrarIngreso(double valor, String categoria)
~~~

~~~java
public boolean registrarGasto(double valor, String categoria)
~~~

También puedes conservar los métodos anteriores si lo deseas, pero el menú del nuevo laboratorio debe utilizar las versiones con categoría.

**Reglas para ingresos:**

- rechazar valores no positivos o no finitos;
- rechazar categorías vacías o compuestas solo por espacios;
- sumar el valor al saldo si es válido;
- actualizar contador y acumulado;
- agregar un movimiento de tipo `INGRESO`;
- no modificar ningún dato cuando la operación se rechace.

**Reglas para gastos:**

- rechazar valores no positivos o no finitos;
- rechazar categoría vacía;
- rechazar valores superiores al saldo;
- descontar el valor del saldo cuando sea válido;
- actualizar contador y acumulado;
- agregar un movimiento de tipo `GASTO`;
- no modificar ningún dato cuando se rechace.

**Importante:** las validaciones deben ocurrir antes de modificar el saldo o agregar el movimiento. Una operación aceptada debe mantener los datos coherentes.

### 6. Historial de movimientos

La billetera debe permitir consultar los movimientos en el orden en que fueron registrados.

Ofrece un método como:

~~~java
public List<Movimiento> getMovimientos() {
    return new ArrayList<>(movimientos);
}
~~~

Como los movimientos no tienen setters y sus datos son finales, esta copia superficial permite consultar los registros sin ofrecer un método directo para cambiar la lista interna.

**No** utilices una lista pública que pueda ser modificada desde `Main`.

### 7. Consultar movimientos por tipo

Incluye un método:

~~~java
public List<Movimiento> buscarPorTipo(String tipo)
~~~

Debe devolver una lista con los movimientos que coincidan con `INGRESO` o `GASTO`.

Puedes apoyarte en este fragmento dentro de `Billetera`, donde ya existe `movimientos`:

~~~java
List<Movimiento> encontrados = new ArrayList<>();

for (Movimiento movimiento : movimientos) {
    if (movimiento.getTipo().equals(tipo)) {
        encontrados.add(movimiento);
    }
}
~~~

Completa el método para devolver el resultado y decidir qué hacer cuando el tipo no sea una opción reconocida.

### 8. Categorías utilizadas

Crea un método que devuelva un `Set<String>` con las categorías de los movimientos, sin duplicados.

Puedes utilizar:

~~~java
Set<String> categorias = new HashSet<>();

for (Movimiento movimiento : movimientos) {
    categorias.add(movimiento.getCategoria());
}
~~~

Importa `Set` y `HashSet`.

Decide si quieres mostrar todas las categorías o exclusivamente las utilizadas en gastos. En la actividad principal, **la opción 6 del menú debe mostrar todas las categorías utilizadas**, tanto en ingresos como en gastos, sin duplicados.

### 9. Reporte de gastos agrupados

Crea en `Billetera` un método que devuelva:

~~~java
public Map<String, Double> gastosPorCategoria()
~~~

Debe incluir únicamente los movimientos de tipo `GASTO`.

Para construir el resultado utiliza un `HashMap` y `getOrDefault()`.

Ejemplo esperado:

~~~text
Transporte: $50000.0
Alimentación: $45000.0
Salud: $40000.0
~~~

Si no hay gastos, devuelve un mapa vacío.

No agregues ingresos al reporte de gastos.

### 10. Menú de la aplicación

`Main.java` debe mostrar repetidamente:

~~~text
================================
       MI BILLETERA JAVA
================================
Titular: Laura
Saldo disponible: $150000.0

1. Registrar ingreso
2. Registrar gasto
3. Consultar saldo
4. Mostrar movimientos
5. Consultar movimientos por tipo
6. Consultar categorías utilizadas
7. Resumen financiero
0. Salir
~~~

La opción `0` termina el programa. Una opción desconocida debe mostrar un mensaje y permitir volver al menú.

### 11. Funcionamiento de las opciones

**Opción 1 — Ingreso.** Solicita categoría y monto. Invoca el método de `Billetera` e informa si la operación fue aceptada.

**Opción 2 — Gasto.** Solicita categoría y monto. Rechaza operaciones inválidas y saldo insuficiente sin alterar el historial.

**Opción 3 — Saldo.** Muestra el saldo actual.

**Opción 4 — Historial.** Lista todos los movimientos numerados según su orden de registro. Si no hay movimientos, informa que está vacío.

**Opción 5 — Filtrar por tipo.** Solicita `INGRESO` o `GASTO` y muestra únicamente los movimientos correspondientes.

**Opción 6 — Categorías utilizadas.** Muestra un conjunto de categorías diferentes, sin depender de un orden específico.

**Opción 7 — Resumen financiero.** Presenta titular, saldo, totales y cantidades de ingresos y gastos y, además, los gastos agrupados por categoría.

### 12. Leer opciones y montos sin interrumpir el menú

Evita mezclar `nextInt()` o `nextDouble()` con `nextLine()` para este ejercicio.

Lee las entradas como texto y convierte cuando sea necesario. Un patrón posible para el menú:

~~~java
System.out.print("Opción: ");
String entrada = scanner.nextLine();

try {
    int opcion = Integer.parseInt(entrada.trim());
    // Utiliza la opción en el switch del menú.
} catch (NumberFormatException e) {
    System.out.println("Debes ingresar una opción numérica.");
}
~~~

Para un monto:

~~~java
System.out.print("Valor: ");
String entrada = scanner.nextLine();

try {
    double valor = Double.parseDouble(entrada.trim());

    if (!Double.isFinite(valor) || valor <= 0) {
        System.out.println("El monto debe ser positivo y finito.");
    } else {
        // Utiliza valor para solicitar una operación a Billetera.
    }
} catch (NumberFormatException e) {
    System.out.println("Debes escribir un monto numérico.");
}
~~~

El código mostrado es un **fragmento**. Debes integrarlo dentro del ciclo de tu menú, utilizando el mismo `Scanner`.

La validación del monto también debe existir en `Billetera`. No confíes solamente en la lectura correcta desde `Main`: otro programa podría llamar al objeto directamente.

### 13. Construcción por etapas

**Etapa 1 — Historial vacío.** Crea la clase `Movimiento` y la lista de `Billetera`. Comprueba que una billetera nueva tenga saldo cero e historial vacío.

**Etapa 2 — Registrar ingresos.** Modifica el método de ingreso para que incorpore la categoría y agregue un movimiento cuando la operación sea válida.

**Etapa 3 — Registrar gastos.** Integra la misma lógica, respetando la comprobación de saldo suficiente.

**Etapa 4 — Mostrar historial.** Implementa la copia de la lista y la opción correspondiente del menú.

**Etapa 5 — Consultas.** Implementa la búsqueda por tipo y la lista de categorías únicas.

**Etapa 6 — Reporte agrupado.** Implementa el mapa de gastos por categoría y compara sus sumas con el total de gastos registrado por la billetera.

**Etapa 7 — Entradas seguras.** Integra `try/catch` para valores numéricos incorrectos y comprueba que el programa continúa en ejecución.

**Etapa 8 — Revisión.** Verifica que toda operación rechazada deje sin cambios saldo, contadores, acumulados e historial.

### 14. Pistas para organizar el código

Dentro de `Billetera`, después de comprobar que el ingreso es válido, puedes construir el movimiento:

~~~java
Movimiento movimiento = new Movimiento("INGRESO", categoria, valor);
~~~

Después agrega el registro al historial cuando se realiza la operación.

Para gastos:

~~~java
Movimiento movimiento = new Movimiento("GASTO", categoria, valor);
~~~

No crees el movimiento con un tipo arbitrario. Mantén solamente las dos opciones definidas.

Para agrupar:

~~~java
double acumulado = totales.getOrDefault(categoria, 0.0);
totales.put(categoria, acumulado + valor);
~~~

Para mostrar el historial numerado, recorre una lista `List<Movimiento>` con un `for` y utiliza `get(i)`.

### 15. Ejemplo de ejecución

~~~text
Titular: Laura

Opción: 1
Categoría: Salario
Valor: 1500000
Ingreso registrado.

Opción: 2
Categoría: Transporte
Valor: 30000
Gasto registrado.

Opción: 2
Categoría: Alimentación
Valor: 45000
Gasto registrado.

Opción: 1
Categoría: Otros
Valor: 100000
Ingreso registrado.

Opción: 4
================================
           HISTORIAL
================================
1. INGRESO | Salario | $1500000.0
2. GASTO | Transporte | $30000.0
3. GASTO | Alimentación | $45000.0
4. INGRESO | Otros | $100000.0

Opción: 7
================================
       RESUMEN FINANCIERO
================================
Titular: Laura
Saldo: $1525000.0
Ingresos realizados: 2
Total ingresado: $1600000.0
Gastos realizados: 2
Total gastado: $75000.0

Gastos por categoría:
Transporte: $30000.0
Alimentación: $45000.0
~~~

El orden de los movimientos debe coincidir con su registro. En cambio, no es obligatorio que las categorías de un `HashSet` o las claves de un `HashMap` aparezcan en un orden concreto.

### 16. Pruebas obligatorias

| Prueba | Resultado esperado |
|---|---|
| Crear billetera | Saldo cero e historial vacío |
| Consultar historial vacío | Mensaje informativo, sin error |
| Registrar ingreso de 200000 | Saldo 200000 y un movimiento |
| Registrar gasto de 50000 | Saldo 150000 y dos movimientos |
| Intentar gastar 300000 | Rechazo; no agrega movimiento |
| Intentar ingreso cero | Rechazo; no modifica los totales |
| Intentar gasto negativo | Rechazo |
| Categoría vacía | Rechazo; no agrega movimiento |
| Ingresar texto como monto | Mensaje, menú continúa |
| Ingresar `NaN` o `Infinity` | Rechazo, sin modificar estado |
| Filtrar tipo INGRESO | Solamente ingresos |
| Filtrar tipo GASTO | Solamente gastos |
| Consultar categorías | Cada categoría aparece una sola vez |
| Agrupar gastos | Suma correcta por categoría |
| Elegir opción desconocida | Mensaje y nuevo menú |
| Elegir 0 | Finaliza el programa |

### 17. Prueba numérica de reporte

Registra estos movimientos en este orden:

| Tipo | Categoría | Valor |
|---|---|---:|
| INGRESO | Salario | 1500000 |
| GASTO | Transporte | 30000 |
| GASTO | Alimentación | 45000 |
| GASTO | Transporte | 20000 |
| INGRESO | Otros | 100000 |
| GASTO | Salud | 40000 |

Calcula manualmente los resultados antes de ejecutar.

Tu programa debe mostrar:

~~~text
Cantidad de movimientos: 6
Ingresos realizados: 2
Gastos realizados: 4
Total ingresado: 1600000.0
Total gastado: 135000.0
Saldo: 1465000.0
Categorías utilizadas: 5
~~~

Y el reporte de gastos:

~~~text
Transporte: 50000.0
Alimentación: 45000.0
Salud: 40000.0
~~~

En esta prueba hay cinco categorías diferentes si incluyes `Salario` y `Otros`.

Comprueba también que:

~~~text
Total ingresado - Total gastado = Saldo
~~~

y que la suma de los valores del mapa de gastos coincide con `Total gastado`.

### 18. Probar la independencia de dos billeteras

Crea, como archivo de comprobación adicional, `PruebaBilleteras.java`.

Desde allí:

1. construye dos objetos `Billetera` con titulares diferentes;
2. registra un ingreso y un gasto en el primero;
3. registra otro ingreso en el segundo;
4. muestra los saldos y cantidades de movimientos de ambos.

El historial del segundo no debe incluir ninguna operación realizada sobre el primero.

### 19. Revisar una copia del historial

Si `getMovimientos()` devuelve una copia de la lista, realiza esta prueba:

~~~java
List<Movimiento> copia = billetera.getMovimientos();

copia.clear();

System.out.println("Copia: " + copia.size());
System.out.println("Original: " + billetera.getMovimientos().size());
~~~

Este fragmento debe usarse dentro de una prueba que ya tenga un objeto `billetera` con movimientos registrados.

La copia quedará vacía, pero el historial interno de la billetera debe conservar todos los movimientos.

### 20. Reto de ampliación

Después de superar las pruebas principales, puedes agregar:

- la cantidad de gastos registrada por cada categoría;
- la categoría con mayor valor acumulado de gastos;
- una búsqueda de movimientos por categoría;
- un método que devuelva el mayor gasto registrado;
- un reporte ordenado alfabéticamente por categoría;
- una comprobación que compare los totales históricos con los acumulados de la billetera.

No necesitas aún `Stream API`, herencia, interfaces propias, archivos CSV o JSON ni bases de datos.

### 21. Depuración

Trabaja con una copia del proyecto. Provoca y corrige estos errores, uno a la vez:

1. agregar el movimiento antes de validar el monto;
2. aceptar un gasto que supera el saldo;
3. devolver directamente la lista interna y agregar un movimiento desde `Main`;
4. crear la lista de movimientos como atributo `static` compartido entre billeteras;
5. agrupar gastos con `put(categoria, valor)` sin sumar el acumulado anterior;
6. calcular las categorías sin tener en cuenta que se repiten;
7. convertir una entrada con `Double.parseDouble()` sin atender texto no numérico.

Explica con un comentario breve qué provocó cada problema.

### 22. Revisión del proyecto

Antes de darlo por terminado comprueba:

- `Movimiento` tiene datos privados y finales, y no ofrece setters;
- las operaciones válidas crean exactamente un movimiento;
- las operaciones rechazadas no modifican saldo, acumulados, contadores ni historial;
- `Billetera` conserva una lista propia por instancia;
- el historial respeta el orden de registro;
- las categorías no aparecen duplicadas al utilizar `Set`;
- el reporte de `Map` agrupa exclusivamente gastos;
- la suma de los gastos agrupados coincide con el total gastado;
- `Main` controla el menú y las entradas;
- las clases de negocio no dependen de `Scanner`;
- las entradas numéricas inválidas no interrumpen el menú;
- el programa compila correctamente desde la terminal;
- puedes explicar por qué utilizaste `ArrayList`, `HashSet` y `HashMap`.

### Evidencia de trabajo

Conserva el proyecto y los archivos de las prácticas en tu repositorio de trabajo.

En `MiBilleteraJava/README.md` incluye:

- descripción breve de la aplicación;
- propósito de cada clase;
- comandos para compilar y ejecutar;
- explicación de las colecciones utilizadas;
- casos de prueba realizados y resultados observados.

No es necesario elaborar un informe PDF ni una presentación.

---

## Cierre del laboratorio

Al finalizar deberías poder crear y recorrer arreglos, trabajar con `ArrayList` de objetos, obtener valores únicos mediante `Set`, agrupar importes con `Map` y manejar entradas incorrectas mediante excepciones.

Compara la billetera del Laboratorio 3 con esta versión. Antes el objeto conservaba saldo y acumulados; ahora también dispone de un historial que permite analizar qué operaciones se realizaron.

En los siguientes laboratorios podrás ampliar esta aplicación con herencia, polimorfismo, interfaces, `Stream API` y persistencia. Ninguno de esos conceptos es obligatorio para completar el reto actual.

### Referencias

- [Java 21 — Collections Framework](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/doc-files/coll-overview.html)
- [Java 21 — List](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html)
- [Java 21 — Set](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Set.html)
- [Java 21 — Map](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Map.html)
- [Java 21 — NumberFormatException](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/NumberFormatException.html)
