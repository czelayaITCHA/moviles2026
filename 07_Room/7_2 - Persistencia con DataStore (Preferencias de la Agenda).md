# 7\_2 — Persistencia de datos con DataStore: Preferencias de la Agenda

Segunda guía de persistencia local, complementaria a `7_1` (Room). Aquí se usa **DataStore** para guardar las **preferencias** del usuario sobre cómo ver su agenda de contactos (no los contactos en sí — eso sigue siendo responsabilidad de Room).

**Requisito:** esta guía continúa el proyecto de la guía `7_1` (la misma app). En cada paso se indica con **CREAR** los archivos nuevos y con **MODIFICAR** los archivos de `7_1` que hay que cambiar o ampliar. Al inicio del Paso 7 hay una tabla con el resumen de todos los cambios.

## **1. Objetivos**

- Comprender qué es DataStore y en qué se diferencia de Room.
- Crear y leer preferencias simples con **Preferences DataStore**.
- Integrar DataStore con un `ViewModel` y aplicar las preferencias en la pantalla de la Agenda.

---

## **2. ¿Qué es DataStore y cuándo usarlo?**

DataStore es la librería moderna de Android para guardar **datos simples tipo clave-valor** (reemplaza a `SharedPreferences`). A diferencia de Room:

|  | **Room** | **DataStore** |
| --- | --- | --- |
| Tipo de dato | Estructurado, en tablas (listas de objetos) | Simple, pares clave-valor |
| Ejemplo en la Agenda | La lista de contactos | ¿Modo oscuro activado? ¿Cómo ordenar la lista? |
| Se consulta con | SQL (`@Query`) | Lectura directa por clave |
| Acceso | `Flow`, a través del DAO | `Flow`, directamente sobre el DataStore |

**Regla práctica:** si el dato es una **lista o tiene relaciones**, va en Room. Si es un **ajuste individual** del usuario, va en DataStore.

Existen dos variantes: **Preferences DataStore** (clave-valor simple, sin definir un esquema) y **Proto DataStore** (con un esquema tipado usando Protocol Buffers, más avanzado). Esta guía usa **Preferences DataStore**, suficiente para la mayoría de los casos.

---

## **3. Dependencia necesaria**

Su proyecto usa **Gradle Version Catalogs** (igual que en la guía `7_1`), así que la versión no va directo en `build.gradle.kts`.

**1. MODIFICAR `gradle/libs.versions.toml`:** agregar una línea en `[versions]` y otra en `[libraries]`.

```toml
# En la sección [versions]
datastore = "1.2.1"

# En la sección [libraries]
androidx-datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }
```

**2. MODIFICAR `build.gradle.kts` del módulo `app`:** agregar la dependencia en el bloque `dependencies`, junto a las que ya tiene.

```kotlin
dependencies {
    // ... Room, Coil y las demás que ya tiene
    implementation(libs.androidx.datastore.preferences)   // NUEVO
}
```

*(La versión estable al escribir esta guía es la 1.2.1. Verifique si hay una más reciente en [developer.android.com/jetpack/androidx/releases/datastore](https://developer.android.com/jetpack/androidx/releases/datastore).)*, aunque ésta es totalmente funcional.

---

## **4. Paso 1 \[CREAR data/AgendaDataStore.kt\] — Crear la instancia de DataStore**

Cree el archivo **`AgendaDataStore.kt`** dentro del paquete `data` (clic derecho sobre `data` → *New* → *Kotlin Class/File*). Este mismo archivo contendrá el Paso 1 y el Paso 2.

Se crea **una sola vez** por app, como una propiedad de extensión sobre `Context`:

```kotlin
package com.example.agendaapp.data

import android.content.Context
import androidx.datastore.preferences.preferencesDataStore

val Context.agendaDataStore by preferencesDataStore(name = "agenda_preferencias")
```

- `preferencesDataStore(name = "agenda_preferencias")`: crea (o abre, si ya existe) un archivo de preferencias con ese nombre.
- Al declararse como propiedad de extensión de `Context` con `by`, Android se encarga de que exista una única instancia durante toda la vida de la app (igual que se hizo manualmente con `synchronized` para la base de datos de Room en la guía `7_1`).

---

## **5. Paso 2 \[MISMO ARCHIVO del Paso 1\] — Definir las claves**

Estas claves van en el **mismo archivo** del Paso 1 (`data/AgendaDataStore.kt`), debajo de la línea `val Context.agendaDataStore ...`; no se crea un archivo aparte. Al final de este paso se muestra cómo queda el archivo completo.

```kotlin
package com.example.agenda.data

import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.stringPreferencesKey

object PreferenciasAgenda {
    val MODO_OSCURO = booleanPreferencesKey("modo_oscuro")
    val SOLO_FAVORITOS = booleanPreferencesKey("solo_favoritos")
    val ORDEN_LISTA = stringPreferencesKey("orden_lista") // "nombre" o "reciente"
}
```

- Cada preferencia necesita una **clave tipada**: `booleanPreferencesKey` para `true`/`false`, `stringPreferencesKey` para texto (también existen `intPreferencesKey`, `doublePreferencesKey`, etc.).
- Agrupar las claves en un `object` evita escribir el nombre (`"modo_oscuro"`) suelto en varias partes del código.

**Archivo completo `data/AgendaDataStore.kt` (Paso 1 + Paso 2 juntos):**

```kotlin
package com.example.agendaapp.data

import android.content.Context
import androidx.datastore.preferences.core.booleanPreferencesKey
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore

// Paso 1: instancia única de DataStore
val Context.agendaDataStore by preferencesDataStore(name = "agenda_preferencias")

// Paso 2: claves tipadas
object PreferenciasAgenda {
    val MODO_OSCURO = booleanPreferencesKey("modo_oscuro")
    val SOLO_FAVORITOS = booleanPreferencesKey("solo_favoritos")
    val ORDEN_LISTA = stringPreferencesKey("orden_lista") // "nombre" o "reciente"
}
```

Use el nombre de paquete de su proyecto en la primera línea (por ejemplo `com.example.agendaapp.data`).

---

## **6. Paso 3 \[CREAR data/AgendaPreferenciasRepository.kt\] — Leer las preferencias, en el package repository**

```kotlin
package com.example.agendaapp.repository

import android.content.Context
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map

class AgendaPreferenciasRepository(private val context: Context) {

    val modoOscuro: Flow<Boolean> = context.agendaDataStore.data
        .map { preferencias -> preferencias[PreferenciasAgenda.MODO_OSCURO] ?: false }

    val soloFavoritos: Flow<Boolean> = context.agendaDataStore.data
        .map { preferencias -> preferencias[PreferenciasAgenda.SOLO_FAVORITOS] ?: false }

    val ordenLista: Flow<String> = context.agendaDataStore.data
        .map { preferencias -> preferencias[PreferenciasAgenda.ORDEN_LISTA] ?: "nombre" }
}
```

- `context.agendaDataStore.data` es un `Flow<Preferences>` que emite un nuevo valor **cada vez que algo cambia**, así que la UI puede reaccionar en tiempo real.
- `preferencias[CLAVE] ?: valorPorDefecto`: como DataStore no obliga a que una clave exista, siempre se da un valor por defecto para el primer uso de la app.

---

## **7. Paso 4 \[MISMO ARCHIVO del Paso 3\] — Guardar las preferencias**

Continuando la misma clase del Paso 3:

```kotlin
    suspend fun actualizarModoOscuro(activado: Boolean) {
        context.agendaDataStore.edit { preferencias ->
            preferencias[PreferenciasAgenda.MODO_OSCURO] = activado
        }
    }

    suspend fun actualizarSoloFavoritos(activado: Boolean) {
        context.agendaDataStore.edit { preferencias ->
            preferencias[PreferenciasAgenda.SOLO_FAVORITOS] = activado
        }
    }

    suspend fun actualizarOrden(orden: String) {
        context.agendaDataStore.edit { preferencias ->
            preferencias[PreferenciasAgenda.ORDEN_LISTA] = orden
        }
    }
}
```

- `edit { ... }` es una función `suspend`: igual que con Room, toda escritura debe hacerse desde una corrutina, nunca directamente en la UI.
- Cada llamada a `edit` reemplaza únicamente la clave indicada; las demás preferencias guardadas no se tocan.

---

## **8. Paso 5 \[CREAR viewmodel/PreferenciasViewModel.kt\] — Integrar en un ViewModel**

```
package com.example.agendaapp.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.agendaapp.repository.AgendaPreferenciasRepository
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class PreferenciasViewModel(
    private val repository: AgendaPreferenciasRepository
) : ViewModel() {

    val modoOscuro = repository.modoOscuro
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), false)

    val soloFavoritos = repository.soloFavoritos
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), false)

    val ordenLista = repository.ordenLista
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), "nombre")

    fun cambiarModoOscuro(activado: Boolean) {
        viewModelScope.launch { repository.actualizarModoOscuro(activado) }
    }

    fun cambiarSoloFavoritos(activado: Boolean) {
        viewModelScope.launch { repository.actualizarSoloFavoritos(activado) }
    }

    fun cambiarOrden(orden: String) {
        viewModelScope.launch { repository.actualizarOrden(orden) }
    }
}
```

Mismo patrón que en la guía de Room (`7_1`): el `ViewModel` expone `StateFlow` que la UI consume con `collectAsState()`, y cada cambio del usuario se guarda lanzando una corrutina con `viewModelScope.launch`.

---

## 9. Paso 6 \[CREAR ui/views/PreferenciasAgendaView.kt\] — Pantalla de preferencias (`PreferenciasAgendaView`)

```kotlin
package com.example.agendaapp.ui.views

import com.example.agendaapp.viewmodel.PreferenciasViewModel

@Composable
fun PreferenciasAgendaView(
    viewModel: PreferenciasViewModel,
    onBack: () -> Unit,
    modifier: Modifier = Modifier
) {
    val modoOscuro by viewModel.modoOscuro.collectAsState()
    val soloFavoritos by viewModel.soloFavoritos.collectAsState()
    val ordenLista by viewModel.ordenLista.collectAsState()

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Preferencias") },
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
                .padding(16.dp),
            verticalArrangement = Arrangement.spacedBy(16.dp)
        ) {
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween,
                verticalAlignment = Alignment.CenterVertically
            ) {
                Text("Modo oscuro")
                Switch(checked = modoOscuro, onCheckedChange = { viewModel.cambiarModoOscuro(it) })
            }

            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween,
                verticalAlignment = Alignment.CenterVertically
            ) {
                Text("Mostrar solo favoritos")
                Switch(checked = soloFavoritos, onCheckedChange = { viewModel.cambiarSoloFavoritos(it) })
            }

            Column {
                Text("Ordenar lista por:")
                Row(verticalAlignment = Alignment.CenterVertically) {
                    RadioButton(
                        selected = ordenLista == "nombre",
                        onClick = { viewModel.cambiarOrden("nombre") }
                    )
                    Text("Nombre")
                    Spacer(modifier = Modifier.width(16.dp))
                    RadioButton(
                        selected = ordenLista == "reciente",
                        onClick = { viewModel.cambiarOrden("reciente") }
                    )
                    Text("Más reciente")
                }
            }
        }
    }
}
```

- No hay botón "Guardar": cada `Switch`/`RadioButton` guarda su valor **inmediatamente** al cambiar, que es el comportamiento esperado de una pantalla de preferencias.
- Por sí sola esta pantalla solo guarda y lee preferencias. Para que esas preferencias afecten la app (tema oscuro, filtro de favoritos, orden de la lista) hay que conectarla con el resto de la Agenda — ver el Paso 7.

---

## 10. Paso 7 — Integrar DataStore con la Agenda (Room)

Hasta aquí DataStore funciona por separado: las preferencias se guardan y se leen, pero ninguna pantalla las usa todavía. No aparecen solas en la app; hay que conectarlas en **cinco puntos**:

1. Crear el `PreferenciasViewModel` en `MainActivity` (igual que el `AgendaViewModel`).
2. Aplicar el **modo oscuro** al tema de la app.
3. Agregar la ruta `"preferencias"` en `NavManager`.
4. Agregar un ícono de **ajustes** en la barra superior de `ListaContactosView` que navegue a esa ruta.
5. Aplicar **solo favoritos** y **orden** a la lista de contactos (esto requiere un campo `favorito` en `Contacto`).

**Resumen de cambios sobre el proyecto de la guía `7_1`:**

| Archivo | Acción | Qué hacer |
| --- | --- | --- |
| `gradle/libs.versions.toml` y `build.gradle.kts` (módulo `app`) | MODIFICAR | Agregar la dependencia de DataStore (sección 3) |
| `data/AgendaDataStore.kt` | CREAR | Instancia de DataStore y claves (Pasos 1 y 2) |
| `repository/AgendaPreferenciasRepository.kt` | CREAR | Leer y guardar preferencias (Pasos 3 y 4) |
| `viewmodel/PreferenciasViewModel.kt` | CREAR | Paso 5 |
| `ui/views/PreferenciasAgendaView.kt` | CREAR | Paso 6 |
| `MainActivity.kt` | MODIFICAR | Crear el `PreferenciasViewModel`, aplicar el modo oscuro y pasarlo a `NavManager` (10.1) |
| `navigation/NavManager.kt` | MODIFICAR | Parámetro nuevo y ruta `"preferencias"` (10.2) |
| `data/Contacto.kt` | MODIFICAR | Agregar el campo `favorito` (10.3) |
| `data/AgendaDatabase.kt` | MODIFICAR | Subir `version` y agregar una `Migration` o `fallbackToDestructiveMigration` (10.3) |
| `repository/ContactoRepository.kt` | MODIFICAR | Agregar la función `actualizar()` (10.3) |
| `viewmodel/AgendaViewModel.kt` | MODIFICAR | Agregar la función `cambiarFavorito()` (10.3) |
| `ui/views/ListaContactosView.kt` | MODIFICAR | Parámetros nuevos, filtro y orden, ícono de ajustes y corazón de favorito (10.4) |
| `AndroidManifest.xml` | MODIFICAR (opcional) | `android:allowBackup="false"`, si la base vieja se restaura al reinstalar (10.3) |
| `ui/views/RegistroContactoView.kt`, `data/ContactoDao.kt` | Sin cambios | — |

### 10.1 \[MODIFICAR\] `MainActivity`: crear el ViewModel de preferencias y aplicar el modo oscuro

```kotlin
class MainActivity : ComponentActivity() {

    // Sin cambios (guía 7_1)
    private val repository by lazy {
        ContactoRepository(AgendaDatabase.obtenerInstancia(applicationContext).contactoDao())
    }

    // Sin cambios (guía 7_1)
    private val viewModel: AgendaViewModel by viewModels {
        viewModelFactory { initializer { AgendaViewModel(repository) } }
    }

    // NUEVO: ViewModel de preferencias
    private val preferenciasViewModel: PreferenciasViewModel by viewModels {
        viewModelFactory {
            initializer { PreferenciasViewModel(AgendaPreferenciasRepository(applicationContext)) }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            // NUEVO: leer el modo oscuro guardado en DataStore
            val modoOscuro by preferenciasViewModel.modoOscuro.collectAsState()

            // CAMBIO: antes era solo AgendaTheme { ... }, ahora recibe darkTheme
            AgendaTheme(darkTheme = modoOscuro) {
                NavManager(
                    viewModel = viewModel,
                    preferenciasViewModel = preferenciasViewModel   // NUEVO: parámetro
                )
            }
        }
    }
}
```

- `PreferenciasViewModel` recibe un `AgendaPreferenciasRepository`, que a su vez necesita un `Context`; por eso también se construye con `viewModelFactory { initializer { ... } }`, igual que el `AgendaViewModel`.
- El tema que genera Android Studio ya trae el parámetro `darkTheme`. Al pasarle el valor guardado en DataStore, el cambio del `Switch` se refleja en toda la app al instante. Su tema puede llamarse distinto (por ejemplo `AgendaAppTheme`).
- Necesita `import androidx.compose.runtime.getValue` y `import androidx.compose.runtime.collectAsState`.

### 10.2 \[MODIFICAR\] `NavManager`: agregar la ruta de preferencias

```kotlin
@Composable
fun NavManager(
    viewModel: AgendaViewModel,
    preferenciasViewModel: PreferenciasViewModel   // NUEVO: parámetro
) {
    val navController = rememberNavController()
    NavHost(navController = navController, startDestination = "lista") {
        composable("lista") {
            ListaContactosView(
                viewModel = viewModel,
                preferenciasViewModel = preferenciasViewModel,                      // NUEVO
                onAgregarClick = { navController.navigate("registro") },
                onPreferenciasClick = { navController.navigate("preferencias") }    // NUEVO
            )
        }
        // Sin cambios (guía 7_1)
        composable("registro") {
            RegistroContactoView(
                viewModel = viewModel,
                onGuardado = { navController.popBackStack() },
                onBack = { navController.popBackStack() }
            )
        }
        // NUEVO: ruta completa
        composable("preferencias") {
            PreferenciasAgendaView(
                viewModel = preferenciasViewModel,
                onBack = { navController.popBackStack() }
            )
        }
    }
}
```

### 10.3 \[MODIFICAR\] Room, repositorio y ViewModel: agregar el campo `favorito`

Las preferencias "solo favoritos" y "orden" necesitan datos sobre los cuales actuar. El orden por "más reciente" no necesita ningún campo nuevo (se usa el `id`, que crece con cada contacto), pero "favoritos" sí necesita un campo en la entidad:

```kotlin
@Entity(tableName = "contactos")
data class Contacto(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val nombre: String,
    val telefono: String,
    val correo: String,
    val fotoUri: String? = null,
    val favorito: Boolean = false   // NUEVO
)
```

**MODIFICAR `data/AgendaDatabase.kt` (guía `7_1`).** Al cambiar la estructura de la tabla, la base de datos que ya existe en el dispositivo deja de coincidir con la entidad y la app se cierra al abrirla con el error *Room cannot verify the data integrity*. La solución siempre es **subir el número de `version`** en `@Database`. Después hay dos caminos: conservar los contactos con una `Migration` (primer bloque de abajo) o, solo para práctica, dejar que Room recree la base y perder los datos (segundo bloque). Desinstalar la app a veces basta, pero en un teléfono real puede fallar (ver más abajo).

**Opción para conservar los datos: subir la versión y escribir una `Migration`.** En `AgendaDatabase.kt`:

```kotlin
import androidx.room.migration.Migration
import androidx.sqlite.db.SupportSQLiteDatabase

val MIGRACION_1_2 = object : Migration(1, 2) {
    override fun migrate(db: SupportSQLiteDatabase) {
        db.execSQL("ALTER TABLE contactos ADD COLUMN favorito INTEGER NOT NULL DEFAULT 0")
    }
}

@Database(entities = [Contacto::class], version = 2, exportSchema = false)
abstract class AgendaDatabase : RoomDatabase() {
    // ... igual que antes, pero el builder lleva la migración:
    // Room.databaseBuilder(context.applicationContext, AgendaDatabase::class.java, "agenda_database")
    //     .addMigrations(MIGRACION_1_2)
    //     .build()
}
```

- Room guarda los booleanos como `INTEGER` (0 o 1), por eso la columna se declara así, con `DEFAULT 0` para que los contactos que ya existían queden como "no favoritos".
- Sin la migración (o sin desinstalar la app), Room se cierra con `IllegalStateException: Room cannot verify the data integrity`.

**Si desinstalar la app no resuelve el error:** en un teléfono real, Android puede **restaurar la base de datos vieja desde la copia de seguridad automática de Google** al reinstalar la app. La señal es que el error muestra los mismos hashes (`found: ...`) que antes de desinstalar. Hay dos soluciones, que se pueden combinar.

**1. Desactivar la copia automática en `AndroidManifest.xml`** mientras desarrolla:

```xml
<application
    android:allowBackup="false"
    ... >
```

**2. Subir la versión a un número mayor que cualquiera usado antes y permitir que Room recree la base** (solo para práctica, porque borra los datos guardados):

```kotlin
@Database(entities = [Contacto::class], version = 3, exportSchema = false)
abstract class AgendaDatabase : RoomDatabase() {
    // ... en el builder:
    // Room.databaseBuilder(context.applicationContext, AgendaDatabase::class.java, "agenda_database")
    //     .fallbackToDestructiveMigration(dropAllTables = true)
    //     .build()
}
```

`fallbackToDestructiveMigration` solo actúa cuando el número de versión de la base guardada es distinto al del código. Si los números son iguales pero la estructura cambió, Room se cierra con el error de integridad.

**AGREGAR** una función en `ContactoRepository.kt` y otra en `AgendaViewModel.kt` (guía `7_1`), para poder marcar un contacto como favorito. Van dentro de cada clase, junto a las funciones que ya tienen:

```kotlin
// ContactoRepository.kt (data) — AGREGAR dentro de la clase
suspend fun actualizar(contacto: Contacto) = dao.actualizar(contacto)

// AgendaViewModel.kt (viewmodel) — AGREGAR dentro de la clase
fun cambiarFavorito(contacto: Contacto) {
    viewModelScope.launch {
        repository.actualizar(contacto.copy(favorito = !contacto.favorito))
    }
}
```

### 10.4 \[MODIFICAR\] `ListaContactosView`: ícono de ajustes, corazón de favorito y filtros

Solo cambian las partes siguientes; el resto de la función queda igual que en la guía `7_1`.

**Firma y estado** (ahora recibe también el `PreferenciasViewModel` y el callback para ir a preferencias):

```kotlin
@Composable
fun ListaContactosView(
    viewModel: AgendaViewModel,
    preferenciasViewModel: PreferenciasViewModel,   // NUEVO: parámetro
    onAgregarClick: () -> Unit,
    onPreferenciasClick: () -> Unit,                 // NUEVO: parámetro
    modifier: Modifier = Modifier
) {
    var busqueda by remember { mutableStateOf("") }
    val contactos by viewModel.contactos.collectAsState()
    val soloFavoritos by preferenciasViewModel.soloFavoritos.collectAsState()   // NUEVO
    val orden by preferenciasViewModel.ordenLista.collectAsState()               // NUEVO

    // CAMBIO: reemplaza al contactosFiltrados de la guía 7_1
    val contactosFiltrados = remember(contactos, busqueda, soloFavoritos, orden) {
        contactos
            .filter { busqueda.isBlank() || it.nombre.contains(busqueda, ignoreCase = true) }
            .filter { !soloFavoritos || it.favorito }
            .let { lista ->
                if (orden == "reciente") lista.sortedByDescending { it.id }
                else lista.sortedBy { it.nombre.lowercase() }
            }
    }
    // ... el Scaffold sigue igual, con los dos cambios de abajo
}
```

**Ícono de ajustes en la barra superior** (se agrega `actions` al `CenterAlignedTopAppBar`):

```kotlin
CenterAlignedTopAppBar(
    title = { Text("Agenda de Contactos") },
    colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
        containerColor = MaterialTheme.colorScheme.primaryContainer,
        titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
    ),
    // NUEVO: ícono de ajustes
    actions = {
        IconButton(onClick = onPreferenciasClick) {
            Icon(Icons.Default.Settings, contentDescription = "Preferencias")
        }
    }
)
```

**Corazón de favorito en cada tarjeta** (dentro del `Row` de la tarjeta, justo antes del botón de eliminar):

```kotlin
// NUEVO: dentro del Row de cada tarjeta, antes del IconButton de eliminar
IconButton(onClick = { viewModel.cambiarFavorito(contacto) }) {
    Icon(
        imageVector = if (contacto.favorito) Icons.Default.Favorite else Icons.Default.FavoriteBorder,
        contentDescription = "Favorito",
        tint = if (contacto.favorito) MaterialTheme.colorScheme.primary
               else MaterialTheme.colorScheme.onSurfaceVariant
    )
}
```

- `Icons.Default.Settings`, `Favorite` y `FavoriteBorder` están en el set básico de íconos, así que no necesitan `material-icons-extended`.
- `remember(contactos, busqueda, soloFavoritos, orden)` vuelve a calcular la lista cuando cambia cualquiera de esos cuatro valores: el texto buscado, los contactos de Room, o las preferencias de DataStore.

### 10.5 Cómo funciona el flujo completo

1. El usuario toca el ícono de ajustes en la lista, que navega a `PreferenciasAgendaView`.
2. Cambia un `Switch` o un `RadioButton`, y el `PreferenciasViewModel` guarda el valor en DataStore.
3. DataStore emite el nuevo valor por su `Flow`: `MainActivity` recompone el tema si cambió el modo oscuro, y `ListaContactosView` vuelve a filtrar y ordenar si cambiaron los otros dos.
4. Al regresar a la lista, los cambios ya se ven. Si cierra y vuelve a abrir la app, las preferencias se conservan.

---

## 11. Estructura de paquetes resultante

```
com.example.agenda
 ├── MainActivity.kt                       (ahora crea también el PreferenciasViewModel)
 ├── data
 │    └── AgendaDataStore.kt               (instancia + claves)
 │    
 ├── viewmodel
 │    └── PreferenciasViewModel.kt
 ├── navigation
 │    └── NavManager.kt                    (ahora con la ruta "preferencias")
 ├── repository
 │    └── AgendaPreferenciasRepository.kt 
 └── ui.views
      └── PreferenciasAgendaView.kt
```

Estos archivos se suman a los de la guía `7_1` (`Contacto`, `ContactoDao`, `AgendaDatabase`, `AgendaViewModel`, `ListaContactosView`, `RegistroContactoView`).

---

## 12. Errores comunes

- **Las preferencias no persisten al cerrar la app:** se está guardando el valor solo en un estado de Compose (`remember`), sin llamar a `repository.actualizarX(...)` — `remember` no sobrevive a cerrar la app; DataStore sí.
- **"This method must be called in a coroutine":** se intentó llamar `.edit { }` directamente desde un `onClick` sin envolverlo en `viewModelScope.launch`.
- **Confundir DataStore con Room:** intentar guardar la lista completa de contactos como una sola clave de texto (por ejemplo, serializada en JSON) funciona, pero pierde las ventajas de consultar, ordenar y actualizar un solo contacto que sí ofrece Room — para listas, siempre preferir Room.
- **"Room cannot verify the data integrity. Looks like you've changed schema but forgot to update the version number":** se agregó el campo `favorito` a `Contacto`, pero en el dispositivo ya existe la base de datos con la estructura anterior. Solución rápida (práctica): desinstalar la app y volver a ejecutarla. Solución que conserva los datos: subir `version = 2` en `@Database` y agregar la `Migration` del Paso 7.
- **Desinstalé la app pero el error sigue, con los mismos hashes:** Android restauró la base de datos desde la copia automática de Google. Ponga `android:allowBackup="false"` en el manifiesto y suba la `version` de `@Database` (en práctica, con `fallbackToDestructiveMigration(dropAllTables = true)`).

---

## 13. Resumen: Room + DataStore juntos

En una misma app es normal (y recomendable) usar ambos a la vez, cada uno para lo que le corresponde:

- **Room:** la lista de contactos (datos estructurados, que se consultan, filtran y relacionan).
- **DataStore:** los ajustes de esa pantalla (modo oscuro, orden, filtros activos) — datos simples que no tendría sentido modelar como una tabla.
