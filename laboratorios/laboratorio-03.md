# Laboratorio 3 — Introducción a la programación orientada a objetos en Java

En el [Laboratorio 2](laboratorio-02.md) construiste una billetera de consola con variables, condiciones, ciclos y métodos estáticos. El programa funciona, pero el saldo y los demás datos se administran desde `main` y deben pasar de un método a otro.

En este laboratorio vas a representar esos datos y operaciones mediante objetos. Comenzarás con clases pequeñas y terminarás con una nueva versión de la billetera, organizada en dos archivos: uno para el comportamiento de la billetera y otro para la interacción con el usuario.

| Práctica | Tema |
|---|---|
| 3.1 | Clases, objetos y métodos de instancia |
| 3.2 | Atributos, constructores y `this` |
| 3.3 | Encapsulamiento y reglas de negocio |
| 3.4 | Colaboración entre clases y separación de responsabilidades |
| 3.5 | Reto integrador: Mi Billetera Java con POO |

Trabaja cada ejemplo en su propia carpeta para no mezclar clases con el mismo nombre. Ejecuta el código antes de modificarlo, predice los resultados de los ejercicios y comprueba el comportamiento con datos diferentes. Si aparece un error, lee primero el mensaje que muestra Java.

## Preparación del trabajo

Conserva la versión de `BilleteraConsola.java` del laboratorio anterior. No la reemplaces: servirá para comparar la solución procedural con la orientada a objetos.

Crea una carpeta `java-laboratorio-03` y, dentro de ella, una subcarpeta por práctica:

~~~text
java-laboratorio-03/
├── practica-3-1/
├── practica-3-2/
├── practica-3-3/
├── practica-3-4/
└── MiBilleteraJava/
~~~

Utilizaremos **Java 21**, Visual Studio Code y la terminal. No necesitas instalar bibliotecas, configurar Maven ni crear paquetes.

Para ejecutar un ejemplo compuesto por dos archivos, abre su carpeta desde VS Code y utiliza **Run** sobre el método `main`. También puedes compilar ambos archivos desde la terminal situada en esa carpeta:

~~~bash
javac Producto.java Main.java
java Main
~~~

El primer comando debe adaptarse a los nombres de los archivos de cada ejercicio. Para ejecutar con `java` se escribe el nombre de la clase principal, sin la extensión `.java`.

Si VS Code no tiene disponible el botón **Run**, utiliza la terminal. Si el equipo no permite ejecutar Java localmente, informa al instructor para utilizar el entorno alternativo autorizado. No instales ni actualices software en los equipos del ambiente por tu cuenta.

---

## Práctica 3.1 — Clases, objetos y métodos de instancia

### Objetivos

Al finalizar esta práctica podrás:

- reconocer una clase como definición de datos y comportamientos;
- crear objetos mediante `new`;
- acceder a sus atributos y métodos;
- diferenciar dos objetos creados desde la misma clase;
- distinguir un método estático de uno de instancia.

### 1. Del dato separado al objeto

En los programas anteriores representábamos un producto mediante variables independientes:

~~~java
String nombre = "Teclado";
double precio = 85000;
int cantidad = 3;
~~~

Imagina que necesitamos dos productos. Habría que crear nuevas variables para cada uno y recordar cuáles pertenecen al mismo producto.

Una clase nos permite definir cómo es un producto y qué operaciones puede realizar.

Crea `Producto.java` dentro de `practica-3-1`:

~~~java
public class Producto {

    String nombre;
    double precio;
    int cantidad;

    void mostrarDetalle() {
        System.out.println("Producto: " + nombre);
        System.out.println("Precio: $" + precio);
        System.out.println("Cantidad: " + cantidad);
    }
}
~~~

Esta clase todavía no tiene un método `main`. No es un programa completo por sí sola. Es la definición que utilizaremos para crear objetos.

Observa sus partes:

- `nombre`, `precio` y `cantidad` son **atributos**;
- `mostrarDetalle()` es un **método de instancia**;
- la clase define qué información y comportamiento tendrá cada producto.

Por ahora usamos atributos accesibles para observar su funcionamiento. Más adelante los protegeremos con `private`.

### 2. Crear el primer objeto

En la misma carpeta crea `Main.java`:

~~~java
public class Main {

    public static void main(String[] args) {
        Producto producto = new Producto();

        producto.nombre = "Teclado";
        producto.precio = 85000;
        producto.cantidad = 3;

        producto.mostrarDetalle();
    }
}
~~~

Ejecuta `Main.java`.

Resultado esperado:

~~~text
Producto: Teclado
Precio: $85000.0
Cantidad: 3
~~~

En:

~~~java
Producto producto = new Producto();
~~~

el primer `Producto` indica el tipo, `producto` es el nombre de la variable y `new Producto()` crea un objeto.

El punto permite utilizar sus miembros:

~~~java
producto.mostrarDetalle();
~~~

### 3. Dos objetos, una clase

Modifica `Main.java` para crear otro producto:

~~~java
Producto segundo = new Producto();

segundo.nombre = "Mouse";
segundo.precio = 45000;
segundo.cantidad = 7;

producto.mostrarDetalle();
segundo.mostrarDetalle();
~~~

Agrega estas líneas **dentro de `main`**, después de crear y configurar el primer producto.

#### Antes de ejecutar

Responde:

1. ¿Cuántas clases definimos?
2. ¿Cuántos objetos creamos?
3. Si cambiamos `segundo.cantidad`, ¿cambiará también `producto.cantidad`?

Ejecuta y comprueba la tercera respuesta.

### 4. Métodos que utilizan los atributos

Agrega a `Producto.java`:

~~~java
double calcularSubtotal() {
    return precio * cantidad;
}
~~~

Desde `Main.java` utiliza:

~~~java
System.out.println("Subtotal primero: $" + producto.calcularSubtotal());
System.out.println("Subtotal segundo: $" + segundo.calcularSubtotal());
~~~

El mismo método produce resultados diferentes porque trabaja con los atributos del objeto sobre el que se invoca.

### Ejercicio — Una clase para representar un libro

En una carpeta diferente, crea `Libro.java` con:

- `titulo`;
- `autor`;
- `paginas`.

Incluye un método `mostrarFicha()` que imprima los tres datos.

Después crea un `Main.java` que construya dos libros, asigne sus datos y muestre ambas fichas.

No copies los nombres de los atributos del ejemplo de productos. Decide qué tipo de dato corresponde a cada atributo del libro.

### 5. ¿Qué diferencia hay entre `static` y un método de instancia?

En el Laboratorio 2 utilizamos métodos como:

~~~java
static double calcularTotal(double precio, int cantidad) {
    return precio * cantidad;
}
~~~

Se podían llamar desde `main` sin crear un objeto.

En `Producto` usamos:

~~~java
double calcularSubtotal() {
    return precio * cantidad;
}
~~~

Aquí el cálculo pertenece a cada objeto. Por eso lo invocamos de este modo:

~~~java
producto.calcularSubtotal();
~~~

Todavía necesitamos `public static void main(String[] args)` como punto de entrada; eso no significa que todos los demás métodos deban ser estáticos.

### Depuración — Objeto que no se creó

Prueba este programa con la clase `Producto` anterior:

~~~java
public class Main {

    public static void main(String[] args) {
        Producto producto = null;
        producto.mostrarDetalle();
    }
}
~~~

Ejecuta y observa el error. ¿Qué ocurre cuando intentas llamar un método sobre una referencia `null`?

Después reemplaza `null` por una instancia creada con `new Producto()`.

### Reto — Catálogo de tres productos

Amplía `Main.java` para:

- crear tres objetos `Producto`;
- asignar nombres, precios y cantidades diferentes;
- mostrar una ficha por objeto;
- calcular el subtotal de cada uno;
- mostrar el valor total de los tres subtotales.

No utilices arreglos ni colecciones. En esta práctica nos interesa observar los objetos por separado.

### Comprobación

Antes de continuar, asegúrate de poder explicar qué es una clase, qué es un objeto, qué hace `new` y por qué los atributos de dos objetos `Producto` no tienen por qué guardar los mismos valores.

---

## Práctica 3.2 — Atributos, constructores y `this`

### Objetivos

Al finalizar esta práctica podrás:

- inicializar atributos desde un constructor;
- reconocer la diferencia entre constructor y método común;
- utilizar `this` para identificar atributos del objeto;
- crear objetos con valores iniciales diferentes;
- comprender qué sucede cuando no existe un constructor sin argumentos.

### 1. Un problema del ejemplo anterior

En `Producto` podíamos crear un objeto y dejarlo sin datos:

~~~java
Producto producto = new Producto();
producto.mostrarDetalle();
~~~

Eso permite mostrar un producto sin nombre, con precio cero y cantidad cero.

Para muchos programas es más útil indicar algunos datos desde el momento en que se crea el objeto.

### 2. Construir el objeto con datos iniciales

Crea en `practica-3-2` un nuevo `Producto.java`:

~~~java
public class Producto {

    String nombre;
    double precio;
    int cantidad;

    public Producto(String nombre, double precio, int cantidad) {
        this.nombre = nombre;
        this.precio = precio;
        this.cantidad = cantidad;
    }

    void mostrarDetalle() {
        System.out.println("Producto: " + nombre);
        System.out.println("Precio: $" + precio);
        System.out.println("Cantidad: " + cantidad);
    }

    double calcularSubtotal() {
        return precio * cantidad;
    }
}
~~~

El constructor:

~~~java
public Producto(String nombre, double precio, int cantidad)
~~~

tiene el mismo nombre de la clase y no declara tipo de retorno.

Se ejecuta al crear un objeto con `new`.

### 3. Crear objetos con el constructor

Crea `Main.java` en esa carpeta:

~~~java
public class Main {

    public static void main(String[] args) {
        Producto primero = new Producto("Teclado", 85000, 3);
        Producto segundo = new Producto("Mouse", 45000, 7);

        primero.mostrarDetalle();
        System.out.println("Subtotal: $" + primero.calcularSubtotal());

        segundo.mostrarDetalle();
        System.out.println("Subtotal: $" + segundo.calcularSubtotal());
    }
}
~~~

Ejecuta y verifica los subtotales.

### 4. ¿Para qué sirve `this`?

Examina:

~~~java
public Producto(String nombre, double precio, int cantidad) {
    this.nombre = nombre;
    this.precio = precio;
    this.cantidad = cantidad;
}
~~~

El nombre a la derecha es el parámetro recibido. El nombre a la izquierda, acompañado de `this.`, es el atributo que pertenece al objeto que se está creando.

Prueba eliminar `this.` solo de esta línea:

~~~java
this.nombre = nombre;
~~~

para dejarla así:

~~~java
nombre = nombre;
~~~

El código puede compilar, pero el atributo no recibe el valor esperado. Ejecuta y observa la ficha. Luego restablece `this.nombre = nombre;`.

### 5. No todo se recibe como parámetro

Un constructor también puede establecer valores iniciales.

Crea `Contador.java`:

~~~java
public class Contador {

    int valor;

    public Contador() {
        this.valor = 0;
    }

    void aumentar() {
        valor++;
    }

    void mostrar() {
        System.out.println("Valor: " + valor);
    }
}
~~~

Crea un `Main.java` y realiza:

~~~java
Contador visitas = new Contador();

visitas.aumentar();
visitas.aumentar();
visitas.mostrar();
~~~

Resultado esperado:

~~~text
Valor: 2
~~~

Ahora crea otro objeto `Contador` y comprueba que empieza en cero.

### Ejercicio — Termómetro

Crea la clase `Termometro` con:

- atributo `temperaturaCelsius` de tipo `double`;
- constructor que reciba una temperatura inicial;
- método `mostrarCelsius()`;
- método `convertirAFahrenheit()` que devuelva la conversión.

Fórmula:

~~~text
F = (C × 9 / 5) + 32
~~~

Desde `Main.java` crea objetos con 0 °C y 25 °C.

Resultados esperados:

~~~text
0 °C -> 32.0 °F
25 °C -> 77.0 °F
~~~

### Depuración — Constructor que no coincide

Con la clase `Producto` que recibe tres parámetros, intenta crear:

~~~java
Producto producto = new Producto();
~~~

Compila y lee el mensaje.

¿Por qué ese código ya no funciona? Corrige la llamada pasando los datos requeridos.

No agregues un constructor vacío solo para ocultar el problema.

### Reto — Registro de estudiantes

Crea `Estudiante.java` con:

- nombre;
- ficha;
- trimestre;
- promedio.

Utiliza un constructor para recibir los cuatro valores y un método de instancia `mostrarPerfil()`.

Desde `Main.java` crea dos estudiantes y muestra sus perfiles. Agrega un método `aprobo()` que devuelva un `boolean` al comparar el promedio con 3.0.

Comprueba un estudiante con promedio 2.8 y otro con 4.1.

### Comprobación

Antes de continuar, explica qué responsabilidad tiene un constructor, por qué `this.nombre` no significa lo mismo que un parámetro `nombre` y cómo pueden crearse dos objetos de la misma clase con estados iniciales diferentes.

---

## Práctica 3.3 — Encapsulamiento y reglas de negocio

### Objetivos

Al finalizar esta práctica podrás:

- proteger atributos con `private`;
- ofrecer operaciones mediante métodos `public`;
- consultar información con métodos de acceso;
- rechazar cambios que incumplan una regla;
- reconocer por qué un atributo privado no debe modificarse desde `main`.

### 1. ¿Qué puede salir mal si el saldo es público?

Observa:

~~~java
Billetera billetera = new Billetera("Laura");

billetera.saldo = -500000;
~~~

Si el atributo fuera accesible, otra clase podría establecer un saldo negativo sin realizar ninguna validación.

Queremos que el objeto controle sus propias reglas: un ingreso debe ser positivo y un gasto no puede superar el saldo.

### 2. Primera versión encapsulada

En `practica-3-3` crea `Billetera.java`:

~~~java
public class Billetera {

    private String titular;
    private double saldo;

    public Billetera(String titular) {
        this.titular = titular;
        this.saldo = 0;
    }

    public String getTitular() {
        return titular;
    }

    public double getSaldo() {
        return saldo;
    }

    public boolean registrarIngreso(double valor) {
        if (valor <= 0) {
            return false;
        }

        saldo += valor;
        return true;
    }
}
~~~

Observa las responsabilidades:

- `private` impide acceder directamente al atributo desde otra clase;
- `public` permite utilizar un constructor o método desde otras clases;
- `getSaldo()` permite consultar el dato sin modificarlo;
- `registrarIngreso()` decide si la operación se realiza.

El método devuelve `true` si la operación fue aceptada y `false` si se rechazó.

### 3. Probar la clase desde `Main`

Crea `Main.java`:

~~~java
public class Main {

    public static void main(String[] args) {
        Billetera billetera = new Billetera("Laura");

        boolean primera = billetera.registrarIngreso(100000);
        boolean segunda = billetera.registrarIngreso(-5000);

        System.out.println("Primer ingreso: " + primera);
        System.out.println("Segundo ingreso: " + segunda);
        System.out.println("Saldo: $" + billetera.getSaldo());
    }
}
~~~

Resultado esperado:

~~~text
Primer ingreso: true
Segundo ingreso: false
Saldo: $100000.0
~~~

### 4. Agregar la operación de gasto

Completa `Billetera.java` con un método público:

~~~java
public boolean registrarGasto(double valor) {
    if (valor <= 0 || valor > saldo) {
        return false;
    }

    saldo -= valor;
    return true;
}
~~~

Prueba:

- ingresar $100.000;
- gastar $30.000;
- intentar gastar $90.000;
- consultar el saldo.

Saldo esperado: **$70.000**.

¿Por qué se rechazó el segundo gasto?

### 5. ¿Debemos crear un `setSaldo()`?

Podríamos escribir un método que permitiera asignar cualquier saldo:

~~~java
public void setSaldo(double saldo) {
    this.saldo = saldo;
}
~~~

Pero eso abriría otra ruta para modificar el saldo sin registrar un ingreso o gasto.

**No agregues ese método a la billetera.** Para esta aplicación el saldo solamente debe cambiar por operaciones válidas.

No todos los atributos necesitan métodos `get` y `set`. La decisión depende de las reglas que debe cumplir el objeto.

### Ejercicio — Cuenta de puntos

En una carpeta separada crea `CuentaPuntos.java`.

Requisitos:

- `titular` y `puntos` privados;
- constructor que reciba el titular e inicie puntos en cero;
- `consultarPuntos()`;
- `sumarPuntos(int cantidad)`, que rechace cantidades no positivas;
- `canjearPuntos(int cantidad)`, que rechace cantidades no positivas o superiores a los puntos disponibles.

Prueba:

~~~text
Puntos iniciales: 0
Sumar: 80        -> aceptado
Canjear: 30      -> aceptado
Canjear: 100     -> rechazado
Puntos finales: 50
~~~

### 6. Cada objeto conserva su propio estado

Utiliza la clase `Billetera` que acabas de construir:

~~~java
Billetera laura = new Billetera("Laura");
Billetera diego = new Billetera("Diego");

laura.registrarIngreso(150000);
diego.registrarIngreso(70000);

laura.registrarGasto(25000);

System.out.println(laura.getSaldo());
System.out.println(diego.getSaldo());
~~~

Antes de ejecutar, escribe ambos saldos.

Resultados esperados:

~~~text
125000.0
70000.0
~~~

Son dos objetos de la misma clase, pero las operaciones realizadas sobre uno no cambian el saldo del otro.

### Depuración — Acceso a un atributo privado

Desde `Main.java` intenta:

~~~java
billetera.saldo = 500000;
~~~

Compila y observa el error.

El atributo existe, pero no es accesible desde esa clase. Elimina esa instrucción y usa `registrarIngreso(500000)` si lo que buscas es efectuar un ingreso.

### Reto — Contadores de operaciones

Amplía `Billetera` con:

~~~java
private int cantidadIngresos;
private int cantidadGastos;
~~~

Estos contadores comienzan en cero.

Modifica los métodos de ingreso y gasto para que se incrementen **solo cuando la operación sea válida**.

Agrega métodos públicos de consulta para ambos contadores.

Prueba:

~~~text
Ingresar 100000   -> aceptado
Ingresar -2000    -> rechazado
Gastar 30000      -> aceptado
Gastar 90000      -> rechazado

Cantidad de ingresos: 1
Cantidad de gastos: 1
Saldo: 70000
~~~

### Comprobación

Antes de continuar asegúrate de poder explicar por qué `saldo` es privado, por qué `getSaldo()` devuelve un dato sin modificarlo y por qué es preferible `registrarGasto()` a permitir que otra clase reste dinero directamente.

---

## Práctica 3.4 — Colaboración entre clases y separación de responsabilidades

### Objetivos

Al finalizar esta práctica podrás:

- trabajar con dos clases en archivos distintos;
- crear un objeto desde una clase que contiene `main`;
- dejar los cálculos dentro del objeto que conoce los datos;
- presentar resultados desde la clase que interactúa con el usuario;
- compilar un programa de varios archivos sin Maven.

### 1. Dos tareas distintas

En una aplicación de consola aparecen dos necesidades:

**Administrar datos y reglas.** Por ejemplo, aumentar o disminuir los puntos de una cuenta.

**Interactuar con el usuario.** Por ejemplo, pedir el valor de una operación y mostrar un mensaje.

Podemos separar esas necesidades. La clase que representa la cuenta decide si una operación es válida; la clase principal lee las entradas y muestra los resultados.

### 2. Ejemplo con dos clases

En `practica-3-4` crea `CuentaPuntos.java`:

~~~java
public class CuentaPuntos {

    private String titular;
    private int puntos;

    public CuentaPuntos(String titular) {
        this.titular = titular;
        this.puntos = 0;
    }

    public boolean sumarPuntos(int cantidad) {
        if (cantidad <= 0) {
            return false;
        }

        puntos += cantidad;
        return true;
    }

    public int getPuntos() {
        return puntos;
    }

    public String getTitular() {
        return titular;
    }
}
~~~

Luego crea `Main.java`:

~~~java
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Titular: ");
        String nombre = scanner.nextLine();

        CuentaPuntos cuenta = new CuentaPuntos(nombre);

        System.out.print("Puntos a registrar: ");
        int cantidad = scanner.nextInt();

        boolean aceptado = cuenta.sumarPuntos(cantidad);

        if (aceptado) {
            System.out.println("Operación registrada.");
        } else {
            System.out.println("La cantidad debe ser positiva.");
        }

        System.out.println("Titular: " + cuenta.getTitular());
        System.out.println("Puntos: " + cuenta.getPuntos());

        scanner.close();
    }
}
~~~

Ejecuta el programa con una cantidad positiva y después con una negativa.

Fíjate en la separación: `CuentaPuntos` no utiliza `Scanner` ni imprime mensajes; `Main` no modifica directamente el atributo `puntos`.

### 3. Compilar ambos archivos

Desde la terminal situada en `practica-3-4`:

~~~bash
javac CuentaPuntos.java Main.java
java Main
~~~

Se generará un archivo `.class` por cada clase compilada.

Si modificas el código y trabajas desde la terminal, **vuelve a compilar antes de ejecutar**. De lo contrario puedes terminar viendo el comportamiento de una versión anterior.

### Ejercicio — Consultar antes y después

Modifica `Main.java` para:

1. mostrar los puntos iniciales;
2. solicitar una primera cantidad y registrarla;
3. solicitar una segunda cantidad y registrarla;
4. mostrar los puntos finales.

Prueba dos valores válidos y una combinación donde uno sea negativo.

### 4. La misma clase, más de un objeto

Crea `PruebaCuentas.java` en la misma carpeta:

~~~java
public class PruebaCuentas {

    public static void main(String[] args) {
        CuentaPuntos primera = new CuentaPuntos("Ana");
        CuentaPuntos segunda = new CuentaPuntos("Luis");

        primera.sumarPuntos(50);
        segunda.sumarPuntos(120);
        primera.sumarPuntos(20);

        System.out.println(primera.getTitular() + ": " + primera.getPuntos());
        System.out.println(segunda.getTitular() + ": " + segunda.getPuntos());
    }
}
~~~

Compila y ejecuta:

~~~bash
javac CuentaPuntos.java PruebaCuentas.java
java PruebaCuentas
~~~

Resultado esperado:

~~~text
Ana: 70
Luis: 120
~~~

### 5. Diagrama de responsabilidades

Revisa el recorrido de una operación:

~~~text
El usuario escribe una cantidad
              |
              v
          Main.java
      Scanner y mensajes
              |
              | cuenta.sumarPuntos(valor)
              v
       CuentaPuntos.java
       comprueba las reglas
       modifica sus atributos
              |
              | devuelve true o false
              v
          Main.java
       informa el resultado
~~~

La clase principal utiliza la clase `CuentaPuntos`: esa es la colaboración que necesitamos en este laboratorio. Más adelante estudiaremos otros tipos de relaciones entre clases.

### Depuración — Método que no existe

Desde `Main.java` intenta llamar:

~~~java
cuenta.agregarPuntos(50);
~~~

La clase define `sumarPuntos()`, no `agregarPuntos()`.

Compila, lee el mensaje y corrige la llamada. Los nombres y parámetros deben corresponder con el método declarado.

### Reto — Separar una calculadora

Construye una clase `Calculadora` con métodos públicos de instancia para:

- sumar dos números;
- restar dos números;
- multiplicar dos números.

Desde `Main.java` solicita dos valores mediante `Scanner`, crea un objeto `Calculadora` y muestra los tres resultados.

Evita colocar `Scanner` o `System.out.println` dentro de los métodos que realizan las operaciones. Su responsabilidad es devolver resultados numéricos.

### Comprobación

Antes de continuar verifica que puedes explicar qué hace cada archivo, qué responsabilidad tiene `Main` y por qué los datos y reglas deben permanecer dentro del objeto que los administra.

---

## Práctica 3.5 — Reto integrador: Mi Billetera Java con POO

Vas a reconstruir la billetera del Laboratorio 2 utilizando programación orientada a objetos.

En la versión anterior `main` mantenía el saldo y llamaba métodos estáticos que recibían y devolvían ese valor. Ahora crearás un objeto `Billetera` que guarda su propio estado y controla los ingresos y gastos.

**No se trata de cambiar los nombres de los métodos del laboratorio anterior.** La aplicación debe seguir funcionando, pero la responsabilidad de modificar el saldo pasará a la clase `Billetera`.

### Objetivo

Construir una aplicación de consola con dos clases, encapsulamiento, constructor, métodos de instancia, validaciones y menú interactivo.

### Archivos del proyecto

Trabaja en la carpeta `MiBilleteraJava`:

~~~text
MiBilleteraJava/
├── Billetera.java
└── Main.java
~~~

Ambos archivos deben estar en la misma carpeta, sin declarar un `package`.

### 1. Responsabilidades de cada clase

**`Billetera.java`**

Representa una billetera y administra sus datos:

- titular;
- saldo;
- cantidad de ingresos;
- total ingresado;
- cantidad de gastos;
- total gastado.

Todos estos atributos deben ser `private`.

El constructor recibe el nombre del titular. El saldo, los contadores y los acumulados comienzan en cero.

La clase deberá ofrecer métodos para:

- consultar el titular;
- consultar el saldo;
- registrar un ingreso;
- registrar un gasto;
- consultar contadores y totales.

**`Main.java`**

Se encarga de:

- crear el `Scanner`;
- solicitar el nombre del titular;
- crear el objeto `Billetera`;
- mostrar un menú mientras el usuario no seleccione salir;
- pedir el monto de cada operación;
- invocar los métodos de la billetera;
- presentar los mensajes y el resumen.

`Main` no debe modificar directamente los atributos de `Billetera`.

### 2. Menú de la aplicación

El programa debe mostrar:

~~~text
================================
        MI BILLETERA JAVA
================================
Titular: Laura
Saldo disponible: $150000.0

1. Registrar ingreso
2. Registrar gasto
3. Consultar saldo
4. Mostrar resumen
0. Salir

Seleccione una opción:
~~~

El menú se repetirá hasta seleccionar `0`.

### 3. Reglas de funcionamiento

**Crear la billetera**

- solicitar el nombre del titular;
- no permitir que el nombre quede vacío o compuesto solamente por espacios;
- crear un único objeto `Billetera` para la sesión;
- iniciar los valores numéricos en cero.

**Registrar ingreso**

- solicitar el valor;
- aceptar solamente montos mayores que cero;
- sumar el monto al saldo cuando sea válido;
- aumentar el contador de ingresos válidos;
- actualizar el acumulado de ingresos;
- mantener todos los valores sin cambios cuando el monto sea inválido.

**Registrar gasto**

- solicitar el valor;
- rechazar montos menores o iguales que cero;
- rechazar montos superiores al saldo disponible;
- restar el valor cuando sea válido;
- aumentar el contador de gastos válidos;
- actualizar el acumulado de gastos;
- no modificar ningún dato cuando el gasto se rechace.

**Consultar saldo**

- mostrar el saldo actual mediante un método público de consulta.

**Mostrar resumen**

Presentar:

~~~text
================================
        RESUMEN DE BILLETERA
================================
Titular: Laura
Saldo actual: $150000.0
Ingresos realizados: 2
Total ingresado: $200000.0
Gastos realizados: 1
Total gastado: $50000.0
~~~

Los contadores solamente deben registrar operaciones aceptadas.

### 4. Métodos mínimos

Puedes utilizar esta propuesta de interfaz pública:

~~~java
public Billetera(String titular)

public String getTitular()

public double getSaldo()

public boolean registrarIngreso(double valor)

public boolean registrarGasto(double valor)

public int getCantidadIngresos()

public int getCantidadGastos()

public double getTotalIngresos()

public double getTotalGastos()
~~~

No es necesario declarar `setSaldo()`.

Los métodos `registrarIngreso()` y `registrarGasto()` deben devolver `true` cuando realizan la operación y `false` cuando la rechazan.

### 5. Antes de escribir el programa

Compara las dos soluciones:

| Laboratorio 2 | Laboratorio 3 |
|---|---|
| `double saldo` en `main` | `private double saldo` en `Billetera` |
| método estático recibe saldo | método de instancia utiliza el saldo del objeto |
| `main` conserva el resultado devuelto | el objeto actualiza su propio estado |
| variables y operaciones dispersas | atributos y comportamientos reunidos en una clase |

Responde en comentarios dentro de `Main.java`:

1. ¿Qué datos deben pertenecer a la billetera?
2. ¿Qué tareas le corresponden solamente al menú?
3. ¿Qué operaciones pueden cambiar el saldo?
4. ¿Qué información debe poder consultarse desde otra clase?

### 6. Esqueleto de `Billetera.java`

Puedes partir de esta estructura. Completa los atributos y métodos pendientes.

~~~java
public class Billetera {

    private String titular;
    private double saldo;

    // Agrega aquí los contadores y acumulados privados.

    public Billetera(String titular) {
        this.titular = titular;
        this.saldo = 0;

        // Inicializa los demás atributos.
    }

    public String getTitular() {
        return titular;
    }

    public double getSaldo() {
        return saldo;
    }

    public boolean registrarIngreso(double valor) {
        // Comprueba el valor.
        // Si es válido, actualiza saldo, contador y acumulado.
        // Devuelve true o false según corresponda.
        return false;
    }

    public boolean registrarGasto(double valor) {
        // Comprueba que sea positivo y que exista saldo suficiente.
        // Si es válido, actualiza los datos de la billetera.
        return false;
    }

    // Agrega los métodos públicos de consulta restantes.
}
~~~

Los `return false` del esqueleto son temporales. **No representan una implementación terminada.**

No copies todos los cálculos de `main` a esta clase sin revisarlos: los métodos ya no deben recibir el saldo como parámetro, porque la billetera conserva ese dato como atributo.

### 7. Esqueleto de `Main.java`

Crea el archivo:

~~~java
import java.util.Scanner;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Nombre del titular: ");
        String titular = scanner.nextLine().trim();

        while (titular.isEmpty()) {
            System.out.print("El nombre es obligatorio. Intenta nuevamente: ");
            titular = scanner.nextLine().trim();
        }

        Billetera billetera = new Billetera(titular);
        int opcion;

        do {
            mostrarMenu(billetera);
            System.out.print("Seleccione una opción: ");
            opcion = scanner.nextInt();

            switch (opcion) {
                case 1 -> {
                    // Solicitar monto e invocar registrarIngreso().
                }
                case 2 -> {
                    // Solicitar monto e invocar registrarGasto().
                }
                case 3 -> {
                    // Mostrar el saldo actual del objeto.
                }
                case 4 -> {
                    // Presentar resumen con los métodos de consulta.
                }
                case 0 -> System.out.println("Programa finalizado.");
                default -> System.out.println("Opción no válida.");
            }

        } while (opcion != 0);

        scanner.close();
    }

    static void mostrarMenu(Billetera billetera) {
        System.out.println();
        System.out.println("================================");
        System.out.println("        MI BILLETERA JAVA");
        System.out.println("================================");
        System.out.println("Titular: " + billetera.getTitular());
        System.out.println("Saldo disponible: $" + billetera.getSaldo());
        System.out.println();
        System.out.println("1. Registrar ingreso");
        System.out.println("2. Registrar gasto");
        System.out.println("3. Consultar saldo");
        System.out.println("4. Mostrar resumen");
        System.out.println("0. Salir");
    }
}
~~~

El método `mostrarMenu` es estático porque se llama directamente desde `main` y su función es presentar información. La lógica de la billetera sigue en métodos de instancia de `Billetera`.

### 8. Construcción por etapas

**Etapa 1 — Crear el objeto.** Completa el constructor y los atributos de `Billetera`. Ejecuta `Main` y confirma que muestra el titular y saldo cero.

**Etapa 2 — Registrar ingresos.** Implementa el método de instancia y la opción 1. Comprueba que un valor positivo cambie el saldo y uno negativo no lo haga.

**Etapa 3 — Registrar gastos.** Implementa las validaciones y la opción 2. Prueba saldo suficiente, insuficiente y valores no positivos.

**Etapa 4 — Contadores y acumulados.** Agrega los cuatro atributos necesarios para cantidades y totales. Actualízalos solamente cuando una operación se complete.

**Etapa 5 — Resumen.** Implementa los métodos de consulta y presenta la información desde `Main`.

**Etapa 6 — Revisión de encapsulamiento.** Comprueba que ninguna instrucción fuera de `Billetera` cambie `saldo`, `totalIngresos` o `totalGastos` directamente.

Compila y ejecuta después de cada etapa. No dejes la revisión de todos los errores para el final.

### 9. Pistas

Para utilizar un método que devuelve un valor lógico:

~~~java
boolean registrado = billetera.registrarIngreso(valor);

if (registrado) {
    System.out.println("Ingreso registrado.");
} else {
    System.out.println("El ingreso no fue aceptado.");
}
~~~

Para pedir un monto:

~~~java
System.out.print("Valor: ");
double valor = scanner.nextDouble();
~~~

Para modificar los datos dentro de `Billetera` cuando la operación sea válida:

~~~java
saldo += valor;
cantidadIngresos++;
totalIngresos += valor;
~~~

Estas tres instrucciones deben ejecutarse juntas **solo después de comprobar el monto**. Una operación rechazada no debe cambiar contadores ni acumulados.

### 10. Compilación y ejecución

Desde la carpeta `MiBilleteraJava`:

~~~bash
javac Billetera.java Main.java
java Main
~~~

Si utilizas **Run** en VS Code, ejecuta la clase `Main` y escribe los datos en la terminal.

No se requieren bibliotecas externas.

### 11. Pruebas obligatorias

Calcula primero los resultados esperados. Después realiza las operaciones en el programa.

| Escenario | Resultado esperado |
|---|---|
| Crear billetera | Saldo y contadores en cero |
| Ingresar $200.000 | Saldo $200.000; 1 ingreso |
| Gastar $50.000 | Saldo $150.000; 1 gasto |
| Gastar $300.000 | Operación rechazada; saldo $150.000 |
| Ingresar $0 | Rechazado; no cambia el contador |
| Ingresar -$5.000 | Rechazado; no cambia el saldo |
| Gastar $0 | Rechazado |
| Gastar -$10.000 | Rechazado |
| Mostrar resumen | Total ingresado $200.000; total gastado $50.000 |
| Salir | Programa finaliza correctamente |

Para esta secuencia, el resumen esperado es:

~~~text
Titular: Laura
Saldo actual: $150000.0
Ingresos realizados: 1
Total ingresado: $200000.0
Gastos realizados: 1
Total gastado: $50000.0
~~~

### 12. Prueba de independencia de objetos

Una ventaja de representar la billetera como clase es poder crear más de una.

Crea un archivo adicional `PruebaBilleteras.java` en la misma carpeta:

~~~java
public class PruebaBilleteras {

    public static void main(String[] args) {
        Billetera laura = new Billetera("Laura");
        Billetera diego = new Billetera("Diego");

        laura.registrarIngreso(100000);
        diego.registrarIngreso(200000);
        laura.registrarGasto(25000);

        System.out.println("Laura: $" + laura.getSaldo());
        System.out.println("Diego: $" + diego.getSaldo());
    }
}
~~~

Compila y ejecuta:

~~~bash
javac Billetera.java PruebaBilleteras.java
java PruebaBilleteras
~~~

Resultado esperado:

~~~text
Laura: $75000.0
Diego: $200000.0
~~~

Las operaciones de Laura no deben alterar la billetera de Diego.

### 13. Reto de ampliación

Una vez que la aplicación principal funcione y supere las pruebas, agrega:

- una opción para mostrar el número de operaciones válidas realizadas;
- un método `getBalanceOperaciones()` que devuelva `totalIngresos - totalGastos` y permita comprobar que coincide con `saldo`;
- un mensaje diferente para gasto con saldo insuficiente y gasto con monto no positivo, sin permitir que `Main` modifique el saldo;
- un tercer objeto de prueba con operaciones diferentes.

No utilices todavía archivos ni bases de datos. Tampoco es necesario crear una colección de billeteras.

### 14. Depuración

Crea una copia antes de introducir errores. Después provoca y corrige, uno por uno:

1. dejar `saldo` público y asignarle un valor negativo desde `Main`;
2. incrementar `cantidadIngresos` antes de validar un ingreso;
3. permitir que un gasto mayor que el saldo se reste de todos modos;
4. olvidar actualizar `totalGastos` después de un gasto válido;
5. utilizar un atributo `static` para el saldo y comprobar cómo afectaría a dos objetos.

En el quinto caso, vuelve a dejar `saldo` como atributo de instancia: `private double saldo;`.

### 15. Revisión del proyecto

Antes de entregar, comprueba lo siguiente:

- la clase `Billetera` tiene atributos privados;
- el constructor recibe el titular y establece los valores iniciales;
- no existe un método público para asignar el saldo libremente;
- los ingresos y gastos se realizan mediante métodos de instancia;
- las operaciones inválidas no cambian ningún acumulado;
- `Main` conserva el menú, la lectura de datos y los mensajes;
- `Billetera` no usa `Scanner` ni depende de un menú;
- se pueden crear dos billeteras con saldos independientes;
- el programa funciona después de volver a compilar desde cero;
- puedes explicar qué hace cada atributo y método.

### Evidencia de trabajo

Conserva los archivos de las prácticas y la carpeta final `MiBilleteraJava` en tu repositorio de trabajo. Incluye un `README.md` breve con:

- nombre y propósito de la aplicación;
- indicaciones para compilar y ejecutar;
- descripción de las clases `Main` y `Billetera`;
- casos de prueba realizados y resultados observados.

No es necesario elaborar un informe en PDF ni una presentación.

---

## Cierre del laboratorio

Al finalizar debes poder explicar la diferencia entre clase y objeto, crear instancias con `new`, inicializar sus atributos mediante constructores, utilizar `this`, proteger datos con `private`, exponer operaciones mediante métodos públicos y separar la lógica de una entidad de la interacción con el usuario.

Compara la nueva `Billetera.java` con tu `BilleteraConsola.java` del laboratorio anterior. Ambas resuelven un problema semejante, pero ahora cada billetera conserva sus datos y es responsable de aplicar sus propias reglas.

En los siguientes laboratorios podremos ampliar esta base con manejo de excepciones, colecciones y persistencia, sin necesitar todavía un framework.
