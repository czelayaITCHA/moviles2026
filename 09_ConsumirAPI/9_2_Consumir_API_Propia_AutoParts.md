# 9\_2 - Retrofit con API propia (AutoParts)

## Objetivo

Al terminar, **AutoParts** es un catálogo de repuestos y servicios con **barra de navegación inferior** (Catálogo, Repuestos y Servicios), un **buscador** que filtra mientras se escribe, una **vista de detalle** que se abre al tocar una tarjeta, y permite crear, editar y eliminar registros consumiendo la API Spring Boot que ya corre en Docker dentro de WSL, sin autenticación (el login se agrega en la guía 9\_3).

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

El proyecto sigue la misma arquitectura de la `AgendaApp`: **Hilt** (guía `8_1`) entrega `Retrofit`, el repositorio y el `ViewModel`, igual que en la guía `9_1`. Para centrarse en Retrofit y en el envío de imágenes, **no usa Room ni DataStore**.

Las pantallas de la app:

| Pantalla | Cómo se llega | Qué muestra o hace |
| --- | --- | --- |
| **Catálogo**, **Repuestos** y **Servicios** | Barra de navegación inferior | Lista de tarjetas con buscador. Catálogo muestra todo; las otras dos filtran por tipo |
| **Detalle** | Tocar una tarjeta | Foto grande, tipo, precio, existencias y descripción completa, con las acciones Editar y Eliminar (`GET /repuestos-servicios/{id}`) |
| **Formulario** | Botón **+** o Editar en el detalle | Crear o editar con foto de galería |

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

* Recuerda recompilar el servicio de la API en docker, de la siguiente manera:
1. Detener los contenedores, en caso de estar iniciados
   ```bash
   docker compose down
   ```
2. compilar el servicio de la API
   ```bash
   docker compose up -d --build --no-deps api
   ```
3. iniciar los contenedores
```bash
docker compose up -d
```

Ahora si haces esto, desde una ventana PowerShell  de la PC anfitrión
```bash
Invoke-RestMethod http://localhost:8080/api/repuestos-servicios | Format-Table id, nombre, precio, tipo
```
Debes ver como resultado algo similar a:

<img width="1101" height="260" alt="image" src="https://github.com/user-attachments/assets/fcfb28cf-a3da-4944-9d2b-aed52f01e200" />


### 0.3 Enlazar el puerto con adb reverse

Conecte el teléfono por USB con la depuración USB activada y ejecute, desde la carpeta `platform-tools` del SDK (normalmente `%LOCALAPPDATA%\Android\Sdk\platform-tools`):

```powershell
adb devices
adb reverse tcp:8080 tcp:8080
```

Prueba rápida: abra Chrome en el teléfono y entre a `http://localhost:8080/api/repuestos-servicios`. Si muestra el JSON, la conexión está lista.

<img width="384" height="853" alt="image" src="https://github.com/user-attachments/assets/fc8c3192-dd7f-413d-aae3-0b96d9cfe064" />

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
├── MainActivity.kt                 (@AndroidEntryPoint; llama a NavManager)
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
├── navigation/
│    ├── Destino.kt                 (las 3 pestañas de la barra inferior)
│    └── NavManager.kt              (Scaffold + NavigationBar + NavHost)
└── ui/
     ├── components/RepuestoCard.kt
     └── views/
          ├── ListaRepuestosView.kt      (buscador y filtro por tipo)
          ├── DetalleRepuestoView.kt
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

class RepuestoRepository @Inject constructor(      
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

Un solo `RepuestosViewModel` atiende la lista, el detalle y el formulario. Expone cuatro estados: la lista (con `cargando` y `error`), el detalle que se edita, si hay un guardado en curso y un mensaje de una sola vez para el `Snackbar`.

Cree `viewmodel/RepuestosViewModel.kt`:

```kotlin
package com.example.autoparts.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.autoparts.data.remote.RepuestoDto
import com.example.autoparts.repository.ImagenSeleccionada
import com.example.autoparts.repository.RepuestoRepository
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.update
import kotlinx.coroutines.launch
import javax.inject.Inject

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

    // Ítem que muestran el detalle y el formulario de edición
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

    // GET /repuestos-servicios/{id}: datos frescos del servidor para el detalle y la edición
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
                .onSuccess { guardado ->
                    _detalle.value = guardado         
                    _mensaje.value =
                        if (id == null) "Registrado correctamente" else "Actualizado correctamente"
                    cargar()
                    alTerminar()
                }
                .onFailure { _mensaje.value = it.message }
        }
    }

    // avisa cuando termina bien, para que el detalle pueda volver a la lista
    fun eliminar(id: Int, alTerminar: () -> Unit = {}) {
        viewModelScope.launch {
            repositorio.eliminar(id)
                .onSuccess { mensaje ->
                    _mensaje.value = mensaje
                    cargar()
                    alTerminar()
                }
                .onFailure { _mensaje.value = it.message }
        }
    }

    fun mensajeMostrado() { _mensaje.value = null }
}
```

Con Hilt, el repositorio llega por el constructor (@Inject) y la anotación @HiltViewModel le enseña a Hilt a construir este ViewModel; por eso se puede crear con `viewModel()` sin fábrica, que es lo que se usa en `MainActivity`.

Después de crear, editar o eliminar, el ViewModel vuelve a pedir la lista completa al servidor (`cargar()`), de modo que la pantalla siempre muestra lo que el servidor tiene realmente.

## Paso 6: la tarjeta, la lista con buscador y el detalle

La lista usa `LazyColumn` con una tarjeta por ítem y un **buscador** arriba. Una misma pantalla, `ListaRepuestosView`, sirve para las tres pestañas: recibe el `tipo` que debe mostrar (`null` para todos) y filtra primero por tipo y luego por el texto escrito. Coil descarga cada foto desde `http://localhost:8080/images/{foto}` y, cuando `foto` es `null` (todos los servicios y algún repuesto), se muestra un icono en su lugar. Al tocar una tarjeta se abre el **detalle** del ítem.

En estos archivos Android Studio importa casi todo con **Alt+Enter**. Estos dos imports son fáciles de equivocar: `coil3.compose.AsyncImage` y `androidx.lifecycle.compose.collectAsStateWithLifecycle`.

### components/RepuestoCard.kt

```kotlin
package com.example.autoparts.ui.components

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun RepuestoCard(
    item: RepuestoDto,
    onClick: () -> Unit,                // toda la tarjeta es tocable: abre el detalle
    modifier: Modifier = Modifier
) {
    ElevatedCard(onClick = onClick, modifier = modifier.fillMaxWidth()) {
        Row(
            modifier = Modifier.padding(12.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Box(
                modifier = Modifier
                    .size(72.dp)
                    .clip(RoundedCornerShape(12.dp))
                    .background(MaterialTheme.colorScheme.surfaceVariant),
                contentAlignment = Alignment.Center
            ) {
                val url = ApiConfig.urlFoto(item.foto)
                if (url != null) {
                    AsyncImage(
                        model = url,
                        contentDescription = item.nombre,
                        contentScale = ContentScale.Crop,
                        modifier = Modifier.fillMaxSize()
                    )
                } else {
                    Icon(Icons.Default.Build, contentDescription = null)
                }
            }

            Column(modifier = Modifier.weight(1f).padding(start = 12.dp)) {
                Text(item.nombre, style = MaterialTheme.typography.titleMedium)
                if (!item.descripcion.isNullOrBlank()) {
                    Text(
                        item.descripcion,
                        style = MaterialTheme.typography.bodySmall,
                        maxLines = 2,
                        overflow = TextOverflow.Ellipsis
                    )
                }
                Text(
                    "$" + "%.2f".format(item.precio),
                    style = MaterialTheme.typography.titleSmall,
                    color = MaterialTheme.colorScheme.primary
                )
                Text(
                    if (item.tipo == TipoItem.REPUESTO) "Repuesto · Stock: ${item.stock}"
                    else "Servicio",
                    style = MaterialTheme.typography.labelMedium
                )
            }
        }
    }
}
```

La tarjeta ya no lleva los botones de editar y eliminar: esas acciones pasan al detalle. No se necesita `material-icons-extended`: `Build`, `Edit`, `Delete`, `Add`, `Refresh`, `Search`, `Clear`, `ShoppingCart`, `Settings` y `ArrowBack` están en el conjunto básico de iconos.

### views/ListaRepuestosView.kt

```kotlin
package com.example.autoparts.ui.views

// Imports poco evidentes: androidx.compose.runtime.saveable.rememberSaveable y
// androidx.compose.foundation.layout.WindowInsets
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ListaRepuestosView(
    viewModel: RepuestosViewModel,
    tipo: TipoItem?,                  // null = todos; REPUESTO o SERVICIO = solo ese tipo
    titulo: String,
    onAbrirDetalle: (Int) -> Unit,
    onNuevo: () -> Unit
) {
    val estado by viewModel.estado.collectAsStateWithLifecycle()
    val mensaje by viewModel.mensaje.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }
    var busqueda by rememberSaveable { mutableStateOf("") }      // sobrevive a girar la pantalla

    // Filtro: primero por tipo (la pestaña) y luego por el texto escrito
    val visibles = remember(estado.repuestos, tipo, busqueda) {
        val texto = busqueda.trim()
        estado.repuestos.filter { item ->
            (tipo == null || item.tipo == tipo) &&
                (texto.isEmpty() ||
                    item.nombre.contains(texto, ignoreCase = true) ||
                    item.descripcion.orEmpty().contains(texto, ignoreCase = true))
        }
    }

    // Muestra cada mensaje del ViewModel una sola vez
    LaunchedEffect(mensaje) {
        mensaje?.let {
            snackbarHostState.showSnackbar(it)
            viewModel.mensajeMostrado()
        }
    }

    Scaffold(
        contentWindowInsets = WindowInsets(0, 0, 0, 0),   // la barra inferior ya cuida ese espacio
        topBar = {
            TopAppBar(
                title = { Text(titulo) },
                actions = {
                    IconButton(onClick = { viewModel.cargar() }) {
                        Icon(Icons.Default.Refresh, contentDescription = "Actualizar")
                    }
                }
            )
        },
        floatingActionButton = {
            FloatingActionButton(onClick = onNuevo) {
                Icon(Icons.Default.Add, contentDescription = "Nuevo")
            }
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { padding ->
        Column(modifier = Modifier.padding(padding)) {
            if (estado.cargando) LinearProgressIndicator(modifier = Modifier.fillMaxWidth())

            // Buscador tipo píldora
            OutlinedTextField(
                value = busqueda,
                onValueChange = { busqueda = it },
                placeholder = { Text("Buscar por nombre o descripción") },
                leadingIcon = { Icon(Icons.Default.Search, contentDescription = null) },
                trailingIcon = {
                    if (busqueda.isNotEmpty()) {
                        IconButton(onClick = { busqueda = "" }) {
                            Icon(Icons.Default.Clear, contentDescription = "Borrar búsqueda")
                        }
                    }
                },
                singleLine = true,
                shape = RoundedCornerShape(50),
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(horizontal = 16.dp, vertical = 8.dp)
            )

            when {
                estado.error != null && estado.repuestos.isEmpty() -> {
                    Column(
                        modifier = Modifier.fillMaxSize().padding(24.dp),
                        verticalArrangement = Arrangement.Center,
                        horizontalAlignment = Alignment.CenterHorizontally
                    ) {
                        Text(estado.error!!, textAlign = TextAlign.Center)
                        Spacer(Modifier.height(12.dp))
                        Button(onClick = { viewModel.cargar() }) { Text("Reintentar") }
                    }
                }
                visibles.isEmpty() && !estado.cargando -> {
                    Box(Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
                        Text(
                            if (busqueda.isBlank()) "Aún no hay registros"
                            else "Sin resultados para \"$busqueda\""
                        )
                    }
                }
                else -> {
                    LazyColumn(
                        contentPadding = PaddingValues(start = 16.dp, end = 16.dp, bottom = 16.dp),
                        verticalArrangement = Arrangement.spacedBy(12.dp)
                    ) {
                        items(visibles, key = { it.id ?: it.nombre }) { item ->
                            RepuestoCard(
                                item = item,
                                onClick = { item.id?.let(onAbrirDetalle) }
                            )
                        }
                    }
                }
            }
        }
    }
}
```

Con los datos de ejemplo del servidor, la pestaña **Catálogo** muestra ocho tarjetas, **Repuestos** muestra las cinco que tienen foto y **Servicios** las tres que usan el icono de herramienta. Si escribe `freno` en el buscador, quedan tres: *Pastillas de freno delanteras*, *Disco de frenos* y *Mantenimiento de frenos*. El buscador ignora mayúsculas y minúsculas, pero no los acentos: `limpieza` encuentra *Limpieza de inyectores*, pero `diametro` no encuentra la descripción "díametro 266mm" de *Disco de frenos*.

La pantalla de error con el botón **Reintentar** es la que verá el estudiante cuando falte `adb reverse` o el contenedor esté detenido; el mensaje ya le indica qué revisar.

### views/DetalleRepuestoView.kt

Al tocar una tarjeta se abre esta pantalla. Pide el ítem completo al servidor con `GET /repuestos-servicios/{id}` (`cargarDetalle`) y muestra todos sus datos. Desde aquí se **edita** (abre el formulario) y se **elimina** (con confirmación).

```kotlin
package com.example.autoparts.ui.views

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun DetalleRepuestoView(
    viewModel: RepuestosViewModel,
    id: Int,
    onVolver: () -> Unit,
    onEditar: () -> Unit
) {
    val detalle by viewModel.detalle.collectAsStateWithLifecycle()
    val mensaje by viewModel.mensaje.collectAsStateWithLifecycle()
    val snackbarHostState = remember { SnackbarHostState() }
    var confirmarBorrado by remember { mutableStateOf(false) }

    // Pide al servidor el ítem completo cada vez que se entra a la pantalla
    LaunchedEffect(id) { viewModel.cargarDetalle(id) }

    // Si el ViewModel todavía guarda el ítem de otra pantalla, se ignora hasta que llegue el correcto
    val item = detalle?.takeIf { it.id == id }

    LaunchedEffect(mensaje) {
        mensaje?.let {
            snackbarHostState.showSnackbar(it)
            viewModel.mensajeMostrado()
        }
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(item?.nombre ?: "Detalle") },
                navigationIcon = {
                    IconButton(onClick = onVolver) {
                        Icon(Icons.AutoMirrored.Filled.ArrowBack, contentDescription = "Volver")
                    }
                },
                actions = {
                    if (item != null) {
                        IconButton(onClick = onEditar) {
                            Icon(Icons.Default.Edit, contentDescription = "Editar")
                        }
                        IconButton(onClick = { confirmarBorrado = true }) {
                            Icon(Icons.Default.Delete, contentDescription = "Eliminar")
                        }
                    }
                }
            )
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { padding ->
        if (item == null) {
            Box(Modifier.fillMaxSize().padding(padding), contentAlignment = Alignment.Center) {
                CircularProgressIndicator()
            }
        } else {
            Column(
                modifier = Modifier
                    .padding(padding)
                    .verticalScroll(rememberScrollState())
            ) {
                // Foto grande
                Box(
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(260.dp)
                        .background(MaterialTheme.colorScheme.surfaceVariant),
                    contentAlignment = Alignment.Center
                ) {
                    val url = ApiConfig.urlFoto(item.foto)
                    if (url != null) {
                        AsyncImage(
                            model = url,
                            contentDescription = item.nombre,
                            contentScale = ContentScale.Crop,
                            modifier = Modifier.fillMaxSize()
                        )
                    } else {
                        Icon(Icons.Default.Build, contentDescription = null, modifier = Modifier.size(64.dp))
                    }
                }

                Column(
                    modifier = Modifier.padding(16.dp),
                    verticalArrangement = Arrangement.spacedBy(12.dp)
                ) {
                    Text(item.nombre, style = MaterialTheme.typography.headlineSmall)

                    FilaDato("Tipo", if (item.tipo == TipoItem.REPUESTO) "Repuesto" else "Servicio")
                    FilaDato("Precio", "$" + "%.2f".format(item.precio))
                    if (item.tipo == TipoItem.REPUESTO) {
                        FilaDato("Existencias", if (item.stock == 0) "Sin existencias" else "${item.stock} unidades")
                    }
                    FilaDato("Código", "#${item.id}")

                    HorizontalDivider()

                    Text("Descripción", style = MaterialTheme.typography.titleMedium)
                    Text(
                        item.descripcion?.takeIf { it.isNotBlank() } ?: "Sin descripción",
                        style = MaterialTheme.typography.bodyLarge
                    )
                }
            }
        }
    }

    if (confirmarBorrado && item != null) {
        AlertDialog(
            onDismissRequest = { confirmarBorrado = false },
            title = { Text("Eliminar registro") },
            text = { Text("¿Eliminar \"${item.nombre}\"? Esta acción no se puede deshacer.") },
            confirmButton = {
                TextButton(onClick = {
                    confirmarBorrado = false
                    viewModel.eliminar(id, onVolver)      // vuelve a la lista solo si se eliminó bien
                }) { Text("Eliminar") }
            },
            dismissButton = {
                TextButton(onClick = { confirmarBorrado = false }) { Text("Cancelar") }
            }
        )
    }
}

@Composable
private fun FilaDato(etiqueta: String, valor: String) {
    Row(
        modifier = Modifier.fillMaxWidth(),
        horizontalArrangement = Arrangement.SpaceBetween
    ) {
        Text(etiqueta, color = MaterialTheme.colorScheme.onSurfaceVariant)
        Text(valor, fontWeight = FontWeight.SemiBold)
    }
}
```

- `detalle?.takeIf { it.id == id }` evita mostrar por un instante los datos de otro ítem que quedaron en el `ViewModel`; mientras llega el correcto se ve el indicador de carga.
- Al **editar y guardar**, el `ViewModel` actualiza `detalle` con la respuesta del servidor, así que al volver a esta pantalla ya aparecen los datos nuevos.
- Al **eliminar**, `eliminar(id, onVolver)` regresa a la lista solo si el servidor respondió bien; si falla, se queda aquí y el mensaje aparece en el `Snackbar`.
- Este `Scaffold` no cambia `contentWindowInsets`: aquí no hay barra inferior, así que la pantalla debe respetar por sí misma el espacio de los botones del sistema.

## Paso 7: formulario para crear y editar con imagen

Un solo formulario sirve para las dos operaciones: con `id == null` crea (POST) y con un `id` edita (PUT). La foto se elige de la galería con `PickVisualMedia`, que no requiere permisos, y sus bytes se leen en segundo plano para enviarlos como parte `imagen`.

Reglas del formulario:

- `nombre` obligatorio, `precio` mayor que 0, `stock` entero desde 0.
- Si el tipo es `SERVICIO`, el campo stock se oculta y se envía `0`.
- Al editar, si no se elige una foto nueva no se envía la parte `imagen` y el servidor conserva la anterior.

### views/FormularioRepuestoView.kt

```kotlin
package com.example.autoparts.ui.views

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun FormularioRepuestoView(
    viewModel: RepuestosViewModel,
    id: Int?,                 // null = nuevo, valor = editar
    onVolver: () -> Unit
) {
    val context = LocalContext.current
    val scope = rememberCoroutineScope()
    val snackbarHostState = remember { SnackbarHostState() }

    val detalle by viewModel.detalle.collectAsStateWithLifecycle()
    val guardando by viewModel.guardando.collectAsStateWithLifecycle()
    val mensaje by viewModel.mensaje.collectAsStateWithLifecycle()

    var nombre by remember { mutableStateOf("") }
    var descripcion by remember { mutableStateOf("") }
    var precioTexto by remember { mutableStateOf("") }
    var stockTexto by remember { mutableStateOf("") }
    var tipo by remember { mutableStateOf(TipoItem.REPUESTO) }
    var fotoActual by remember { mutableStateOf<String?>(null) }
    var imagenUri by remember { mutableStateOf<android.net.Uri?>(null) }
    var imagen by remember { mutableStateOf<ImagenSeleccionada?>(null) }

    // Al editar: pedir el ítem al servidor (GET por id) y llenar los campos
    LaunchedEffect(id) {
        if (id != null) viewModel.cargarDetalle(id) else viewModel.limpiarDetalle()
    }
    LaunchedEffect(detalle) {
        detalle?.takeIf { it.id == id }?.let {
            nombre = it.nombre
            descripcion = it.descripcion.orEmpty()
            precioTexto = it.precio.toString()
            stockTexto = it.stock.toString()
            tipo = it.tipo
            fotoActual = it.foto
        }
    }
    

    LaunchedEffect(mensaje) {
        mensaje?.let {
            snackbarHostState.showSnackbar(it)
            viewModel.mensajeMostrado()
        }
    }

    // Galería: lee los bytes de la imagen elegida fuera del hilo principal
    val selector = rememberLauncherForActivityResult(
        ActivityResultContracts.PickVisualMedia()
    ) { uri ->
        if (uri != null) {
            imagenUri = uri
            scope.launch {
                imagen = withContext(Dispatchers.IO) {
                    val mime = context.contentResolver.getType(uri) ?: "image/jpeg"
                    context.contentResolver.openInputStream(uri)
                        ?.use { ImagenSeleccionada(it.readBytes(), mime) }
                }
            }
        }
    }

    val precio = precioTexto.toDoubleOrNull()
    val stock = if (tipo == TipoItem.SERVICIO) 0 else stockTexto.toIntOrNull()
    val valido = nombre.isNotBlank() && precio != null && precio > 0 &&
        stock != null && stock >= 0

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(if (id == null) "Nuevo registro" else "Editar registro") },
                navigationIcon = {
                    IconButton(onClick = onVolver) {
                        Icon(Icons.AutoMirrored.Filled.ArrowBack, contentDescription = "Volver")
                    }
                }
            )
        },
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { padding ->
        Column(
            modifier = Modifier
                .padding(padding)
                .padding(16.dp)
                .verticalScroll(rememberScrollState()),
            verticalArrangement = Arrangement.spacedBy(12.dp)
        ) {
            // Vista previa: la foto recién elegida o, si no hay, la que ya tiene en el servidor
            Box(
                modifier = Modifier
                    .fillMaxWidth()
                    .height(180.dp)
                    .clip(RoundedCornerShape(12.dp))
                    .background(MaterialTheme.colorScheme.surfaceVariant),
                contentAlignment = Alignment.Center
            ) {
                val modelo: Any? = imagenUri ?: ApiConfig.urlFoto(fotoActual)
                if (modelo != null) {
                    AsyncImage(
                        model = modelo,
                        contentDescription = "Foto",
                        contentScale = ContentScale.Crop,
                        modifier = Modifier.fillMaxSize()
                    )
                } else {
                    Text("Sin foto")
                }
            }
            OutlinedButton(
                onClick = {
                    selector.launch(
                        PickVisualMediaRequest(ActivityResultContracts.PickVisualMedia.ImageOnly)
                    )
                },
                modifier = Modifier.fillMaxWidth()
            ) { Text(if (imagenUri == null) "Elegir foto" else "Cambiar foto") }

            // Tipo
            Row(verticalAlignment = Alignment.CenterVertically) {
                TipoItem.entries.forEach { opcion ->
                    RadioButton(selected = tipo == opcion, onClick = { tipo = opcion })
                    Text(
                        opcion.name.lowercase().replaceFirstChar { it.uppercase() },
                        modifier = Modifier.padding(end = 16.dp)
                    )
                }
            }

            OutlinedTextField(
                value = nombre,
                onValueChange = { if (it.length <= 100) nombre = it },
                label = { Text("Nombre") },
                singleLine = true,
                modifier = Modifier.fillMaxWidth()
            )
            OutlinedTextField(
                value = descripcion,
                onValueChange = { if (it.length <= 250) descripcion = it },
                label = { Text("Descripción (opcional)") },
                modifier = Modifier.fillMaxWidth()
            )
            OutlinedTextField(
                value = precioTexto,
                onValueChange = { precioTexto = it },
                label = { Text("Precio") },
                singleLine = true,
                keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Decimal),
                modifier = Modifier.fillMaxWidth()
            )
            if (tipo == TipoItem.REPUESTO) {
                OutlinedTextField(
                    value = stockTexto,
                    onValueChange = { stockTexto = it },
                    label = { Text("Stock") },
                    singleLine = true,
                    keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number),
                    modifier = Modifier.fillMaxWidth()
                )
            }

            Button(
                enabled = valido && !guardando,
                onClick = {
                    val dto = RepuestoDto(
                        id = id,
                        nombre = nombre.trim(),
                        descripcion = descripcion.trim().ifBlank { null },
                        precio = precio!!,
                        stock = stock!!,
                        foto = fotoActual,
                        tipo = tipo
                    )
                    viewModel.guardar(id, dto, imagen, onVolver)
                },
                modifier = Modifier.fillMaxWidth()
            ) { Text(if (guardando) "Guardando..." else "Guardar") }
        }
    }
}
```

Puntos que conviene explicar en clase:

- `imagenUri` sirve solo para la vista previa; lo que viaja al servidor es `imagen` (los bytes).
- Si el usuario no toca la galería, `imagen` queda en `null` y el repositorio no envía la parte `imagen`.
- `TipoItem.entries` recorre el enum para dibujar una opción por cada valor, de modo que si el servidor agrega un tipo nuevo, solo cambia el enum.

### Límite de tamaño en el servidor

Spring Boot acepta por defecto archivos de hasta 1 MB, y una foto de teléfono suele pasar de ese tamaño. Si la subida falla con `413` o `500`, el docente sube el límite en `application.properties` del proyecto Spring:

```properties
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

## Paso 8: barra de navegación inferior y navegación

Este paso conecta todo: la **barra inferior** (`NavigationBar`) con las tres pestañas y un `NavHost` con cinco rutas. Es la misma estructura que se usó en la guía `5_1`: un `Scaffold` raíz con la barra y, dentro, el `NavHost` con las pantallas.

### navigation/Destino.kt

Cada pestaña de la barra inferior se describe con una ruta, un título y un icono. Una `sealed class` agrupa las tres y evita escribir las rutas como texto suelto en varios archivos. El detalle y el formulario no están aquí porque no son pestañas.

```kotlin
package com.example.autoparts.navigation

import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Build
import androidx.compose.material.icons.filled.Settings
import androidx.compose.material.icons.filled.ShoppingCart
import androidx.compose.ui.graphics.vector.ImageVector

sealed class Destino(val ruta: String, val titulo: String, val icono: ImageVector) {
    data object Catalogo : Destino("catalogo", "Catálogo", Icons.Default.ShoppingCart)
    data object Repuestos : Destino("repuestos", "Repuestos", Icons.Default.Build)
    data object Servicios : Destino("servicios", "Servicios", Icons.Default.Settings)
}

val destinosInferiores = listOf(Destino.Catalogo, Destino.Repuestos, Destino.Servicios)
```

### navigation/NavManager.kt

Aquí vive el `Scaffold` raíz. La barra inferior se dibuja **solo cuando la ruta actual es una pestaña**: en el detalle y en el formulario desaparece, para dejar espacio al contenido.

```kotlin
package com.example.autoparts.navigation

@Composable
fun NavManager(viewModel: RepuestosViewModel) {
    val navController = rememberNavController()
    val entradaActual by navController.currentBackStackEntryAsState()
    val rutaActual = entradaActual?.destination?.route

    val abrirDetalle = { id: Int -> navController.navigate("detalle/$id") }
    val abrirNuevo = { navController.navigate("formulario") }

    Scaffold(
        contentWindowInsets = WindowInsets(0, 0, 0, 0),
        bottomBar = {
            if (destinosInferiores.any { it.ruta == rutaActual }) {
                NavigationBar {
                    destinosInferiores.forEach { destino ->
                        NavigationBarItem(
                            selected = rutaActual == destino.ruta,
                            onClick = {
                                navController.navigate(destino.ruta) {
                                    // Evita apilar pestañas y conserva el estado de cada una (por ejemplo, el buscador)
                                    popUpTo(navController.graph.findStartDestination().id) {
                                        saveState = true
                                    }
                                    launchSingleTop = true
                                    restoreState = true
                                }
                            },
                            icon = { Icon(destino.icono, contentDescription = destino.titulo) },
                            label = { Text(destino.titulo) }
                        )
                    }
                }
            }
        }
    ) { innerPadding ->
        NavHost(
            navController = navController,
            startDestination = Destino.Catalogo.ruta,
            modifier = Modifier.padding(innerPadding)
        ) {
            // Las tres pestañas usan la misma pantalla, con distinto tipo
            composable(Destino.Catalogo.ruta) {
                ListaRepuestosView(viewModel, tipo = null, titulo = Destino.Catalogo.titulo,
                    onAbrirDetalle = abrirDetalle, onNuevo = abrirNuevo)
            }
            composable(Destino.Repuestos.ruta) {
                ListaRepuestosView(viewModel, tipo = TipoItem.REPUESTO, titulo = Destino.Repuestos.titulo,
                    onAbrirDetalle = abrirDetalle, onNuevo = abrirNuevo)
            }
            composable(Destino.Servicios.ruta) {
                ListaRepuestosView(viewModel, tipo = TipoItem.SERVICIO, titulo = Destino.Servicios.titulo,
                    onAbrirDetalle = abrirDetalle, onNuevo = abrirNuevo)
            }

            composable(
                route = "detalle/{id}",
                arguments = listOf(navArgument("id") { type = NavType.IntType })
            ) { entrada ->
                val id = entrada.arguments?.getInt("id") ?: return@composable
                DetalleRepuestoView(
                    viewModel = viewModel,
                    id = id,
                    onVolver = { navController.popBackStack() },
                    onEditar = { navController.navigate("formulario?id=$id") }
                )
            }

            composable(
                route = "formulario?id={id}",
                arguments = listOf(navArgument("id") {
                    type = NavType.IntType
                    defaultValue = -1          // -1 significa "registro nuevo"
                })
            ) { entrada ->
                val id = entrada.arguments?.getInt("id")?.takeIf { it != -1 }
                FormularioRepuestoView(
                    viewModel = viewModel,
                    id = id,
                    onVolver = { navController.popBackStack() }
                )
            }
        }
    }
}
```

- **Las rutas:** `catalogo`, `repuestos` y `servicios` son las pestañas; `detalle/{id}` recibe el identificador del ítem; `formulario` abre un registro nuevo y `formulario?id=5` edita el ítem 5.
- **`saveState` y `restoreState`:** al cambiar de pestaña y volver, la lista conserva su posición y el texto del buscador, porque se guardan junto con la entrada de la pila.
- **Márgenes del sistema (`WindowInsets`):** el `Scaffold` raíz y el de `ListaRepuestosView` usan `WindowInsets(0, 0, 0, 0)` porque la `NavigationBar` ya reserva el espacio de los botones del sistema; si se dejara el valor por defecto en ambos, quedaría un espacio doble debajo de la lista. El detalle y el formulario sí lo conservan, porque ahí no hay barra inferior.
- **Un solo `ViewModel`:** las pantallas lo reciben como parámetro y comparten la lista, el detalle y los mensajes.

### MainActivity.kt

`MainActivity` queda muy corta: solo crea el `ViewModel` una vez, a nivel de la actividad, y se lo entrega a `NavManager`. Como la actividad lleva `@AndroidEntryPoint`, `viewModel()` usa a Hilt de forma automática y entrega el `ViewModel` con su repositorio ya armado.

```kotlin
package com.example.autoparts

@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            AutoPartsTheme {
                val viewModel: RepuestosViewModel = viewModel()
                NavManager(viewModel)
            }
        }
    }
}
```

El nombre del tema (`AutoPartsTheme`) es el que generó la plantilla para este proyecto; si el suyo se llama distinto, use ese.

### Lista de comprobación

Conecte el teléfono, ejecute `adb reverse tcp:8080 tcp:8080` y pruebe cada caso:

| Acción | Resultado esperado |
| --- | --- |
| Abrir la app | Pestaña **Catálogo** con 8 tarjetas y la barra inferior con las 3 pestañas |
| Tocar **Repuestos** y luego **Servicios** | Cinco tarjetas con foto; luego tres con el icono de herramienta |
| Escribir `freno` en el buscador | Quedan 3 tarjetas; la **X** borra el texto |
| Escribir un texto que no existe | Aparece "Sin resultados para ..." |
| Tocar una tarjeta | Se abre el **detalle**: foto grande, tipo, precio, existencias y descripción completa; la barra inferior desaparece |
| Flecha de retroceso en el detalle | Vuelve a la misma pestaña, con el buscador como se dejó |
| Botón **+** y crear un servicio sin foto | "Registrado correctamente"; el nuevo servicio aparece en Catálogo y en Servicios |
| Detalle, **Editar** y cambiar el precio | Al guardar vuelve al detalle con el precio nuevo |
| Detalle y **Eliminar** | Pide confirmación, regresa a la lista y el servidor responde `200` |
| Detener el contenedor y tocar Actualizar | Mensaje de error con el botón Reintentar |

## Errores frecuentes y soluciones

La mayoría de los fallos en el primer intento son de conexión o de configuración del servidor, no del código Kotlin. Cuando algo falle, mire primero Logcat (filtre por `OkHttp`) para ver el código HTTP que respondió el servidor.

| Síntoma | Causa probable | Solución |
| --- | --- | --- |
| `CLEARTEXT communication to localhost not permitted` | Falta habilitar HTTP sin cifrar | Agregar `android:usesCleartextTraffic="true"` en `<application>` (Paso 0.4) |
| `Failed to connect to localhost/127.0.0.1:8080` | `adb reverse` no se ejecutó o se perdió, o el contenedor está detenido | Repetir `adb reverse tcp:8080 tcp:8080`, revisar `docker ps` y probar la URL en Chrome del teléfono |
| `IllegalArgumentException: baseUrl must end in /` | La URL base no termina en `/` | Escribir `http://localhost:8080/api/` |
| La lista carga pero las fotos salen en blanco | Falta `coil-network-okhttp`, o `/images/` exige autenticación | Agregar la dependencia (Paso 1.2) y probar `Invoke-WebRequest -Method Head` sobre una foto |
| `Unresolved reference: AsyncImage` | Import de Coil 2 | Usar `import coil3.compose.AsyncImage` |
| `JsonSyntaxException: Expected BEGIN_ARRAY but was ...` | El servidor devolvió algo que no es la lista (página de login, objeto de error) | Probar el endpoint con `Invoke-WebRequest` y revisar el código HTTP |
| `401` o `403` en cualquier petición | La API exige token, o CSRF está activo en los `POST`, `PUT` y `DELETE` | Aplicar el Paso 0.2 o continuar con la guía 9\_3 |
| `415` al crear o editar | La parte `dto` no viaja como `application/json`, o falta `@Multipart` | Revisar `aParteJson()` en el repositorio y la anotación en `RepuestoApi` |
| `400` al crear o editar | Un campo no cumple lo que valida el servidor (nombre vacío, tipo fuera del enum) | Revisar los datos enviados en Logcat y el mensaje de la respuesta |
| `413` o `500` al subir una foto | El servidor limita los archivos a 1 MB por defecto | Subir `spring.servlet.multipart.max-file-size` (Paso 7) |
| `500` al eliminar un registro | El ítem está relacionado con otros datos en la base | Verificar en el servidor si otra tabla lo referencia; no es un fallo de la app |

Errores propios de Hilt en este proyecto (la guía `8_1` los explica con más detalle):

- **La app se cierra al abrir, con *Hilt Activity must be attached to an @HiltAndroidApp Application*:** falta `@HiltAndroidApp` en `AutoPartsApplication` o `android:name=".AutoPartsApplication"` en el manifiesto.
- **`NoSuchMethodException: RepuestosViewModel.<init> []`:** si en el registro no aparece `HiltViewModelFactory`, falta `@AndroidEntryPoint` en `MainActivity`; si aparece, al `ViewModel` le falta `@HiltViewModel` o `@Inject constructor`.
- **`[Dagger/MissingBinding] RepuestoApi cannot be provided`:** `NetworkModule` no tiene `@Module` o `@InstallIn(SingletonComponent::class)`, o falta la función `@Provides` de `RepuestoApi`.
- **`[Hilt] @HiltViewModel is only supported on types that subclass androidx.lifecycle.ViewModel`:** la anotación quedó sobre el repositorio. Los repositorios llevan solo `@Inject constructor`.
- **Errores en `..._Impl.kt` o *incompatible version of Kotlin*:** el valor de `kotlin` en `libs.versions.toml` es anterior al que piden las librerías; use `2.4.21`.

Errores propios de la navegación y del buscador:

- **Espacio doble debajo de la lista, o el último elemento queda tapado por la barra:** falta `contentWindowInsets = WindowInsets(0, 0, 0, 0)` en el `Scaffold` raíz o en el de `ListaRepuestosView` (explicado en el Paso 8).
- **`IllegalArgumentException: Navigation destination that matches request ... cannot be found in the navigation graph`:** la ruta usada en `navigate(...)` no coincide con la declarada en `composable(...)`; revise `"detalle/$id"` y `"formulario?id=$id"`.
- **El detalle se queda con el indicador de carga:** `GET /repuestos-servicios/{id}` falló (el ítem ya no existe o no hay conexión). El motivo aparece en el `Snackbar`; vuelva a la lista y actualice.
- **El buscador no encuentra un registro que sí existe:** la comparación ignora mayúsculas pero no acentos (`diametro` no encuentra `díametro`). Normalizar los acentos con `java.text.Normalizer` queda como ejercicio.
- **Al cambiar de pestaña se pierde lo escrito en el buscador:** falta `saveState = true` y `restoreState = true` en la navegación de la barra, o el texto se guarda con `remember` en lugar de `rememberSaveable`.

## Ejercicio de cierre y adelanto de la guía 9\_3

### Ejercicio

Los tres primeros se resuelven solo en la capa de interfaz y en el ViewModel; el cuarto aplica todo lo aprendido a otra API.

1. **Filtro por tipo:** tres opciones (Todos, Repuestos, Servicios) sobre la lista, filtrando en la app los datos que ya llegaron.
2. **Aviso de stock:** en las tarjetas de repuestos con `stock == 0`, mostrar "Sin existencias" en color de error.
3. **Buscador por nombre:** un campo de búsqueda tipo píldora en la `TopAppBar`, como el de AgendaApp, que filtre mientras se escribe.
4. **Reto con su propia API:** copiar el proyecto y conectarlo a la API PHP o Spring de otro módulo, cambiando la `baseUrl`, el DTO y las rutas de `RepuestoApi`. Entregar una captura de la lista y otra del formulario guardando un registro.

### Qué sigue en la guía 9\_3

La API real no tiene los endpoints abiertos: `POST`, `PUT` y `DELETE` exigen rol, y el login vive en otro servicio (`http://127.0.0.1:8000/login`, método `POST` con `username` y `password`). La guía 9\_3 parte de este mismo proyecto y agrega:

1. `LoginApi`, con su propio Retrofit apuntando al puerto 8000.
2. Guardar el token en DataStore, como en la guía 7\_2.
3. Un interceptor de OkHttp que añade `Authorization: Bearer <token>` a cada petición al catálogo; con Hilt se agrega en NetworkModule, al construir el OkHttpClient.
4. Una pantalla de inicio de sesión y el manejo de `401` y `403`.
5. Revertir los cambios temporales del Paso 0.2 en el servidor.

Como hay dos servicios, hará falta un segundo enlace: `adb reverse tcp:8000 tcp:8000`.
