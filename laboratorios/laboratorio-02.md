# Laboratorio 2 — Decisiones, ciclos y métodos en Java

En este laboratorio vas a construir programas capaces de tomar decisiones, repetir operaciones y organizar parte de la lógica en métodos. El objetivo es pasar de programas que siguen una secuencia fija a programas que responden a diferentes entradas y pueden mantenerse activos hasta que el usuario decida terminar.

| Práctica | Tema |
|---|---|
| 2.1 | Comparaciones y expresiones booleanas |
| 2.2 | Decisiones con `if`, `else if` y `else` |
| 2.3 | Selección de opciones con `switch` |
| 2.4 | Repetición con `for`, `while` y `do while` |
| 2.5 | Métodos y reto integrador: billetera de consola |

> Ejecuta cada ejemplo antes de modificarlo. Cuando un resultado no sea el esperado, revisa primero los valores de las variables y las condiciones antes de cambiar varias partes del programa.

---

## Práctica 2.1 — Comparaciones y expresiones booleanas

### Objetivos

Al finalizar esta práctica podrás:

- comparar valores numéricos y texto;
- utilizar operadores relacionales;
- almacenar el resultado de una comparación en una variable `boolean`;
- combinar condiciones con operadores lógicos;
- predecir el resultado de expresiones antes de ejecutarlas.

Hasta ahora los programas del laboratorio anterior realizaban operaciones y mostraban resultados. Ahora necesitamos que puedan responder preguntas como:

- ¿el saldo es suficiente?
- ¿la nota alcanza para aprobar?
- ¿la edad cumple un requisito?
- ¿dos valores son iguales?

Estas preguntas producen un resultado de tipo `boolean`: `true` o `false`.

### 1. Operadores de comparación

| Operador | Significado |
|---|---|
| `>` | mayor que |
| `<` | menor que |
| `>=` | mayor o igual que |
| `<=` | menor o igual que |
| `==` | igual a |
| `!=` | diferente de |

Crea:

```text
Comparaciones.java
```

Escribe:

```java
public class Comparaciones {

    public static void main(String[] args) {
        int edad = 20;

        System.out.println(edad > 18);
        System.out.println(edad < 18);
        System.out.println(edad >= 20);
        System.out.println(edad == 20);
        System.out.println(edad != 20);
    }
}
```

### Antes de ejecutar

Escribe qué esperas ver en cada línea.

Después ejecuta y compara tu predicción con el resultado.

### 2. Guardar una comparación

Una comparación puede almacenarse:

```java
double nota = 3.8;

boolean aprobo = nota >= 3.0;

System.out.println(aprobo);
```

Prueba también:

```java
double saldo = 120000;
double compra = 150000;

boolean puedeComprar = saldo >= compra;

System.out.println(puedeComprar);
```

Modifica los valores de `saldo` y `compra` hasta obtener ambos resultados posibles.

### Ejercicio — Comparar cantidades

Crea:

```text
CompararCantidades.java
```

Declara:

```java
int inventario = 12;
int cantidadSolicitada = 8;
```

Calcula y muestra:

- si hay inventario suficiente;
- si la cantidad solicitada es exactamente igual al inventario;
- si la cantidad solicitada supera el inventario.

Guarda cada comparación en una variable `boolean`.

### 3. Comparar texto

Con números podemos utilizar `==` para comparar valores. Con objetos como `String`, en estas prácticas utilizaremos `.equals()`.

Ejecuta:

```java
String ciudad = "Bogotá";

System.out.println(ciudad.equals("Bogotá"));
System.out.println(ciudad.equals("Medellín"));
```

Para ignorar diferencias entre mayúsculas y minúsculas puedes utilizar:

```java
System.out.println(ciudad.equalsIgnoreCase("bogotá"));
```

### Importante

Evita utilizar:

```java
ciudad == "Bogotá"
```

para decidir si dos textos tienen el mismo contenido.

Para esta etapa usa:

```java
ciudad.equals("Bogotá")
```

### Ejercicio — Validación simple de usuario

Crea:

```text
ValidacionUsuario.java
```

Declara:

```java
String usuarioRegistrado = "aprendiz";
String usuarioIngresado = "Aprendiz";
```

Muestra:

- el resultado usando `.equals()`;
- el resultado usando `.equalsIgnoreCase()`.

Explica qué diferencia observaste.

### 4. Operadores lógicos

Podemos combinar varias condiciones.

| Operador | Significado |
|---|---|
| `&&` | y |
| `||` | o |
| `!` | negación |

Ejemplo:

```java
int edad = 21;
boolean documentoValido = true;

boolean puedeIngresar = edad >= 18 && documentoValido;

System.out.println(puedeIngresar);
```

En este caso ambas condiciones deben ser verdaderas.

Prueba:

```java
boolean tieneCarnet = false;
boolean tieneAutorizacion = true;

boolean puedeEntrar = tieneCarnet || tieneAutorizacion;

System.out.println(puedeEntrar);
```

Aquí basta con que una de las condiciones sea verdadera.

### Antes de ejecutar — Predice

Analiza:

```java
int edad = 17;
boolean acompanado = true;

boolean resultado1 = edad >= 18;
boolean resultado2 = edad >= 18 || acompanado;
boolean resultado3 = edad >= 18 && acompanado;
boolean resultado4 = !acompanado;

System.out.println(resultado1);
System.out.println(resultado2);
System.out.println(resultado3);
System.out.println(resultado4);
```

Escribe los cuatro resultados antes de ejecutar.

### Reto — Acceso a una actividad

Crea:

```text
AccesoActividad.java
```

Utiliza variables para representar:

- edad;
- inscripción activa;
- documento presentado.

Calcula expresiones booleanas que permitan conocer:

- si la persona es mayor de edad;
- si cumple inscripción y documento;
- si cumple los tres requisitos.

Todavía no necesitas utilizar `if`. Muestra directamente los valores `true` o `false`.

### Depuración — Encuentra los errores

Copia:

```java
public class ErrorComparaciones {

    public static void main(String[] args) {
        int edad = 20;
        boolean activo = true;

        boolean mayorEdad = edad = 18;
        boolean habilitado = edad >= 18 & activo;

        System.out.println(mayorEdad);
        System.out.println(habilitado);
    }
}
```

Revisa:

- qué operador debería utilizarse para comparar igualdad;
- qué operador lógico utilizaremos para expresar “y”.

Corrige el código y comprueba los resultados.

### Comprobación

Antes de continuar asegúrate de poder explicar:

- qué diferencia existe entre `=` y `==`;
- qué resultado produce una comparación;
- cuándo usar `&&`;
- cuándo usar `||`;
- cómo comparar el contenido de dos `String`.

---

## Práctica 2.2 — Decisiones con `if`, `else if` y `else`

### Objetivos

Al finalizar esta práctica podrás:

- ejecutar código solamente cuando se cumpla una condición;
- construir decisiones de dos alternativas;
- trabajar con varias posibilidades mediante `else if`;
- combinar condiciones;
- identificar problemas comunes en estructuras condicionales.

### 1. Ejecutar código según una condición

Crea:

```text
ValidarEdad.java
```

Escribe:

```java
public class ValidarEdad {

    public static void main(String[] args) {
        int edad = 20;

        if (edad >= 18) {
            System.out.println("Es mayor de edad.");
        }
    }
}
```

Ejecuta.

Después cambia:

```java
int edad = 16;
```

Vuelve a ejecutar.

Observa que el mensaje solamente aparece cuando la condición es verdadera.

### 2. Dos posibles caminos

Agrega `else`:

```java
if (edad >= 18) {
    System.out.println("Es mayor de edad.");
} else {
    System.out.println("Es menor de edad.");
}
```

Prueba al menos estos valores:

```text
17
18
25
```

### Ejercicio — Número positivo, negativo o cero

Crea:

```text
ClasificarNumero.java
```

Solicita un número entero utilizando `Scanner`.

El programa debe mostrar:

- `El número es positivo`;
- `El número es negativo`;
- o `El número es cero`.

Piensa cuántos caminos posibles existen antes de escribir el código.

### 3. Varias alternativas con `else if`

Crea:

```text
ClasificarNota.java
```

Utiliza:

```java
double nota = 4.2;
```

Completa:

```java
if (nota >= 4.5) {
    System.out.println("Desempeño sobresaliente");
} else if (nota >= 4.0) {
    System.out.println("Desempeño alto");
} else if (nota >= 3.0) {
    System.out.println("Desempeño aprobatorio");
} else {
    System.out.println("No aprobado");
}
```

Prueba:

```text
4.8
4.2
3.5
2.7
```

### Pregunta de análisis

¿Por qué comenzamos por la condición:

```java
nota >= 4.5
```

y no por:

```java
nota >= 3.0
```

Cambia temporalmente el orden y observa el resultado.

### 4. Condiciones compuestas

Una aplicación permite aplicar un beneficio cuando el cliente tiene membresía activa y la compra supera cierto valor.

```java
boolean membresiaActiva = true;
double totalCompra = 250000;

if (membresiaActiva && totalCompra >= 200000) {
    System.out.println("Aplica beneficio.");
} else {
    System.out.println("No aplica beneficio.");
}
```

Prueba las cuatro combinaciones posibles cambiando:

- membresía activa / inactiva;
- compra mayor / menor al valor requerido.

### Ejercicio — Acceso al sistema

Solicita:

- nombre de usuario;
- contraseña.

Compara con:

```java
String usuarioCorrecto = "admin";
String claveCorrecta = "java21";
```

El programa debe indicar si las credenciales coinciden.

Utiliza `.equals()` para comparar texto.

No almacenes contraseñas reales. Este ejercicio solamente practica condiciones.

### 5. Condiciones anidadas

A veces una segunda decisión solamente tiene sentido después de cumplir una primera condición.

Ejemplo:

```java
int edad = 20;
boolean tieneDocumento = true;

if (edad >= 18) {
    if (tieneDocumento) {
        System.out.println("Ingreso autorizado.");
    } else {
        System.out.println("Debe presentar documento.");
    }
} else {
    System.out.println("No cumple la edad requerida.");
}
```

Ahora expresa la misma regla utilizando `&&`.

Compara ambas soluciones.

No siempre es mejor anidar. Si una condición compuesta se entiende fácilmente, suele resultar más clara.

### Reto — Calculadora de descuento

Crea:

```text
DescuentoCompra.java
```

Solicita:

- nombre del cliente;
- valor de la compra.

Aplica estas reglas:

| Valor de compra | Descuento |
|---:|---:|
| $500.000 o más | 15 % |
| $250.000 o más | 10 % |
| $100.000 o más | 5 % |
| Menos de $100.000 | 0 % |

El programa debe calcular:

- valor original;
- porcentaje aplicado;
- valor del descuento;
- total final.

Prueba valores cercanos a los límites:

```text
99999
100000
249999
250000
499999
500000
```

### Depuración — ¿Por qué siempre entra?

Analiza:

```java
int edad = 15;

if (edad >= 18);
{
    System.out.println("Mayor de edad");
}
```

Ejecuta.

Localiza el carácter que cambia el comportamiento de la estructura.

Corrige y vuelve a probar.

### Comprobación

Antes de continuar asegúrate de poder explicar:

- qué ocurre cuando la condición de un `if` es falsa;
- cuándo utilizar `else`;
- para qué sirve `else if`;
- por qué importa el orden de las condiciones;
- cómo combinar dos requisitos en una misma condición.

---

## Práctica 2.3 — Selección de opciones con `switch`

### Objetivos

Al finalizar esta práctica podrás:

- utilizar `switch` cuando una variable puede tomar varias opciones concretas;
- construir menús sencillos;
- diferenciar un problema apropiado para `switch` de uno más adecuado para `if`;
- utilizar `default` para opciones no contempladas.

### 1. Seleccionar una opción

Crea:

```text
MenuBasico.java
```

Escribe:

```java
import java.util.Scanner;

public class MenuBasico {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.println("1. Consultar");
        System.out.println("2. Registrar");
        System.out.println("3. Eliminar");

        System.out.print("Seleccione una opción: ");
        int opcion = scanner.nextInt();

        switch (opcion) {
            case 1 -> System.out.println("Seleccionó consultar.");
            case 2 -> System.out.println("Seleccionó registrar.");
            case 3 -> System.out.println("Seleccionó eliminar.");
            default -> System.out.println("Opción no válida.");
        }

        scanner.close();
    }
}
```

Prueba:

```text
1
2
3
8
```

En Java 21 podemos utilizar esta forma de `switch` con `->`, que evita el uso de `break` en cada caso.

### 2. ¿Cuándo usar `switch`?

Un `switch` funciona bien cuando queremos comparar una misma variable contra opciones concretas.

Por ejemplo:

```text
1
2
3
0
```

o:

```text
A
B
C
```

En cambio, una condición como:

```java
edad >= 18 && activo
```

se expresa mejor con `if`.

### Ejercicio — Día de la semana

Crea:

```text
DiaSemana.java
```

Solicita un número del 1 al 7.

Utiliza `switch` para mostrar el nombre del día correspondiente.

Para cualquier otro valor muestra:

```text
Día no válido
```

### 3. `switch` con texto

También puedes utilizar `String`:

```java
String comando = "guardar";

switch (comando) {
    case "guardar" -> System.out.println("Guardando...");
    case "buscar" -> System.out.println("Buscando...");
    case "salir" -> System.out.println("Saliendo...");
    default -> System.out.println("Comando desconocido.");
}
```

### Ejercicio — Conversor por menú

Crea:

```text
ConversorMenu.java
```

Muestra:

```text
CONVERSOR
1. Kilómetros a metros
2. Metros a centímetros
3. Celsius a Fahrenheit
0. Salir
```

Solicita una opción.

Dependiendo de la opción, pide el dato necesario y realiza la conversión.

Por ahora el menú se ejecutará una sola vez. En la siguiente práctica haremos que permanezca activo.

### Reto — Calculadora con `switch`

Crea:

```text
CalculadoraMenu.java
```

Solicita dos números y luego muestra:

```text
1. Sumar
2. Restar
3. Multiplicar
4. Dividir
```

Utiliza `switch` para ejecutar la operación seleccionada.

Antes de dividir, comprueba con `if` que el segundo número no sea cero.

### Depuración — Opción no contemplada

Construye un `switch` con opciones 1, 2 y 3, pero omite temporalmente `default`.

Ejecuta con:

```text
9
```

¿Qué recibe el usuario?

Agrega `default` y vuelve a probar.

### Comprobación

Antes de continuar asegúrate de poder decidir cuándo utilizar:

- `if`;
- `else if`;
- `switch`;
- `default`.

---

## Práctica 2.4 — Repetición con `for`, `while` y `do while`

### Objetivos

Al finalizar esta práctica podrás:

- repetir instrucciones un número conocido de veces;
- mantener un programa activo mientras se cumpla una condición;
- utilizar contadores y acumuladores;
- construir un menú repetitivo;
- reconocer un ciclo infinito básico.

### 1. Repetir con `for`

Crea:

```text
ContadorFor.java
```

Escribe:

```java
public class ContadorFor {

    public static void main(String[] args) {

        for (int numero = 1; numero <= 5; numero++) {
            System.out.println(numero);
        }
    }
}
```

Antes de ejecutar identifica:

- valor inicial;
- condición;
- cambio realizado en cada repetición.

Prueba después:

```java
for (int numero = 10; numero >= 1; numero--) {
    System.out.println(numero);
}
```

### Ejercicio — Tabla de multiplicar

Solicita un número.

Utiliza `for` para mostrar su tabla del 1 al 10.

Ejemplo:

```text
Número: 7

7 x 1 = 7
7 x 2 = 14
...
7 x 10 = 70
```

### 2. Acumuladores

Un ciclo también puede ir construyendo un resultado.

Ejecuta:

```java
int suma = 0;

for (int numero = 1; numero <= 5; numero++) {
    suma = suma + numero;
}

System.out.println("Suma: " + suma);
```

Haz una tabla en papel o comentarios:

```text
numero | suma
1      | ?
2      | ?
3      | ?
4      | ?
5      | ?
```

Completa los valores antes de ejecutar.

### Ejercicio — Promedio de notas

Solicita cuántas notas se van a registrar.

Luego utiliza `for` para:

- solicitar cada nota;
- acumularlas;
- calcular el promedio.

Ejemplo de interacción:

```text
Cantidad de notas: 3
Nota 1: 4.0
Nota 2: 3.5
Nota 3: 4.5
Promedio: 4.0
```

### 3. Repetir con `while`

Crea:

```text
ContadorWhile.java
```

Escribe:

```java
int numero = 1;

while (numero <= 5) {
    System.out.println(numero);
    numero++;
}
```

Compara este ciclo con el primer `for`.

Ambos pueden producir el mismo resultado, pero `while` resulta especialmente útil cuando no sabemos de antemano cuántas repeticiones serán necesarias.

### Ejercicio — Repetir hasta escribir cero

Solicita números al usuario.

Mientras el número sea diferente de cero:

- muéstralo;
- solicita otro.

Cuando escriba `0`, termina.

### 4. Menú persistente

Ahora podemos mantener un menú activo.

Crea:

```text
MenuRepetitivo.java
```

Escribe:

```java
import java.util.Scanner;

public class MenuRepetitivo {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int opcion = -1;

        while (opcion != 0) {
            System.out.println();
            System.out.println("1. Saludar");
            System.out.println("2. Mostrar mensaje");
            System.out.println("0. Salir");

            System.out.print("Opción: ");
            opcion = scanner.nextInt();

            switch (opcion) {
                case 1 -> System.out.println("Hola.");
                case 2 -> System.out.println("Java continúa ejecutándose.");
                case 0 -> System.out.println("Programa finalizado.");
                default -> System.out.println("Opción no válida.");
            }
        }

        scanner.close();
    }
}
```

Ejecuta varias opciones antes de salir.

Observa que el `switch` decide qué hacer y el `while` decide si el menú vuelve a mostrarse.

### 5. `do while`

Existe otra forma de ciclo:

```java
int opcion;

do {
    System.out.println("1. Continuar");
    System.out.println("0. Salir");

    opcion = scanner.nextInt();

} while (opcion != 0);
```

La diferencia principal es que el bloque se ejecuta al menos una vez antes de revisar la condición.

### Ejercicio — Comparar `while` y `do while`

Construye dos versiones pequeñas de un menú:

- una con `while`;
- otra con `do while`.

Escribe en un comentario cuál te resulta más natural para mostrar un menú al menos una vez.

### Depuración — Ciclo infinito

Analiza:

```java
int numero = 1;

while (numero <= 5) {
    System.out.println(numero);
}
```

Antes de ejecutarlo responde:

- ¿qué variable controla el ciclo?
- ¿esa variable cambia?
- ¿la condición llegará a ser falsa?

Corrige el programa.

Si alguna vez ejecutas un ciclo infinito en la terminal, puedes detenerlo normalmente con **Ctrl + C**.

### Reto — Suma hasta salir

Crea:

```text
AcumuladorInteractivo.java
```

El programa debe solicitar valores y mantener una suma acumulada.

Ejemplo:

```text
Valor: 100
Acumulado: 100

Valor: 50
Acumulado: 150

Valor: 25
Acumulado: 175

Valor: 0
Programa finalizado.
```

El valor `0` termina el ciclo y no debe alterar el acumulado.

### Comprobación

Antes de continuar asegúrate de poder explicar:

- cuándo elegir `for`;
- cuándo elegir `while`;
- qué diferencia básica tiene `do while`;
- qué es un contador;
- qué es un acumulador;
- cómo puede aparecer un ciclo infinito.

---

## Práctica 2.5 — Métodos y reto integrador: billetera de consola

Hasta este punto hemos escrito casi toda la lógica dentro de `main`. Cuando un programa empieza a crecer, conviene separar operaciones que tienen una responsabilidad concreta.

En esta práctica utilizaremos métodos estáticos sencillos. El objetivo todavía no es trabajar programación orientada a objetos; eso se abordará en el siguiente laboratorio.

### Objetivos

Al finalizar esta práctica podrás:

- declarar y llamar métodos;
- enviar datos mediante parámetros;
- devolver resultados con `return`;
- diferenciar un método que retorna un valor de uno `void`;
- separar responsabilidades dentro de un programa;
- integrar condiciones, ciclos y métodos en una aplicación de consola.

### 1. Primer método

Crea:

```text
PrimerMetodo.java
```

Escribe:

```java
public class PrimerMetodo {

    public static void main(String[] args) {
        mostrarTitulo();
        mostrarTitulo();
    }

    static void mostrarTitulo() {
        System.out.println("====================");
        System.out.println("     APLICACIÓN");
        System.out.println("====================");
    }
}
```

Ejecuta.

El método:

```java
mostrarTitulo()
```

puede invocarse cada vez que necesitamos ese comportamiento.

### 2. Métodos con parámetros

Ejecuta:

```java
public class Saludos {

    public static void main(String[] args) {
        saludar("Laura");
        saludar("Andrés");
    }

    static void saludar(String nombre) {
        System.out.println("Hola, " + nombre);
    }
}
```

El parámetro:

```java
String nombre
```

permite que el mismo método trabaje con valores diferentes.

### Ejercicio — Mostrar producto

Crea un método:

```java
static void mostrarProducto(String nombre, double precio)
```

Llámalo varias veces con productos diferentes.

### 3. Métodos que retornan valores

Crea:

```text
MetodosCalculo.java
```

Escribe:

```java
public class MetodosCalculo {

    public static void main(String[] args) {
        double total = calcularTotal(85000, 3);

        System.out.println("Total: $" + total);
    }

    static double calcularTotal(double precio, int cantidad) {
        return precio * cantidad;
    }
}
```

El método recibe datos, realiza una operación y devuelve el resultado.

### Ejercicio — Descuento

Crea:

```java
static double calcularDescuento(double subtotal, double porcentaje)
```

Prueba:

```java
double subtotal = 200000;
double descuento = calcularDescuento(subtotal, 0.10);

System.out.println(descuento);
```

Resultado:

```text
20000.0
```

### 4. Separar el menú

Crea:

```java
static void mostrarMenu() {
    System.out.println();
    System.out.println("1. Registrar ingreso");
    System.out.println("2. Registrar gasto");
    System.out.println("3. Consultar saldo");
    System.out.println("0. Salir");
}
```

Luego desde `main`:

```java
mostrarMenu();
```

El método no necesita devolver nada, por eso utiliza `void`.

### Ejercicio — Identifica el tipo de método

Para cada necesidad decide si utilizarías `void` o un valor de retorno:

1. mostrar un título;
2. calcular un subtotal;
3. mostrar un menú;
4. calcular un descuento;
5. mostrar un mensaje de despedida.

Escribe la respuesta antes de programar.

### 5. Reto integrador — Billetera de consola

Crea:

```text
BilleteraConsola.java
```

Vas a construir una aplicación que permita manejar un saldo durante la ejecución.

Todavía no utilizaremos clases propias para representar la billetera. El objetivo es resolver el problema con variables, decisiones, ciclos y métodos.

### Requisitos

Al iniciar, solicita:

```text
Nombre del titular:
```

El saldo comienza en:

```text
0
```

Después muestra repetidamente:

```text
=========================
       MI BILLETERA
=========================
1. Registrar ingreso
2. Registrar gasto
3. Consultar saldo
4. Mostrar resumen
0. Salir
```

La aplicación debe permanecer activa hasta seleccionar `0`.

### Reglas

**Registrar ingreso**

- solicitar un valor;
- aceptar únicamente valores mayores que cero;
- sumar el valor al saldo.

**Registrar gasto**

- solicitar un valor;
- comprobar que sea mayor que cero;
- comprobar que no sea superior al saldo;
- descontarlo cuando sea válido.

**Consultar saldo**

Mostrar el saldo actual.

**Mostrar resumen**

Mostrar:

- titular;
- saldo actual.

### Métodos mínimos

Organiza la solución utilizando al menos:

```java
static void mostrarMenu()
```

```java
static double registrarIngreso(Scanner scanner, double saldo)
```

```java
static double registrarGasto(Scanner scanner, double saldo)
```

```java
static void mostrarSaldo(double saldo)
```

También puedes crear otros métodos si encuentras una responsabilidad que tenga sentido separar.

### Construcción por etapas

No intentes escribir toda la aplicación de una vez.

#### Etapa 1 — Menú

Haz que el menú se muestre repetidamente y permita salir.

Todavía no implementes ingresos ni gastos.

#### Etapa 2 — Consulta

Agrega:

```text
3. Consultar saldo
```

El saldo inicial debe mostrar:

```text
$0.0
```

#### Etapa 3 — Ingresos

Implementa el registro de ingresos.

Prueba:

```text
Ingreso: 100000
Saldo: 100000
```

Luego:

```text
Ingreso: 50000
Saldo: 150000
```

Comprueba también un ingreso:

```text
-20000
```

El saldo no debe cambiar.

#### Etapa 4 — Gastos

Implementa el gasto.

Prueba con saldo de:

```text
150000
```

y un gasto:

```text
40000
```

Saldo esperado:

```text
110000
```

Después intenta gastar:

```text
200000
```

El programa debe indicar que el saldo es insuficiente y conservar:

```text
110000
```

#### Etapa 5 — Métodos

Si inicialmente escribiste la lógica directamente en `main`, sepárala ahora utilizando los métodos indicados.

Después de cada cambio vuelve a ejecutar todas las opciones.

### Esqueleto inicial

Puedes comenzar desde esta estructura:

```java
import java.util.Scanner;

public class BilleteraConsola {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        System.out.print("Nombre del titular: ");
        String titular = scanner.nextLine();

        double saldo = 0;
        int opcion;

        do {
            mostrarMenu();

            System.out.print("Opción: ");
            opcion = scanner.nextInt();

            switch (opcion) {
                case 1 -> {
                    // Registrar ingreso
                }
                case 2 -> {
                    // Registrar gasto
                }
                case 3 -> {
                    // Consultar saldo
                }
                case 4 -> {
                    // Mostrar resumen
                }
                case 0 -> System.out.println("Programa finalizado.");
                default -> System.out.println("Opción no válida.");
            }

        } while (opcion != 0);

        scanner.close();
    }

    static void mostrarMenu() {
        System.out.println();
        System.out.println("=========================");
        System.out.println("       MI BILLETERA");
        System.out.println("=========================");
        System.out.println("1. Registrar ingreso");
        System.out.println("2. Registrar gasto");
        System.out.println("3. Consultar saldo");
        System.out.println("4. Mostrar resumen");
        System.out.println("0. Salir");
    }
}
```

El resto de la solución debes construirlo a partir de las prácticas anteriores.

### Pistas

Un método que modifica conceptualmente el saldo puede devolver el nuevo valor:

```java
saldo = registrarIngreso(scanner, saldo);
```

Dentro del método:

```java
return nuevoSaldo;
```

Si una operación no es válida, puedes devolver el saldo sin modificar:

```java
return saldo;
```

### Pruebas obligatorias

Comprueba al menos estos escenarios.

#### Caso 1 — Uso normal

```text
Saldo inicial: 0
Ingreso: 200000
Gasto: 50000
Saldo esperado: 150000
```

#### Caso 2 — Gasto superior al saldo

```text
Saldo: 100000
Gasto: 150000
Saldo esperado: 100000
```

#### Caso 3 — Valores inválidos

Prueba:

```text
Ingreso: 0
Ingreso: -50000
Gasto: 0
Gasto: -10000
```

Ninguno debe modificar el saldo.

#### Caso 4 — Varias operaciones

Realiza:

```text
+100000
+80000
-30000
+20000
-50000
```

Calcula primero el saldo esperado sin ejecutar.

Después comprueba el programa.

### Revisión del código

Antes de terminar revisa:

- ¿el menú permanece activo hasta seleccionar salir?
- ¿cada opción ejecuta solamente la operación correspondiente?
- ¿los ingresos inválidos se rechazan?
- ¿los gastos superiores al saldo se rechazan?
- ¿el saldo se conserva entre iteraciones?
- ¿los cálculos están separados de la salida cuando corresponde?
- ¿los métodos tienen nombres que explican su responsabilidad?
- ¿puedes explicar qué datos recibe y qué devuelve cada método?

### Reto de ampliación

Agrega:

- contador de ingresos realizados;
- contador de gastos realizados;
- acumulado total de ingresos;
- acumulado total de gastos.

El resumen debe mostrar:

```text
Titular:
Saldo:
Cantidad de ingresos:
Total ingresado:
Cantidad de gastos:
Total gastado:
```

No utilices todavía arreglos, listas, archivos ni bases de datos.

### Reto de depuración

Crea una copia de la aplicación y provoca intencionalmente:

1. una condición incorrecta;
2. un ciclo que no termine;
3. un `switch` que ejecute una opción equivocada;
4. un método que devuelva un saldo incorrecto.

Corrige cada problema y escribe un comentario breve explicando qué lo causaba.

---

## Cierre del laboratorio

Al finalizar deberías poder:

- construir expresiones booleanas;
- tomar decisiones con `if`, `else if` y `else`;
- combinar condiciones con `&&`, `||` y `!`;
- utilizar `switch` para opciones concretas;
- repetir operaciones con `for`, `while` y `do while`;
- trabajar con contadores y acumuladores;
- declarar y llamar métodos;
- utilizar parámetros y valores de retorno;
- mantener un programa interactivo activo mediante un menú;
- dividir una aplicación pequeña en responsabilidades más claras.

Conserva `BilleteraConsola.java`. En el siguiente laboratorio volveremos sobre este problema para comenzar a representar sus datos y comportamientos mediante objetos.
