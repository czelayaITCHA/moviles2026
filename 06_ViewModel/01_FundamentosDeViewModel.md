# Fundamentos de ViewModel

## Concepto básico

El `ViewModel` es una Clase que mantiene los datos y la lógica de la interfaz de usuario (UI). Su propósito es asegurarse de que los datos no se pierdan aunque la UI se reconstruya, promoviendo la separación entre la UI y los datos.
Dentro de una función componible, utiliza una instancia de `ViewModel` para acceder a los datos y a la lógica necesarios.

### Ventajas de ViewModel
- **Mantener datos durante la rotación o redibujado**:
  <br>Las pantallas de Android pueden destruirse y regenerarse durante eventos como la rotación del dispositivo o cambios en la configuración, pero el `ViewModel` no se ve afectado por estos eventos del ciclo de vida. Mientras que las variables o estados normales se restablecen cuando la pantalla se regenera, `ViewModel` mantiene los datos, por lo que, por ejemplo, los datos en un formulario de entrada no se pierden aunque se rote la pantalla durante la escritura.
- **Lógica reutilizable**:
  <br>Mantener el código de la interfaz de usuario simple y reutilizable consolidando la lógica de negocio y la recuperación de datos en el `ViewModel`.
- **Compartir datos entre múltiples pantallas**:
  <br>Con `ViewModel`, los datos se pueden reutilizar en varias pantallas o componentes. Por ejemplo, varias pantallas que muestran la misma información del usuario o configuraciones pueden utilizar un `ViewModel` común. Esto evita la necesidad de obtener y gestionar los datos en cada pantalla por separado.
- **Manejo seguro de tareas asíncronas**:
  <br>El `ViewModel` es útil para manejar procesos asíncronos (como el acceso a la base de datos o solicitudes de red). No depende del ciclo de vida de Android, por lo que puede continuar procesando de manera segura incluso si la pantalla se destruye o regenera. En combinación con `Coroutine` o `Flow`, se pueden gestionar tareas asíncronas de manera eficiente.

## Pasos para implementar
Vamos a probar el flujo básico de implementación de ViewModel mirando un poco de código de ejemplo.

### 0. Crea un nuevo proyecto llamado `EjemploViewModel`.
Crea un nuevo proyecto. Una vez compilado, elimina la función por defecto `Greeting` y su punto de llamada en la `MainActivity`.

### 1. **Agregar dependencias de ViewModel al proyecto**  
Primero, necesitas agregar la biblioteca para utilizar `ViewModel`. Añade lo siguiente a las dependencias en tu archivo `build.gradle.kts` (:app).

```diff
dependencies {
    ...

+    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.11.0")
}
```
### 2. **Crear la estructura de paquetes**  
Crea la siguiente estructura de paquetes para organizar el código

<img width="357" height="281" alt="image" src="https://github.com/user-attachments/assets/6074cebf-762c-4bf9-b0e1-4c50b94e3d6b" />

### 3. **Crear la clase ViewModel**  
A continuación, crea una clase `ViewModel` en el package **viewmodel** para mantener los datos necesarios. Consideremos el ejemplo de una aplicación de contador.

```kotlin
// Clase ViewModel
class ContadorViewModel : ViewModel() {
    //Mantener el estado del contador
    var contador by mutableStateOf(0)
        private set
    
    //Función para incrementar el contador
    fun incrementarContador(){
        contador++
    }
}
```
#### Explicación:
- Se crea la clase `ViewModel` y se define la variable entera `contador` utilizando `mutableStateOf`. Esto permite que Compose redibuje la interfaz automáticamente.
- La función `incrementarContador()` incrementa el valor de `contador`.

### 4. **Usar ViewModel en una función Composable**  
Ahora, utiliza `ViewModel` dentro de una función `Composable`, en el package **components**. Nombra al archivo **ContadorView**. Se puede obtener una instancia del `ViewModel` usando la función `viewModel()`.
```kotlin
// Utilizar ViewModel dentro de una función Composable.
// Se puede obtener una instancia del ViewModel usando la función viewModel()
@Composable
fun ContadorView(viewModel: ContadorViewModel = viewModel()){
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) { 
        //Mostramos el valor del contador
        Text(text = "Contador: ${viewModel.contador}", fontSize = 32.sp)
        Spacer(modifier = Modifier.height(16.dp))
        
        //Boton para incrementar el contador
        Button(onClick = {viewModel.incrementarContador()}) { 
            Text(text="Incrementar Contador")
        }
        
    }
}
```
#### Explicación:
- Se usa `viewModel()` para obtener una instancia de `ContadorViewModel`.
- La función `Text` muestra el valor de `viewModel.contador` y el botón llama a `incrementarContador()` para incrementar el contador.
- Al usar la función `viewModel()` en Compose, el estado se mantiene incluso cuando se producen eventos como la rotación de la pantalla o el redibujado.

### 5. **Mostrar la función Composable que usa ViewModel en la aplicación**  
Finalmente, incluye la función `ContadorView` en la actividad o en otras funciones `Composable` para mostrarla en la aplicación.
```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            EjemploViewModelTheme {
                ContadorView()
            }
        }
    }
}
```
#### Explicación:
- Dentro de la clase `MainActivity`, se utiliza `setContent` para mostrar `ContadorView`.
- Argumento por defecto `ContadorView`, establecido para asignar el valor de retorno del método `viewModel()`. Por lo tanto, el constructor puede ser llamado sin argumentos.

### Resumen

- **Creación de ViewModel**: Se crea una clase `ViewModel` y se definen las variables de estado y funciones para operar sobre ellas.
- **Uso de ViewModel en Composable**: Se utiliza la función `viewModel()` para aprovechar el `ViewModel` dentro de las funciones `Composable`.
- **Retención de estado**: Usando `ViewModel`, el estado se mantiene incluso durante eventos del ciclo de vida como la rotación de la pantalla o redibujos.

## Comparación con código sin ViewModel
En los códigos siguientes, el valor del contador se pone a cero al girar la pantalla.
```kotlin
@Composable
fun PantallaContadorSinViewModel() {
    // Definir estado en lugar de ViewModel
    var contador by remember { mutableStateOf(0) }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        // Mostrar el valor del contador
        Text(text = "Contador: ${contador}", fontSize = 32.sp)
        Spacer(modifier = Modifier.height(16.dp))

        // Botón para incrementar el contador
        Button(onClick = { contador++ }) {
            Text(text = "Incrementar Contador")
        }
    }
}
```

## Ejercicios
### Ejercicio 1
- Debajo del botón para incrementar el contador, agregue un botón para disminuir el contador.
- En el `ContadorViewModel`, agregue también el método correspondiente.
<!--
<details>
  <summary>Ejemplo de solución</summary>

  ```kotlin
  // Botón para decrementar el contador
  Button(onClick = { viewModel.decrementarContador() }) {
      Text(text = "Decrementar Contador")
  }

  // Función para decrementar el contador
  fun decrementarContador() {
      contador--
  }
  ```
</details>
-->
### Ejercicio 2
- Debajo del botón del ejercicio 1, agregue un botón para reiniciar el contador.
- En el `ContadorViewModel`, agregue también el método correspondiente.

<!--
<details>
  <summary>Ejemplo de solución</summary>

  ```kotlin
  // Botón para reiniciar el contador
  Button(onClick = { viewModel.reiniciarContador() }) {
      Text(text = "Reiniciar Contador")
  }

  // Función para reiniciar el contador
  fun reiniciarContador() {
      contador = 0
  }
  ```
</details>
-->
