# Variables en Kotlin

Aprenda la sintaxis ejecutando código de ejemplo y ejercicios en **Kotlin Playground**.

Kotlin Playground: https://play.kotlinlang.org/

## 1. Declaración de variables
Para definir una nueva variable, comience con la palabra clave `val` o `var` de Kotlin.
- Palabra clave `val`: úsela cuando espere que el valor de la variable no cambie.
- Palabra clave `var`: úsela cuando espere que el valor de la variable pueda cambiar.

Código de ejemplo:
```kotlin
val nombre: String = "Juan"  // Inmutable
var edad: Int = 25           // Mutable
```

Sintaxis:

![image](https://github.com/user-attachments/assets/3287d540-ef9c-471a-92bf-835fcaad40e8)

## 2. Tipos de datos (`Int`, `String`, `Boolean`, etc.)
Los tipos de datos básicos en Kotlin se muestran en la siguiente tabla.

![image](https://github.com/user-attachments/assets/2605e4c1-4a5a-40e4-b65f-34ea95a43647)

### Plantillas de strings
En Kotlin, las plantillas de strings permiten incrustar variables o expresiones directamente dentro de una cadena de texto utilizando el símbolo `$`. Si la expresión es más compleja, se encierra entre llaves `{}`.

```kotlin
val nombre: String = "Juan"
var edad: Int = 25

// $nombre inserta el valor de la variable nombre en la cadena.
// $edad inserta el valor de la variable edad en la cadena.
println("$nombre tiene $edad años.")

// ${edad + 1} evalúa la expresión antes de insertarla.
println("El año que viene $nombre cumplirá ${edad + 1} años.")
```

### Inferencia de tipo
La inferencia de tipo es cuando el compilador de Kotlin puede inferir (o determinar) qué tipo de datos debe tener una variable, sin que el tipo se escriba de manera explícita en el código. Esto significa que se puede omitir el tipo de datos en una declaración de variable si se proporciona un valor inicial para ella.

Pruebe a omitir el tipo de datos en el código de ejemplo para comprobar que no se produce ningún error.
```kotlin
val nombre = "Juan"  // String
var edad = 25        // Int
```

### Convención de codificación
Algunas convenciones de codificación recomendadas por Google para Kotlin:

- Los nombres de las variables deben seguir la convención **camelCase** (primera palabra en minúscula y cada palabra siguiente con inicial mayúscula) y comenzar con una letra minúscula.

  ![image](https://github.com/user-attachments/assets/16f6db52-a936-4ee4-94fa-a615949d9ace)

- Debe haber un espacio antes y después de un operador como la asignación (`=`), la suma (`+`), la resta (`-`), la multiplicación (`*`), la división (`/`) y muchos más.

  ![image](https://github.com/user-attachments/assets/c9a816c4-eb0a-4bf0-9fad-f0db4efd306b)

## 3. Ejercicios: definir variables y constantes, y realizar operaciones

### Ejercicio 1: interpolación de variables en un string
1. Defina una variable inmutable `carrera` de tipo `String` y asígnele la cadena `"Computación"`.
2. Defina una variable mutable `grado` de tipo `Int` y asígnele el número `2`.
3. Usando las variables, muestre `Soy 2do grado de Computación.` en la consola.
<!--
<details>
  <summary>Respuesta</summary>

  ```kotlin
  fun main() {
      val carrera: String = "Computación"  // Inmutable
      var grado: Int = 2                   // Mutable

      // $grado inserta el valor de la variable grado en la cadena (antes de "do grado").
      // $carrera inserta el valor de la variable carrera en la cadena.
      println("Soy ${grado}do grado de $carrera.")
  }
  ```
</details>
-->
### Ejercicio 2: operaciones aritméticas básicas
1. Defina tres variables `a`, `b`, `c` y asígneles cualquier número entero.
2. Luego, use estas variables para realizar los siguientes cálculos y muestre los resultados en la consola:
   - `a + b + c`
   - `a * b - c`
   - `(a + b) / c`
   - `b % a`
<!--
<details>
  <summary>Respuesta</summary>

   ```kotlin
   fun main() {
       // Asignar valores enteros a 3 variables
       val a = 10
       val b = 5
       val c = 2

       // Realizar varias operaciones y mostrar los resultados
       println("a + b + c = ${a + b + c}")     // 10 + 5 + 2 = 17
       println("a * b - c = ${a * b - c}")     // 10 * 5 - 2 = 48
       println("(a + b) / c = ${(a + b) / c}") // (10 + 5) / 2 = 7
       println("b % a = ${b % a}")             // 5 % 10 = 5
   }
   ```

   **Explicación:**
   - Se asignan valores a las variables `a`, `b` y `c`, y se realizan operaciones aritméticas básicas.
   - Como `a`, `b` y `c` son `Int`, la división `(a + b) / c` es división entera: el resultado se trunca (15 / 2 = 7, no 7.5).
   - Los resultados de las operaciones se muestran en la consola utilizando la función `println`.
</details>
-->
### Ejercicio 3: cálculo de la media
1. Defina cinco variables `x1`, `x2`, `x3`, `x4`, `x5` y asígneles cualquier número entero.
2. Calcule la media de estos cinco valores y muestre el resultado en la consola.
<!--
<details>
  <summary>Respuesta</summary>

   ```kotlin
   fun main() {
       // Definir 5 números enteros
       val x1 = 10
       val x2 = 20
       val x3 = 30
       val x4 = 40
       val x5 = 50

       // Calcular el promedio
       val promedio = (x1 + x2 + x3 + x4 + x5) / 5

       // Mostrar el resultado
       println("Promedio: $promedio") // Salida: Promedio: 30
   }
   ```

   **Explicación:**
   - Se calcula el promedio de 5 números enteros.
   - La suma de los números se divide entre 5 y se muestra el resultado.
   - Nota: al ser todos los valores `Int`, si la suma no fuera múltiplo de 5 el resultado también se truncaría (división entera).
</details>
-->
## Referencia
- [Crea y usa variables en Kotlin — Android Basics with Compose (español)](https://developer.android.com/codelabs/basic-android-kotlin-compose-variables?hl=es-419#0)
