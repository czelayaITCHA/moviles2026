# 8\_1 — Inyección de dependencias con Hilt: AgendaApp

Siguiente tema del módulo (carpeta `08_hilt`). Se aplica a la misma **AgendaApp** (paquete `com.example.agendaapp`) de las guías `7_1`, `7_2` y `5_1`. Como en las guías anteriores, cada paso indica si hay que **CREAR** un archivo nuevo o **MODIFICAR** uno que ya existe.

**Lo importante:** al terminar, la app se ve y funciona **exactamente igual** que antes. Lo que cambia es cómo se construyen los objetos por dentro.

## 1. Objetivos

- Comprender qué es una dependencia y qué problema resuelve la inyección de dependencias.
- Configurar Hilt en un proyecto con Version Catalog, KSP y AGP 9.
- Reemplazar la construcción manual de la base de datos, los repositorios y los `ViewModel` por anotaciones de Hilt.

---

## 2. ¿Qué problema resuelve Hilt?

Una **dependencia** es un objeto que otra clase necesita para funcionar. En la AgendaApp:

- `AgendaViewModel` necesita un `ContactoRepository`.
- `ContactoRepository` necesita un `ContactoDao`.
- `ContactoDao` sale de la `AgendaDatabase`, que necesita un `Context`.

Hoy esa cadena se arma **a mano** en `MainActivity`:

```kotlin
// MainActivity actual (guias 7_2 y 5_1)
private val repository by lazy {
    ContactoRepository(AgendaDatabase.obtenerInstancia(applicationContext).contactoDao())
}

private val viewModel: AgendaViewModel by viewModels {
    viewModelFactory { initializer { AgendaViewModel(repository) } }
}

private val preferenciasViewModel: PreferenciasViewModel by viewModels {
    viewModelFactory {
        initializer { PreferenciasViewModel(AgendaPreferenciasRepository(applicationContext)) }
    }
}
```

Eso tiene tres inconvenientes:

- `MainActivity` tiene que **conocer** cómo se construye todo: la base de datos, el DAO, los repositorios.
- Cada dependencia nueva agrega más líneas aquí. En la siguiente guía se agregará Retrofit, y la cadena crece.
- Es difícil sustituir una pieza por otra falsa para hacer pruebas.

**Hilt** (construido sobre Dagger) resuelve esto: cada clase **declara lo que necesita** y Hilt arma la cadena completa. `MainActivity` solo dice "dáme un `AgendaViewModel`".

### Anotaciones que se usan

| Anotación | Dónde va | Qué hace |
| --- | --- | --- |
| `@HiltAndroidApp` | La clase `Application` | Crea el contenedor de Hilt para toda la app |
| `@AndroidEntryPoint` | La `Activity` | Permite que Hilt entregue objetos a esa pantalla |
| `@HiltViewModel` + `@Inject constructor` | Cada `ViewModel` | Hilt sabe construirlo y le pasa lo que pide |
| `@Inject constructor` | Clases propias (repositorios) | Hilt sabe construirlas |
| `@Module` + `@InstallIn` | Un `object` | Agrupa "recetas" para lo que Hilt no puede construir solo |
| `@Provides` | Función dentro del módulo | La receta para crear un objeto (la base de datos, el DAO, luego Retrofit) |
| `@Singleton` | Junto a `@Provides` | Una sola instancia en toda la app |
| `@ApplicationContext` | Parámetro `Context` | Pide el `Context` de la aplicación |

**Regla práctica:** si la clase es **suya**, se le pone `@Inject constructor`. Si viene de una **librería** (Room, Retrofit) o no se puede construir con un constructor simple, se escribe una receta con `@Provides`.

---

## 3. Resumen de cambios sobre el proyecto

| Archivo | Acción |
| --- | --- |
| `libs.versions.toml`, `build.gradle.kts` (raíz) y `build.gradle.kts` (app) | MODIFICAR |
| `AgendaApplication.kt` | CREAR |
| `AndroidManifest.xml` | MODIFICAR |
| `di/AppModule.kt` | CREAR |
| `data/AgendaDatabase.kt` | MODIFICAR (quitar el singleton manual) |
| `data/ContactoRepository.kt` | MODIFICAR (`@Inject`) |
| `data/AgendaPreferenciasRepository.kt` | MODIFICAR (`@Inject`) |
| `viewmodel/AgendaViewModel.kt` y `viewmodel/PreferenciasViewModel.kt` | MODIFICAR (`@HiltViewModel`) |
| `MainActivity.kt` | MODIFICAR (`@AndroidEntryPoint`, se simplifica) |
| `NavManager`, las vistas, la entidad, el DAO y las claves de DataStore | Sin cambios |

---

## 4. Paso 1 \[MODIFICAR libs.versions.toml + build.gradle.kts (raíz y app)\] — Dependencias

**`gradle/libs.versions.toml`:** agregar una línea en cada sección.

```toml
# En la sección [versions]
hilt = "2.60.1"

# En la sección [plugins]
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }

# En la sección [libraries]
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-android-compiler", version.ref = "hilt" }
```

**`build.gradle.kts` de la RAÍZ del proyecto:** agregar el plugin junto a los que ya tiene.

```kotlin
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.ksp) apply false
    alias(libs.plugins.hilt) apply false      // NUEVO
}
```

**`build.gradle.kts` del módulo `app`:**

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.compose)
    alias(libs.plugins.ksp)
    alias(libs.plugins.hilt)                  // NUEVO
}

dependencies {
    // ... las dependencias que ya tiene (Room, Coil, DataStore, etc.)
    implementation(libs.hilt.android)         // NUEVO
    ksp(libs.hilt.compiler)                   // NUEVO
}
```

- El compilador de Hilt se agrega con `ksp(...)`, igual que el de Room. El plugin KSP ya está en el proyecto desde la guía `7_1`.
- Hilt `2.59` en adelante exige **AGP 9** y Gradle 9.1 o superior; a partir de `2.60` también exige `minSdk` 23 o mayor. El proyecto usa AGP 9.4.1, así que cumple. Si su `minSdk` es menor a 23, súbalo.
- Después de editar, pulse *Sync Now*.

## 5. Paso 2 \[CREAR AgendaApplication.kt + MODIFICAR AndroidManifest.xml\] — El punto de entrada de Hilt

Hilt necesita una clase `Application` anotada. Se crea en el paquete raíz, junto a `MainActivity`:

```kotlin
package com.example.agendaapp

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class AgendaApplication : Application()
```

Y se declara en el manifiesto, agregando `android:name` a la etiqueta `<application>`:

```xml
<application
    android:name=".AgendaApplication"
    ... >
```

- `@HiltAndroidApp` genera el contenedor donde Hilt guarda todo lo que sabe construir. Se crea una sola vez, al arrancar la app.
- Si se olvida el `android:name` en el manifiesto, la app se cierra al abrir con el error *Hilt Activity must be attached to an @HiltAndroidApp Application*.

## 6. Paso 3 \[CREAR di/AppModule.kt\] — Las recetas de la base de datos y el DAO

Room es una librería: Hilt no sabe construir una `AgendaDatabase` por sí solo. Se le da la receta en un módulo. Se crea un paquete nuevo `di`:

```kotlin
package com.example.agendaapp.di

import android.content.Context
import androidx.room.Room
import com.example.agendaapp.data.AgendaDatabase
import com.example.agendaapp.data.ContactoDao
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.android.qualifiers.ApplicationContext
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
object AppModule {

    @Provides
    @Singleton
    fun provideAgendaDatabase(@ApplicationContext context: Context): AgendaDatabase =
        Room.databaseBuilder(context, AgendaDatabase::class.java, "agenda_database")
            .fallbackToDestructiveMigration(dropAllTables = true)   // solo práctica: borra los datos si cambia la versión (ver Errores comunes)
            .build()

    @Provides
    fun provideContactoDao(database: AgendaDatabase): ContactoDao = database.contactoDao()
}
```

- `@InstallIn(SingletonComponent::class)` significa que estas recetas viven durante toda la vida de la app.
- `@Singleton` asegura **una sola** base de datos. Esto reemplaza al `companion object` con `@Volatile` y `synchronized` que se escribió a mano en la guía `7_1`.
- `provideContactoDao` recibe la `AgendaDatabase` como parámetro: Hilt ve que la necesita, usa la receta de arriba y se la entrega. No hay que llamar a una función a la otra.
- En el `builder` deje la misma configuración de migración que ya tenía en `obtenerInstancia` (`Migration` o `fallbackToDestructiveMigration`). Ese código se mueve de aquí.
- El nombre `"agenda_database"` debe ser el mismo que antes; si se cambia, Room crea una base nueva y vacía.

## 7. Paso 4 \[MODIFICAR data/AgendaDatabase.kt\] — Quitar el singleton manual

Como ahora la base de datos la crea el módulo, se elimina todo el `companion object` de la clase. Queda así:

```kotlin
@Database(entities = [Contacto::class], version = 3, exportSchema = false)   // deje su número de versión actual
abstract class AgendaDatabase : RoomDatabase() {
    abstract fun contactoDao(): ContactoDao
}
```

- Si se deja el `companion object` y además se agrega el módulo, la app tendría **dos** bases de datos abiertas sobre el mismo archivo, que es un error difícil de detectar.
- Si usaba una `Migration` (por ejemplo `MIGRACION_1_2`), déjela declarada en este archivo, fuera de la clase, y referéncela en el `builder` del módulo.

## 8. Paso 5 \[MODIFICAR los repositorios\] — `@Inject constructor`

Se agrega `@Inject` al constructor de cada repositorio. Con eso Hilt sabe construirlos **sin escribir ninguna receta**:

```kotlin
// data/ContactoRepository.kt
class ContactoRepository @Inject constructor(
    private val dao: ContactoDao
) {
    // ... el cuerpo queda igual
}
```

```kotlin
// data/AgendaPreferenciasRepository.kt
class AgendaPreferenciasRepository @Inject constructor(
    @ApplicationContext private val context: Context   // NUEVO: @ApplicationContext
) {
    // ... el cuerpo queda igual
}
```

- Imports: `javax.inject.Inject` y `dagger.hilt.android.qualifiers.ApplicationContext`.
- Hilt lee `ContactoRepository @Inject constructor(dao: ContactoDao)`, ve que necesita un `ContactoDao`, y lo busca en `AppModule`. Esa es la cadena que antes se armaba a mano.
- `@ApplicationContext` le indica a Hilt que el `Context` pedido es el de la aplicación. Es el adecuado para DataStore porque no se queda sujeto a una pantalla que ya se cerró.

## 9. Paso 6 \[MODIFICAR los ViewModel\] — `@HiltViewModel`

```kotlin
// viewmodel/AgendaViewModel.kt
@HiltViewModel                                        // NUEVO
class AgendaViewModel @Inject constructor(            // CAMBIO: @Inject constructor
    private val repository: ContactoRepository
) : ViewModel() {
    // ... el cuerpo queda igual
}
```

```kotlin
// viewmodel/PreferenciasViewModel.kt
@HiltViewModel                                        // NUEVO
class PreferenciasViewModel @Inject constructor(      // CAMBIO: @Inject constructor
    private val repository: AgendaPreferenciasRepository
) : ViewModel() {
    // ... el cuerpo queda igual
}
```

- Imports: `dagger.hilt.android.lifecycle.HiltViewModel` y `javax.inject.Inject`.
- Los dos `ViewModel` ya reciben su repositorio por el constructor, así que no hay que cambiar nada más: solo se agregan las dos anotaciones.
- Este mismo `@HiltViewModel` es el que corrige el error `Cannot create an instance of class AgendaViewModel` que apareció en la guía `7_1`, pero ahora sin escribir una `Factory`.

## 10. Paso 7 \[MODIFICAR MainActivity.kt\] — Se simplifica

```kotlin
@AndroidEntryPoint                                    // NUEVO
class MainActivity : ComponentActivity() {

    // CAMBIO: se eliminó el repository y las dos viewModelFactory { ... }
    private val viewModel: AgendaViewModel by viewModels()
    private val preferenciasViewModel: PreferenciasViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            val modoOscuro by preferenciasViewModel.modoOscuro.collectAsState()

            AgendaTheme(darkTheme = modoOscuro) {
                NavManager(
                    viewModel = viewModel,
                    preferenciasViewModel = preferenciasViewModel
                )
            }
        }
    }
}
```

- Import: `dagger.hilt.android.AndroidEntryPoint`. Se pueden borrar los imports de `viewModelFactory` e `initializer`, que ya no se usan.
- `by viewModels()` sin llaves ni fábrica: al estar la `Activity` anotada con `@AndroidEntryPoint`, Hilt se encarga de construir el `ViewModel`.
- `MainActivity` ya no sabe que existen una base de datos, un DAO o un repositorio.

---

## 11. Cómo ve Hilt la AgendaApp

Al compilar, Hilt revisa todas las anotaciones y arma este árbol. Cada flecha es una dependencia que "alguien pidió":

```
Context (@ApplicationContext)
 ├── AgendaDatabase                 ← receta en AppModule (@Provides @Singleton)
 │     └── ContactoDao              ← receta en AppModule (@Provides)
 │           └── ContactoRepository ← @Inject constructor
 │                 └── AgendaViewModel ← @HiltViewModel
 └── AgendaPreferenciasRepository   ← @Inject constructor
       └── PreferenciasViewModel    ← @HiltViewModel
```

Si alguna pieza del árbol no tiene receta ni `@Inject`, Hilt avisa **al compilar** (no cuando el usuario abre la app) con un error `MissingBinding` que dice exactamente qué falta.

**Cómo probar:** compile y ejecute. No debe haber ningún cambio visible: la lista, los favoritos, las preferencias, el idioma y la cámara funcionan igual. Si hay contactos guardados, siguen ahí (siempre que el nombre del archivo de la base de datos no haya cambiado).

---

## 12. Opcional — `hiltViewModel()` en `NavManager`

Hasta aquí `MainActivity` sigue entregando los `ViewModel` a `NavManager`, que se los pasa a cada pantalla. Con Hilt, cada pantalla puede **pedir el suyo** con `hiltViewModel()`, y `NavManager` deja de recibir parámetros.

**Dependencia** (desde `androidx.hilt` 1.3 la función `hiltViewModel()` para Compose está en un artefacto propio):

```toml
# En la sección [versions]
hiltAndroidx = "1.4.0"

# En la sección [libraries]
androidx-hilt-lifecycle-viewmodel-compose = { group = "androidx.hilt", name = "hilt-lifecycle-viewmodel-compose", version.ref = "hiltAndroidx" }
```

```kotlin
// build.gradle.kts (app)
implementation(libs.androidx.hilt.lifecycle.viewmodel.compose)
```

*(Verifique la versión más reciente en [developer.android.com/jetpack/androidx/releases/hilt](https://developer.android.com/jetpack/androidx/releases/hilt).)*

**`NavManager`, con el cambio:**

```kotlin
// import androidx.hilt.lifecycle.viewmodel.compose.hiltViewModel

@Composable
fun NavManager() {                                   // CAMBIO: sin parámetros
    // ... navController, Scaffold y NavHost igual que antes
    composable(Destino.Contactos.ruta) {
        ListaContactosView(
            viewModel = hiltViewModel(),             // CAMBIO
            preferenciasViewModel = hiltViewModel(), // CAMBIO
            onAgregarClick = { navController.navigate("registro") }
        )
    }
    composable(Destino.Favoritos.ruta) {
        FavoritosView(viewModel = hiltViewModel())   // CAMBIO
    }
    // lo mismo en "registro" y en Destino.Ajustes.ruta
}
```

Y en `MainActivity`: `NavManager()`. El `preferenciasViewModel` de `MainActivity` se queda, porque lo necesita para el modo oscuro.

- `hiltViewModel()` guarda el `ViewModel` en la pantalla de navegación actual: **cada pantalla recibe su propia instancia**, y se destruye cuando esa pantalla sale de la pila.
- Aquí eso no causa problemas porque la fuente de verdad son Room y DataStore, que son compartidos. Pero es una diferencia a tener presente: si un `ViewModel` guardara estado propio, dos pantallas ya no lo compartirían.
- Es opcional: la app funciona igual sin este paso.

---

## 13. Errores comunes

- **"Hilt Activity must be attached to an @HiltAndroidApp Application":** falta `@HiltAndroidApp` en `AgendaApplication` o falta `android:name=".AgendaApplication"` en el manifiesto.
- **"Cannot create an instance of class ...ViewModel" / `NoSuchMethodException`:** al `ViewModel` le falta `@HiltViewModel` o `@Inject constructor`, o a `MainActivity` le falta `@AndroidEntryPoint`.
- **"\[Dagger/MissingBinding\] ContactoDao cannot be provided without an @Provides-annotated method":** falta la receta en `AppModule`, o el módulo no tiene `@Module` y `@InstallIn(SingletonComponent::class)`.
- **Error de Gradle al sincronizar mencionando AGP o Gradle:** Hilt 2.59 o posterior exige AGP 9 y Gradle 9.1 o superior. Revise las versiones.
- **Se usa `kapt` en lugar de `ksp`:** el proyecto no tiene `kapt`. El compilador de Hilt va con `ksp(libs.hilt.compiler)`.
- **Los contactos desaparecieron:** el nombre del archivo de la base de datos en `AppModule` no coincide con el que se usaba antes (`"agenda_database"`), o la versión de `@Database` cambió y se activó la migración destructiva.
- **Dos bases de datos abiertas:** quedó el `companion object` con `obtenerInstancia` en `AgendaDatabase` junto con el módulo de Hilt.
- **Imports equivocados:** `HiltViewModel` es `dagger.hilt.android.lifecycle.HiltViewModel`, `Inject` es `javax.inject.Inject`, y `InstallIn` es `dagger.hilt.InstallIn`.

* **"A migration from 2 to 3 was required but not found" (los números pueden variar):** la base de datos guardada en el teléfono tiene una versión distinta a la de `@Database`, y el `builder` de `AppModule` no tiene ni una `Migration` para ese salto ni `fallbackToDestructiveMigration`. Solución para práctica: agregar `.fallbackToDestructiveMigration(dropAllTables = true)` al `builder` (borra los contactos guardados). Para conservar los datos: si la estructura de la tabla no cambió entre esas versiones, agregar una migración vacía, `object : Migration(2, 3) { override fun migrate(db: SupportSQLiteDatabase) {} }`, y registrarla con `.addMigrations(...)`.
* **No se puede "bajar" el número de versión:** si la base del teléfono ya está en la versión 3, poner `version = 2` hace que Room falle por intentar retroceder. Siempre se sube.

## 14. Ejercicio propuesto

1. **Convertir `ContactoRepository` en una interfaz.** Crear `interface ContactoRepository` con las funciones que ya existen, y una clase `ContactoRepositoryImpl @Inject constructor(dao: ContactoDao)` que la implemente. Como Hilt no puede elegir una implementación a partir de una interfaz, hay que agregar un módulo con `@Binds` (en una clase `abstract`) que le diga "cuando pidan `ContactoRepository`, entrega `ContactoRepositoryImpl`".
2. **Preparar la siguiente guía.** Crear un `di/NetworkModule` (un `object` con `@Module` y `@InstallIn(SingletonComponent::class)`) que tenga una función `@Provides` que devuelva la URL base de la API como `String`. En la guía de Retrofit se completará con el cliente y el servicio.
3. **Explicar con sus palabras** qué cambió en `MainActivity` y por qué ya no necesita conocer la base de datos.
