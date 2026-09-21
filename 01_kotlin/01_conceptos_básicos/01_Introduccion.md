# Introducción a Kotlin para el desarrollo de aplicaciones Android

## Objetivos
Al finalizar esta unidad, el estudiante será capaz de:
- Explicar qué es Kotlin y por qué es el lenguaje recomendado para el desarrollo Android.
- Comparar la sintaxis básica de Kotlin con la de C#.
- Ubicar a Kotlin dentro del ecosistema Android actual (Jetpack Compose, Kotlin Multiplatform).

## ¿Qué es Kotlin?
¿Sabe qué es Kotlin? Descúbralo a continuación y coméntelo con sus compañeros.

## 1. Características y ventajas de Kotlin

### 1.1 Un lenguaje de programación moderno y fácil de usar
Kotlin es un lenguaje de programación moderno con una sintaxis concisa y fácil de entender. Esto permite crear rápidamente programas que realmente funcionan sin tener que escribir código complejo. Es posible ver resultados inmediatos a medida que se aprende, lo cual hace que el aprendizaje sea más ameno.

### 1.2 El lenguaje oficial de desarrollo de Android
Kotlin ha sido adoptado por Google como el lenguaje oficial para el desarrollo de aplicaciones Android. Hoy en día, Jetpack Compose —el kit de herramientas moderno de Android para construir interfaces de usuario, escrito enteramente en Kotlin— es el estándar recomendado para nuevos proyectos. Por eso, al aprender Kotlin, el estudiante adquiere habilidades directamente aplicables al desarrollo real de aplicaciones.

### 1.3 Un lenguaje popular y útil para la carrera profesional
Kotlin sigue creciendo en popularidad y está especialmente demandado en el sector del desarrollo de aplicaciones móviles. Las empresas también buscan ingenieros con conocimientos de Kotlin, lo que representa un gran activo para la futura carrera profesional del estudiante.

### 1.4 Buena comunidad y soporte
Kotlin tiene una comunidad activa, y hay muchos recursos y soporte disponibles en caso de surgir alguna pregunta o problema. Si se presenta alguna dificultad durante el aprendizaje, es fácil encontrar una solución, lo cual hace menos probable que el estudiante se quede atrás.

## 2. Kotlin en el desarrollo Android actual
El ecosistema ha madurado bastante desde las primeras versiones de este material. Algunos puntos que vale la pena destacar en clase:

- **Jetpack Compose** ya no es "la alternativa nueva": es el enfoque estándar para construir interfaces en Android, y ha ido reemplazando gradualmente al sistema de vistas basado en XML.
- El compilador de Compose ahora se desarrolla y distribuye junto con el propio compilador de Kotlin (K2), lo que eliminó muchos de los antiguos problemas de compatibilidad de versiones entre Kotlin y Compose.
- **Kotlin Multiplatform (KMP)** permite compartir lógica de negocio entre Android, iOS y otras plataformas, lo cual amplía el alcance profesional de aprender Kotlin más allá del desarrollo Android puro.

## 3. Comparación de sintaxis: Kotlin y C#
La sintaxis de Kotlin y C# es, en términos generales, similar. A continuación se presentan algunos ejemplos comparativos.

### 3.1 Variables

| | Kotlin | C# |
|---|---|---|
| Inmutable | `val` | No existe una palabra clave equivalente para variables locales; simplemente no se reasigna el valor |
| Mutable | `var` | `var` (o el tipo explícito) |

> **Nota:** en C#, `const` no es un equivalente directo de `val`, ya que se reserva para constantes de tiempo de compilación. Para campos de instancia de solo lectura se usa `readonly`.

**Kotlin**
```kotlin
val nombre: String = "Juan"  // Inmutable
var edad: Int = 25           // Mutable
```

**C#**
```csharp
string nombre = "Juan";  // Se trata como inmutable: no se reasigna
int edad = 25;           // Mutable
```

### 3.2 Funciones
Ambos lenguajes definen funciones de manera similar, aunque la sintaxis varía.

**Kotlin**
```kotlin
fun saludar(nombre: String): String {
    return "Hola, $nombre"
}
```

**C#**
```csharp
string Saludar(string nombre) {
    return $"Hola, {nombre}";
}
```

### 3.3 Clases
Ambos lenguajes usan una sintaxis similar para definir clases, aunque con ligeras diferencias.

**Kotlin**
```kotlin
class Persona(val nombre: String, var edad: Int)
```

**C#**
```csharp
class Persona {
    public string Nombre { get; }
    public int Edad { get; set; }

    public Persona(string nombre, int edad) {
        Nombre = nombre;
        Edad = edad;
    }
}
```

### 3.4 Herencia de clases
Ambos lenguajes permiten herencia, pero en Kotlin las clases son `final` por defecto: se debe usar `open` para permitir que una clase sea heredada.

**Kotlin**
```kotlin
open class Animal
class Perro: Animal()
```

**C#**
```csharp
class Animal {}
class Perro : Animal {}
```

### 3.5 Null safety (seguridad ante valores nulos)
Kotlin cuenta con un sistema de seguridad ante `null` que obliga a manejar explícitamente los posibles valores nulos. C# también ofrece tipos anulables (`nullable`) mediante `?`.

**Kotlin**
```kotlin
val nombre: String? = null  // Tipo nullable
```

**C#**
```csharp
string? nombre = null;  // Tipo nullable
```

### 3.6 Bucles for
Los bucles `for` son muy similares en ambos lenguajes.

**Kotlin**
```kotlin
for (i in 1..10) {
    println(i)
}
```

**C#**
```csharp
for (int i = 1; i <= 10; i++) {
    Console.WriteLine(i);
}
```

## Referencia
- [Introducción a la programación en Kotlin — Android Basics with Compose (español)](https://developer.android.com/courses/pathways/android-basics-compose-unit-1-pathway-1?hl=es-419)
