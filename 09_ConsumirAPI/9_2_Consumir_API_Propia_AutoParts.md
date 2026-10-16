# 9\_2 - Retrofit con API propia (AutoParts)

## Objetivo

Al terminar, **AutoParts** lista, crea, edita y elimina repuestos y servicios consumiendo la API Spring Boot que ya corre en Docker dentro de WSL, sin autenticación (el login se agrega en la guía 9\_3).

La app usa estos cinco endpoints de `RepuestoServicioController`:

| Método | Ruta (base `http://localhost:8080/api/`) | Qué hace en la app | Cuerpo |
| --- | --- | --- | --- |
| GET | `repuestos-servicios` | Lista con foto, precio y tipo | — |
| GET | `repuestos-servicios/{id}` | Detalle de un ítem | — |
| POST | `repuestos-servicios` | Crear | `multipart/form-data`: parte `dto` (JSON) + parte `imagen` (opcional) |
| PUT | `repuestos-servicios/{id}` | Editar | Igual que POST; si no llega `imagen`, el servidor conserva la anterior |
| DELETE | `repuestos-servicios/{id}` | Eliminar | — |

Lo nuevo respecto a la guía 9\_1:

- La API es propia, así que el DTO se arma a partir de la entidad Java y no de un JSON ajeno.
- Crear y editar usan `multipart`: un JSON y un archivo viajan juntos en una sola petición.
- Las fotos se sirven desde `http://localhost:8080/images/{foto}`, fuera de `/api`.
- El servidor corre en la PC del estudiante, por lo que la conexión se prepara con `adb reverse` (Paso 0).

El proyecto sigue la misma arquitectura de `AgendaApp`: **Hilt** (guía `8_1`) entrega `Retrofit`, el repositorio y el `ViewModel`, igual que en la guía `9_1`. Para centrarse en Retrofit y en el envío de imágenes, **no usa Room ni DataStore**.

## Paso 0: preparar la conexión

Dentro del teléfono o del emulador, `localhost` es el propio dispositivo y no la PC donde corre Docker. La solución más estable para clase es `adb reverse`, que enlaza un puerto del dispositivo con el mismo puerto de la PC por USB, sin depender de Wi-Fi, firewall ni direcciones IP.

### 0.1 Probar la API desde Windows

Antes de escribir código, compruebe lo que responde el servidor. No hace falta `curl`: hay cuatro formas de hacerlo en Windows, de la más simple a la más completa. Con cualquiera basta.

**A. El navegador de la PC (la más simple).** Abra en Chrome o Edge:

```text
http://localhost:8080/api/repuestos-servicios
http://localhost:8080/images/08605714-820e-415b-9e31-fea1bdb00321.jpg
```

La primera dirección debe mostrar el JSON con los ocho registros, y la segunda, la foto. Si el servidor exige autenticación, el navegador muestra una página de error `401` o `403`. Esta prueba solo sirve para los `GET`.

**B. PowerShell, con sus propios comandos** (no `curl`, que en PowerShell es un alias de otro comando):

```powershell
# Lista legible: Invoke-RestMethod convierte el JSON en objetos
Invoke-RestMethod http://localhost:8080/api/repuestos-servicios | Format-Table id, nombre, precio, tipo

# Solo el código de respuesta (también cuando es 401, 403 o 404)
try {
    (Invoke-WebRequest http://localhost:8080/api/repuestos-servicios -UseBasicParsing).StatusCode
} catch {
    $_.Exception.Response.StatusCode.value__
}

# ¿Existe la foto? Pide solo los encabezados
(Invoke-WebRequest -Method Head -UseBasicParsing `
    http://localhost:8080/images/08605714-820e-415b-9e31-fea1bdb00321.jpg).StatusCode
```

`-UseBasicParsing` evita una advertencia de Windows PowerShell 5.1; en PowerShell 7 no hace falta.

**C. Postman** (ya lo usa para el login): una petición `GET` a `http://localhost:8080/api/repuestos-servicios` muestra el código de estado, el tiempo y el JSON. Es la mejor opción para probar después los `POST` y `PUT` con la imagen, con el cuerpo en *form-data*.

**D. Desde WSL**, si su distribución tiene `curl`:

```bash
curl -i http://localhost:8080/api/repuestos-servicios
```

Esta prueba confirma que el contenedor responde, pero **no** que Windows lo vea, y es Windows quien lo comparte con el teléfono por `adb reverse`. Por eso conviene probar siempre una de las opciones A, B o C.

**Si ninguna responde**, revise en este orden:

1. `docker ps`: la columna de puertos debe mostrar `8080->8080` (por ejemplo, `0.0.0.0:8080->8080/tcp`). Si no aparece, el contenedor no publica el puerto.
2. Cambie `localhost` por `127.0.0.1`. Si así funciona, Windows está resolviendo `localhost` a la dirección IPv6 (`::1`) y Docker solo escucha en IPv4; use `127.0.0.1` en las pruebas de la PC.
3. `docker logs <nombre-del-contenedor>`: confirma que Spring arrancó sin errores y en qué puerto.
4. Un proxy o antivirus puede interceptar las peticiones: pruebe desde el navegador o desde Postman.

| Respuesta | Qué significa | Qué hacer |
| --- | --- | --- |
| `200` y un JSON | La ruta está abierta | Seguir con 0.3 |
| `401` o `403` | La ruta exige token | Abrirla temporalmente (0.2) o pasar a la guía 9\_3 |
| `Connection refused` | El contenedor no publica el puerto | `docker ps` y revisar que muestre `8080->8080` |

### 0.2 Abrir la API para esta práctica

Si los `GET`, `POST`, `PUT` o `DELETE` responden `401` o `403`, el docente puede preparar una rama de práctica del proyecto Spring con tres cambios temporales (los nombres dependen de cada `SecurityConfig`):

```java
http
    .csrf(csrf -> csrf.disable())
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/repuestos-servicios/**", "/images/**").permitAll()
        .anyRequest().authenticated());
```

Además, mientras `@EnableMethodSecurity` esté activo, cada `@PreAuthorize` sigue exigiendo rol aunque la ruta esté permitida. Para la práctica hay que comentar esas anotaciones del controlador (o desactivar la seguridad por método). Estos cambios nunca se llevan a producción: la guía 9\_3 los revierte y usa el login.

### 0.3 Enlazar el puerto con adb reverse

Conecte el teléfono por USB con la depuración USB activada y ejecute, desde la carpeta `platform-tools` del SDK (normalmente `%LOCALAPPDATA%\Android\Sdk\platform-tools`):

```powershell
adb devices
adb reverse tcp:8080 tcp:8080
```

Prueba rápida: abra Chrome en el teléfono y entre a `http://localhost:8080/api/repuestos-servicios`. Si muestra el JSON, la conexión está lista.

- El enlace se pierde al desconectar el cable o reiniciar `adb`: vuelva a ejecutar `adb reverse`.
- En el emulador también funciona; la alternativa es usar `http://10.0.2.2:8080/`, pero conviene que todos usen la misma URL.

### 0.4 Permisos en el manifiesto

Android bloquea `http://` por defecto. Para desarrollo agregue el permiso de Internet y habilite el tráfico sin cifrar en `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />

<application
    android:usesCleartextTraffic="true"
    ... >
```

`usesCleartextTraffic` es solo para pruebas locales: en una app publicada se usa HTTPS.

## Paso 1: crear el proyecto y las dependencias

En Android Studio cree un proyecto **Empty Activity** (Compose) llamado `AutoParts`, con paquete `com.example.autoparts` y minSdk 24. Se agregan cuatro grupos de librerías: Retrofit con OkHttp, Hilt (como en la guía 8\_1), Coil para las fotos y Navigation para pasar de la lista al formulario.

### 1.1 libs.versions.toml

Agregue las versiones y librerías nuevas a las que ya trae el archivo:

```toml
[versions]
retrofit = "3.0.0"
okhttp = "4.12.0"
coil = "3.6.3"
lifecycleRuntimeKtx = "2.11.0"
navigationCompose = "2.10.2"

[libraries]
retrofit = { module = "com.squareup.retrofit2:retrofit", version.ref = "retrofit" }
retrofit-gson = { module = "com.squareup.retrofit2:converter-gson", version.ref = "retrofit" }
okhttp-logging = { module = "com.squareup.okhttp3:logging-interceptor", version.ref = "okhttp" }
coil-compose = { module = "io.coil-kt.coil3:coil-compose", version.ref = "coil" }
coil-network-okhttp = { module = "io.coil-kt.coil3:coil-network-okhttp", version.ref = "coil" }
androidx-lifecycle-viewmodel-compose = { module = "androidx.lifecycle:lifecycle-viewmodel-compose", version.ref = "lifecycleRuntimeKtx" }
androidx-lifecycle-runtime-compose = { module = "androidx.lifecycle:lifecycle-runtime-compose", version.ref = "lifecycleRuntimeKtx" }
androidx-navigation-compose = { module = "androidx.navigation:navigation-compose", version.ref = "navigationCompose" }
```

Si `lifecycleRuntimeKtx`, `navigationCompose` o `coil` ya existen en el archivo (por ejemplo, porque el proyecto se creó con la plantilla de Android Studio), no las duplique: deje la versión que ya tienen. Con estas versiones recientes el proyecto necesita Kotlin 2.4: compruebe que en `[versions]` esté `kotlin = "2.4.21"` (ver Paso 1 de la guía 8\_1 si aparecen errores de *incompatible version of Kotlin*).

### 1.2 build.gradle.kts (módulo app)

```kotlin
dependencies {
    // ... las que ya trae la plantilla

    implementation(libs.retrofit)
    implementation(libs.retrofit.gson)
    implementation(libs.okhttp.logging)

    implementation(libs.coil.compose)
    implementation(libs.coil.network.okhttp)   // sin esta, las fotos salen en blanco

    implementation(libs.androidx.lifecycle.viewmodel.compose)
    implementation(libs.androidx.lifecycle.runtime.compose)
    implementation(libs.androidx.navigation.compose)
}
```

Este proyecto **no** usa Room, así que no hace falta el compilador de Room, pero sí **KSP** para Hilt. Sincronice con **Sync Now** después de agregar lo siguiente.

### Hilt: plugins, dependencias y la clase Application

La configuración es la misma del Paso 1 de la guía `8_1`. En `libs.versions.toml` use los valores con los que se probó esa guía (si `kotlin` o `ksp` ya existen, solo cambie el valor):

```toml
[versions]
kotlin = "2.4.21"
ksp = "2.3.5"
hilt = "2.60.1"

[libraries]
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-android-compiler", version.ref = "hilt" }

[plugins]
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

En el `build.gradle.kts` de la raíz del proyecto, si ahí se declaran los plugins (como hace la plantilla), agregue:

```kotlin
plugins {
    // ... los que ya tiene
    alias(libs.plugins.ksp) apply false
    alias(libs.plugins.hilt) apply false
}
```

Y en el `build.gradle.kts` del módulo `app`:

```kotlin
plugins {
    // ... los que ya tiene
    alias(libs.plugins.ksp)
    alias(libs.plugins.hilt)
}

dependencies {
    // ... las que ya tiene
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
}
```

Cree la clase `Application`, que es donde Hilt arma sus componentes:

```kotlin
// AutoPartsApplication.kt
package com.example.autoparts

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class AutoPartsApplication : Application()
```

Y regístrela en el manifiesto, junto con el permiso de Internet y el tráfico sin cifrar del Paso 0.4:

```xml
<application
    android:name=".AutoPartsApplication"
    android:usesCleartextTraffic="true"
    ... >
```

- Sin `android:name`, la app se cierra al abrir con el error *Hilt Activity must be attached to an @HiltAndroidApp Application*.
- Si el proyecto no compila y aparecen errores de *incompatible version of Kotlin*, revise el valor de `kotlin` (explicado en la guía `8_1`).
### 1.3 Estructura de paquetes

```text
com.example.autoparts
├── AutoPartsApplication.kt         (@HiltAndroidApp)
├── MainActivity.kt                 (@AndroidEntryPoint; NavHost: lista y formulario)
├── data/
│    └── remote/
│         ├── RepuestoDto.kt        (DTOs y enum TipoItem)
│         ├── RepuestoApi.kt        (interfaz Retrofit)
│         └── ApiConfig.kt          (direcciones del servidor y urlFoto)
├── di/
│    └── NetworkModule.kt           (OkHttp, Retrofit y RepuestoApi con Hilt)
├── repository/
│    └── RepuestoRepository.kt      (@Inject constructor; también ImagenSeleccionada)
├── viewmodel/
│    └── RepuestosViewModel.kt      (@HiltViewModel)
└── ui/
     ├── components/RepuestoCard.kt
     └── views/
          ├── ListaRepuestosView.kt
          └── FormularioRepuestoView.kt
```

## Paso 2: DTO y enum

El DTO de la app replica el `RepuestoServicioDTO` del servidor campo por campo, con los mismos nombres: Gson relaciona el JSON con la clase por el nombre del campo.

| Java (servidor) | Kotlin (app) | Nota |
| --- | --- | --- |
| `Integer id` | `Int?` | Es `null` al crear; Gson no envía los campos nulos |
| `String nombre` | `String` | Obligatorio, máximo 100 caracteres |
| `String descripcion` | `String?` | Opcional, máximo 250 |
| `BigDecimal precio` | `Double` | Suficiente para la práctica; en producción conviene `BigDecimal` |
| `Integer stock` | `Int` | Los servicios llegan con `0` |
| `String foto` | `String?` | Solo el nombre del archivo, o `null` si no tiene imagen |
| `TipoItem tipo` | `TipoItem` | Los nombres del enum deben coincidir exactamente con el servidor |

Cree `data/remote/RepuestoDto.kt`:

```kotlin
package com.example.autoparts.data.remote

enum class TipoItem { REPUESTO, SERVICIO }

data class RepuestoDto(
    val id: Int? = null,
    val nombre: String,
    val descripcion: String? = null,
    val precio: Double,
    val stock: Int,
    val foto: String? = null,
    val tipo: TipoItem
)

// Respuesta de POST y PUT: {"message": "...", "repuestoServicio": {...}}
data class RespuestaGuardadoDto(
    val message: String,
    val repuestoServicio: RepuestoDto
)

// Respuesta de DELETE: {"message": "..."}
data class MensajeDto(
    val message: String
)
```

Gson no respeta la nulabilidad de Kotlin: si el servidor omite un campo declarado como no nulo, la app no falla al leer el JSON sino más tarde, al usar ese campo. Por eso `descripcion` y `foto`, que llegan como `null`, están declarados con `?`.

## Paso 3: la interfaz Retrofit

Cada método de `RepuestoApi` corresponde a un endpoint del controlador. Los `GET` y el `DELETE` son como en la guía 9\_1; `POST` y `PUT` son nuevos porque el servidor los declara con `consumes = "multipart/form-data"`.

### Qué envía un multipart

Una petición multipart es un solo cuerpo dividido en partes, cada una con su nombre y su tipo de contenido. El controlador espera dos, que llegan a `@RequestPart("dto")` y `@RequestPart("imagen")`:

```text
POST /api/repuestos-servicios
Content-Type: multipart/form-data; boundary=XYZ

--XYZ
Content-Disposition: form-data; name="dto"
Content-Type: application/json

{"nombre":"Bujías","precio":15.0,"stock":12,"tipo":"REPUESTO"}
--XYZ
Content-Disposition: form-data; name="imagen"; filename="foto.jpg"
Content-Type: image/jpeg

<bytes de la imagen>
--XYZ--
```

Dos detalles deciden si funciona:

- La parte `dto` debe llevar `Content-Type: application/json`; si viaja como texto plano, Spring responde `415`.
- Los nombres de las partes (`dto` e `imagen`) deben ser exactamente esos.

### RepuestoApi.kt

```kotlin
package com.example.autoparts.data.remote

import okhttp3.MultipartBody
import okhttp3.RequestBody
import retrofit2.http.DELETE
import retrofit2.http.GET
import retrofit2.http.Multipart
import retrofit2.http.POST
import retrofit2.http.PUT
import retrofit2.http.Part
import retrofit2.http.Path

interface RepuestoApi {

    @GET("repuestos-servicios")
    suspend fun obtenerTodos(): List<RepuestoDto>

    @GET("repuestos-servicios/{id}")
    suspend fun obtenerPorId(@Path("id") id: Int): RepuestoDto

    @Multipart
    @POST("repuestos-servicios")
    suspend fun crear(
        @Part("dto") dto: RequestBody,
        @Part imagen: MultipartBody.Part?          // opcional
    ): RespuestaGuardadoDto

    @Multipart
    @PUT("repuestos-servicios/{id}")
    suspend fun actualizar(
        @Path("id") id: Int,
        @Part("dto") dto: RequestBody,
        @Part imagen: MultipartBody.Part?          // si es null, el servidor conserva la foto
    ): RespuestaGuardadoDto

    @DELETE("repuestos-servicios/{id}")
    suspend fun eliminar(@Path("id") id: Int): MensajeDto
}
```

Cuando un `@Part` es `null`, Retrofit simplemente no incluye esa parte en la petición, que es lo que el servidor espera cuando no hay imagen. Si el servidor responde con un código de error (`400`, `404`, `500`), el método `suspend` lanza una `HttpException`, igual que en la guía 9\_1.

## Paso 4: el cliente Retrofit y el repositorio

`NetworkModule` le entrega a Hilt las piezas de red, igual que en la guía `9_1`: `RepuestoApi` necesita un `Retrofit`, que necesita un `OkHttpClient`, y Hilt los crea una sola vez (`@Singleton`) y los reutiliza. Las direcciones del servidor y la función `urlFoto` quedan en un `object ApiConfig`, porque las pantallas también la necesitan para mostrar las fotos.

### ApiConfig.kt y NetworkModule.kt

Guarde en un solo lugar las dos direcciones del servidor: la API (`/api/`) y las fotos (`/images/`). La `baseUrl` de Retrofit debe terminar en `/`.

```kotlin
// data/remote/ApiConfig.kt
package com.example.autoparts.data.remote

object ApiConfig {

    // Con adb reverse, localhost del dispositivo llega a la PC
    private const val SERVIDOR = "http://localhost:8080/"
    const val BASE_URL = SERVIDOR + "api/"          // debe terminar en /
    private const val BASE_IMAGENES = SERVIDOR + "images/"

    // "abc.jpg" -> "http://localhost:8080/images/abc.jpg"; null si el ítem no tiene foto
    fun urlFoto(foto: String?): String? = foto?.let { BASE_IMAGENES + it }
}
```

```kotlin
// di/NetworkModule.kt
package com.example.autoparts.di

import com.example.autoparts.data.remote.ApiConfig
import com.example.autoparts.data.remote.RepuestoApi
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
    fun provideOkHttpClient(): OkHttpClient =
        OkHttpClient.Builder()
            .addInterceptor(
                HttpLoggingInterceptor().apply { level = HttpLoggingInterceptor.Level.BASIC }
            )
            .connectTimeout(15, TimeUnit.SECONDS)
            .readTimeout(15, TimeUnit.SECONDS)
            .writeTimeout(30, TimeUnit.SECONDS)    // subir una foto tarda más
            .build()

    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit =
        Retrofit.Builder()
            .baseUrl(ApiConfig.BASE_URL)
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()

    @Provides
    @Singleton
    fun provideRepuestoApi(retrofit: Retrofit): RepuestoApi =
        retrofit.create(RepuestoApi::class.java)
}
```

El nivel `BASIC` registra solo método, ruta y código. Con `BODY` el registro incluiría los bytes de las imágenes y llenaría Logcat.

### RepuestoRepository.kt

El repositorio hace tres cosas: arma el `RequestBody` JSON de la parte `dto`, arma la parte `imagen` y convierte las excepciones de red en mensajes legibles dentro de un `Result`.

```kotlin
// repository/RepuestoRepository.kt
package com.example.autoparts.repository

import com.example.autoparts.data.remote.RepuestoApi
import com.example.autoparts.data.remote.RepuestoDto
import com.google.gson.Gson
import com.google.gson.JsonParseException
import okhttp3.MediaType.Companion.toMediaType
import okhttp3.MultipartBody
import okhttp3.RequestBody
import okhttp3.RequestBody.Companion.toRequestBody
import retrofit2.HttpException
import java.io.IOException
import javax.inject.Inject

// Foto elegida por el usuario, ya leída de la galería
class ImagenSeleccionada(val bytes: ByteArray, val mime: String)

class RepuestoRepository @Inject constructor(      // CAMBIO: Hilt entrega la API
    private val api: RepuestoApi
) {
    private val gson = Gson()

    suspend fun obtenerTodos(): Result<List<RepuestoDto>> =
        llamar { api.obtenerTodos() }

    suspend fun obtenerPorId(id: Int): Result<RepuestoDto> =
        llamar { api.obtenerPorId(id) }

    suspend fun crear(dto: RepuestoDto, imagen: ImagenSeleccionada?): Result<RepuestoDto> =
        llamar { api.crear(dto.aParteJson(), imagen?.aParte()).repuestoServicio }

    suspend fun actualizar(id: Int, dto: RepuestoDto, imagen: ImagenSeleccionada?): Result<RepuestoDto> =
        llamar { api.actualizar(id, dto.aParteJson(), imagen?.aParte()).repuestoServicio }

    suspend fun eliminar(id: Int): Result<String> =
        llamar { api.eliminar(id).message }

    // Parte "dto": el JSON debe declararse como application/json
    private fun RepuestoDto.aParteJson(): RequestBody =
        gson.toJson(this).toRequestBody("application/json".toMediaType())

    // Parte "imagen": un archivo con su nombre y su tipo
    private fun ImagenSeleccionada.aParte(): MultipartBody.Part {
        val extension = if (mime == "image/png") "png" else "jpg"
        return MultipartBody.Part.createFormData(
            "imagen", "imagen.$extension", bytes.toRequestBody(mime.toMediaType())
        )
    }

    private suspend fun <T> llamar(bloque: suspend () -> T): Result<T> = try {
        Result.success(bloque())
    } catch (e: HttpException) {
        Result.failure(Exception(mensajeHttp(e.code())))
    } catch (e: IOException) {
        Result.failure(Exception("No se pudo conectar con el servidor. Revise adb reverse y el contenedor."))
    } catch (e: JsonParseException) {
        Result.failure(Exception("La respuesta del servidor no tiene el formato esperado."))
    }

    private fun mensajeHttp(codigo: Int) = when (codigo) {
        400 -> "Datos inválidos: revise los campos (nombre, precio, stock, tipo)."
        401, 403 -> "Sin permiso: la API exige autenticación."
        404 -> "El repuesto o servicio ya no existe."
        413 -> "La imagen es demasiado grande para el servidor."
        415 -> "El servidor rechazó el tipo de contenido de la petición."
        else -> "Error del servidor ($codigo)."
    }
}
```

- `@Inject constructor` le dice a Hilt cómo construir el repositorio. Su único parámetro, `RepuestoApi`, lo entrega `NetworkModule`: es la cadena que en la versión anterior se armaba con un valor por defecto.
- Los imports nuevos son `javax.inject.Inject` y, desde el paquete `data.remote`, `RepuestoApi` y `RepuestoDto`; ya no se importa nada de red como `RetrofitClient`.
- Los repositorios llevan **solo** `@Inject constructor`. La anotación `@HiltViewModel` va únicamente en el `ViewModel`.

## Paso 5: el ViewModel

Un solo `RepuestosViewModel` atiende la lista y el formulario. Expone cuatro estados: la lista (con `cargando` y `error`), el detalle que se edita, si hay un guardado en curso y un mensaje de una sola vez para el `Snackbar`.

Cree `viewmodel/RepuestosViewModel.kt`:

```kotlin
package com.example.autoparts.viewmodel

import androidx.lifecycle.ViewModel
import dagger.hilt.android.lifecycle.HiltViewModel
import javax.inject.Inject
import androidx.lifecycle.viewModelScope
import com.example.autoparts.repository.ImagenSeleccionada
import com.example.autoparts.repository.RepuestoRepository
import com.example.autoparts.data.remote.RepuestoDto
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch

data class ListaUiState(
    val cargando: Boolean = false,
    val repuestos: List<RepuestoDto> = emptyList(),
    val error: String? = null
)

@HiltViewModel
class RepuestosViewModel @Inject constructor(
    private val repositorio: RepuestoRepository
) : ViewModel() {

    private val _estado = MutableStateFlow(ListaUiState())
    val estado: StateFlow<ListaUiState> = _estado.asStateFlow()

    private val _detalle = MutableStateFlow<RepuestoDto?>(null)
    val detalle: StateFlow<RepuestoDto?> = _detalle.asStateFlow()

    private val _guardando = MutableStateFlow(false)
    val guardando: StateFlow<Boolean> = _guardando.asStateFlow()

    private val _mensaje = MutableStateFlow<String?>(null)
    val mensaje: StateFlow<String?> = _mensaje.asStateFlow()

    init { cargar() }

    fun cargar() {
        viewModelScope.launch {
            _estado.update { it.copy(cargando = true, error = null) }
            repositorio.obtenerTodos()
                .onSuccess { lista ->
                    _estado.update { it.copy(cargando = false, repuestos = lista) }
                }
                .onFailure { e ->
                    _estado.update { it.copy(cargando = false, error = e.message) }
                }
        }
    }

    // GET /repuestos-servicios/{id}: datos frescos del servidor para el formulario de edición
    fun cargarDetalle(id: Int) {
        viewModelScope.launch {
            repositorio.obtenerPorId(id)
                .onSuccess { _detalle.value = it }
                .onFailure { _mensaje.value = it.message }
        }
    }

    fun limpiarDetalle() { _detalle.value = null }

    // id == null: crea (POST); id != null: edita (PUT)
    fun guardar(
        id: Int?,
        dto: RepuestoDto,
        imagen: ImagenSeleccionada?,
        alTerminar: () -> Unit
    ) {
        viewModelScope.launch {
            _guardando.value = true
            val resultado =
                if (id == null) repositorio.crear(dto, imagen)
                else repositorio.actualizar(id, dto, imagen)
            _guardando.value = false

            resultado
                .onSuccess {
                    _mensaje.value =
                        if (id == null) "Registrado correctamente" else "Actualizado correctamente"
                    cargar()
                    alTerminar()
                }
                .onFailure { _mensaje.value = it.message }
        }
    }

    fun eliminar(id: Int) {
        viewModelScope.launch {
            repositorio.eliminar(id)
                .onSuccess { mensaje -> _mensaje.value = mensaje; cargar() }
                .onFailure { _mensaje.value = it.message }
        }
    }

    fun mensajeMostrado() { _mensaje.value = null }
}
```

Con Hilt, el repositorio llega por el constructor (@Inject) y la anotación @HiltViewModel le enseña a Hilt a construir este ViewModel; por eso se puede crear con `viewModel()` sin fábrica, que es lo que se usa en `MainActivity`.

Después de crear, editar o eliminar, el ViewModel vuelve a pedir la lista completa al servidor (`cargar()`), de modo que la pantalla siempre muestra lo que el servidor tiene realmente.
