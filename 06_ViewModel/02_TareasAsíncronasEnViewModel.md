# Tarea Asincrona En ViewModel

## Kotlin Coroutines

**Kotlin Coroutines** es una librería oficial de Kotlin para manejar tareas asíncronas.

### Características principales:
1. **Escritura sencilla de procesamiento asíncrono**: Permite escribir código asíncrono de manera secuencial, sin necesidad de callbacks o manejo directo de hilos.
2. **Fácil manejo de tareas asíncronas**: Usando funciones `suspend`, puedes escribir tareas asíncronas como funciones normales.

### Elementos básicos:
- **Funciones `suspend`**: Funciones que realizan tareas asíncronas.
- **`launch`**: Método para iniciar coroutines(tareas asíncronas).

### Ejemplo:

La función de `delay`, que suspende el procesamiento durante un número de segundos dado como argumento, se utiliza para simular un procesamiento que requiere mucho tiempo.
<br>El proceso se interrumpe durante 1 segundo por `delay`, por lo que 3 sale antes que 2.

https://pl.kotl.in/aedeR4JQu

```kotlin
import kotlinx.coroutines.*

fun main() {
    runBlocking {
        launch {
            println("1")
            delay(1000L)
            println("2")
        }
        launch {
            println("3")
        }
    }
}
```

### 1. **Comparación con el procesamiento asíncrono en C# (`async`/`await`)**

En C#, el procesamiento asíncrono se maneja utilizando las palabras clave `async` y `await`. Esta estructura es muy similar a las funciones `launch` y `suspend` en Kotlin Coroutines, ya que ambas permiten escribir operaciones asíncronas de manera secuencial y fácil de leer.

#### Ejemplo en C#:
```csharp
public async Task FetchDataAsync()
{
    var result = await GetDataFromNetworkAsync();
    Console.WriteLine(result);
}

public async Task<string> GetDataFromNetworkAsync()
{
    await Task.Delay(1000);  // Operación asíncrona
    return "Data fetched";
}
```

#### Ejemplo en Kotlin Coroutines:
```kotlin
suspend fun fetchData() {
    val result = getDataFromNetwork()
    println(result)
}

suspend fun getDataFromNetwork(): String {
    delay(1000)  // Operación asíncrona
    return "Data fetched"
}
```

**Puntos de correspondencia**:
- En C#, `async` y `await` son muy similares a las funciones `suspend` en Kotlin, ya que ambas permiten escribir operaciones asíncronas de manera secuencial.
- Ambas estructuras permiten esperar una operación asíncrona de manera intuitiva.

## Pasos para realizar tareas asíncronas en ViewModel

### Preparación previa: Crear `ViewModel` de ejemplo sin procesamiento asíncrono.

- Primero, necesitas agregar la biblioteca para utilizar `ViewModel`. Añade lo siguiente a las dependencias en tu archivo `build.gradle.kts` (:app).

```diff
dependencies {
    ...

+    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.11.0")
}
```

- Crea una clase Kotlin `CargarDatoViewModel` y un archivo Kotlin `CargarDatoScreen` y pega lo siguiente.
- Llama a `CargarDatoScreen` en `MainActivity`.
```
class CargarDatoViewModel : ViewModel() {

    var mensaje by mutableStateOf("Estado inicial")
        private set

    fun obtenerDatos() {
        mensaje = "Cargando..."
        val resultado = cargarDatos()
        mensaje = resultado
    }

    fun cargarDatos(): String {
        // Se detiene durante 2 segundos
        // para representar el tiempo de espera de una conexión a Internet.
//        delay(2000)
        return "Datos cargados"
    }
}
```
```
@Composable
fun CargarDatoScreen(viewModel: CargarDatoViewModel = viewModel()) {
    val mensaje = viewModel.mensaje

    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(text = mensaje, fontSize = 24.sp)
        Button(onClick = { viewModel.obtenerDatos() }) {
            Text("Obtener datos")
        }
    }
}
```
```diff
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            EjemploAsincrono2Theme {
+                CargarDatoScreen()
            }
        }
    }
}
```

### ✅ Paso 1: Configurar el entorno de Coroutines en `ViewModel`

Añade las importaciones necesarias.

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.launch

class CargarDatoViewModel : ViewModel() {
...
}
```



### ✅ Paso 2: Hacer las funciones de procesamiento asincrónico `suspend`

Funciones como `delay()` o las que involucran solicitudes de red deben ser declaradas como `suspend` para que puedan ejecutarse de manera asincrónica.

```kotlin
// Definir suspend para funciones de procesamiento asíncrono.
suspend fun cargarDatos(): String {
    // Se detiene durante 2 segundos
    // para representar el tiempo de espera de una conexión a Internet.
    delay(2000)
    return "Datos cargados"
}
```



### ✅ Paso 3: Llamar la función `suspend` en `launch`

Como las funciones `suspend` no se pueden ejecutar directamente en un hilo normal, usaremos `viewModelScope.launch {}` para ejecutarlas dentro del `ViewModel`.

```kotlin
fun obtenerDatos() {
    // La función　con `suspend` se puede ejecutar dentro del bloque `viewModelScope.launch`
    viewModelScope.launch {
        _mensaje.value = "Cargando..."
        val resultado = cargarDatos()
        _mensaje.value = resultado
    }
}
```



### ✅ Paso 4: Notificar el estado a la UI con `StateFlow`

Usamos `MutableStateFlow` para manejar el estado (como el mensaje que se muestra en la UI), y lo exponemos como `StateFlow` para que sea solo de lectura.

```kotlin
private val _mensaje = MutableStateFlow("Estado inicial") // Variable privada modificable
val mensaje: StateFlow<String> = _mensaje // Variable de publicación de sólo lectura

fun obtenerDatos() {
...
}
```




<details>
  <summary>📦Código completo del `ViewModel`</summary>
    
```kotlin
class CargarDatoViewModel : ViewModel() {

    private val _mensaje = MutableStateFlow("Estado inicial") // Variable privada modificable
    val mensaje: StateFlow<String> = _mensaje // Variable de publicación de sólo lectura

    fun obtenerDatos() {
        // La función　con `suspend` se puede ejecutar dentro del bloque `viewModelScope.launch`
        viewModelScope.launch {
            _mensaje.value = "Cargando..."
            val resultado = cargarDatos()
            _mensaje.value = resultado
        }
    }

    // Definir suspend para funciones de procesamiento asíncrono.
    suspend fun cargarDatos(): String {
        // Se detiene durante 2 segundos
        // para representar el tiempo de espera de una conexión a Internet.
        delay(2000)
        return "Datos cargados"
    }
}
```

</details>


## 💡 Uso en la UI (Jetpack Compose)

```kotlin
@Composable
fun CargarDatoScreen(viewModel: CargarDatoViewModel = viewModel()) {
    // collectAsState()` convierte el valor de `StateFlow` en un `State`.
    // Cada vez que cambia el valor de `StateFlow`, se refleja en `State`.
    val mensaje by viewModel.mensaje.collectAsState()

    Column(
        modifier = Modifier.fillMaxSize(),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(text = mensaje, fontSize = 24.sp)
        Button(onClick = { viewModel.obtenerDatos() }) {
            Text("Obtener datos")
        }
    }
}
```



## ✅ Resumen

| Elemento                        | Descripción |
|-|-|
| `viewModelScope.launch`          | Lanza una corutina (dentro del ciclo de vida del `ViewModel`) |
| Función `suspend`                | Define tareas asincrónicas, puede usar `delay()` o procesos que no bloquean el hilo principal |
| `withContext(Dispatchers.IO)`    | Ejecuta tareas en un hilo de entrada/salida (para redes, bases de datos) |
| `MutableStateFlow` / `StateFlow` | Gestiona el estado de manera reactiva, manteniendo los datos actualizados para la UI |
| `collectAsState()`               | Suscribe la UI a un `Flow` para observar cambios en los datos |
