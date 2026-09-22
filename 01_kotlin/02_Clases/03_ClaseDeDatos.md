## Clases de Datos

## 1. Introducción a las Clases de Datos

Las **clases de datos** en Kotlin son clases que se utilizan principalmente para almacenar datos. Su propósito principal es contener **información** y no tener comportamientos complejos. Kotlin proporciona una forma concisa y eficiente de crear este tipo de clases con la palabra clave `data`.

Cuando defines una clase de datos, Kotlin genera automáticamente varios métodos útiles para ti, como:
- `equals()`: Para comparar objetos por sus valores.
- `hashCode()`: Para generar un código hash basado en los valores.
- `toString()`: Para proporcionar una representación en cadena del objeto.
- `copy()`: Para crear una copia del objeto con la opción de modificar algunas propiedades.

## 2. Sintaxis de una Clase de Datos

```kotlin
data class Persona(val nombre: String, val edad: Int)
```
- `data class Persona` define una clase de datos.
- `val nombre` y `val edad` son las **propiedades** de la clase `Persona`.

## 3. Ejemplos de Clases de Datos

**Ejemplo 1: Creación y uso básico**
```kotlin
data class Persona(val nombre: String, val edad: Int)

fun main() {
    val persona1 = Persona("Ana", 25)
    val persona2 = Persona("Ana", 25)

    println(persona1.toString())
    println(persona1)  // Salida: Persona(nombre=Ana, edad=25)

    // Comparación basada en los valores
    println(persona1.equals(persona2))
    println(persona1 == persona2)  // Salida: true
}
```
- Aquí hemos creado una clase de datos `Persona` con dos propiedades: `nombre` y `edad`.
- Cuando creamos dos instancias (`persona1` y `persona2`) con los mismos valores, la clase de datos genera automáticamente un método `equals()`, lo que permite que se comparen por **valores** y no por la referencia de los objetos. Por eso, `persona1 == persona2` es `true`.
- Además, gracias al método `toString()` generado automáticamente, cuando imprimimos `persona1`, obtenemos una representación clara de sus valores.

**Ejemplo 2: Uso del método `copy()`**
```kotlin
data class Persona(val nombre: String, val edad: Int)

fun main() {
    val persona1 = Persona("Ana", 25)
    
    // Crear una copia de persona1, pero con una edad diferente
    val persona2 = persona1.copy(edad = 30)

    println(persona2)  // Salida: Persona(nombre=Ana, edad=30)
}
```
- El método `copy()` permite hacer una **copia** exacta de un objeto, pero con la opción de modificar algunas propiedades.
- En este caso, se crea una copia de `persona1`, pero cambiamos la propiedad `edad` de `25` a `30`, manteniendo el mismo `nombre`.
- Este es un método útil cuando trabajas con objetos **inmutables** y necesitas crear variantes de un objeto sin modificar el original.

**Ejemplo 3: Descomposición de una Clase de Datos**
```kotlin
data class Persona(val nombre: String, val edad: Int)

fun main() {
    val persona = Persona("Carlos", 32)

    // Descomposición en sus propiedades
    val (nombre, edad) = persona

    println("Nombre: $nombre, Edad: $edad")  // Salida: Nombre: Carlos, Edad: 32
}
```
- En este ejemplo, estamos utilizando una característica de Kotlin conocida como **descomposición**.
- Con las clases de datos, puedes extraer las propiedades directamente en variables separadas. Aquí, las propiedades `nombre` y `edad` se extraen de la instancia `persona` y se asignan a dos variables (`nombre` y `edad`).
- Esto es muy útil para evitar escribir código redundante al acceder a las propiedades de un objeto y hace que el código sea más legible.

## 4. Minicuestionario

1. **¿Qué métodos se generan automáticamente en la siguiente clase `Persona`?**

```kotlin
data class Persona(val nombre: String, val edad: Int)
```

A) `equals`, `hashCode`, `toString`, `copy`  
B) `equals`, `toString`, `finalize`  
C) `equals`, `toString`, `hashCode`, `finalize`  
D) `equals`, `hashCode`, `copy`, `destroy`

<!--
<details>
  <summary>Respuesta</summary>
  
  **A) `equals`, `hashCode`, `toString`, `copy`**

  **Explicación:**
  - Las clases de datos en Kotlin generan automáticamente varios métodos útiles, como `equals` para comparar objetos, `hashCode` para generar un valor hash, `toString` para representar el objeto como cadena, y `copy` para crear copias con modificaciones.
</details>
-->
---

2. **¿Cuál es el resultado de `persona2` en el siguiente código utilizando el método `copy`?**

```kotlin
data class Persona(val nombre: String, val edad: Int)

fun main() {
    val persona1 = Persona("Ana", 25)
    val persona2 = persona1.copy(edad = 30)
    println(persona2)
}
```

A) `Persona(nombre=Ana, edad=30)`  
B) `Persona(nombre=Ana, edad=25)`  
C) `Persona(nombre=Maria, edad=30)`  
D) Error de compilación
<!--
<details>
  <summary>Respuesta</summary>
  
  **A) `Persona(nombre=Ana, edad=30)`**

  **Explicación:**
  - El método `copy` permite crear una copia del objeto, cambiando solo los valores especificados. En este caso, se copia `persona1` y solo se modifica el valor de `edad` a 30.
</details>
-->
---

3. **¿Cuál es el resultado del siguiente código?**

```kotlin
data class Producto(val nombre: String, val precio: Double)

fun main() {
    val producto1 = Producto("Manzana", 1.5)
    val producto2 = Producto("Manzana", 1.5)
    println(producto1 == producto2)
}
```

A) `true`  
B) `false`  
C) Error de compilación  
D) Error en tiempo de ejecución
<!--
<details>
  <summary>Respuesta</summary>
  
  **A) `true`**

  **Explicación:**
  - El método `equals` generado automáticamente por las clases de datos compara todas las propiedades. Como ambos objetos tienen las mismas propiedades (`nombre` y `precio`), se consideran iguales.
</details>
-->
---

4. **¿Qué imprime el método `toString` en el siguiente código?**

```kotlin
data class Libro(val titulo: String, val autor: String)

fun main() {
    val libro = Libro("1984", "George Orwell")
    println(libro.toString())
}
```

A) `Libro@1984`  
B) `Libro(titulo=1984, autor=George Orwell)`  
C) `Libro("1984", "George Orwell")`  
D) `Libro{1984, George Orwell}`
<!--
<details>
  <summary>Respuesta</summary>
  
  **B) `Libro(titulo=1984, autor=George Orwell)`**

  **Explicación:**
  - El método `toString` en una clase de datos genera automáticamente una representación de cadena del objeto, mostrando el nombre de la clase y sus propiedades con sus valores.
</details>
-->
---

5. **¿Cuál es el resultado del uso del método `copy` en el siguiente código?**

```kotlin
data class Carro(val marca: String, val modelo: String)

fun main() {
    val carro1 = Carro("Honda", "Civic")
    val carro2 = carro1.copy(modelo = "Accord")
    println(carro2)
}
```

A) `Carro(marca=Honda, modelo=Civic)`  
B) `Carro(marca=Honda, modelo=Accord)`  
C) Error de compilación  
D) `Carro(marca=Accord, modelo=Accord)`
<!--
<details>
  <summary>Respuesta</summary>
  
  **B) `Carro(marca=Honda, modelo=Accord)`**

  **Explicación:**
  - Al utilizar el método `copy`, se crea una copia del objeto `carro1` con el valor del `modelo` modificado a "Accord", manteniendo el valor de `marca` como "Honda".
</details>
-->
---

6. **¿Cuál es el resultado del siguiente código?**

```kotlin
data class Ciudad(val nombre: String, val pais: String)

fun main() {
    val ciudad1 = Ciudad("Tokio", "Japón")
    val ciudad2 = Ciudad("Tokio", "Japón")
    println(ciudad1 == ciudad2)
}
```

A) `true`  
B) `false`  
C) Error de compilación  
D) Error en tiempo de ejecución
<!--
<details>
  <summary>Respuesta</summary>
  
  **A) `true`**

  **Explicación:**
  - El método `equals` compara las propiedades de los objetos. Como `ciudad1` y `ciudad2` tienen las mismas propiedades (`nombre` y `pais`), se consideran iguales, por lo que `equals` devuelve `true`.
</details>
-->
