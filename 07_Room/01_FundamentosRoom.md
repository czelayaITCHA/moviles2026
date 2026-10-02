# 7\_1 — Persistencia de datos con Room: Agenda de Contactos

Guía para persistir datos localmente con **Room**, usando como ejemplo una agenda de contactos (nombre, teléfono, correo y una foto tomada con la cámara o elegida de la galería).

## **1. Objetivos**

- Comprender qué es Room y cuándo conviene usarlo.
- Crear una entidad, un DAO y una base de datos con Room.
- Conectar Room con la UI a través de un `ViewModel`.
- Capturar una foto con la cámara o seleccionarla de la galería y guardar su referencia en la base de datos.

---

## **2. ¿Qué es Room y por qué usarlo?**

Room es la librería oficial de Android para trabajar con **SQLite** de forma más segura y sencilla: en vez de escribir SQL a mano y manejar cursores, se definen clases Kotlin y Room genera el código necesario en tiempo de compilación.

Se usa Room cuando los datos son **estructurados y se repiten** (una lista de contactos, tareas, productos, etc.). 

---

## **3. Dependencias necesarias**

Cuando se crea el proyecto usa **Gradle Version Catalogs** (`libs.versions.toml`), así que las versiones no van directo en `build.gradle.kts` sino en ese archivo. Hay que agregar tres cosas: el plugin **KSP** (procesador de anotaciones de Room), las librerías de **Room**, y **Coil** (para mostrar la foto del contacto).

**1. En `gradle/libs.versions.toml`, agregar a `[versions]`:**

```toml
ksp = "2.3.5"
room = "2.8.5"
coil = "3.0.4"
```

**Agregar a `[plugins]`:**

```toml
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

**Agregar a `[libraries]`:**

```toml
androidx-room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
androidx-room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
androidx-room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
coil-compose = { group = "io.coil-kt.coil3", name = "coil-compose", version.ref = "coil" }
```

**2. En el `build.gradle.kts` de la RAÍZ del proyecto**, agregar el plugin KSP junto a los que ya tiene (con `apply false`):

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.ksp) apply false
}
```

**3. En el `build.gradle.kts` del módulo `app`:**

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.ksp)
}

dependencies {
    implementation(libs.androidx.room.runtime)
    implementation(libs.androidx.room.ktx)
    ksp(libs.androidx.room.compiler)

    implementation(libs.coil.compose)
    
}
```

Note que los guiones del TOML (`androidx-room-runtime`) se convierten en puntos al usarlos (`libs.androidx.room.runtime`) — eso lo genera Gradle automáticamente, no hay que escribirlo distinto a como está en el archivo.

---
## **4. Paso 1 — Modelo de datos (`@Entity`)**

```kotlin
package com.example.agendaapp.data

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "contactos")
data class Contacto(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val nombre: String,
    val telefono: String,
    val correo: String,
    val fotoUri: String? = null // se guarda la URI de la imagen, NO la imagen en sí
)
```

- `@Entity(tableName = "contactos")`: cada fila de esta tabla es un objeto `Contacto`.
- `@PrimaryKey(autoGenerate = true)`: Room asigna el `id` automáticamente al insertar.
- `fotoUri`: Room (y SQLite) no guardan archivos de imagen, solo texto/números; por eso se guarda la **ruta (URI)** de la foto, no la foto en sí.

---

## **5. Paso 2 — DAO (acceso a datos)**

```kotlin
package com.example.agendaapp.data

import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.Query
import androidx.room.Update
import kotlinx.coroutines.flow.Flow

@Dao
interface ContactoDao {

    @Query("SELECT * FROM contactos ORDER BY nombre ASC")
    fun obtenerTodos(): Flow<List<Contacto>>

    @Insert
    suspend fun insertar(contacto: Contacto)

    @Update
    suspend fun actualizar(contacto: Contacto)

    @Delete
    suspend fun eliminar(contacto: Contacto)
}
```

- `@Dao`: marca la interfaz como el lugar donde se definen las operaciones sobre la tabla; Room genera la implementación real.
- `Flow<List<Contacto>>`: al usar `Flow`, la lista se actualiza **automáticamente** en la UI cada vez que la tabla cambia (insertar/eliminar), sin necesidad de volver a consultar manualmente.
- `suspend fun`: las operaciones que escriben (`insertar`, `actualizar`, `eliminar`) deben ejecutarse en una corrutina, nunca en el hilo principal.

---

## **6. Paso 3 — Base de datos (`@Database`)**

```kotlin
package com.example.agendaapp.data

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(entities = [Contacto::class], version = 1, exportSchema = false)
abstract class AgendaDatabase : RoomDatabase() {

    abstract fun contactoDao(): ContactoDao

    companion object {
        @Volatile
        private var INSTANCIA: AgendaDatabase? = null

        fun obtenerInstancia(context: Context): AgendaDatabase {
            return INSTANCIA ?: synchronized(this) {
                val instancia = Room.databaseBuilder(
                    context.applicationContext,
                    AgendaDatabase::class.java,
                    "agenda_db"
                ).build()
                INSTANCIA = instancia
                instancia
            }
        }
    }
}
```

- `entities = [Contacto::class]`: le indica a Room qué tablas debe crear.
- El patrón `companion object` + `@Volatile` + `synchronized` asegura que **exista una sola instancia** de la base de datos en toda la app (crear varias instancias es un error común y costoso en rendimiento).

---

## **7. Paso 4 — Repositorio (capa intermedia)**

```kotlin
package com.example.agendaapp.repository

import com.example.agendaapp.data.Contacto
import com.example.agendaapp.data.ContactoDao

class ContactoRepository(private val dao: ContactoDao) {
    val contactos = dao.obtenerTodos()

    suspend fun guardar(contacto: Contacto) = dao.insertar(contacto)
    suspend fun eliminar(contacto: Contacto) = dao.eliminar(contacto)
}
```

El repositorio no es obligatorio, pero es buena práctica: el `ViewModel` no habla directamente con el DAO, sino con el repositorio — así, si más adelante se agrega una API remota, solo cambia el repositorio, no el `ViewModel` ni la UI.

---
## **8. Paso 5 — ViewModel**

```kotlin
package com.example.agendaapp.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.agendaapp.data.Contacto
import com.example.agendaapp.repository.ContactoRepository
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class AgendaViewModel(private val repository: ContactoRepository) : ViewModel() {

    val contactos = repository.contactos.stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        initialValue = emptyList()
    )

    fun guardarContacto(nombre: String, telefono: String, correo: String, fotoUri: String?) {
        viewModelScope.launch {
            repository.guardar(
                Contacto(nombre = nombre, telefono = telefono, correo = correo, fotoUri = fotoUri)
            )
        }
    }

    fun eliminarContacto(contacto: Contacto) {
        viewModelScope.launch {
            repository.eliminar(contacto)
        }
    }
}
```

- `stateIn(...)`: convierte el `Flow` del repositorio en un `StateFlow`, que es lo que la UI de Compose puede leer cómodamente con `collectAsState()`.
- Cada operación de escritura se lanza con `viewModelScope.launch { }` porque son funciones `suspend`.

*(Esta `AgendaViewModel` recibe el repositorio por parámetro; en la Sesión de Hilt se simplificará esta construcción con inyección de dependencias. Mientras tanto, hace falta una Factory manual — ver más abajo — porque `AgendaViewModel` no tiene un constructor sin parámetros.)*

**En `MainActivity`, crear el repositorio y construir el ViewModel con el DSL `viewModelFactory` (sin crear una clase Factory aparte):**

```kotlin
package com.example.agendaapp

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.activity.viewModels
import androidx.lifecycle.viewmodel.initializer
import androidx.lifecycle.viewmodel.viewModelFactory
import com.example.agendaapp.data.AgendaDatabase
import com.example.agendaapp.navegation.NavManager
import com.example.agendaapp.repository.ContactoRepository
import com.example.agendaapp.ui.theme.AgendaAppTheme
import com.example.agendaapp.viewmodel.AgendaViewModel
import kotlin.getValue

class MainActivity : ComponentActivity() {
    private val repository by lazy {
        ContactoRepository(AgendaDatabase.obtenerInstancia(applicationContext).contactoDao())
    }

    private val viewModel: AgendaViewModel by viewModels {
        viewModelFactory {
            initializer { AgendaViewModel(repository) }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            AgendaAppTheme {
                NavManager(viewModel = viewModel)
            }
        }
    }
}
```

- Sin esto, `by viewModels()` (o `viewModel()` en Compose) usa el constructor **sin parámetros** por defecto, y como `AgendaViewModel` no tiene uno, falla con `NoSuchMethodException: AgendaViewModel.<init> []`.
- `viewModelFactory { initializer { AgendaViewModel(repository) } }` (de `androidx.lifecycle.viewmodel`) construye el `ViewModel` pasándole el `repository`, sin necesidad de crear una clase `Factory` aparte — y conserva el mismo `AgendaViewModel` aunque la pantalla se recree (por ejemplo, al rotar).

---
## **9. Paso 6 — Capturar o elegir la foto del contacto**

### 9.1 Seleccionar de la galería (sin permisos especiales)

Desde Android 13, el **Photo Picker** del sistema no requiere el permiso `READ_MEDIA_IMAGES`:

```kotlin
val seleccionarImagenLauncher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.PickVisualMedia()
) { uri: Uri? ->
    uri?.let { fotoUri = it.toString() }
}

Button(onClick = {
    seleccionarImagenLauncher.launch(
        PickVisualMediaRequest(ActivityResultContracts.PickVisualMedia.ImageOnly)
    )
}) {
    Text("Elegir de galería")
}
```

### 9.2 Tomar una foto con la cámara

Tomar una foto requiere **tres piezas adicionales**: permiso de cámara, un `FileProvider` (para compartir el archivo de forma segura con la app de cámara) y un archivo temporal donde guardarla.

**a) Permiso en `AndroidManifest.xml`:**

```xml
<uses-permission android:name="android.permission.CAMERA" />
```

**b) Declarar el `FileProvider` en `AndroidManifest.xml`** (dentro de `<application>`):

```xml
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

**c) Crear `res/xml/file_paths.xml`:**

```xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <cache-path name="fotos" path="fotos/" />
</paths>
```

**d) Código para lanzar la cámara:**

```kotlin
val context = LocalContext.current
var uriTemporal by remember { mutableStateOf<Uri?>(null) }

val tomarFotoLauncher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.TakePicture()
) { exito: Boolean ->
    if (exito) {
        fotoUri = uriTemporal.toString()
    }
}

val permisoCamaraLauncher = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.RequestPermission()
) { concedido ->
    if (concedido) {
        val archivo = File(context.cacheDir, "fotos/contacto_${System.currentTimeMillis()}.jpg")
        archivo.parentFile?.mkdirs()
        uriTemporal = FileProvider.getUriForFile(
            context, "${context.packageName}.fileprovider", archivo
        )
        tomarFotoLauncher.launch(uriTemporal)
    }
}

Button(onClick = { permisoCamaraLauncher.launch(Manifest.permission.CAMERA) }) {
    Text("Tomar foto")
}
```

- El flujo es: pedir permiso → crear un archivo vacío en `cacheDir` → obtener su `Uri` segura con `FileProvider` → lanzar la cámara pasándole esa `Uri` (la foto se guarda directamente ahí).
- `fotoUri` (el estado que se guarda luego en Room) es el mismo en ambos casos: una cadena de texto con la URI de la imagen.

**NOTA:** los puntos 9.1 y 9.2 literal d, son explicativos, se implementan en la vista de registro del contacto (RegistroContactoView)
---
## 10. Paso 7 — Dos pantallas: Lista (buscador + FAB) y Registro

En vez de una sola pantalla con todo junto, se separa en **dos**: `ListaContactosView` (lista con buscador y botón flotante) y `RegistroContactoView` (el formulario, con los *launchers* de cámara/galería del Paso 6), ambas dentro del paquete `ui.views`. La navegación entre ambas se centraliza en un `NavManager`, dentro de un paquete `navigation` — igual que se vio en la Sesión 7 del temario, pero ahora en su propio archivo en vez de ir dentro de `MainActivity`.

**`NavManager`, dentro del paquete `navigation`:**

```kotlin
package com.example.agendaapp.navegation

import androidx.compose.runtime.Composable
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import com.example.agendaapp.ui.views.ListaContactosView
import com.example.agendaapp.ui.views.RegistroContactoView
import com.example.agendaapp.viewmodel.AgendaViewModel

@Composable
fun NavManager(viewModel: AgendaViewModel) {
    val navController = rememberNavController()
    NavHost(navController = navController, startDestination = "lista") {
        composable("lista") {
            ListaContactosView(
                viewModel = viewModel,
                onAgregarClick = { navController.navigate("registro") }
            )
        }
        composable("registro") {
            RegistroContactoView(
                viewModel = viewModel,
                onGuardado = { navController.popBackStack() },
                onBack = { navController.popBackStack() }
            )
        }
    }
}
```

**`ListaContactosView` — lista, buscador y FAB:**

```kotlin
package com.example.agendaapp.ui.views

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Add
import androidx.compose.material.icons.filled.Delete
import androidx.compose.material.icons.filled.Search
import androidx.compose.material3.Card
import androidx.compose.material3.CenterAlignedTopAppBar
import androidx.compose.material3.ElevatedCard
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.FloatingActionButton
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.OutlinedTextFieldDefaults
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBarDefaults
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import coil3.compose.AsyncImage
import com.example.agendaapp.viewmodel.AgendaViewModel

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ListaContactosView(
    viewModel: AgendaViewModel,
    onAgregarClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    var busqueda by remember { mutableStateOf("") }
    val contactos by viewModel.contactos.collectAsState()

    val contactosFiltrados = remember(contactos, busqueda) {
        if (busqueda.isBlank()) contactos
        else contactos.filter { it.nombre.contains(busqueda, ignoreCase = true) }
    }

    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(
                title = { Text("Agenda de Contactos") },
                colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                )
            )
        },
        floatingActionButton = {
            FloatingActionButton(onClick = onAgregarClick) {
                Icon(Icons.Default.Add, contentDescription = "Agregar contacto")
            }
        }
    ) { innerPadding ->
        Column(
            modifier = modifier
                .fillMaxSize()
                .padding(innerPadding)
                .padding(horizontal = 16.dp)
        ) {
            OutlinedTextField(
                value = busqueda,
                onValueChange = { busqueda = it },
                placeholder = { Text("Buscar contacto") },
                leadingIcon = { Icon(Icons.Default.Search, contentDescription = null) },
                singleLine = true,
                shape = RoundedCornerShape(28.dp),
                colors = OutlinedTextFieldDefaults.colors(
                    unfocusedBorderColor = Color.Transparent,
                    focusedBorderColor = Color.Transparent,
                    unfocusedContainerColor = MaterialTheme.colorScheme.surfaceVariant,
                    focusedContainerColor = MaterialTheme.colorScheme.surfaceVariant
                ),
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(vertical = 12.dp)
            )

            if (contactosFiltrados.isEmpty()) {
                Text(
                    "No se encontraron contactos",
                    modifier = Modifier.padding(top = 24.dp),
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            } else {
                LazyColumn(verticalArrangement = Arrangement.spacedBy(10.dp)) {
                    items(contactosFiltrados, key = { it.id }) { contacto ->
                        ElevatedCard(shape = RoundedCornerShape(16.dp)) {
                            Row(
                                modifier = Modifier.padding(14.dp),
                                verticalAlignment = Alignment.CenterVertically,
                                horizontalArrangement = Arrangement.spacedBy(14.dp)
                            ) {
                                AsyncImage(
                                    model = contacto.fotoUri,
                                    contentDescription = null,
                                    modifier = Modifier.size(56.dp).clip(CircleShape),
                                    contentScale = ContentScale.Crop
                                )
                                Column(modifier = Modifier.weight(1f)) {
                                    Text(
                                        text = contacto.nombre,
                                        style = MaterialTheme.typography.titleMedium,
                                        fontWeight = FontWeight.SemiBold
                                    )
                                    Text(
                                        text = contacto.telefono,
                                        style = MaterialTheme.typography.bodyMedium,
                                        color = MaterialTheme.colorScheme.onSurfaceVariant
                                    )
                                    Text(
                                        text = contacto.correo,
                                        style = MaterialTheme.typography.bodySmall,
                                        color = MaterialTheme.colorScheme.onSurfaceVariant
                                    )
                                }
                                IconButton(onClick = { viewModel.eliminarContacto(contacto) }) {
                                    Icon(
                                        Icons.Default.Delete,
                                        contentDescription = "Eliminar",
                                        tint = MaterialTheme.colorScheme.error
                                    )
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```

**`RegistroContactoView` — el formulario, con los *launchers* de cámara/galería:**

```kotlin
package com.example.agendaapp.ui.views

import android.Manifest
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import android.net.Uri
import androidx.activity.result.PickVisualMediaRequest
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.automirrored.filled.ArrowBack
import androidx.compose.material3.Button
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBar
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.unit.dp
import androidx.core.content.FileProvider
import coil3.compose.AsyncImage
import com.example.agendaapp.viewmodel.AgendaViewModel
import java.io.File


@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun RegistroContactoView(
    viewModel: AgendaViewModel,
    onGuardado: () -> Unit,
    onBack: () -> Unit,
    modifier: Modifier = Modifier
) {
    var nombre by remember { mutableStateOf("") }
    var telefono by remember { mutableStateOf("") }
    var correo by remember { mutableStateOf("") }
    var fotoUri by remember { mutableStateOf<String?>(null) }

    val context = LocalContext.current
    var uriTemporal by remember { mutableStateOf<Uri?>(null) }

    // --- Launchers: deben declararse aquí, al nivel superior del composable ---
    val seleccionarImagenLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.PickVisualMedia()
    ) { uri: Uri? ->
        uri?.let { fotoUri = it.toString() }
    }

    val tomarFotoLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.TakePicture()
    ) { exito: Boolean ->
        if (exito) fotoUri = uriTemporal.toString()
    }

    val permisoCamaraLauncher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.RequestPermission()
    ) { concedido ->
        if (concedido) {
            val archivo = File(context.cacheDir, "fotos/contacto_${System.currentTimeMillis()}.jpg")
            archivo.parentFile?.mkdirs()
            uriTemporal = FileProvider.getUriForFile(
                context, "${context.packageName}.fileprovider", archivo
            )
            tomarFotoLauncher.launch(uriTemporal!!)
        }
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Nuevo contacto") },
                navigationIcon = {
                    IconButton(onClick = onBack) {
                        Icon(
                            imageVector = Icons.AutoMirrored.Filled.ArrowBack,
                            contentDescription = "Volver"
                        )
                    }
                }
            )
        }
    ) { innerPadding ->
        Column(
            modifier = modifier
                .fillMaxSize()
                .padding(innerPadding)
                .padding(16.dp)
        ) {
            fotoUri?.let { uri ->
                AsyncImage(
                    model = uri,
                    contentDescription = "Foto del contacto",
                    modifier = Modifier.size(80.dp).clip(CircleShape),
                    contentScale = ContentScale.Crop
                )
            }

            Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
                Button(onClick = {
                    seleccionarImagenLauncher.launch(
                        PickVisualMediaRequest(ActivityResultContracts.PickVisualMedia.ImageOnly)
                    )
                }) { Text("Galería") }
                Button(onClick = {
                    permisoCamaraLauncher.launch(Manifest.permission.CAMERA)
                }) { Text("Cámara") }
            }

            OutlinedTextField(
                value = nombre, onValueChange = { nombre = it },
                label = { Text("Nombre") }, modifier = Modifier.fillMaxWidth()
            )
            OutlinedTextField(
                value = telefono, onValueChange = { telefono = it },
                label = { Text("Teléfono") },
                keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Phone),
                modifier = Modifier.fillMaxWidth()
            )
            OutlinedTextField(
                value = correo, onValueChange = { correo = it },
                label = { Text("Correo") },
                keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
                modifier = Modifier.fillMaxWidth()
            )

            Button(
                onClick = {
                    viewModel.guardarContacto(nombre, telefono, correo, fotoUri)
                    onGuardado()
                },
                modifier = Modifier.fillMaxWidth()
            ) { Text("Guardar contacto") }
        }
    }
}
```

- Los *launchers* (`seleccionarImagenLauncher`, `tomarFotoLauncher`, `permisoCamaraLauncher`) se declaran al nivel superior de `RegistroContactoView` — no dentro de un `onClick` — porque `rememberLauncherForActivityResult` es una llamada `@Composable` y debe ejecutarse durante la composición; dentro del `onClick` del botón solo se llama `.launch(...)`.
- El buscador ya no va dentro del `TopAppBar` (se veía plano, pegado al borde); ahora el `TopAppBar` tiene un título fijo ("Agenda de Contactos") con color `primaryContainer`, y el buscador es un `OutlinedTextField` con forma de píldora (`RoundedCornerShape(28.dp)`) y borde transparente, debajo de la barra — el estilo típico de apps como Contactos de Google.
- `RegistroContactoView` recibe dos callbacks distintos: `onBack` (flecha de retroceso, solo cierra la pantalla sin guardar) y `onGuardado` (botón "Guardar contacto", guarda y luego cierra) — en este ejemplo ambos hacen `popBackStack()`, pero quedan separados por si más adelante se quiere confirmar antes de salir sin guardar.
- `NavManager` importa las dos vistas desde `ui.views` y el `ViewModel` desde `viewmodel`; `MainActivity` ya no conoce la navegación, solo llama a `NavManager(viewModel)`.
- `Icons.Default.Add`, `Icons.Default.Search` e `Icons.AutoMirrored.Filled.ArrowBack` son parte del set básico de íconos (igual que `Delete`), en caso de ser necesario debes agregar la dependencia de `material-icons-extended`, como hemos visto en guias anteriores.
- `remember(contactos, busqueda) { ... }` recalcula la lista filtrada solo cuando cambian los contactos o el texto buscado, evitando filtrar en cada recomposición sin necesidad.

---

## **11. Estructura de paquetes resultante**
```
com.example.agenda
 ├── MainActivity.kt
 ├── data
 │    ├── Contacto.kt          (@Entity)
 │    ├── ContactoDao.kt       (@Dao)
 │    └── AgendaDatabase.kt    (@Database)
 ├── viewmodel
 │    └── AgendaViewModel.kt
 ├── navigation
 │    └── NavManager.kt
 ├── repository
 │    └── ContactoRepository.kt 
 └── ui.views
      ├── ListaContactosView.kt
      └── RegistroContactoView.kt
```
## **12. Ejercicio adicional**
**a) Agregar mensaje con Toast** que muestre al usuario que se ha guardado correctamente el contacto
**b) Implementar dialog de confirm** investigue como implementar dialog de confirmación en la opción de eliminar contacto
**c) Función editar** implementar la función de editar los datos del contacto
