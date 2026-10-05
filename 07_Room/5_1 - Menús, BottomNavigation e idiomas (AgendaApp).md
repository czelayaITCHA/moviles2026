# 5\_1 — Menús, BottomNavigation y múltiples idiomas: AgendaApp

Esta guía amplía la **AgendaApp** (paquete `com.example.agendaapp`) construida en las guías `7_1` (Room) y `7_2` (DataStore). Se agregan tres cosas: **menús**, una **barra de navegación inferior** y soporte para **varios idiomas**. Igual que en `7_2`, cada paso indica si hay que **CREAR** un archivo nuevo o **MODIFICAR** uno que ya existe.

**No hace falta agregar ninguna dependencia nueva:** los íconos usados están en el set básico, `NavigationBar` y `DropdownMenu` vienen en Material 3, y `toUri` viene en `core-ktx`, que el proyecto ya tiene.

## 1. Objetivos

- Crear menús desplegables (menú de tres puntos y menú por tarjeta) con `DropdownMenu`.
- Mostrar un diálogo con `AlertDialog` y lanzar acciones del sistema (llamar, enviar correo) con `Intent`.
- Organizar la app con una `NavigationBar` (BottomNavigation) de tres pestañas.
- Traducir la interfaz a español e inglés con recursos `strings.xml` y permitir cambiar el idioma desde la app.

---

## 2. Resumen de cambios sobre el proyecto

| Archivo | Acción | Parte |
| --- | --- | --- |
| `ui/components/ContactoCard.kt` | CREAR | A |
| `ui/views/ListaContactosView.kt` | MODIFICAR | A, B y C |
| `navigation/Destino.kt` | CREAR | B |
| `navigation/BarraInferior.kt` | CREAR | B |
| `navigation/NavManager.kt` | MODIFICAR | B |
| `ui/views/FavoritosView.kt` | CREAR | B |
| `ui/views/PreferenciasAgendaView.kt` | MODIFICAR | B y C |
| `res/values/strings.xml` | MODIFICAR (ampliar) | C |
| `res/values-en/strings.xml` | CREAR | C |
| `res/xml/locales_config.xml` | CREAR | C |
| `AndroidManifest.xml` | MODIFICAR | C |
| `ui/components/SelectorIdioma.kt` | CREAR | C |
| `ui/views/RegistroContactoView.kt`, `ui/components/ContactoCard.kt`, `navigation/Destino.kt`, `navigation/BarraInferior.kt`, `ui/views/FavoritosView.kt` | MODIFICAR (solo textos) | C |

---

## 3. Parte A — Menús

En Compose, un menú desplegable se arma con tres piezas: un botón que lo abre, un estado `Boolean` que indica si está abierto, y un `DropdownMenu` con sus `DropdownMenuItem`. En esta parte se crean dos menús:

- **Menú de tres puntos** en la barra superior de la lista, con las opciones "Ordenar por nombre", "Ordenar por más reciente" y "Acerca de".
- **Menú por tarjeta**, con las opciones "Llamar", "Enviar correo" y "Eliminar".

*(Existen otros menús que no se ven en esta guía: `ModalNavigationDrawer` para el menú lateral y `ExposedDropdownMenuBox` para listas desplegables dentro de un formulario.)*

## 4. Paso 1 \[CREAR ui/components/ContactoCard.kt\] — Tarjeta reutilizable con menú

Hasta ahora la tarjeta del contacto estaba escrita dentro de `ListaContactosView`. Como la pestaña de Favoritos (Parte B) necesita la misma tarjeta, se extrae a un composable propio:

```kotlin
package com.example.agendaapp.ui.components

// Imports principales (el resto los sugiere Android Studio con Alt+Enter):
// android.content.Intent, androidx.core.net.toUri, coil3.compose.AsyncImage,
// com.example.agendaapp.data.Contacto

@Composable
fun ContactoCard(
    contacto: Contacto,
    onFavoritoClick: () -> Unit,
    onEliminarClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    val context = LocalContext.current
    var menuAbierto by remember { mutableStateOf(false) }

    ElevatedCard(modifier = modifier.fillMaxWidth(), shape = RoundedCornerShape(16.dp)) {
        Row(
            modifier = Modifier.padding(14.dp),
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.spacedBy(10.dp)
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

            IconButton(onClick = onFavoritoClick) {
                Icon(
                    imageVector = if (contacto.favorito) Icons.Default.Favorite else Icons.Default.FavoriteBorder,
                    contentDescription = "Favorito",
                    tint = if (contacto.favorito) MaterialTheme.colorScheme.primary
                           else MaterialTheme.colorScheme.onSurfaceVariant
                )
            }

            // Menú de la tarjeta: el Box hace que el menú se abra junto al botón
            Box {
                IconButton(onClick = { menuAbierto = true }) {
                    Icon(Icons.Default.MoreVert, contentDescription = "Más opciones")
                }
                DropdownMenu(expanded = menuAbierto, onDismissRequest = { menuAbierto = false }) {
                    DropdownMenuItem(
                        text = { Text("Llamar") },
                        leadingIcon = { Icon(Icons.Default.Call, contentDescription = null) },
                        onClick = {
                            menuAbierto = false
                            val intent = Intent(Intent.ACTION_DIAL, "tel:${contacto.telefono}".toUri())
                            runCatching { context.startActivity(intent) }
                        }
                    )
                    DropdownMenuItem(
                        text = { Text("Enviar correo") },
                        leadingIcon = { Icon(Icons.Default.Email, contentDescription = null) },
                        onClick = {
                            menuAbierto = false
                            val intent = Intent(Intent.ACTION_SENDTO, "mailto:${contacto.correo}".toUri())
                            runCatching { context.startActivity(intent) }
                        }
                    )
                    DropdownMenuItem(
                        text = { Text("Eliminar") },
                        leadingIcon = { Icon(Icons.Default.Delete, contentDescription = null) },
                        onClick = {
                            menuAbierto = false
                            onEliminarClick()
                        }
                    )
                }
            }
        }
    }
}
```

- `menuAbierto` es el estado que controla el menú: el botón de tres puntos lo pone en `true`, y `onDismissRequest` (tocar fuera del menú) o elegir una opción lo regresa a `false`.
- `Intent.ACTION_DIAL` abre el marcador del teléfono con el número ya escrito; **no necesita ningún permiso**, porque es el usuario quien pulsa "llamar". `ACTION_SENDTO` con `mailto:` abre la app de correo.
- `runCatching { ... }` evita que la app se cierre si el dispositivo no tiene ninguna app que atienda el `Intent` (por ejemplo, un emulador sin app de correo).
- `"tel:...".toUri()` (de `androidx.core.net`) crea el `Uri` de Android. Se usa en lugar de `Uri.parse` para no confundirlo con `coil3.Uri`, el choque de nombres que apareció en la guía `7_1`.
- La tarjeta ya no decide qué hacer con el favorito ni con eliminar: recibe `onFavoritoClick` y `onEliminarClick` desde afuera. Así es reutilizable en cualquier pantalla.

## 5. Paso 2 \[MODIFICAR ui/views/ListaContactosView.kt\] — Usar la tarjeta y agregar el menú de tres puntos

**Estado nuevo**, junto a los demás `remember` de la función:

```kotlin
var menuAbierto by remember { mutableStateOf(false) }       // NUEVO
var mostrarAcercaDe by remember { mutableStateOf(false) }   // NUEVO
```

**`actions` de la barra superior.** Se deja el ícono de ajustes de `7_2` y se agrega el menú de tres puntos a su lado:

```kotlin
actions = {
    // Sin cambios por ahora (se retira en la Parte B)
    IconButton(onClick = onPreferenciasClick) {
        Icon(Icons.Default.Settings, contentDescription = "Preferencias")
    }
    // NUEVO: menú de tres puntos
    Box {
        IconButton(onClick = { menuAbierto = true }) {
            Icon(Icons.Default.MoreVert, contentDescription = "Más opciones")
        }
        DropdownMenu(expanded = menuAbierto, onDismissRequest = { menuAbierto = false }) {
            DropdownMenuItem(
                text = { Text("Ordenar por nombre") },
                onClick = {
                    preferenciasViewModel.cambiarOrden("nombre")
                    menuAbierto = false
                }
            )
            DropdownMenuItem(
                text = { Text("Ordenar por más reciente") },
                onClick = {
                    preferenciasViewModel.cambiarOrden("reciente")
                    menuAbierto = false
                }
            )
            HorizontalDivider()
            DropdownMenuItem(
                text = { Text("Acerca de") },
                onClick = {
                    menuAbierto = false
                    mostrarAcercaDe = true
                }
            )
        }
    }
}
```

**Elementos de la lista.** Se reemplaza todo el `Card` que estaba dentro de `items(...)` por la tarjeta nueva:

```kotlin
LazyColumn(verticalArrangement = Arrangement.spacedBy(10.dp)) {
    items(contactosFiltrados, key = { it.id }) { contacto ->
        ContactoCard(                                              // CAMBIO
            contacto = contacto,
            onFavoritoClick = { viewModel.cambiarFavorito(contacto) },
            onEliminarClick = { viewModel.eliminarContacto(contacto) }
        )
    }
}
```

**Diálogo "Acerca de"**, al final de la función, después del `Scaffold`:

```kotlin
if (mostrarAcercaDe) {
    AlertDialog(
        onDismissRequest = { mostrarAcercaDe = false },
        title = { Text("Acerca de") },
        text = { Text("AgendaApp 1.0 — práctica de Jetpack Compose, Room y DataStore.") },
        confirmButton = {
            TextButton(onClick = { mostrarAcercaDe = false }) { Text("Cerrar") }
        }
    )
}
```

- "Ordenar por nombre" y "Ordenar por más reciente" **reutilizan el DataStore de la guía `7_2`**: llaman a `preferenciasViewModel.cambiarOrden(...)`, la misma función que usa la pantalla de preferencias. Cambiar el orden desde aquí cambia también el `RadioButton` de esa pantalla.
- Un `AlertDialog` no es un menú, pero se usa en el mismo flujo: un estado `Boolean` decide si se dibuja o no. Cuando `mostrarAcercaDe` es `false`, el diálogo simplemente no existe en la composición.

**Cómo probar:** toque los tres puntos de una tarjeta y elija "Llamar"; debe abrirse el marcador con el número. Luego toque los tres puntos de la barra superior y cambie el orden de la lista.

---

## 6. Parte B — BottomNavigation

La app pasa de tener un ícono de ajustes en la barra superior a tener una **barra inferior con tres pestañas**:

| Pestaña | Ruta | Pantalla |
| --- | --- | --- |
| Contactos | `"lista"` | `ListaContactosView` (la que ya existe) |
| Favoritos | `"favoritos"` | `FavoritosView` (nueva) |
| Ajustes | `"preferencias"` | `PreferenciasAgendaView` (la de `7_2`) |

La pantalla de registro (`"registro"`) **no** muestra la barra: es una pantalla secundaria a la que se llega con el botón "+" y de la que se sale con la flecha de volver.

## 7. Paso 3 \[CREAR navigation/Destino.kt\] — Las pestañas como objetos

```kotlin
package com.example.agendaapp.navigation

import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material.icons.filled.Person
import androidx.compose.material.icons.filled.Settings
import androidx.compose.ui.graphics.vector.ImageVector

sealed class Destino(val ruta: String, val titulo: String, val icono: ImageVector) {
    object Contactos : Destino("lista", "Contactos", Icons.Default.Person)
    object Favoritos : Destino("favoritos", "Favoritos", Icons.Default.Favorite)
    object Ajustes : Destino("preferencias", "Ajustes", Icons.Default.Settings)
}

val destinosInferiores = listOf(Destino.Contactos, Destino.Favoritos, Destino.Ajustes)
```

- Una `sealed class` agrupa las pestañas en un solo lugar: ruta, texto e ícono de cada una. Agregar una cuarta pestaña es agregar un `object` y ponerlo en la lista.
- Las rutas `"lista"` y `"preferencias"` son las mismas que ya usaba `NavManager`; solo se agrega `"favoritos"`.

## 8. Paso 4 \[CREAR navigation/BarraInferior.kt\] — La barra

```kotlin
package com.example.agendaapp.navigation

// Imports que conviene revisar:
// androidx.navigation.NavHostController
// androidx.navigation.NavGraph.Companion.findStartDestination
// androidx.navigation.compose.currentBackStackEntryAsState

@Composable
fun BarraInferior(navController: NavHostController) {
    val entradaActual by navController.currentBackStackEntryAsState()
    val rutaActual = entradaActual?.destination?.route

    NavigationBar {
        destinosInferiores.forEach { destino ->
            NavigationBarItem(
                selected = rutaActual == destino.ruta,
                onClick = {
                    navController.navigate(destino.ruta) {
                        popUpTo(navController.graph.findStartDestination().id) { saveState = true }
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
```

- `currentBackStackEntryAsState()` entrega la pantalla actual como un estado de Compose; cuando cambia, la barra se redibuja y marca la pestaña correcta con `selected`.
- Las tres opciones dentro de `navigate { ... }` evitan los problemas típicos de una barra inferior: `popUpTo(inicio)` impide que el botón "atrás" recorra todas las pestañas que se tocaron, `launchSingleTop` evita abrir dos copias de la misma pestaña, y `saveState`/`restoreState` conservan el estado de cada pestaña (por ejemplo, el texto del buscador) al volver a ella.

## 9. Paso 5 \[MODIFICAR navigation/NavManager.kt\] — Un `Scaffold` raíz con la barra

La barra inferior se coloca **una sola vez**, en un `Scaffold` que envuelve al `NavHost`, y solo se muestra en las rutas de las pestañas:

```kotlin
package com.example.agendaapp.navigation

@Composable
fun NavManager(
    viewModel: AgendaViewModel,
    preferenciasViewModel: PreferenciasViewModel
) {
    val navController = rememberNavController()

    // NUEVO: saber en qué ruta estamos para decidir si se muestra la barra
    val entradaActual by navController.currentBackStackEntryAsState()
    val rutaActual = entradaActual?.destination?.route
    val mostrarBarra = destinosInferiores.any { it.ruta == rutaActual }

    // NUEVO: Scaffold raíz
    Scaffold(
        bottomBar = { if (mostrarBarra) BarraInferior(navController) },
        contentWindowInsets = WindowInsets(0.dp)
    ) { innerPadding ->
        NavHost(
            navController = navController,
            startDestination = Destino.Contactos.ruta,                         // CAMBIO
            modifier = Modifier.padding(innerPadding).consumeWindowInsets(innerPadding)  // NUEVO
        ) {
            composable(Destino.Contactos.ruta) {
                ListaContactosView(
                    viewModel = viewModel,
                    preferenciasViewModel = preferenciasViewModel,
                    onAgregarClick = { navController.navigate("registro") }
                    // CAMBIO: se quitó onPreferenciasClick (ver Paso 7)
                )
            }
            composable(Destino.Favoritos.ruta) {                                    // NUEVO
                FavoritosView(viewModel = viewModel)
            }
            // Sin cambios (guía 7_1)
            composable("registro") {
                RegistroContactoView(
                    viewModel = viewModel,
                    onGuardado = { navController.popBackStack() },
                    onBack = { navController.popBackStack() }
                )
            }
            composable(Destino.Ajustes.ruta) {
                PreferenciasAgendaView(viewModel = preferenciasViewModel)          // CAMBIO: sin onBack
            }
        }
    }
}
```

- `mostrarBarra` es `true` solo si la ruta actual es una de las tres pestañas, por eso la barra desaparece al entrar a `"registro"`.
- **Por qué `contentWindowInsets = WindowInsets(0.dp)` y `consumeWindowInsets(innerPadding)`:** cada pantalla ya tiene su propio `Scaffold` con su `TopAppBar`, que se encarga de la barra de estado. Con estas dos líneas el `Scaffold` raíz no agrega espacio arriba (la barra superior de cada pantalla sigue pintando bajo la barra de estado) y le avisa a los `Scaffold` internos que el espacio inferior ya lo ocupa la barra, para que no dejen un hueco doble sobre ella.
- `startDestination` usa `Destino.Contactos.ruta`, que vale `"lista"`: el mismo valor de antes.

## 10. Paso 6 \[CREAR ui/views/FavoritosView.kt\] — La pestaña de favoritos

```kotlin
package com.example.agendaapp.ui.views

@Composable
fun FavoritosView(viewModel: AgendaViewModel, modifier: Modifier = Modifier) {
    val contactos by viewModel.contactos.collectAsState()
    val favoritos = remember(contactos) { contactos.filter { it.favorito } }

    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(
                title = { Text("Favoritos") },
                colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                )
            )
        }
    ) { innerPadding ->
        if (favoritos.isEmpty()) {
            Box(
                modifier = modifier.fillMaxSize().padding(innerPadding),
                contentAlignment = Alignment.Center
            ) {
                Text(
                    text = "Aún no tiene contactos favoritos",
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        } else {
            LazyColumn(
                modifier = modifier.fillMaxSize().padding(innerPadding),
                contentPadding = PaddingValues(16.dp),
                verticalArrangement = Arrangement.spacedBy(10.dp)
            ) {
                items(favoritos, key = { it.id }) { contacto ->
                    ContactoCard(
                        contacto = contacto,
                        onFavoritoClick = { viewModel.cambiarFavorito(contacto) },
                        onEliminarClick = { viewModel.eliminarContacto(contacto) }
                    )
                }
            }
        }
    }
}
```

- Reutiliza el mismo `AgendaViewModel` y la misma `ContactoCard`. Desde esta pestaña, quitar el corazón de un contacto hace que desaparezca de la lista al instante, porque `viewModel.contactos` es un `StateFlow` que viene de Room.
- Esta pestaña muestra **siempre** solo los favoritos. La preferencia "Mostrar solo favoritos" de `7_2` sigue funcionando en la pestaña Contactos, y no se contradicen.

## 11. Paso 7 \[MODIFICAR ListaContactosView.kt y PreferenciasAgendaView.kt\] — Quitar lo que sobra

Como "Ajustes" ahora es una pestaña, ya no hace falta el ícono de ajustes ni la flecha de volver.

**En `ListaContactosView`:** se elimina el parámetro `onPreferenciasClick` de la firma y el primer `IconButton` (el de `Icons.Default.Settings`) dentro de `actions`. El menú de tres puntos del Paso 2 se queda:

```kotlin
@Composable
fun ListaContactosView(
    viewModel: AgendaViewModel,
    preferenciasViewModel: PreferenciasViewModel,
    onAgregarClick: () -> Unit,
    // CAMBIO: se eliminó onPreferenciasClick
    modifier: Modifier = Modifier
) { ... }
```

**En `PreferenciasAgendaView`:** se elimina el parámetro `onBack` y el `navigationIcon` de la barra superior:

```kotlin
@Composable
fun PreferenciasAgendaView(
    viewModel: PreferenciasViewModel,
    modifier: Modifier = Modifier          // CAMBIO: se eliminó onBack
) {
    val modoOscuro by viewModel.modoOscuro.collectAsState()
    val soloFavoritos by viewModel.soloFavoritos.collectAsState()
    val ordenLista by viewModel.ordenLista.collectAsState()

    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(        // CAMBIO: ya no lleva flecha de volver
                title = { Text("Ajustes") },
                colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                )
            )
        }
    ) { innerPadding ->
        // ... el contenido (Switch y RadioButton) queda igual que en la guía 7_2
    }
}
```

**Cómo probar:** marque un contacto como favorito en la pestaña Contactos y vaya a Favoritos. Toque "+" para abrir el registro y compruebe que la barra inferior desaparece. Al guardar debe volver a la lista con la barra visible.

---

## 12. Parte C — Múltiples idiomas

Android elige el idioma de la app según la carpeta de recursos que coincida con el idioma del dispositivo:

```
res/
 ├── values/strings.xml       ← idioma por defecto (español). También es el de respaldo
 └── values-en/strings.xml    ← inglés
```

Si el dispositivo está en inglés se usa `values-en`. Si está en un idioma que la app no tiene (por ejemplo, francés), se usa `values`. En el código no hay textos escritos a mano: cada texto se pide por su clave con `stringResource(R.string.clave)`.

## 13. Paso 8 \[MODIFICAR res/values/strings.xml + CREAR res/values-en/strings.xml\] — Los textos

**`res/values/strings.xml` (español).** El proyecto ya trae `app_name`; se deja esa línea y se agregan las demás:

```xml
<resources>
    <string name="app_name">AgendaApp</string>

    <!-- Pestañas -->
    <string name="tab_contactos">Contactos</string>
    <string name="tab_favoritos">Favoritos</string>
    <string name="tab_ajustes">Ajustes</string>

    <!-- Lista de contactos -->
    <string name="titulo_agenda">Agenda de Contactos</string>
    <string name="buscar_contacto">Buscar contacto</string>
    <string name="sin_resultados">No se encontraron contactos</string>
    <string name="agregar_contacto">Agregar contacto</string>
    <plurals name="total_contactos">
        <item quantity="one">%d contacto</item>
        <item quantity="other">%d contactos</item>
    </plurals>

    <!-- Favoritos -->
    <string name="titulo_favoritos">Favoritos</string>
    <string name="sin_favoritos">Aún no tiene contactos favoritos</string>

    <!-- Tarjeta y menús -->
    <string name="favorito">Favorito</string>
    <string name="mas_opciones">Más opciones</string>
    <string name="llamar">Llamar</string>
    <string name="enviar_correo">Enviar correo</string>
    <string name="eliminar">Eliminar</string>
    <string name="ordenar_por_nombre">Ordenar por nombre</string>
    <string name="ordenar_por_reciente">Ordenar por más reciente</string>
    <string name="acerca_de">Acerca de</string>
    <string name="acerca_de_texto">AgendaApp %1$s — práctica de Jetpack Compose, Room y DataStore.</string>
    <string name="cerrar">Cerrar</string>

    <!-- Registro -->
    <string name="nuevo_contacto">Nuevo contacto</string>
    <string name="volver">Volver</string>
    <string name="galeria">Galería</string>
    <string name="camara">Cámara</string>
    <string name="foto_contacto">Foto del contacto</string>
    <string name="nombre">Nombre</string>
    <string name="telefono">Teléfono</string>
    <string name="correo">Correo</string>
    <string name="guardar_contacto">Guardar contacto</string>

    <!-- Ajustes -->
    <string name="titulo_ajustes">Ajustes</string>
    <string name="modo_oscuro">Modo oscuro</string>
    <string name="solo_favoritos">Mostrar solo favoritos</string>
    <string name="ordenar_lista_por">Ordenar lista por:</string>
    <string name="orden_nombre">Nombre</string>
    <string name="orden_reciente">Más reciente</string>
    <string name="idioma">Idioma</string>
    <string name="idioma_es">Español</string>
    <string name="idioma_en">English</string>
</resources>
```

**`res/values-en/strings.xml` (inglés).** En Android Studio: clic derecho sobre `res` → *New* → *Android Resource File*; en *File name* escriba `strings.xml`, en *Available qualifiers* elija *Locale* y luego `en`. Cada clave debe tener **exactamente el mismo nombre** que en español:

```xml
<resources>
    <string name="app_name">AgendaApp</string>

    <string name="tab_contactos">Contacts</string>
    <string name="tab_favoritos">Favorites</string>
    <string name="tab_ajustes">Settings</string>

    <string name="titulo_agenda">Contact Book</string>
    <string name="buscar_contacto">Search contact</string>
    <string name="sin_resultados">No contacts found</string>
    <string name="agregar_contacto">Add contact</string>
    <plurals name="total_contactos">
        <item quantity="one">%d contact</item>
        <item quantity="other">%d contacts</item>
    </plurals>

    <string name="titulo_favoritos">Favorites</string>
    <string name="sin_favoritos">You have no favorite contacts yet</string>

    <string name="favorito">Favorite</string>
    <string name="mas_opciones">More options</string>
    <string name="llamar">Call</string>
    <string name="enviar_correo">Send email</string>
    <string name="eliminar">Delete</string>
    <string name="ordenar_por_nombre">Sort by name</string>
    <string name="ordenar_por_reciente">Sort by most recent</string>
    <string name="acerca_de">About</string>
    <string name="acerca_de_texto">AgendaApp %1$s — Jetpack Compose, Room and DataStore practice project.</string>
    <string name="cerrar">Close</string>

    <string name="nuevo_contacto">New contact</string>
    <string name="volver">Back</string>
    <string name="galeria">Gallery</string>
    <string name="camara">Camera</string>
    <string name="foto_contacto">Contact photo</string>
    <string name="nombre">Name</string>
    <string name="telefono">Phone</string>
    <string name="correo">Email</string>
    <string name="guardar_contacto">Save contact</string>

    <string name="titulo_ajustes">Settings</string>
    <string name="modo_oscuro">Dark mode</string>
    <string name="solo_favoritos">Show favorites only</string>
    <string name="ordenar_lista_por">Sort list by:</string>
    <string name="orden_nombre">Name</string>
    <string name="orden_reciente">Most recent</string>
    <string name="idioma">Language</string>
    <string name="idioma_es">Español</string>
    <string name="idioma_en">English</string>
</resources>
```

- Los nombres de los idiomas ("Español", "English") se escriben igual en ambos archivos a propósito: cada idioma se muestra con su propio nombre, para que el usuario lo reconozca aunque la app esté en el otro.
- `%1$s` es un **marcador con argumento**: el texto real se pasa al pedir la cadena (ver `acerca_de_texto` en el Paso 9). `%d` dentro de `<plurals>` recibe un número.
- Un apóstrofe dentro de un texto XML debe escribirse con barra (`\'`); en esta lista no hay ninguno, pero es el primer error al traducir al inglés (por ejemplo, *Don't*).

## 14. Paso 9 \[MODIFICAR todas las vistas\] — Cambiar los textos fijos por `stringResource`

En cada archivo se reemplaza el texto escrito a mano por la llamada a `stringResource`, y se agrega `import com.example.agendaapp.R` y `import androidx.compose.ui.res.stringResource`.

```kotlin
// Antes
Text("Buscar contacto")

// Después
Text(stringResource(R.string.buscar_contacto))
```

Android Studio ayuda a hacerlo: con el cursor sobre un texto escrito a mano, `Alt+Enter` → *Extract string resource*.

**Tabla de reemplazos por archivo:**

| Archivo | Texto actual | Clave |
| --- | --- | --- |
| `ContactoCard` | `"Favorito"` (contentDescription) | `R.string.favorito` |
| `ContactoCard` | `"Más opciones"` | `R.string.mas_opciones` |
| `ContactoCard` | `"Llamar"`, `"Enviar correo"`, `"Eliminar"` | `R.string.llamar`, `R.string.enviar_correo`, `R.string.eliminar` |
| `FavoritosView` | `"Favoritos"` | `R.string.titulo_favoritos` |
| `FavoritosView` | `"Aún no tiene contactos favoritos"` | `R.string.sin_favoritos` |
| `RegistroContactoView` | `"Nuevo contacto"`, `"Volver"`, `"Foto del contacto"` | `R.string.nuevo_contacto`, `R.string.volver`, `R.string.foto_contacto` |
| `RegistroContactoView` | `"Galería"`, `"Cámara"` | `R.string.galeria`, `R.string.camara` |
| `RegistroContactoView` | `"Nombre"`, `"Teléfono"`, `"Correo"` (labels) | `R.string.nombre`, `R.string.telefono`, `R.string.correo` |
| `RegistroContactoView` | `"Guardar contacto"` | `R.string.guardar_contacto` |
| `PreferenciasAgendaView` | `"Ajustes"`, `"Modo oscuro"`, `"Mostrar solo favoritos"` | `R.string.titulo_ajustes`, `R.string.modo_oscuro`, `R.string.solo_favoritos` |
| `PreferenciasAgendaView` | `"Ordenar lista por:"`, `"Nombre"`, `"Más reciente"` | `R.string.ordenar_lista_por`, `R.string.orden_nombre`, `R.string.orden_reciente` |

**No traduzca los valores internos** `"nombre"` y `"reciente"` que se pasan a `cambiarOrden(...)` y que DataStore guarda: no son texto para el usuario, son una clave. Si se cambian, las preferencias ya guardadas dejan de coincidir.

**`Destino` y `BarraInferior`:** el título de las pestañas ahora es una referencia a un recurso, no un `String`. Se usa la anotación `@StringRes`:

```kotlin
// Destino.kt
sealed class Destino(val ruta: String, @StringRes val titulo: Int, val icono: ImageVector) {
    object Contactos : Destino("lista", R.string.tab_contactos, Icons.Default.Person)
    object Favoritos : Destino("favoritos", R.string.tab_favoritos, Icons.Default.Favorite)
    object Ajustes : Destino("preferencias", R.string.tab_ajustes, Icons.Default.Settings)
}
```

```kotlin
// BarraInferior.kt — dentro del forEach
val titulo = stringResource(destino.titulo)
NavigationBarItem(
    selected = rutaActual == destino.ruta,
    onClick = { /* sin cambios */ },
    icon = { Icon(destino.icono, contentDescription = titulo) },
    label = { Text(titulo) }
)
```

**`ListaContactosView` completa, ya con textos traducibles.** Es el archivo con más cambios, así que se muestra completo:

```kotlin
@Composable
fun ListaContactosView(
    viewModel: AgendaViewModel,
    preferenciasViewModel: PreferenciasViewModel,
    onAgregarClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    var busqueda by remember { mutableStateOf("") }
    var menuAbierto by remember { mutableStateOf(false) }
    var mostrarAcercaDe by remember { mutableStateOf(false) }
    val contactos by viewModel.contactos.collectAsState()
    val soloFavoritos by preferenciasViewModel.soloFavoritos.collectAsState()
    val orden by preferenciasViewModel.ordenLista.collectAsState()

    val contactosFiltrados = remember(contactos, busqueda, soloFavoritos, orden) {
        contactos
            .filter { busqueda.isBlank() || it.nombre.contains(busqueda, ignoreCase = true) }
            .filter { !soloFavoritos || it.favorito }
            .let { lista ->
                if (orden == "reciente") lista.sortedByDescending { it.id }
                else lista.sortedBy { it.nombre.lowercase() }
            }
    }

    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(
                title = { Text(stringResource(R.string.titulo_agenda)) },
                colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primaryContainer,
                    titleContentColor = MaterialTheme.colorScheme.onPrimaryContainer
                ),
                actions = {
                    Box {
                        IconButton(onClick = { menuAbierto = true }) {
                            Icon(
                                Icons.Default.MoreVert,
                                contentDescription = stringResource(R.string.mas_opciones)
                            )
                        }
                        DropdownMenu(expanded = menuAbierto, onDismissRequest = { menuAbierto = false }) {
                            DropdownMenuItem(
                                text = { Text(stringResource(R.string.ordenar_por_nombre)) },
                                onClick = {
                                    preferenciasViewModel.cambiarOrden("nombre")
                                    menuAbierto = false
                                }
                            )
                            DropdownMenuItem(
                                text = { Text(stringResource(R.string.ordenar_por_reciente)) },
                                onClick = {
                                    preferenciasViewModel.cambiarOrden("reciente")
                                    menuAbierto = false
                                }
                            )
                            HorizontalDivider()
                            DropdownMenuItem(
                                text = { Text(stringResource(R.string.acerca_de)) },
                                onClick = {
                                    menuAbierto = false
                                    mostrarAcercaDe = true
                                }
                            )
                        }
                    }
                }
            )
        },
        floatingActionButton = {
            FloatingActionButton(onClick = onAgregarClick) {
                Icon(
                    Icons.Default.Add,
                    contentDescription = stringResource(R.string.agregar_contacto)
                )
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
                placeholder = { Text(stringResource(R.string.buscar_contacto)) },
                leadingIcon = { Icon(Icons.Default.Search, contentDescription = null) },
                singleLine = true,
                shape = RoundedCornerShape(28.dp),
                colors = OutlinedTextFieldDefaults.colors(
                    unfocusedBorderColor = Color.Transparent,
                    focusedBorderColor = Color.Transparent,
                    unfocusedContainerColor = MaterialTheme.colorScheme.surfaceVariant,
                    focusedContainerColor = MaterialTheme.colorScheme.surfaceVariant
                ),
                modifier = Modifier.fillMaxWidth().padding(vertical = 12.dp)
            )

            if (contactosFiltrados.isEmpty()) {
                Text(
                    text = stringResource(R.string.sin_resultados),
                    modifier = Modifier.padding(top = 24.dp),
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            } else {
                LazyColumn(verticalArrangement = Arrangement.spacedBy(10.dp)) {
                    items(contactosFiltrados, key = { it.id }) { contacto ->
                        ContactoCard(
                            contacto = contacto,
                            onFavoritoClick = { viewModel.cambiarFavorito(contacto) },
                            onEliminarClick = { viewModel.eliminarContacto(contacto) }
                        )
                    }
                }
            }
        }
    }

    if (mostrarAcercaDe) {
        AlertDialog(
            onDismissRequest = { mostrarAcercaDe = false },
            title = { Text(stringResource(R.string.acerca_de)) },
            text = { Text(stringResource(R.string.acerca_de_texto, "1.0")) },
            confirmButton = {
                TextButton(onClick = { mostrarAcercaDe = false }) {
                    Text(stringResource(R.string.cerrar))
                }
            }
        )
    }
}
```

- `stringResource(R.string.acerca_de_texto, "1.0")` rellena el marcador `%1$s` del XML con la versión.
- `stringResource` solo puede llamarse dentro de un composable. En un `onClick` (por ejemplo, para un `Toast`) se usa `context.getString(R.string.clave)`.
- Para ver la vista previa en otro idioma: `@Preview(showBackground = true, locale = "en")`.

## 15. Paso 10 \[CREAR res/xml/locales\_config.xml + MODIFICAR AndroidManifest.xml\] — Declarar los idiomas de la app

Desde Android 13, el sistema ofrece un selector de idioma por app en *Ajustes → Sistema → Idiomas → Idiomas de las apps*. Para que AgendaApp aparezca ahí hay que declarar los idiomas que soporta.

**`res/xml/locales_config.xml`** (la carpeta `xml` ya existe por el `file_paths.xml` de la guía `7_1`):

```xml
<?xml version="1.0" encoding="utf-8"?>
<locale-config xmlns:android="http://schemas.android.com/apk/res/android">
    <locale android:name="es" />
    <locale android:name="en" />
</locale-config>
```

**`AndroidManifest.xml`**, agregar un atributo a la etiqueta `<application>`:

```xml
<application
    android:localeConfig="@xml/locales_config"
    ... >
```

## 16. Paso 11 \[CREAR ui/components/SelectorIdioma.kt + MODIFICAR PreferenciasAgendaView.kt\] — Cambiar el idioma desde la app

```kotlin
package com.example.agendaapp.ui.components

// Imports principales: android.app.LocaleManager, android.os.Build, android.os.LocaleList,
// androidx.compose.ui.platform.LocalContext, androidx.compose.ui.res.stringResource,
// java.util.Locale, com.example.agendaapp.R

@Composable
fun SelectorIdioma(modifier: Modifier = Modifier) {
    // LocaleManager existe desde Android 13 (API 33)
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        val context = LocalContext.current
        val localeManager = remember { context.getSystemService(LocaleManager::class.java) }

        val idiomaActual = localeManager.applicationLocales.let { locales ->
            if (locales.isEmpty) Locale.getDefault().language else locales[0].language
        }

        val idiomas = listOf(
            "es" to R.string.idioma_es,
            "en" to R.string.idioma_en
        )

        Column(modifier = modifier) {
            Text(stringResource(R.string.idioma))
            idiomas.forEach { (codigo, nombreRes) ->
                Row(verticalAlignment = Alignment.CenterVertically) {
                    RadioButton(
                        selected = idiomaActual == codigo,
                        onClick = {
                            localeManager.applicationLocales = LocaleList.forLanguageTags(codigo)
                        }
                    )
                    Text(stringResource(nombreRes))
                }
            }
        }
    }
}
```

Y en `PreferenciasAgendaView`, dentro del `Column` de las preferencias, después del bloque de "Ordenar lista por":

```kotlin
SelectorIdioma()   // NUEVO
```

- Al elegir un idioma, Android **recrea la pantalla** con los textos del idioma elegido; no hace falta guardar nada en DataStore, porque el sistema recuerda el idioma de cada app.
- Este selector solo funciona en **Android 13 o superior**. En versiones anteriores el `if` lo oculta y la app sigue el idioma del dispositivo, que también funciona gracias a `values-en`.
- *Para soportar Android 12 o inferior* se necesita la librería `androidx.appcompat`, cambiar `MainActivity` a `AppCompatActivity` y usar un tema que herede de `Theme.AppCompat`; después se llama a `AppCompatDelegate.setApplicationLocales(LocaleListCompat.forLanguageTags("en"))`. Con una `ComponentActivity` normal esa llamada no tiene efecto, por eso esta guía usa `LocaleManager` directamente.

## 17. Opcional — Plurales y argumentos

En el XML se definió `total_contactos` con una forma para uno (`%d contacto`) y otra para varios (`%d contactos`). Para mostrar el total encima de la lista, en `ListaContactosView`, debajo del buscador:

```kotlin
Text(
    text = pluralStringResource(
        R.plurals.total_contactos,
        contactosFiltrados.size,
        contactosFiltrados.size
    ),
    style = MaterialTheme.typography.labelMedium,
    color = MaterialTheme.colorScheme.onSurfaceVariant
)
```

- El segundo parámetro (`contactosFiltrados.size`) decide **qué forma** se usa ("1 contacto" o "3 contactos"); el tercero es el valor que se pone en el `%d`.
- Se importa `androidx.compose.ui.res.pluralStringResource`.

---

## 18. Cómo probar los tres idiomas

1. **Idioma del dispositivo:** *Ajustes → Sistema → Idiomas* y ponga el inglés como primero. Abra AgendaApp: todos los textos, incluidas las pestañas y los menús, deben estar en inglés.
2. **Idioma por app (Android 13+):** *Ajustes → Aplicaciones → AgendaApp → Idioma*, o *Ajustes → Sistema → Idiomas → Idiomas de las apps*.
3. **Desde la app:** pestaña *Ajustes* → *Idioma* → elija *English*.
4. Cierre y abra la app: el idioma elegido se conserva.

## 19. Errores comunes

- **La barra inferior aparece también en la pantalla de registro:** `mostrarBarra` no está filtrando por ruta. Revise que la condición sea `destinosInferiores.any { it.ruta == rutaActual }` y que `"registro"` no esté en esa lista.
- **Queda un hueco entre el contenido y la barra inferior:** falta `Modifier.padding(innerPadding).consumeWindowInsets(innerPadding)` en el `NavHost`.
- **La pestaña no se resalta:** `selected` compara la ruta de la pestaña con `entradaActual?.destination?.route`. Si una ruta se escribe distinto en `Destino` y en `NavHost`, nunca coinciden.
- **Al volver a una pestaña se pierde lo que estaba escrito:** faltan `saveState = true` y `restoreState = true` en el `navigate` de la barra.
- **"Unresolved reference: R":** falta `import com.example.agendaapp.R` (cuidado de no importar el `R` de `android` ni el de una librería).
- **Un texto sigue en español aunque el dispositivo esté en inglés:** esa clave no existe en `values-en/strings.xml` (Android usa el español como respaldo), o el texto sigue escrito a mano en el código. Android Studio avisa con la advertencia *MissingTranslation*.
- **El selector de idioma no aparece o no hace nada:** el dispositivo o emulador tiene Android 12 o inferior (API menor a 33).
- **AgendaApp no aparece en "Idiomas de las apps":** falta `android:localeConfig` en el manifiesto o el archivo `locales_config.xml` está mal escrito.
- **Al llamar o enviar correo no pasa nada en el emulador:** el emulador no tiene app de marcador o de correo. En un teléfono real sí funciona.

## 20. Ejercicio propuesto

Sobre la AgendaApp, debe agregar:

1. **Un tercer idioma** (por ejemplo portugués, `pt`): carpeta `values-pt`, una línea más en `locales_config.xml` y una opción más en `SelectorIdioma`.
2. **Una cuarta opción en el menú de la tarjeta, "Compartir"**, que envíe por otra app el nombre y el teléfono del contacto con `Intent.ACTION_SEND` (`type = "text/plain"` y `Intent.EXTRA_TEXT`). Debe estar traducida en los dos idiomas.
3. **Una insignia con el número de favoritos** sobre el ícono de la pestaña Favoritos, usando `BadgedBox` y `Badge`. Pista: el número sale de `contactos.count { it.favorito }`.
