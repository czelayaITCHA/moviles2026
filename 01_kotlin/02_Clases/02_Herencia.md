# Herencia en Kotlin

## 1. Herencia
En Kotlin, las clases pueden heredar de otras clases. Las clases hijas pueden utilizar las propiedades y métodos de las clases padres. Sin embargo, por defecto, las clases en Kotlin no se pueden heredar a menos que se marque con la palabra clave `open`.

### **Ejemplo: Herencia de clases**

```kotlin
fun main() {
    // Crear una instancia
    val miPerro = Perro("Firulais")
    miPerro.hacerSonido()  // Salida: Firulais ladra.
}

// Clase padre
open class Animal(val nombre: String) {
    
    open fun hacerSonido() {
        println("$nombre hace un sonido.")
    }
}

// Clase hija
class Perro(nombre: String) : Animal(nombre) {
    
    // Sobrescribir el método de la clase padre
    override fun hacerSonido() {
        println("$nombre ladra.")
    }
}
```
- La palabra clave **`open`** permite que una clase o método sea heredado.
- El paréntesis de cierre del constructor va seguido de un espacio, dos puntos, otro espacio, el nombre de la superclase y un conjunto de paréntesis.
- **`override`** se utiliza para sobrescribir métodos de la clase padre.

## 2. Polimorfismo
El polimorfismo es un concepto de la programación orientada a objetos en el que diferentes tipos de objetos pueden ser manipulados a través de una misma interfaz. Esto se logra mediante herencia y sobrescritura de métodos.

### **Ejemplo: Fundamentos del polimorfismo**

```kotlin
fun main() {
    // Usar polimorfismo
    val vehiculo1: Vehiculo = Bicicleta()
    val vehiculo2: Vehiculo = Coche()

    moverVehiculo(vehiculo1)  // Salida: La bicicleta está pedaleando.
    moverVehiculo(vehiculo2)  // Salida: El coche está conduciendo.
}

open class Vehiculo {
    open fun moverse() {
        println("El vehículo se está moviendo.")
    }
}

class Bicicleta : Vehiculo() {
    override fun moverse() {
        println("La bicicleta está pedaleando.")
    }
}

class Coche : Vehiculo() {
    override fun moverse() {
        println("El coche está conduciendo.")
    }
}

fun moverVehiculo(vehiculo: Vehiculo) {
    vehiculo.moverse()
}
```
- La función `moverVehiculo()` toma un parámetro de tipo `Vehiculo`, lo que permite que tanto `Bicicleta` como `Coche` se utilicen.
- Cada clase sobrescribe el método `moverse()`, y se llama al comportamiento adecuado en función del tipo de objeto.

## 3. Minicuestionario
```kotlin
// Clase Padre
open class Vehiculo(val marca: String, val modelo: String) {
    // Método que será heredado
    open fun conducir() {
        println("Conduciendo un vehículo $marca $modelo.")
    }
}

// Clase Hija que hereda de Vehiculo
class Coche(marca: String, modelo: String, val puertas: Int) : Vehiculo(marca, modelo) {
    // Sobrescribir el método
    override fun conducir() {
        println("Conduciendo un coche $marca $modelo con $puertas puertas.")
    }

    // Método adicional de la clase Coche
    fun tocarClaxon() {
        println("¡Beep Beep!")
    }
}

// Clase Hija que hereda de Vehiculo
class Bicicleta(marca: String, modelo: String, val tipo: String) : Vehiculo(marca, modelo) {
    // Sobrescribir el método
    override fun conducir() {
        println("Montando una bicicleta $tipo marca $marca modelo $modelo.")
    }

    // Método adicional de la clase Bicicleta
    fun pedalear() {
        println("¡Pedaleando!")
    }
}

// Uso
fun main() {
    val coche = Coche("Toyota", "Corolla", 4)
    val bicicleta = Bicicleta("Giant", "Escape", "de montaña")

    coche.conducir()  // Salida: Conduciendo un coche Toyota Corolla con 4 puertas.
    coche.tocarClaxon()  // Salida: ¡Beep Beep!

    bicicleta.conducir()  // Salida: Montando una bicicleta de montaña marca Giant modelo Escape.
    bicicleta.pedalear()  // Salida: ¡Pedaleando!
}
```
1. ¿Qué clase es la clase padre de la clase `Bicicleta`?
1. ¿Qué significa la palabra clave `open`?
1. ¿Qué significa la palabra clave `override`?
1. ¿Qué es el resultado de ejecutar el siguiente código?
   ```kotlin
   fun main() {
      val coche = Coche("Toyota", "Corolla", 4)
      coche.conducir()
   }
   ```
1. ¿Qué es el resultado de ejecutar el siguiente código?
   ```kotlin
   fun main() {
      val bicicleta = Bicicleta("Giant", "Escape", "de montaña")
      bicicleta.conducir()
   }
   ```

## 4. Ejercicios
