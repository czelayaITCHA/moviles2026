# Funciones en Kotlin

## 1. Cómo definir una función y cómo llamarla
Para definir una función, comience con la palabra clave `fun` en Kotlin.

Sintaxis:

![image](https://github.com/user-attachments/assets/0b881cbe-3625-409e-872b-60f99a55f4b4)

Ejecute el siguiente código de ejemplo para ver cómo se definen y llaman las funciones.

```kotlin
fun main() {
    // Llamada a la función
    println(saludoCumpleaños("Juan", 25))
}

// Definición de la función
fun saludoCumpleaños(nombre: String, edad: Int): String {
    val saludoNombre = "¡Feliz cumpleaños, $nombre!"
    val saludoEdad = "¡Ahora tienes $edad años!"
    return "$saludoNombre\n$saludoEdad"
}
```

### El tipo `Unit`
De forma predeterminada, si no se especifica un tipo de dato de retorno, el predeterminado es `Unit`. `Unit` significa que la función no devuelve ningún valor. `Unit` es equivalente al tipo `void` en Java y C#.

Ejemplo de código para una función que no devuelve ningún valor:
```kotlin
fun saludoCumpleaños(): Unit { // El tipo de retorno es Unit
    println("¡Feliz cumpleaños, Rex!")
    println("¡Ahora tienes 5 años!")
}
```

> En la práctica, casi nunca se escribe `Unit` de forma explícita: si una función no tiene tipo de retorno declarado, Kotlin asume `Unit` automáticamente.

## 2. Argumentos con nombre
En Kotlin, puede llamar a una función con varios parámetros pasando los argumentos en un orden diferente al de la definición; por ejemplo, colocando el parámetro `edad` antes del parámetro `nombre`. Cuando se incluye el nombre del parámetro al llamar a una función, esto se denomina **argumento con nombre**.

```kotlin
fun main() {
    // El orden en que se pasan los parámetros es diferente,
    // pero el resultado es el mismo.
    println(saludoCumpleaños(nombre = "Juan", edad = 25))
    println(saludoCumpleaños(edad = 25, nombre = "Juan"))
}

fun saludoCumpleaños(nombre: String, edad: Int): String {
    val saludoNombre = "¡Feliz cumpleaños, $nombre!"
    val saludoEdad = "¡Ahora tienes $edad años!"
    return "$saludoNombre\n$saludoEdad"
}
```

## 3. Argumentos predeterminados
Los parámetros de una función también pueden tener argumentos predeterminados. Al llamar a la función, se puede omitir el argumento para el cual existe un valor predeterminado; en ese caso, se usa el valor predeterminado.

Para agregar un argumento predeterminado, agregue un operador de asignación (`=`) después del tipo de dato del parámetro y asígnele un valor, igual que con una variable.

```kotlin
fun main() {
    // Si se llama a la función sin argumentos, se aplican los valores por defecto.
    println(saludoCumpleaños())
}

// Los valores por defecto se asignan en la definición del parámetro.
fun saludoCumpleaños(nombre: String = "Juan", edad: Int = 25): String {
    val saludoNombre = "¡Feliz cumpleaños, $nombre!"
    val saludoEdad = "¡Ahora tienes $edad años!"
    return "$saludoNombre\n$saludoEdad"
}
```

## 4. Ejercicios: crear programas de cálculo básico utilizando funciones

### Ejercicio 1: crear una función que sume dos números
1. Cree una función que reciba dos números enteros y devuelva su suma. El nombre de la función será `sumar`.
2. Imprima el resultado de ejecutar la función con la siguiente entrada utilizando `println()`.
   - Entrada: `sumar(5, 10)`
   - Salida: `15`

Pista:
- Copie este código para empezar.
   ```kotlin
   fun sumar(a: Int, b: Int): Int {
       // TODO: crear el cuerpo de la función
   }
   fun main() {
       println(sumar(5, 10))  // Salida: 15
   }
   ```
<!--
<details>
  <summary>Respuesta</summary>

   ```kotlin
   fun sumar(a: Int, b: Int): Int {
       return a + b
   }

   fun main() {
       println(sumar(5, 10))  // Salida: 15
   }
   ```

   **Explicación:**
   - La función `sumar` recibe dos enteros y devuelve su suma.
   - `a` y `b` son los parámetros de la función, y el resultado de la suma se devuelve con la palabra clave `return`.
</details>
-->
---

### Ejercicio 2: crear una función que calcule el área de un círculo
1. Cree una función que reciba el radio de un círculo y devuelva su área. El nombre de la función será `calcularAreaCirculo`.
2. Imprima el resultado de ejecutar la función con la siguiente entrada utilizando `println()`.
   - Entrada: `calcularAreaCirculo(5.0)`
   - Salida: `78.53981633974483`

Pista:
- La fórmula para el área de un círculo es `π * radio * radio`. En Kotlin, puede usar `Math.PI` para obtener el valor de π.
- Copie este código para empezar.
   ```kotlin
   fun calcularAreaCirculo(radio: Double): Double {
       // el valor de π
       val pi = Math.PI
       // TODO: la lógica de cálculo
   }

   fun main() {
       println(calcularAreaCirculo(5.0))  // Salida: 78.53981633974483
   }
   ```
<!--
<details>
  <summary>Respuesta</summary>

   ```kotlin
   fun calcularAreaCirculo(radio: Double): Double {
       // el valor de π
       val pi = Math.PI
       return pi * radio * radio
   }

   fun main() {
       println(calcularAreaCirculo(5.0))  // Salida: 78.53981633974483
   }
   ```

   **Explicación:**
   - La función `calcularAreaCirculo` toma el radio de un círculo como argumento y devuelve el área.
   - Se utiliza la fórmula `π * radio²` para calcular el área, donde la variable `pi` proporciona el valor de π.
</details>
-->
---

### Ejercicio 3: crear una función que determine si un número es par o impar
1. Cree una función que reciba un número entero y devuelva un valor booleano: `true` si el entero es par, `false` si es impar. El nombre de la función será `esPar`.
2. Imprima el resultado de ejecutar la función con la siguiente entrada utilizando `println()`.
   - Entrada: `esPar(7)`
   - Salida: `false`

<!--
<details>
  <summary>Respuesta</summary>

   ```kotlin
   fun esPar(numero: Int): Boolean {
       return numero % 2 == 0
   }

   fun main() {
       println(esPar(7))  // Salida: false
   }
   ```

   **Explicación:**
   - La función `esPar` determina si un número es par o impar.
   - Si el resto de la división del número entre 2 es 0, entonces es par; de lo contrario, es impar.
</details>
-->
## Referencia
- [Cómo crear y usar funciones en Kotlin — Android Basics with Compose (español)](https://developer.android.com/codelabs/basic-android-kotlin-compose-functions?hl=es-419#0)
