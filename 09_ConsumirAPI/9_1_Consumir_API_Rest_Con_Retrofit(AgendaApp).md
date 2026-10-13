# 9\_1 — Consumo de API REST con Retrofit: AgendaApp

Siguiente tema del módulo (carpeta `09_ConsumirAPI`). Se aplica a la misma **AgendaApp** (paquete `com.example.agendaapp`) y se apoya en lo anterior: Room (`7_1`), DataStore (`7_2`), menús (`5_1`) y Hilt (`8_1`). Como en las guías anteriores, cada paso indica si hay que **CREAR** un archivo nuevo o **MODIFICAR** uno que ya existe.

**Qué se va a construir:** una opción **"Importar contactos de ejemplo"** en el menú de tres puntos de la lista. Al tocarla, la app pide por Internet unos usuarios ficticios (nombre, teléfono, correo y foto), los convierte en `Contacto` y los guarda en Room. Como la lista ya está conectada a Room mediante un `Flow`, los contactos nuevos **aparecen solos**.

## 1. Objetivos

- Comprender cómo se comunica una app con un servidor mediante una API REST y JSON.
- Definir una API con Retrofit y configurarla con Hilt.
- Separar el modelo que llega de Internet (DTO) del modelo que se guarda (entidad de Room).
- Manejar los tres estados de una llamada de red (cargando, éxito, error) y mostrarlos en la pantalla.

---

## 2. Conceptos previos

### 2.1 ¿Qué es una API REST?

Una **API** es una forma acordada de solicitar datos a un programa que está en otra computadora. Una API **REST** lo hace usando **HTTP**, el mismo protocolo de las páginas web: la app (el *cliente*) envía una **petición** a una url (un *endpoint*) y el servidor devuelve una **respuesta**, casi siempre en formato JSON.

Toda petición lleva un **método HTTP** que dice qué se quiere hacer:

| Método | Para qué se usa | Ejemplo en una agenda |
| --- | --- | --- |
| `GET` | Leer datos | Obtener la lista de contactos |
| `POST` | Crear un dato nuevo | Registrar un contacto en el servidor |
| `PUT` / `PATCH` | Modificar un dato existente | Cambiar el teléfono de un contacto |
| `DELETE` | Borrar un dato | Eliminar un contacto |

En esta guía solo se usa `GET`.

La respuesta trae un **código de estado** que indica cómo salió la petición:

- **200 a 299:** salió bien (`200 OK`).
- **400 a 499:** el error es de quien pidió (`404`: esa dirección no existe; `429`: se hicieron demasiadas peticiones seguidas).
- **500 a 599:** el error es del servidor.

### 2.2 JSON

JSON es el formato de texto en que casi todas las APIs devuelven los datos. Tiene solo tres piezas:

- **Objeto** `{ ... }`: un conjunto de pares *clave: valor*.
- **Lista** `[ ... ]`: varios elementos seguidos.
- **Valores simples:** texto entre comillas, números, `true`/`false` y `null`.

```json
{
  "nombre": "Laura",
  "telefono": "7000-1234",
  "favorito": false,
  "etiquetas": ["trabajo", "familia"]
}
```

La app no trabaja con ese texto directamente: una librería lo convierte en objetos de Kotlin (una `data class`), y de regreso.

### 2.3 Qué hace cada librería

| Librería | Qué hace en la AgendaApp |
| --- | --- |
| **Retrofit** | Permite describir la API como una **interfaz de Kotlin** con anotaciones (`@GET`, `@Query`). Retrofit arma y ejecuta las peticiones por usted |
| **OkHttp** | Es el cliente HTTP que realmente envía la petición (Retrofit lo usa por debajo). Permite agregar *interceptores*, por ejemplo uno que escribe cada petición en Logcat |
| **Gson** (con `converter-gson`) | Convierte el JSON recibido en objetos de Kotlin |
| **Coil** (`coil-network-okhttp`) | Descarga y muestra las fotos que vienen como una URL |
| **Hilt** | Entrega ya configurados el `Retrofit` y la API a quien los necesite (`NetworkModule`) |

### 2.4 La API que se usa: randomuser.me

[randomuser.me](https://randomuser.me/documentation) es una API gratuita, **sin clave**, que genera usuarios **ficticios**. Una petición de ejemplo:

```
GET
?results=2&inc=name,email,phone,picture&nat=us,es,mx
```

- `results`: cuántos usuarios se piden.
- `inc`: solo incluir esos campos. Si se omite, la API devuelve muchos más (dirección, usuario, contraseña, etc.) que la app no necesita.
- `nat`: nacionalidades de los usuarios. Afecta nombres y teléfonos; las fotos no cambian.

La respuesta, resumida:

```json
{
  "results": [
    {
      "name": { "title": "Ms", "first": "Laura", "last": "Mendez" },
      "email": "laura.mendez@example.com",
      "phone": "(272) 790-0888",
      "picture": {
        "large": "https://randomuser.me/api/portraits/women/75.jpg",
        "medium": "https://randomuser.me/api/portraits/med/women/75.jpg",
        "thumbnail": "https://randomuser.me/api/portraits/thumb/women/75.jpg"
      }
    }
  ]
}
```

Los datos son **ficticios**: no llame ni escriba a esos teléfonos o correos.

### 2.5 Dos modelos distintos: DTO y entidad

En la app habrá **dos clases** que describen a una persona, y es a propósito:

|  | **DTO** (`UsuarioDto`) | **Entidad** (`Contacto`) |
| --- | --- | --- |
| De dónde viene su forma | Del JSON del servidor | De la tabla de Room |
| Ejemplo | `name.first` y `name.last` por separado | Un solo campo `nombre` |
| Quién la controla | El dueño de la API (puede cambiarla) | Usted |

Si se usara la misma clase para las dos cosas, un cambio en el servidor obligaría a cambiar la base de datos. Con dos clases, solo cambia una **función de conversión** (`toContacto()`), que se escribe en el Paso 2. (*DTO* significa *Data Transfer Object*: un objeto que solo sirve para transportar datos.)

### 2.6 El recorrido completo

```
[Menú] Importar contactos de ejemplo
   │
AgendaViewModel.importarContactos()        ← el estado pasa a "Cargando"
   │
ContactoRepository.importarContactosDeEjemplo()
   │
RandomUserApi (Retrofit) ── HTTP GET ──►  randomuser.me
   │                                            │
   │◄──────────────  JSON  ◄────────────────────┘
   │
RandomUserResponse (DTO) ── toContacto() ──►  List<Contacto>
   │
ContactoDao.insertarTodos()  ──►  Room
   │
El Flow de Room avisa del cambio  ──►  la lista se redibuja sola
   │
ViewModel: estado "Exito(n)"  ──►  Snackbar "Se importaron n contactos"
```

### 2.7 Room como fuente única de verdad

La pantalla **nunca muestra directamente lo que llega de Internet**: siempre lee de Room. La red solo **alimenta** a Room. Esto tiene tres ventajas:

- Los contactos importados quedan guardados, así que **se ven sin conexión**.
- La lista funciona igual sin importar de dónde vino cada contacto (escrito a mano, con foto de la cámara o importado).
- Se escribe menos código: no hay que manejar una lista "de la red" y otra "local".

---

## 3. Resumen de cambios sobre el proyecto

| Archivo | Acción |
| --- | --- |
| `libs.versions.toml` y `build.gradle.kts` (app) | MODIFICAR |
| `AndroidManifest.xml` | MODIFICAR (permiso de Internet) |
| `data/remote/RandomUserDto.kt` | CREAR |
| `data/remote/RandomUserApi.kt` | CREAR |
| `di/NetworkModule.kt` | CREAR (o MODIFICAR, si ya lo creó en el ejercicio de `8_1`) |
| `data/ContactoDao.kt` | MODIFICAR (`insertarTodos`) |
| `repository/ContactoRepository.kt` | MODIFICAR (recibe la API e importa) |
| `viewmodel/EstadoImportacion.kt` | CREAR |
| `viewmodel/AgendaViewModel.kt` | MODIFICAR (estado e `importarContactos`) |
| `ui/views/ListaContactosView.kt` | MODIFICAR (menú, barra de carga y Snackbar) |
| `res/values/strings.xml` y `res/values-en/strings.xml` | MODIFICAR (textos nuevos) |
| `MainActivity`, `NavManager`, `AppModule` y `AgendaDatabase` | **Sin cambios** |

---

## 4. Paso 1 \[MODIFICAR libs.versions.toml + build.gradle.kts (app) + AndroidManifest.xml\] — Dependencias y permiso

**`gradle/libs.versions.toml`:**

```toml
# En la sección [versions]  (coil ya existe desde la guía 7_1)
retrofit = "3.0.0"
okhttp = "4.12.0"

# En la sección [libraries]
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }
coil-network-okhttp = { group = "io.coil-kt.coil3", name = "coil-network-okhttp", version.ref = "coil" }
```

**`build.gradle.kts` del módulo `app`:**

```kotlin
dependencies {
    // ... lo que ya tiene (Room, Hilt, DataStore, Coil, etc.)
    implementation(libs.retrofit)                  // NUEVO
    implementation(libs.retrofit.converter.gson)   // NUEVO
    implementation(libs.okhttp.logging)            // NUEVO
    implementation(libs.coil.network.okhttp)       // NUEVO
}
```

**`AndroidManifest.xml`:** el permiso de Internet va **antes** de la etiqueta `<application>`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

- `okhttp = "4.12.0"` es la versión de OkHttp con la que trabaja Retrofit 3.0.0, así que no hay mezcla de versiones. (OkHttp 5.x también es compatible, pero no hace falta.)
- **Por qué `coil-network-okhttp`:** hasta ahora las fotos eran archivos del teléfono. Los contactos importados traen la foto como una **URL de Internet**, y Coil 3 no trae de fábrica el componente que descarga de la red. Sin esta dependencia, las fotos importadas se verían en blanco, sin ningún error en pantalla.
- **Por qué el permiso:** Android no deja que una app use Internet si no lo declara. Es un permiso "normal": se concede al instalar y el usuario no ve ningún diálogo. Sin él, la llamada falla con `UnknownHostException` o `SecurityException`.
- Después de editar, pulse *Sync Now*.

## 5. Paso 2 \[CREAR data/remote/RandomUserDto.kt\] — Los DTO y la conversión a `Contacto`

Se crea un paquete nuevo `data.remote`, para separar lo que viene de la red de lo que se guarda en Room:

```kotlin
package com.example.agendaapp.data.remote

import com.example.agendaapp.data.Contacto

// Forma exacta del JSON de randomuser.me (solo los campos que interesan)
data class RandomUserResponse(val results: List<UsuarioDto>)

data class UsuarioDto(
    val name: NombreDto,
    val email: String,
    val phone: String,
    val picture: FotoDto
)

data class NombreDto(val first: String, val last: String)

data class FotoDto(val large: String)

// Conversión: del modelo de la red al modelo de la base de datos
fun UsuarioDto.toContacto() = Contacto(
    nombre = "${name.first} ${name.last}",
    telefono = phone,
    correo = email,
    fotoUri = picture.large
)
```

- **Los nombres de las propiedades deben coincidir con las claves del JSON** (`name`, `email`, `phone`, `picture`, `first`, `last`, `large`). Gson las une por nombre. Si un nombre de Kotlin tuviera que ser distinto, se usa `@SerializedName("clave_del_json")` encima de la propiedad.
- **Lo que no se declara se ignora.** El JSON trae también `title`, `medium` y `thumbnail`, pero como los DTO no los tienen, Gson los descarta sin error.
- `toContacto()` es una *función de extensión*: se llama sobre un `UsuarioDto` (`usuario.toContacto()`). Aquí se unen el nombre y el apellido en un solo `nombre`, y la URL de la foto se guarda en `fotoUri`, el mismo campo donde antes se guardaba la ruta de un archivo local.
- `id` y `favorito` no se indican: toman sus valores por defecto de `Contacto` (`id = 0` para que Room lo genere, `favorito = false`).
- **Una advertencia sobre Gson y Kotlin:** Gson no conoce las propiedades no nulas de Kotlin. Si el JSON no trajera un campo, Gson dejaría un `null` aunque la propiedad sea `String`, y la app fallaría más adelante con un `NullPointerException`. Por eso se pide con `inc=name,email,phone,picture` exactamente lo que los DTO necesitan.

## 6. Paso 3 \[CREAR data/remote/RandomUserApi.kt\] — La API como una interfaz

```kotlin
package com.example.agendaapp.data.remote

import retrofit2.http.GET
import retrofit2.http.Query

interface RandomUserApi {

    @GET("api/")
    suspend fun obtenerUsuarios(
        @Query("results") resultados: Int,
        @Query("inc") campos: String,
        @Query("nat") nacionalidades: String
    ): RandomUserResponse
}
```

No se escribe ninguna implementación: **Retrofit genera la clase por usted** al ejecutar `retrofit.create(RandomUserApi::class.java)`. Lo que hace cada parte:

- `@GET("api/")` indica el método HTTP y la **ruta**, que se une a la dirección base (`https://randomuser.me/`, que se configura en el Paso 4). La URL final es `https://randomuser.me/api/`.
- `@Query("results")` agrega un parámetro a la dirección: lo que se pase en `resultados` queda como `?results=5`. Los tres `@Query` juntos arman `?results=5&inc=...&nat=...`. En Logcat esa dirección puede verse con las comas escritas como `%2C`; es lo mismo.
- `suspend fun`: la llamada se hace dentro de una corrutina, igual que las funciones del DAO de Room. Mientras espera la respuesta, el hilo principal queda libre y la pantalla no se congela.
- **El tipo de retorno** `RandomUserResponse` le dice a Retrofit en qué clase convertir el JSON de la respuesta (con Gson, que se configura en el Paso 4).
- **Qué pasa si el servidor responde un error:** como la función devuelve directamente `RandomUserResponse` y no una `Response<...>`, si el código de estado no es 2xx (por ejemplo `404` o `500`), Retrofit lanza una `HttpException`. Si no hay conexión, lanza una `IOException`. El repositorio (Paso 5) atrapa las dos.

## 7. Paso 4 \[CREAR di/NetworkModule.kt\] — Configurar Retrofit con Hilt

Igual que `AppModule` entrega la base de datos, este módulo entrega todo lo necesario para hablar con Internet. Si ya creó un `NetworkModule` vacío en el ejercicio de la guía `8_1`, reemplace su contenido por este:

```kotlin
package com.example.agendaapp.di

import com.example.agendaapp.data.remote.RandomUserApi
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.util.concurrent.TimeUnit
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        val logging = HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        }
        return OkHttpClient.Builder()
            .addInterceptor(logging)
            .connectTimeout(15, TimeUnit.SECONDS)
            .readTimeout(15, TimeUnit.SECONDS)
            .build()
    }

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit =
        Retrofit.Builder()
            .baseUrl("https://randomuser.me/")
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()

    @Provides
    @Singleton
    fun provideRandomUserApi(retrofit: Retrofit): RandomUserApi =
        retrofit.create(RandomUserApi::class.java)
}
```

Se arma una cadena, y Hilt la resuelve de atrás hacia adelante: `RandomUserApi` necesita un `Retrofit`, que necesita un `OkHttpClient`.

- **`OkHttpClient`:** es el cliente que envía las peticiones. Se le agregan dos cosas:
  - El **interceptor de registro** (`HttpLoggingInterceptor`) con nivel `BODY`: escribe en Logcat cada petición y cada respuesta completa, incluido el JSON. Es muy útil para aprender y depurar. En una app real se usa solo en versiones de prueba, porque el registro puede mostrar datos privados.
  - Los **tiempos de espera** de 15 segundos: si el servidor no responde, la petición falla con `SocketTimeoutException` (que es un tipo de `IOException`) en lugar de quedarse esperando para siempre.
- **`baseUrl`:** es solo la dirección del servidor, y **debe terminar en `/`**; si no, la app se cierra con `IllegalArgumentException: baseUrl must end in /`. La ruta (`api/`) va en el `@GET` de la interfaz.
- **`addConverterFactory(GsonConverterFactory.create())`:** le indica a Retrofit que use Gson para convertir el JSON en objetos y viceversa. Sin esta línea, Retrofit no sabe convertir y falla con *Unable to create converter*.
- **`@Singleton`:** el `OkHttpClient` y el `Retrofit` son costosos de crear y guardan conexiones abiertas para reutilizarlas. Por eso hay uno solo para toda la app.
- **`retrofit.create(RandomUserApi::class.java)`:** aquí es donde Retrofit genera la clase que implementa la interfaz del Paso 3.

## 8. Paso 5 \[MODIFICAR data/ContactoDao.kt + repository/ContactoRepository.kt\] — Pedir los datos y guardarlos

**`ContactoDao`:** se agrega una función que inserta una lista completa:

```kotlin
@Insert
suspend fun insertarTodos(contactos: List<Contacto>)    // NUEVO
```

Insertar la lista con una sola llamada es mejor que insertar uno por uno: Room la ejecuta en **una sola transacción**, es decir, o se guardan todos o no se guarda ninguno, y es más rápido.

**`ContactoRepository`:** ahora recibe también la API y tiene una función nueva:

```kotlin
// repository/ContactoRepository.kt  (package com.example.agendaapp.repository)
// Imports que ya tiene: com.example.agendaapp.data.ContactoDao, javax.inject.Inject
// Imports nuevos: com.example.agendaapp.data.remote.RandomUserApi,
// com.example.agendaapp.data.remote.toContacto, com.google.gson.JsonParseException,
// java.io.IOException, retrofit2.HttpException

class ContactoRepository @Inject constructor(
    private val dao: ContactoDao,
    private val api: RandomUserApi                  // NUEVO
) {
    // ... contactos, guardar, actualizar y eliminar quedan igual

    // NUEVO
    suspend fun importarContactosDeEjemplo(cantidad: Int = 5): Result<Int> =
        try {
            val respuesta = api.obtenerUsuarios(
                resultados = cantidad,
                campos = "name,email,phone,picture",
                nacionalidades = "us,es,mx"
            )
            val contactos = respuesta.results.map { it.toContacto() }
            dao.insertarTodos(contactos)
            Result.success(contactos.size)
        } catch (e: IOException) {
            Result.failure(e)        // sin conexión, tiempo agotado, servidor inalcanzable
        } catch (e: HttpException) {
            Result.failure(e)        // el servidor respondió con un código de error (4xx o 5xx)
        } catch (e: JsonParseException) {
            Result.failure(e)        // el JSON recibido no coincide con los DTO
        }
}
```

- **Hilt ya sabe darle la API:** `RandomUserApi` tiene su receta en `NetworkModule`, así que Hilt la entrega al constructor sin que se cambie nada en `MainActivity` ni en `AppModule`. Este es el beneficio de la guía `8_1`: agregar una dependencia nueva es agregar un parámetro.
- **`Result<Int>`** es el tipo de Kotlin para "terminó bien con un valor, o falló con una excepción". Aquí el valor es cuántos contactos se importaron. Así el `ViewModel` no necesita escribir `try/catch`: usa `onSuccess` y `onFailure`.
- **Las tres excepciones, y qué significa cada una:**
  - `IOException`: problemas de red (sin Internet, tiempo agotado, no se encontró el servidor).
  - `HttpException`: el servidor sí respondió, pero con un código que no es 2xx.
  - `JsonParseException`: la respuesta llegó bien, pero no se pudo convertir con los DTO.
- **Por qué no `catch (e: Exception)`:** también atraparía la `CancellationException`, que las corrutinas usan para cancelarse (por ejemplo, cuando el `ViewModel` se destruye). Si se atrapa, esa cancelación se ignora y la corrutina sigue trabajando cuando ya no debería.
- **No hace falta cambiar de hilo.** Las funciones `suspend` de Retrofit hacen la petición en hilos de fondo, y las de Room también. Por eso no se usa `withContext(Dispatchers.IO)`: se pueden llamar directamente desde `viewModelScope`.
- La conversión `respuesta.results.map { it.toContacto() }` transforma la lista de DTO en la lista de entidades antes de tocar Room. La base de datos nunca ve un DTO.

## 9. Paso 6 \[CREAR viewmodel/EstadoImportacion.kt + MODIFICAR AgendaViewModel.kt\] — Los estados de la llamada

Una llamada de red tarda y puede fallar, así que la pantalla debe saber en cuál de estas situaciones está. Se modelan como una `sealed interface`:

```kotlin
package com.example.agendaapp.viewmodel

sealed interface EstadoImportacion {
    data object Inactivo : EstadoImportacion                  // no se está haciendo nada
    data object Cargando : EstadoImportacion                  // petición en curso
    data class Exito(val cantidad: Int) : EstadoImportacion   // terminó bien, con cuántos
    data object Error : EstadoImportacion                     // falló
}
```

Y en `AgendaViewModel` se agrega:

```kotlin
// Imports nuevos: kotlinx.coroutines.flow.MutableStateFlow,
// kotlinx.coroutines.flow.StateFlow, kotlinx.coroutines.flow.asStateFlow

// NUEVO
private val _estadoImportacion = MutableStateFlow<EstadoImportacion>(EstadoImportacion.Inactivo)
val estadoImportacion: StateFlow<EstadoImportacion> = _estadoImportacion.asStateFlow()

// NUEVO
fun importarContactos() {
    if (_estadoImportacion.value is EstadoImportacion.Cargando) return   // ignora toques repetidos
    viewModelScope.launch {
        _estadoImportacion.value = EstadoImportacion.Cargando
        repository.importarContactosDeEjemplo()
            .onSuccess { _estadoImportacion.value = EstadoImportacion.Exito(it) }
            .onFailure { _estadoImportacion.value = EstadoImportacion.Error }
    }
}

// NUEVO: la pantalla avisa que ya mostró el mensaje
fun importacionMostrada() {
    _estadoImportacion.value = EstadoImportacion.Inactivo
}
```

- **Por qué una `sealed interface`:** solo puede estar en uno de esos cuatro estados a la vez. Eso evita combinaciones absurdas como "cargando y con error al mismo tiempo", que aparecen cuando se usan varias variables `Boolean` sueltas.
- **`_estadoImportacion` (privado) y `estadoImportacion` (público):** el `MutableStateFlow` solo lo puede cambiar el `ViewModel`; la pantalla recibe una versión de solo lectura con `asStateFlow()`. Es el mismo patrón de `contactos` en la guía `7_1`.
- **El primer `if`** evita que dos toques seguidos en el menú lancen dos peticiones.
- **El recorrido de los estados:** `Inactivo` → `Cargando` → `Exito(n)` o `Error` → (la pantalla muestra el mensaje) → `importacionMostrada()` → `Inactivo`.
- **Por qué regresar a `Inactivo`:** un `StateFlow` conserva siempre su último valor. Si el estado se quedara en `Exito(5)`, al girar el teléfono la pantalla se volvería a crear y mostraría el mensaje otra vez. Al volver a `Inactivo`, cada mensaje aparece **una sola vez**.

## 10. Paso 7 \[MODIFICAR ListaContactosView.kt + res/values/strings.xml + res/values-en/strings.xml\] — Mostrar el resultado

**Textos nuevos.** Si todavía no aplicó la parte de idiomas de la guía `5_1`, escriba estos textos directamente en el código en lugar de usar `stringResource`.

```xml
<!-- res/values/strings.xml (español) -->
<string name="importar_contactos">Importar contactos de ejemplo</string>
<string name="error_importar">No se pudo importar. Revise su conexión a Internet.</string>
<plurals name="importados_ok">
    <item quantity="one">Se importó %d contacto</item>
    <item quantity="other">Se importaron %d contactos</item>
</plurals>
```

```xml
<!-- res/values-en/strings.xml (inglés) -->
<string name="importar_contactos">Import sample contacts</string>
<string name="error_importar">Could not import. Check your Internet connection.</string>
<plurals name="importados_ok">
    <item quantity="one">%d contact imported</item>
    <item quantity="other">%d contacts imported</item>
</plurals>
```

**`ListaContactosView`.** Hay cuatro cambios.

**1. Estado y mensajes**, junto a los demás `remember` de la función:

```kotlin
val estadoImportacion by viewModel.estadoImportacion.collectAsState()   // NUEVO
val snackbarHostState = remember { SnackbarHostState() }               // NUEVO
val context = LocalContext.current                                      // NUEVO

LaunchedEffect(estadoImportacion) {                                     // NUEVO
    when (val estado = estadoImportacion) {
        is EstadoImportacion.Exito -> {
            snackbarHostState.showSnackbar(
                context.resources.getQuantityString(
                    R.plurals.importados_ok, estado.cantidad, estado.cantidad
                )
            )
            viewModel.importacionMostrada()
        }
        EstadoImportacion.Error -> {
            snackbarHostState.showSnackbar(context.getString(R.string.error_importar))
            viewModel.importacionMostrada()
        }
        else -> Unit
    }
}
```

**2. El `Scaffold` recibe el lugar donde se dibuja el Snackbar:**

```kotlin
Scaffold(
    snackbarHost = { SnackbarHost(snackbarHostState) },    // NUEVO
    topBar = { ... },
    floatingActionButton = { ... }
) { innerPadding -> ... }
```

**3. Opción nueva en el menú de tres puntos**, después de "Ordenar por más reciente" y antes del `HorizontalDivider`:

```kotlin
DropdownMenuItem(
    text = { Text(stringResource(R.string.importar_contactos)) },
    onClick = {
        menuAbierto = false
        viewModel.importarContactos()
    }
)
```

**4. Barra de progreso**, como primer elemento del `Column` que está dentro del `Scaffold`:

```kotlin
if (estadoImportacion is EstadoImportacion.Cargando) {
    LinearProgressIndicator(modifier = Modifier.fillMaxWidth())
}
```

- **Imports nuevos:** `androidx.compose.runtime.LaunchedEffect`, `androidx.compose.material3.SnackbarHost`, `androidx.compose.material3.SnackbarHostState`, `androidx.compose.material3.LinearProgressIndicator`, `androidx.compose.ui.platform.LocalContext` y `com.example.agendaapp.viewmodel.EstadoImportacion`.
- **Por qué `LaunchedEffect(estadoImportacion)`:** un Snackbar es un aviso que aparece **una vez** cuando algo ocurre, no algo que se dibuja siempre. `showSnackbar` es una función `suspend`, así que necesita una corrutina, y `LaunchedEffect` la da. Se vuelve a ejecutar cada vez que cambia el valor `estadoImportacion` (la "clave").
- **Por qué `context.getString(...)` y no `stringResource(...)` dentro del efecto:** `stringResource` solo se puede llamar dentro de un composable, y el cuerpo de un `LaunchedEffect` no lo es. Por eso se obtiene el texto con el `Context`.
- **`importacionMostrada()` va después de `showSnackbar`:** esa función espera hasta que el Snackbar desaparece. Si se reiniciara el estado antes, el efecto se cancelaría y el mensaje nunca llegaría a verse.
- `LinearProgressIndicator` sin parámetro `progress` es una barra **indeterminada**: se anima mientras dura la espera, porque no se sabe cuánto falta.

---

## 11. Cómo probar

1. Ejecute la app. En el menú de tres puntos elija **Importar contactos de ejemplo**. Debe verse la barra de progreso un instante, aparecer 5 contactos con foto en la lista, y un Snackbar con "Se importaron 5 contactos".
2. Abra **Logcat** y filtre por `okhttp.OkHttpClient`. Se ven la petición (`--> GET https://randomuser.me/api/?results=5...`), el código de respuesta (`<-- 200`) y el JSON completo.
3. Active el **modo avión** y vuelva a importar. Debe aparecer el Snackbar de error y no agregarse ningún contacto. La app **no se cierra**: el error se atrapó en el repositorio.
4. Con el modo avión aún activo, cierre y abra la app. Los contactos ya importados siguen ahí, porque están en Room.

## 12. Detalles que conviene entender

- **Cada importación agrega contactos nuevos.** No se revisa si ya existe un contacto con el mismo correo; el ejercicio 2 lo resuelve.
- **Las fotos importadas son direcciones de Internet.** Coil las descarga y las guarda en su caché: las que ya se vieron aparecen aunque no haya conexión, las demás no.
- **La API es gratuita y compartida.** Si se hacen muchas peticiones seguidas puede responder `429`, y la app mostrará el mensaje de error.
- **Los datos son ficticios.** Las opciones "Llamar" y "Enviar correo" del menú de la tarjeta funcionan con ellos, pero esos números y correos no pertenecen a nadie: no los use.
- **Gson en una app publicada:** al activar la minificación (R8), los DTO necesitan reglas de conservación para que Gson los siga encontrando. Para la práctica, en modo *debug*, no es necesario.

### Si los contactos anteriores perdieron su foto de la galería

Al importar contactos, es normal descubrir que los contactos que se agregaron antes con una foto **de la galería** aparecen sin foto, mientras que los importados (y los de la cámara) sí la muestran. No es un problema de Retrofit ni de Coil: la foto de la galería se guardó como una dirección temporal del selector de fotos (`content://media/picker/...`), y Android deja de dar acceso a ella. Si activa un `onError` en el `AsyncImage` y filtra Logcat por esa etiqueta, el motivo aparece así:

```
SecurityException: Calling uid (...) does not have permission to access picker uri:
content://media/picker/0/com.android.providers.media.photopicker/media/...
```

La corrección es copiar la foto a la carpeta interna de la app en el momento de elegirla y guardar la ruta de la copia. Son dos cambios:

**1. CREAR `utils/ImagenUtil.kt`** (paquete `utils`):

```kotlin
package com.example.agendaapp.utils

import android.content.Context
import android.net.Uri
import androidx.core.net.toUri
import java.io.File
import java.util.UUID

// Copia la imagen elegida a la carpeta interna de la app y devuelve su dirección (file://...)
fun copiarImagenALocal(context: Context, uri: Uri): String? = try {
    val destino = File(context.filesDir, "contacto_${UUID.randomUUID()}.jpg")
    val entrada = context.contentResolver.openInputStream(uri) ?: return null
    entrada.use { origen -> destino.outputStream().use { salida -> origen.copyTo(salida) } }
    destino.toUri().toString()
} catch (e: Exception) {
    null
}
```

**2. MODIFICAR `ui/views/RegistroContactoView.kt`:** el *launcher* de la galería copia la imagen antes de asignar `fotoUri`.

```kotlin
// Imports nuevos: androidx.compose.runtime.rememberCoroutineScope, kotlinx.coroutines.Dispatchers,
// kotlinx.coroutines.launch, kotlinx.coroutines.withContext,
// com.example.agendaapp.utils.copiarImagenALocal
val scope = rememberCoroutineScope()                        // NUEVO, junto a val context = LocalContext.current

val seleccionarImagenLauncher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.PickVisualMedia()
) { uri: Uri? ->
    if (uri != null) {                                      // CAMBIO
        scope.launch {
            val ruta = withContext(Dispatchers.IO) { copiarImagenALocal(context, uri) }
            if (ruta != null) fotoUri = ruta
        }
    }
}
```

- Es el mismo cambio que se explica en la guía `7_1` (sección 9.1); aquí se repite porque es en esta guía, al ver juntas las fotos locales y las de Internet, donde el problema se hace visible.
- Los contactos que ya se guardaron con la dirección antigua **no se recuperan**: hay que editarlos y elegir la foto otra vez, una sola vez. Las fotos importadas de randomuser.me no se ven afectadas, porque son direcciones `https://`.

## 13. Errores comunes

- **`IllegalArgumentException: baseUrl must end in /`:** a la dirección base de `NetworkModule` le falta la barra final.
- **"Unable to create converter" o "Could not locate ResponseBody converter":** falta `addConverterFactory(GsonConverterFactory.create())` o la dependencia `converter-gson`.
- **`UnknownHostException`:** no hay Internet en el dispositivo o emulador. Si el mensaje es `SecurityException: Permission denied (missing INTERNET permission?)`, falta `<uses-permission android:name="android.permission.INTERNET" />` en el manifiesto.
- **`CLEARTEXT communication not permitted`:** la dirección empieza con `http://` en lugar de `https://`. Android bloquea por defecto el tráfico sin cifrar.
- **`JsonSyntaxException: Expected BEGIN_OBJECT but was BEGIN_ARRAY`:** un DTO no coincide con la forma del JSON, por ejemplo una propiedad declarada como objeto cuando el JSON trae una lista.
- **`NullPointerException` al usar `usuario.name.first`:** el JSON no trajo ese campo y Gson dejó un `null`. Revise el parámetro `inc` y los nombres de las propiedades de los DTO.
- **`HttpException: HTTP 429`:** se hicieron demasiadas peticiones. Espere un momento.
- **`[Dagger/MissingBinding] RandomUserApi cannot be provided`:** `NetworkModule` no tiene `@Module` o `@InstallIn(SingletonComponent::class)`, o falta la función `@Provides` de la API.
- **Las fotos importadas se ven en blanco:** falta `coil-network-okhttp` o el permiso de Internet.
- **`NetworkOnMainThreadException`:** se hizo una llamada síncrona de red en el hilo principal. La función de la interfaz debe ser `suspend`.
- **"Unresolved reference: toContacto":** falta `import com.example.agendaapp.data.remote.toContacto` en el repositorio. Las funciones de extensión se importan por separado.
- **El Snackbar sale cada vez que se gira la pantalla:** falta llamar a `viewModel.importacionMostrada()` después de mostrarlo.

* **Los contactos agregados antes con foto de la galería aparecen sin foto, pero los importados sí la muestran (`SecurityException: ... does not have permission to access picker uri`):** se guardó la dirección temporal del selector de fotos. Copie la imagen a `filesDir` con `copiarImagenALocal` (ver la sección 12, *Si los contactos anteriores perdieron su foto de la galería*).
* **`Unresolved reference: copiarImagenALocal`:** falta el archivo `utils/ImagenUtil.kt`, su `package` no es `com.example.agendaapp.utils`, o falta el `import` en `RegistroContactoView`.

## 14. Ejercicio propuesto

Sobre la AgendaApp, el estudiante debe agregar:

1. **Elegir cuántos contactos importar.** Reemplazar la opción del menú por un submenú o un diálogo con las opciones 5, 10 y 20, y pasar la cantidad desde `importarContactos(cantidad)` hasta el repositorio.
2. **Evitar contactos duplicados por correo.** En `Contacto`, agregar `indices = [Index(value = ["correo"], unique = true)]` en la anotación `@Entity`; en el DAO, usar `@Insert(onConflict = OnConflictStrategy.IGNORE)` y que `insertarTodos` devuelva `List<Long>`. Room devuelve `-1` en las filas ignoradas, lo que permite contar cuántos se agregaron de verdad. Recuerde subir la `version` de la base de datos, como se vio en la guía `7_2`.
3. **Importar por país.** Parametrizar `nacionalidades` y ofrecer en el menú "Importar de México", "Importar de España" e "Importar de Estados Unidos" (`mx`, `es`, `us`). Todo el texto debe estar traducido a los dos idiomas.
4. **Botón "Reintentar" en el Snackbar de error.** `showSnackbar` acepta un `actionLabel` y devuelve un `SnackbarResult`; si es `ActionPerformed`, se vuelve a llamar a `importarContactos()`.
