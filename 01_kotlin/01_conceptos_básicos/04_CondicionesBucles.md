# Condiciones y bucles en Kotlin

## 1. Cómo usar sentencias `if/else` para expresar condiciones
En Kotlin, las sentencias `if/else` funcionan de manera similar a otros lenguajes. A continuación se muestra cómo se usan en diferentes contextos.

### Uso básico de `if/else`
Este es el uso más común para verificar una condición:
```kotlin
val numero = 10

if (numero > 5) {
    println("El número es mayor que 5")
} else {
    println("El número es menor o igual a 5")
}
```

### Anidación de `if/else`
Se pueden anidar múltiples condiciones `if/else` para manejar más casos:
```kotlin
val numero = 10

if (numero > 10) {
    println("El número es mayor que 10")
} else if (numero == 10) {
    println("El número es igual a 10")
} else {
    println("El número es menor que 10")
}
```

### Condiciones más complejas
Se pueden combinar varias condiciones en un `if` utilizando operadores lógicos como `&&` (y) y `||` (o):
```kotlin
val numero = 10

if (numero > 5 && numero < 15) {
    println("El número está entre 5 y 15")
}
```
```kotlin
val edad = 20

if (edad < 18 || edad > 65) {
    println("Es menor de 18 o mayor de 65 años.")
} else {
    println("Tiene entre 18 y 65 años.")
}
```

### Minicuestionario

**Pregunta 1:** ¿Cuál será la salida del siguiente código?
```kotlin
val numero = 7
val resultado = if (numero % 2 == 0) {
    "Par"
} else {
    "Impar"
}
println(resultado)
```
1. `Par`
2. `Impar`
3. `Error`
<!--
<details>
  <summary>Respuesta</summary>

  Opción 2: `Impar`. Como `7 % 2` es `1` (no `0`), la condición `numero % 2 == 0` es falsa y se ejecuta la rama `else`.
</details>
-->
**Pregunta 2:** Complete el siguiente código para que muestre "Adulto" si el valor de `edad` es mayor o igual a 18, y "Menor de edad" en caso contrario.
```kotlin
val edad = 20
if (edad >= 18) {
    // Añada aquí el código
} else {
    // Añada aquí el código
}
```
<!--
<details>
  <summary>Respuesta</summary>

  ```kotlin
  val edad = 20
  if (edad >= 18) {
      println("Adulto")
  } else {
      println("Menor de edad")
  }
  ```
</details>
-->
**Pregunta 3:** Cree una función `verificarAprobado` que muestre "Aprobado" si `nota` es 60 o más, y "Reprobado" en caso contrario.
```kotlin
fun verificarAprobado(nota: Int) {
    // Añada aquí el código
}
```
<!--
<details>
  <summary>Respuesta</summary>

  ```kotlin
  fun verificarAprobado(nota: Int) {
      if (nota >= 60) {
          println("Aprobado")
      } else {
          println("Reprobado")
      }
  }
  ```
</details>
-->

## 2. Cómo usar una sentencia `when` para varias ramas
En Kotlin, la sentencia `when` es una estructura de control muy flexible que se utiliza para manejar múltiples condiciones. Puede usarse como un reemplazo más potente y expresivo de las sentencias `switch` de otros lenguajes.

### Sintaxis básica de `when`
```kotlin
val numero = 3

when (numero) {
    1 -> println("El número es 1")
    2 -> println("El número es 2")
    3 -> println("El número es 3")
    else -> println("El número no es 1, 2, ni 3")
}
```
- `when (numero)` evalúa la variable `numero`.
- Cada rama del `when` compara el valor de `numero` con el valor especificado (`1`, `2`, `3`).
- `else` es una rama opcional que se ejecuta si ninguna de las otras condiciones es verdadera.

### Ejemplo con múltiples valores por rama
Se pueden manejar múltiples valores en una sola rama:
```kotlin
val dia = 3

when (dia) {
    1, 2, 3 -> println("Es el inicio de la semana")
    4, 5 -> println("Es el final de la semana laboral")
    6, 7 -> println("Es el fin de semana")
    else -> println("Día no válido")
}
```

### Ejemplo con rangos
También se pueden usar rangos con la palabra clave `in` en las condiciones de `when`:
```kotlin
val edad = 25

when (edad) {
    in 0..12 -> println("Es un niño")
    in 13..19 -> println("Es un adolescente")
    in 20..64 -> println("Es un adulto")
    else -> println("Es un adulto mayor")
}
```

### Ejemplo con expresiones booleanas
En lugar de valores simples, se pueden usar expresiones booleanas para las ramas:
```kotlin
val numero = 15

when {
    numero % 2 == 0 -> println("El número es par")
    numero % 2 != 0 -> println("El número es impar")
    else -> println("Número desconocido")
}
// Salida: El número es impar
```

### Ejemplo con tipo de dato
`when` también puede evaluar el tipo de dato de una variable usando la palabra clave `is`:
```kotlin
fun describir(objeto: Any) {
    when (objeto) {
        is String -> println("Es una cadena de texto")
        is Int -> println("Es un entero")
        else -> println("Tipo desconocido")
    }
}

describir("Hola")   // Salida: Es una cadena de texto
describir(42)       // Salida: Es un entero
describir(3.14)     // Salida: Tipo desconocido
```

### Minicuestionario

**Pregunta 1:** ¿Cuál será la salida de la siguiente declaración `when`?
```kotlin
val x = 2
when (x) {
    1 -> println("Uno")
    2 -> println("Dos")
    3 -> println("Tres")
    else -> println("Desconocido")
}
```
1. Uno
2. Dos
3. Tres
4. Desconocido
<!--
<details>
  <summary>Respuesta</summary>

  Opción 2: `Dos`. Como `x` es `2`, se ejecuta la rama que compara con `2`.
</details>
-->
**Pregunta 2:** ¿Cuál será la salida de la siguiente declaración `when`?
```kotlin
val color = "azul"
when (color) {
    "rojo" -> println("El color es rojo")
    "azul" -> println("El color es azul")
    "verde" -> println("El color es verde")
    else -> println("Color desconocido")
}
```
1. El color es rojo
2. El color es azul
3. El color es verde
4. Color desconocido
<!--
<details>
  <summary>Respuesta</summary>

  Opción 2: `El color es azul`. `when` también puede comparar valores de tipo `String`, no solo números.
</details>
-->

**Pregunta 3:** Use `when` para completar el siguiente código de modo que cumpla con las siguientes condiciones:
- Si el número es 0, imprima "Cero".
- Si el número está entre 1 y 5, imprima "Número pequeño".
- En cualquier otro caso, imprima "Número grande".

```kotlin
val numero = 4
when (numero) {
    _______________ -> println("Cero")
    in _______________ -> println("Número pequeño")
    else -> println("Número grande")
}
```
<!--
<details>
  <summary>Respuesta</summary>

  ```kotlin
  val numero = 4
  when (numero) {
      0 -> println("Cero")
      in 1..5 -> println("Número pequeño")
      else -> println("Número grande")
  }
  ```

  **Explicación:** La declaración `when` evalúa `numero`. Si el número es 0, imprime "Cero". Si está en el rango de 1 a 5 (con la expresión `in 1..5`), imprime "Número pequeño". Para cualquier otro valor fuera de ese rango, se ejecuta el bloque `else` con el mensaje "Número grande".
</details>
-->
**Pregunta 4:** Use `when` para completar la siguiente función `determinarLongitud`, que debe imprimir un mensaje según la longitud de una cadena de texto:
```kotlin
fun determinarLongitud(texto: String) {
    val longitud: Int = texto.length
    // TODO: crear el cuerpo de la función usando `when`
}

fun main() {
    determinarLongitud("hola")  // Salida: Corto
}
```
Las condiciones del bloque `when` deben ser:
- Si la longitud es 0, imprima "Vacío".
- Si la longitud está entre 1 y 5, imprima "Corto".
- Si la longitud está entre 6 y 10, imprima "Medio".
- Si la longitud es mayor a 10, imprima "Largo".

<!--
<details>
  <summary>Respuesta</summary>

  ```kotlin
  fun determinarLongitud(texto: String) {
      val longitud: Int = texto.length
      when (longitud) {
          0 -> println("Vacío")
          in 1..5 -> println("Corto")
          in 6..10 -> println("Medio")
          else -> println("Largo")
      }
  }

  fun main() {
      determinarLongitud("hola")  // Salida: Corto
  }
  ```

  **Explicación:** la propiedad `length` devuelve la longitud de la cadena `texto`. Si la longitud es 0, imprime "Vacío"; si está en el rango de 1 a 5, imprime "Corto"; entre 6 y 10, imprime "Medio"; y para cualquier valor mayor a 10, se ejecuta el bloque `else`, que imprime "Largo". En este caso, la longitud de "hola" es 4, por lo que se imprime "Corto".
</details>
-->

## 3. Cómo usar `if/else` y `when` como expresiones
En Kotlin, tanto `if/else` como `when` pueden usarse como **expresiones**: pueden devolver un valor y asignarse a una variable. Esto los hace más flexibles y elegantes que en otros lenguajes. La sintaxis es similar a la de las sentencias, pero la última línea del cuerpo de cada rama debe ser el valor que se desea devolver.

### `if/else` como expresión
Al usar `if/else` como expresión, se puede asignar directamente el valor resultante a una variable:
```kotlin
val edad = 18

val estado = if (edad >= 18) {
    "Mayor de edad"
} else {
    "Menor de edad"
}

println(estado)  // Salida: Mayor de edad
```
- El resultado de la expresión `if/else` se asigna a la variable `estado`.
- Si la condición `edad >= 18` es verdadera, se asigna `"Mayor de edad"`; si es falsa, se asigna `"Menor de edad"`.

### `when` como expresión
De manera similar, `when` también puede devolver un valor. Esto es útil cuando hay varias condiciones y se quiere que la expresión devuelva diferentes valores:
```kotlin
val dia = 3

val mensaje = when (dia) {
    1 -> "Lunes"
    2 -> "Martes"
    3 -> "Miércoles"
    4 -> "Jueves"
    5 -> "Viernes"
    6, 7 -> "Fin de semana"
    else -> "Día no válido"
}

println(mensaje)  // Salida: Miércoles
```
- `when` devuelve un valor dependiendo de `dia`, y ese valor se asigna a la variable `mensaje`.

> **Nota:** cuando se usa `when` como expresión (y no cubre todos los casos posibles, como un enum), el compilador exige incluir una rama `else` para garantizar que siempre se devuelva un valor.

## 4. Cómo utilizar los bucles
En Kotlin, al igual que en otros lenguajes, se pueden usar bucles para repetir bloques de código. Los bucles más comunes en Kotlin son `for` y `while`.

A continuación se explica cómo se usa cada uno de ellos con ejemplos.

### Bucle `for`
El bucle `for` en Kotlin se utiliza para recorrer rangos, colecciones (como listas o arreglos) y otras estructuras iterables.

**Ejemplo: iterar sobre un rango de números**
```kotlin
for (i in 1..5) {
    println(i)
}
```
- Esto imprime los números del 1 al 5.
- `1..5` es un rango que incluye tanto el 1 como el 5.

**Comparación con el código en C#:**
```csharp
for (int i = 1; i <= 5; i++)
{
    Console.WriteLine(i);
}
```

**Ejemplo: recorrer una lista**
```kotlin
// listOf() crea una instancia de List (lista de solo lectura)
val frutas = listOf("Manzana", "Banana", "Cereza")

for (fruta in frutas) {
    println(fruta)
}
```
- Este código imprime cada elemento de la lista `frutas`.

**Comparación con el código en C#:**
```csharp
var frutas = new List<string> { "Manzana", "Banana", "Cereza" };

foreach (var fruta in frutas)
{
    Console.WriteLine(fruta);
}
```

### Bucle `while`
El bucle `while` continúa ejecutándose mientras una condición sea verdadera.

**Ejemplo: usar `while`**
```kotlin
var contador = 5

while (contador > 0) {
    println(contador)
    contador--
}
```
- Este bucle imprime los números del 5 al 1, disminuyendo el valor de `contador` en cada iteración.

### Minicuestionario

**Pregunta 1:** ¿Qué imprimiría el siguiente código?
```kotlin
for (i in 6..10) {
    println(i)
}
```
<!--
<details>
  <summary>Respuesta</summary>

  Imprime los números del 6 al 10, uno por línea (6, 7, 8, 9, 10).

  **Explicación:** `6..10` es un rango que incluye tanto el 6 como el 10.
</details>
-->
**Pregunta 2:** ¿Qué imprimiría el siguiente código?
```kotlin
val meses = listOf("Septiembre", "Octubre", "Noviembre", "Diciembre")

for (mes in meses) {
    println(mes)
}
```
<!--
<details>
  <summary>Respuesta</summary>

  Imprime los meses Septiembre, Octubre, Noviembre y Diciembre, uno por línea, en ese orden.
</details>
-->

## 5. Ejercicios opcionales
Para seguir practicando, resuelva los ejercicios de la web oficial de Android:
- [Problemas prácticos: Conceptos básicos de Kotlin — Android Basics with Compose (español)](https://developer.android.com/codelabs/basic-android-kotlin-compose-intro-kotlin-practice-problems?hl=es-419#0)
